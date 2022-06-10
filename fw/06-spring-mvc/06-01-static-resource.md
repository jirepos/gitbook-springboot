# 정적 자원 처리
## 정적 리소스 폴더 문제

그런데 @EnableWebMvc 어노테이션을 적용하면 src/main/resources 소스 폴더의 /public, /static 등 정적 파일을 두는 폴더에 있는 파일들을 브라우저에서 직접 호출하지 못한다. 예를 들어, /static 폴더에 test.html이 있다고 가정하면 다음과 같이 직접 브라우저에서 호출할 수 있다.

```java
http://localhost:8080/test.html
```

그러나 **@EnableWebMvc 어노테이션을 적용하면 브라우저에서 직접 호출이 되지 않는다.** 스프링 MVC 자동구성은 WebMvcAutoConfiguration이 담당한다. 이 구성이 활성화되는 조건 중에 WebMvcConfigurationSupport 타입의 빈을 찾을 수 없을 때 라는 조건이 있다. @EnableWebMvc를 스프링 MVC 구성을 위한 클래스에 선언하면 WebMvcConfigurationSupport를 불러와 스프링 MVC를 구성한다. WebMvcConfigurationSupport가 컴포넌트로 등록되면 WebMvcAutoConfiguration은 비활성화 되기 때문이다.

이전과 같이 정적 리소스를 직접 접근하려면 addResourceHandlers() 를 오버라이드해야 한다.

```java
@Override
  public void addResourceHandlers(ResourceHandlerRegistry registry) {
     registry.addResourceHandler("/**")
             .addResourceLocations( "classpath:/public/", "classpath:/static/" );
  }
```

SpringBoot에서 정적 자원은 public, static에 둔다. 

- public과 static 폴더는 브라우저에서 직접 접근할 수 있다.
- templates는 Controller를 통해서 만 접근이 가능하다.

```java

🗂️src/main/resources
  🗂️public
  🗂️static
  🗂️templates
```

Spring Boot에서 html,CSS, javascript 파일과 같은 정적 파일들을 Conroller를 거치지 않고 서비스하기 위해서는 src/main/resources 파일 아래에 폴더를 만든다. 일단, static 으로 폴더를 만든다.

```java
src/main/resources
  /static
```

static 폴더 아래에 test.html을 생성한다. WebMvcConfigurer 인터페이스를 구현한 클래스에 addResourceHandlers() 메소드를 구현한다.

addResouresHandler() 메소드에 URL에서 사용할 path를 설정하고 addResourceLocations() 메소드에 클래스 패스를 적용한다

## application.yml에서 설정

```java
spring:
  web:
    resources:
      static-locations:
        - classpath:/static
        - classpath:/public
```