# Spring Boot

- [SpringBoot 특징](#feature)
- [Spring Boot auto-configuration](#config)
- [빌드툴(Maven/Gradle)이 하는 일](#build-tools)
- [어노테이션](#annotaions)
- [에러페이지 핸들링](#error)
- [@GetMapping 어노테이션으로 다중맵핑하기](#get-mapping-multi)
- [h2 데이터베이스 마이그레이션](#h2-databse)
- [DB 에러발생 무시하고 프로젝트 실행하기](#datasource-autocofig)
- [Gradle 버전확인하는 법](#gradlew-version)
- [프로젝트에서 Gradle 버전 올리기](#upgrade-gradle)
- [Gradle 다운그레이드](#gradlew-which-version)
- [Unit Test](#unit-test)

<br>

## <a name="feature"></a>Spring Boot 특징

![](https://miro.medium.com/max/1904/1*4ZPi1b_ca54pUE9xRB-IFQ.jpeg)

스프링을 보다 간편하게 사용할 수 있는 프레임워크이다.

기존의 스프링 프레임워크는 안정성과 성능, DI(의존성 주입) 등의 장점이 있었던 반면, 개발 환경 설정이 복잡하고 어렵다는 단점이 있었다. 스프링부트는 환경 설정을 최소화하고, 개발자가 비즈니스 로직에 집중할 수 있도록 도와 생산성을 향상시키는 프레임 워크이다.

[김지현](https://github.com/ihoneymon)님이 번역해주신 [스프링부트 설명서](https://gist.github.com/ihoneymon/8a905e1dd8393b6b9298)에 의하면, 스프링부트는 다음과 같은 특징을 갖고 있다. 김지현님이 정리해주신 문서는 [스프링부트 공식문서](https://spring.io/projects/spring-boot)를 기반으로 한다.

- **Crete stand-alone Spring application.**
  단독실행(stand-alone)가능한 스프링 애플리케이션을 실행한다.
- **Embed Tomcat, Jetty or Undertow directly (no need to deploy WAR files)**
  내장형 톰캣, Jetty 혹은 Undertow를 내장한다. (WAR파일로 배포할 필요가 없음.)
- **Provide opinionated 'starter' component to simplify your build configuration**
  기본 설정되어 있는 'starter' 컴포넌트들을 쉽게 추가할 수 있다.
- **Automatically configure Spring whenever possible.**
  항상 자동으로 설정(configure)해준다.
- **Provide production-ready features such as metrics, health checks and externalized configuration.**
  상용화에 필요한 통계, 상태 점검 및 외부설정을 제공한다.
- **Absoluely no code generation and no requirement for XMl configuration.**
  설정을 위한 XML 코드를 생성하거나 요구하지 않는다.

<br>

이와 함께 책 <처음 배우는 스프링 부트2>에서 정리한 것도 함께 작성해둔다.

- Embeded Tomcat, Jetty, Undertow를 사용하여 독립실행이 가능한 스프링 애플리케이션 개발
- 통합 스타터를 제공하여 Maven/Gradle 구성 간소화
- 스타터를 통한 자동화된 스프링 설정 제공
- 번거로운 XML 설정을 요구하지 않음
- JAR을 사용하여 자바 옵션만으로도 배포 가능
- 애플리케이션의 모니터링과 관리를 위한 스프링 액츄에이터(Spring Actuator)제공

<br>

## <a name="config"></a>Spring Boot auto-configuration

자동환경설정은 스프링부트의 가장 큰 장점이자 매우 중요한 역할을 수행한다. 약 Web, H2, JDBC 등 100여개의 자동 설정을  제공하며 새로 추가되는 라이브러리(jar)은 스프링부트 자동 설정 의존성에 따라 설정이 자동 적용된다.

스프링부트의 자동환경설정은 `@EnableAutoConfiguration` 또는 `@SpringBootApplication` 중 하나를 사용함으로써 할 수 있다.

[Spring-Boot Application](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/SpringBootApplication.java) 코드는 다음으로 구성되어 있다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { 
   @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
   @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
   ...
}
```

- `@SpringBootConfiguration`
  - 스프링부트의 설정을 나타내는 어노테이션. 스프링 프레임워크의 `@Configuration`을 대체하며 **스프링부트 전용**이다.
- `@EnableAutoConfiguration`
  - 자동 설정의 핵심 어노테이션. **클래스 경로에 지정된 내용을 기반으로 자동 설정을 수행**. 특별한 설정값을 추가하지 않으면 기본값으로 작동한다.
- `@ComponentScan`
  - 특정 패키지 경로를 기반으로 `@Configuration`**에서 사용할** `@Component` **설정 클래스를 찾는다.** `@ComponenetScan`의 basePackages 프로퍼티값에 별도의 경로를 설정하지 않으면 `@ComponentScan`이 위치한 패키지가 루트 경로(Base Package)로 설정. 

<br>

`@SpringBootApplication` 어노테이션은 위 3가지 **어노테이션의 조합**이다.

<br>

### SpringBootApplication <span style="color: red;">패키지 경로</span> 주의할 것!

또 하나, 패키지 hierarchy를 구성하면서 SpringBootApplication 실행 클래스를 다른 경로로 둔 실수를 공유한다.

아래 이미지가 처음 작성한 프로젝트 패키지 경로이다.

![](https://lh3.googleusercontent.com/pw/ACtC-3c0Hkc7PJIePEHW-hfUZuhzz587GdEmdalBU37y2AKBkUb5AjF5rYowQwuyo-Ismdt1hZh2zrYyFclOOJHZ1aBU9z2uQiM-HDD-3E1EsnF8fomRxpXG153VLkYqkXJZxSqr11G4qa4vF5osRgNUjg4B5w=w720-h480-no?authuser=0)

위의 패키지 구조로 톰캣을 실행해서 postman으로 api를 확인했다. 

![](https://lh3.googleusercontent.com/pw/ACtC-3dbuFEH9OuQJHg_B3vzDYh9DzKCxTejY8EYRj88Scv9-v_C0Gc8Ub9bA2XWnDNPFw96k1q4E9mpG69aT8DD7p72dzsvO-LP83p4r-IQjWB7bK5URD8gS2WTxmNnHOot_M7D1EnZMXRAqW5NcuoyfxN9qw=w1442-h903-no?authuser=0)

그런데 index페이지가 작동하고, 나머지 페이지는 모두 404 에러가 발생했다. 컨트롤러가 리소스를 제대로 못찾고 있는 것이다.

아무리 찾아도 이해를 못하다가 다른 분이 작성한 프로젝트의 패키지 경로를 보고 내가 잘못 생성했음을 인지했다. 이번엔 계층구조를 아래처럼 수정해보았다.

![](https://lh3.googleusercontent.com/pw/ACtC-3ey8sxu7uKzX79qr_KYDn3FgNPBFU366cDpi9a3r1mfnm_M_iraKM2oy5jiGmIwRAKTqSMe8G9ea9JFEOqMCvpmfAq2SgR3h0j38agC0Pym4sHp4EwoL-fhMgVHWTC6sCcKdRVFGZ8_oe419oR7JcwUFw=w720-h480-no?authuser=0)

그리고 톰캣을 다시 실행해서 postman으로 확인했다.

![](https://lh3.googleusercontent.com/pw/ACtC-3cMBKPol7vETGyScZSSYVstRXi7etY4mMXyzRZiQYfi40BgvRqIV-uU8ZyEbXRw2Yo83eeUF5RaospGXyjpJhZj29vFLm0xE5SxSEQV-AFLeyDH1f4x7GRossMoF12SIivjTSKPlxrV0hi5IIHyxYh-sA=w1442-h903-no?authuser=0)

패키지 계층 구조를 달리하니 API가 정상적으로 작동하는 것을 확인했다.

<br>

## <a name="build-tools"></a>빌드툴(Maven/Gradle)이 하는 일

Maven, Gradle같은 빌드 툴은 아래의 역할을 순차적으로 수행한다.

- **Manage libraries.**
  - 프로그램에 필요한 라이브러리들을 관리한다.
- **Build program. (Compile source into binary)**
  - 프로그램을 컴파일하고, 지정된 디렉토리에 resource를 모아서 프로그램을 빌드한다. 이 때 필요한 라이브러리가 있을 경우, 파일을 설치하기도 한다.
- **Test and run.**
  - 빌드가 끝나면 프로그램의 실행 전, 테스트를 진행한다. 테스트가 끝나면, 비로서 프로그램을 실행한다. 
- **Deployment**
  - 빌드한 프로그램을 배포하는 기능을 제공한다.



**같이 읽어보면 좋을 글**

- [Stackoverflow - What is a build tool?](https://stackoverflow.com/questions/7249871/what-is-a-build-tool)

- [별의역사 - 빌드 툴(Build Tool)](https://starrykss.tistory.com/276)

<br>

## <a name="annotations"></a>어노테이션

스프링 프레임워크에서 어노테이션이 하는 역할

XML을 분리하는 역할을 수행한다. 

결합도를 낮추고 유지보수성을 높이기 위해 xml로 설정하였으나 xml이 너무 많아지면 오히려 유지보수성이 낮아지는 아이러니한 상황이 발생할 수 있다. 

어노테이션은 유지보수성에 방점을 둔다.

어노테이션을 한 문장으로 정의한다면, 클래스와 필드, 메서드 등의 애플리케이션 요소의 다양한 종류의 정보를 주입하는 방법이라고 할 수 있다.



### 스프링에서 자주 사용하는 어노테이션

**의존성 주입용도**

- @Required
  - setter 메서드에 사용하는 어노테이션이다.
  - bean property 구성시 XML 설정파일에 반드시 property를 채우도록 하는 어노테이션이다. 만약 이를 지키지 않을시, BeanInitializationException이 발생한다.
- @Autowired
  - org.springframework.beans.factory.annotation.Autowired
  - Type에 따라 알아서 Bean을 주입한다.
    - type을 먼저 확인하고, type을 찾지 못하면 name에 따라 주입한다.
    - name으로 강제하려면 @Qualifier를 함께 명시하면 됨.
  - bean을 주입하는 3가지 방법
    - @Autowired
    - setter
    - 생성자 (@AllArgsConstructor)
- @Inject
  - @Autowired와 비슷한 역할을 한다.

**컨트롤러 관련**

- @Controller

- @RestController

  - = @Controller + @ResponseBody

  - @ResponseBody 어노테이션을 모든 메서드에 적용하는 어노테이션이다.

  - 메서드의 반환 결과를 JSON 형태로 반환한다.

  - @RequestMapping 이 기본적으로 @ResponseBody 의미를 가정한다.

  - @Controller와 @RestController의 차이

    - @Controller는 API와 View를 동시에 사용하는 경우에 사용하는 어노테이션.
    - @RestController는 View가 필요없는 API만 지원하는 서비스에 사용하는 어노테이션.

    

- @RequestMapping

- @PathVariable

- @RequestBody

- @RequestParam

- @ResponseBody

**데이터 접근 관련**

- @Service

- @Repository

- @Entity

  - 실제 DB의 테이블과 매칭될 클래스를 명시
  - DTO 클래스와 Entity 클래스를 분리하는 이유
    - 테이블과 매핑되는 Entity 클래스가 변경되면 여러 클래스에 영향을 끼치게 되는 반면 View와 통신하는 DTO 클래스는 자주 변경되므로 이 둘을 분리하는 것이 좋다.
    - DTO 클래스 = View Layer
    - Entity 클래스 = DB Layer

- @Table

  - Entity 클래스에 매핑할 테이블 정보를 알려준다.
  - @Table 어노테이션 생략시, 클래스 이름이 테이블 이름으로 매핑된다.

- @Id

  - 테이블의 PK필드

- @GeneratedValue

  - PK의 생성규칙이다.

    





- @Component
  - 해당 클래스가 Controller/Service/Repository로 사용됨을 Spring에 알려주는 어노테이션.
  - @Component는 아래의 어노테이션으로 구체화 할 수 있다.
    - @Controller
    - @Serivce
    - @Repository
- @Qualifier
  - 같은 타입의 빈이 두개 이상 존재할 경우에 스프링이 어떤 빈을 주입해야 할지 알 수 없어서 스프링 컨테이너를 초기화하는 과정에서 예외를 발생시킨다.
- @Transactional
  - 메서드 내에서 Exception이 발생하면 해당 메서드에서 이루어진 모든 DB 작업을 초기화(Rollback)한다.
  - 즉 모든 커밋이 정상적일때에만 커밋이 이루어진다.
  - DB를 등록/수정/삭제하는 Service 메서드에 필수적으로 필요한 어노테이션.
- @Resource
- @Scope
- @PostConstruct
- @PreDestory
- @RequestHeader
- @CookieValue
- @ModelAttribute
- @SessionAttibute
- @InitBinder
- @ControllerAdvice

**참고 문서**

- [이재민 - Spring Framework Annotation](https://medium.com/@2xel/spring-framework-annotation-%EA%B0%9C%EB%85%90-c26c15716538)
- [평범한개발자노트 - 스프링 어노테이션 종류](https://cornswrold.tistory.com/8)
- [Aaaalpooo - 많이쓰는 스프링 프레임워크 어노테이션 정리](https://medium.com/@aaaalpooo/%EB%A7%8E%EC%9D%B4-%EC%93%B0%EB%8A%94-spring-framework-annotation-%EC%A0%95%EB%A6%AC-summary-of-annotations-frequently-used-in-spring-framework-935e1c1a4877)
- [권희정 - Spring Annotation의 종류와 그 역할](https://gmlwjd9405.github.io/2018/12/02/spring-annotation-types.html)
- [Spring Framework Guru](https://springframework.guru/spring-framework-annotations/)

<br>

## <a name="error"></a>에러페이지 핸들링 (a.k.a 404 페이지 커스터마이징)

솔직히 이걸 현재의 수준으로 작성해도 될지 자신없으나.. 일단 **"기록"**에 의미를 두며 남겨본다. 혹시나 다른 분들이 이 부분을 보시게된다면, 그냥 건너띄시거나 구글에서 다른 글을 자세히 읽어보시길 꼭 권한다.

현재 진행중인 프로젝트에서 `resource/static/` 에 `/error` 라는 디렉토리를 만들고 여기에 404.html 파일과 500.html파일을 넣었다. 

![](https://www.mediafire.com/convkey/941e/5d7kwvr34i90pwuzg.jpg)

그럼 이것만으로도 HTTP 상태코드에 맞춰서 페이지를 띄어준다. 

이 때 서버에서 에러 메세지를 정확히 보고싶다면, 프로젝트의 `/main/java/` 디렉토리 아래에 `error/` 패키지를 생성하고, `ErrorController.java` 와 같은 이름의 컨트롤러를 생성해준다.

`ErrorController.java`

```java
package devandy.til.error;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class ErrorHandler {
   @GetMapping("/error")
   public String errorpage(){
      throw new IllegalStateException("Error");
   }
}
```

위의 코드를 입력하면 사용자가 에러를 만났을때, 에러 메세지를 콘솔로 출력받을 수 있다.

추가적인 학습/설명이 필요하다면 이 링크[(갱그리-Spring Boot 에러페이지 Customizing)](https://brunch.co.kr/@ourlove/70)를 통해 꼭 해보길 바란다.

<br>

## <a name="get-mapping-multi"></a>@GetMapping 어노테이션으로 다중맵핑하기

파라미터안에 `value = { , }` 형식으로 작성하면 `@GetMapping` 어노테이션으로 다중맵핑이 가능하다.

웹 애플리케이션의 메인 페이지로 이동하는 3개의 키워드를 모두 index.html로 넘기고 싶어서 작성한 코드이다.

~~~java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class homeController {
   @GetMapping(value = {"/", "/index", "/main"})
   public String index(){
      return "index";
   }
}
~~~

<br>

## <a name="h2-databse"></a>h2 데이터베이스 마이그레이션

h2데이터 베이스란 **컴퓨터에 내장된 램(RAM)메모리에 의지하는 데이터베이스**를 말한다. 테스트 또는 지금의 나처럼 실습을 위해서는 간편하고 빠르기 때문에 좋은 옵션이 될 수 있다.

다만, 램에 데이터를 저장하다보니 웹서버를 재부팅하면 기존의 데이터가 사라진다는 단점이 있다. 따라서 이 때엔 테스트에 필요한 데이터를 미리 sql로 작성해두고 웹서버 재부팅시마다 데이터를 인위적으로 주입하여 테스트 해볼 수 있다.

스프링부트 프로젝트에서 h2 데이터 베이스를 사용하는 방법은 다음과 같다.



### 의존성 설정

**Maven**

~~~xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
   <groupId>com.h2database</groupId>
   <artifactId>h2</artifactId>
</dependency>
~~~

**Gradle**

~~~gradle
// https://mvnrepository.com/artifact/com.h2database/h2
testCompile group: 'com.h2database', name: 'h2', version: '1.4.200'
~~~



### application.yml 설정

~~~yml
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

spring.datasource.url=jdbc:h2:mem:devandy;
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
~~~

이제 데이터베이스가 사용가능하며, `localhost:8080/h2-console` 에서 대시보드를 이용할 수도 있다.

![](https://lh3.googleusercontent.com/VD97biPg5fcx8KwvetWuiwQ9Atr81zPmhwlMntynUvyK7ICh1Qq8Pk34_9Wa6_YgJAd_hDRDvEy14LYYVALxi-E0_Jrv7wBChRywueHMVyuJfOj2fRlIT2H6vxhLv30jklpyrOXDCmMoy6dWncPF0rC3CSvE4_pfkbi4qIWBScKBl_Y69eNoJv-JZAGgxd1RcA-rbk63cEfPFEgrCl3b8_v0_VCV3vTEbIgwV_huEb-7BGIbvZxAtdcaNTkv1LsJaquRis_vkPjkfzsu2LzgPXGfcBTNg8KWqtvNMtq1b3fcrU49tmzX_oIvKYU9IXeA011fG6oI6hsEmpY4BNW91Sn10Q0vgXRtxgsRV1DWySyJD_jW0_aFURz__PqEc5Xn3WVAERTso52Autvn07ejtg21fr84d3lk3qTaKozgbEJgwBJDflpvrV64ODEmjeXquMTQ9FwTsi2-NFSjrazPlak4LNHPXsCP69SdEUZ5STE8JKD99fiD9a96UYeml-EyHyvDzNz6MnRnXmkLjgrnd4Zj7sV8qUad990r6EQNp9JVtY4mGIL-zCXMhvC2SVLO-PEyirrUZ7DEnL5UbnvpHWLfUluFSdk09BupqW9H_Vt35slM0iOKbliBTsxo7MCSzAjVTEP3kXaNkqUiLTLZUpSP79ZA6GBCcUaUiqslpvfdLxMIZoFqOY2sITcDRSKZpTURnKtXbjo5aEURXdaxBDNinAr1oy7rzywUkf1_IjaN3dLKoQ=w509-h402-no)

![](https://lh3.googleusercontent.com/7g4emF9IGksv-m0sPpFAR3zI979_RKRss2Ebksu4bCBxCAZ-XOVqS62PyLWFyX4qXQ8VnP7iFhlu2xfqW0XU4AL63SG9uoYtsWY68bUm4P1AqC-0A2nX0JQZnqD5JkwwCcR9yx1fY2QSDeYvnTPJDxf9QntTAX95xGi1iIsdYIKDd9Eg9zkgaRguWEnpksmoUuJTj4FIgeOQjveE1f1YuVRLRgSwOSBU-WMVhvmd6O0KuVFpNra3i934fhG3gUa8RohlhHSdXlKzaleHwZYYnJEHI1m601NBY_h2LQinHkCmx_wNIzSReWKJHHW_wNmTlRBS1WlG1NvUgvUTy8AEbaeMThJmQm3QUbLHxmFK8B3E-XcG-NLRYjyvVppUcMn3_hngSfPcDvzjRT1bkSZ3fWv6k_3o5vyXd5b1amWmAxVS_rZJ_RGZOh6iaLPLarDaD4NQ55D8w-8mjXm7I7ZDJQh3Ta2qWfOlQb0ta0mB2yzvuC3JjoLWsf4V6o9lv07q6kO7IJ2yTNC6YPz3UGJlLHgIbUomuvJsflbuNSOJPIO58WSThBLRUdQQRTuQnKhbXa6_grfdFmQVqrB87rs11LkZVIpSCGELnf5U5kDZUHa643w1HIbPcMKX1x75OThFKUOr-amtqFBPA48mpzsodaW79YqNXSoTHuJvDAEgelED-wKxoXScnGoBWIYWlPbPmJkJtvnnkOe2qwt7duRGAa8SXjA2cNOTZ46IduCRpd76B4pokA=w864-h778-no)

<br>

## <a name="datasource-autocofig"></a>DB 에러발생 무시하고 프로젝트 실행하기

스프링부트 프로젝트의 main클래스에 어노테이션을 추가한다.

~~~java
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
@SpringBootApplication
public class DevAndy {
   public static void main(String[] args) {
      SpringApplication.run(ToDoApplication.class, args);
   }
}
~~~

이렇게 하면, Datasource가 없어도, 즉 DB 연결을 필요로 하는 프로젝트인데, DB가 연결되지 않은 경우에도 프로젝트를 실행할 수 있다.

<br>

## <a name="gradlew-version"></a>Gradle 버전확인하는 법

Gradle 프로젝트 디렉토리 내에서 아래의 명령어를 입력하면 Gradle의 버전 및 개발환경을 콘솔에 출력한다.

~~~bash
$ ./gradlew --version
~~~

![](https://lh3.googleusercontent.com/pw/ACtC-3fwbH6tqvImZYqrGcBw-Hm6XnkHU5m7jalTIKoOHZXESRTolKdFRghMai1EQoYZSuyTP6QPKIa8U2yGi6HhGpL4xnWcjxrGp9XyWZBMPNEZ6qa7rKwE-SsjK1hiCxCQ6hnn_RNuEkAnQxdGyzgmWST5FQ=w585-h492-no?authuser=0)

위의 방법은 프로젝트에 적용된 gradle 버전을 확인하는 방법이고, 운영체제에 있는 gradle 버전을 확인하는 방법은 다음과 같다.

~~~bash
$ gradle -v
~~~

<br>

## <a name="upgrade-gradle"></a>프로젝트에서 gradle 버전 올리기

프로젝트 디렉토리에 [gradle] - [wrapper] 하위에 gradle-wrapper.properties 라는 파일이 있다. 여기서 원하는 버전으로 변경후 gradle을 refresh하면 새로운 버전의 gradle로 변경된다.

![](https://lh3.googleusercontent.com/pw/ACtC-3fRfZOwWUHdp5aDKfKZ_MgHKTCJGquZKIluhn6Fj99a40mk8cIkJUAPW2fT_DKE2py-GxZNjU-15SQ-raWuMM1JivslcDRdmueMiNw6Lp1wHIyErafahHQAXPTAarUTRMo3WiyAgxCGlvgtW9GWX5cOqQ=w1440-h480-no?authuser=0)

<br>

## <a name="gradle-which-version"></a>Gradle 다운그레이드

이동욱님이 작성하신 스프링 부트와 AWS로 혼자구현하는 서비스를 보며 따라 만들어보는 중인데, gradle 버전의 차이로 책에서 작성한 코드가 컴파일 되지 않았다. 그래서 프로젝트 저장소에 등록된 이슈를 통해 gradle 버전을 동욱님이 책을 집필하시던 때의 버전과 맞추었다. 이 때 사용한 명령어는 다음과 같다.

~~~bash
$ ./gradlew wrapper --gradle-version 4.10.2
~~~

이 때 위의 명령어는 gradle 프로젝트 내부의 콘솔에서 입력해야한다. 꼭 다운그레이드 뿐 아니라 원하는 버전으로 gradle 버전을 변경할 수 있을것 같다.

<br>

## <a name="unit-test"></a>Unit Test

Unit Test의 FIRST 원칙

- Fast 테스트 코드를 실행하는 일은 오래걸리면 안된다.
- Independent 독립적으로 실행되어야 한다.
- Repeatable 반복가능해야한다.
- Self Validating 메뉴얼 없이 테스트 코드만 실행해도 성공/실패 여부를 알 수 있어야 한다.
- Timely 바로 사용 가능해야 한다.

출처 : https://brunch.co.kr/@springboot/207

<br>