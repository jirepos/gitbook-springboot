# Spring MVC 구성 변경

Spring MVC를 사용하면 많은 부분을 Spring이 자동으로 설정해주기 때문에 편리하다.  앱을 개발하다 보면 자동으로 구성된 스프링 MVC 구성에 Formatter, MessageConverter 등을 큰 변경 없이 추가하거나 변경할 일이 생기는데 WebMvcConfigurer 인터페이스를 구현하면 된다. 

자동 구성된 스프링 MVC 구성을 변경하려면 com.sogood.core.config 패키지를 생성하고, DispatcherClass.java 파일을 생성한 다음에 다음과 같이 코드를 정의한다.

 

- 자동 구성된 스프링 MVC 구성의 제언권을 가져 오려면 EnableWebMvc 어노테이션을 적용해야 한다.

```java
@Configuration
@EnableWebMvc
public class DispatcherConfig implements WebMvcConfigurer {
}
```

@EnableWebMvc를 선언하면 WebMvcConfigurationSupport에서 구성한 스프링 MVC 구성을 불러온다.@EnableWebMvc는 SpringMvc를 구성할 때 필요한 Bean 설정 등를 자동으로 등록해주는 어노테이션인데, handlerMapping,HandlerAdapter 등을 자동 구성한다.