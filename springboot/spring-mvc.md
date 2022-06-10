# Spring MVC 구성 변경

Spring MVC를 사용하면 많은 부분을 Spring이 자동으로 설정해주기 때문에 편리하다. 앱을 개발하다보면 자동으로 구성된 스프링 MVC 구성에 Formatter, MessageConverter 등을 큰 변경없이 추가하거나 변경할 일이 생기는데 WebMvcConfigurer 인터페이스를 구현하면 된다. WebMvcRegistrations는 RequestMappingHandlerMapping, RequestMappingHandlerAdapter와 ExceptionHandlerExceptionResolver를 재정의할 때 사용한다.

## 자동 구성된 Spring MVC 구성 변경

자동 구성된 스프링 MVC 구성을 변경하려면 com.sogood.core.config 패키지를 생성하고, DispatcherClass.java 파일을 생성한 다음에 다음과 같이 코드를 정의한다.

```java
@Configuration
public class DispatcherConfig implements WebMvcConfigurer, WebMvcRegistrations {
}
```

자동 구성된 스프링 MVC 구성의 제어권을 가져오려면 다음과 같이 @EnableWebMvc를 함께 적용해야 한다.

```java
@Configuration
@EnableWebMvc
public class DispatcherConfig implements WebMvcConfigurer, WebMvcRegistrations {
}
```

@EnableWebMvc를 선언하면 WebMvcConfigurationSupport에서 구성한 스프링 MVC 구성을 불러온다.@EnableWebMvc는 SpringMvc를 구성할 때 필요한 Bean 설정등를 자동으로 등록해주는 어노테이션인데, handlerMapping,HandlerAdapter 등을 자동 구성한다.

### 정적 리소스 폴더 문제

그런데 @EnableWebMvc 어노테이션을 적용하면 src/main/resources 소스 폴더의 /public, /static 등 정적 파일을 두는 폴더에 있는 파일들을 브라우저에서 직접 호출하지 못한다. 예를 들어, /static 폴더에 test.html이 있다고 가정하면 다음과 같이 직접 브라우저에서 호출할 수 있다.

```shell
http://localhost:8080/test.html
```

그러나 @EnableWebMvc 어노테이션을 적용하면 브라우저에서 직접 호출이 되지 않는다. 스프링 MVC 자동구성은 WebMvcAutoConfiguration이 담당한다. 이 구성이 활성화되는 조건 중에 WebMvcConfigurationSupport 타입의 빈을 찾을 수 없을 때 라는 조건이 있다. @EnableWebMvc를 스프링 MVC 구성을 위한 클래스에 선언하면 WebMvcConfigurationSupport를 불러와 스프링 MVC를 구성한다. WebMvcConfigurationSupport가 컴포넌트로 등록되면 WebMvcAutoConfiguration은 비활성화 되기 때문이다.

이전과 같이 정적 리소스를 직접 접근하려면 addResourceHandlers() 를 오버라이드해야 한다.

```java
  @Override
  public void addResourceHandlers(ResourceHandlerRegistry registry) {
     registry.addResourceHandler("/**").addResourceLocations( "classpath:/public/", "classpath:/static/" );
  }
```

### DefaultServletHandler 설정

DispatcherServlet의 매핑이 "/"로 지정하면 JSP를 제외한 모든 요청이 DispatcherServlet으로 가기 때문에 WAS가 제공하는 Default Servlet이 \*.html, \*.css같은 요청을 처리할 수 없게된다. Default ServletHandler는 이런 요청들을 Default Servlet에게 전달해주는 Handler이다. Deafult ServletHandler를 활성화 하려면 다음과 같이 한다.

```java
  public void configureDefaultServletHandling(org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer configurer) {
    configurer.enable();
  }
```

**defaultServletName 오류**\
@EnableWebMvc 어노테이션을 설정 후에 Spring Boot 2.4로 업그레드한 후 더이상 시작되지 않는다. 다음과 같은 오류가 있다.

```shell
nable to locate the default servlet for serving static content. Please set the 'defaultServletName' property explicitly.
```

Spring 2.4 릴리스 노트에 기술된 내용에는 DefaultServlet은 임베디드 서블릿 컨테이너에 더 이상 기본적으로 등록되지 않는다. applicaion.yml 파일에 다음과 같이 등록한다.

```yaml
server:
  servlet:
    register-default-servlet: true
```
