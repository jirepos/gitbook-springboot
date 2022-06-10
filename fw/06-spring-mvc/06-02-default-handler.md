# DefaultServletHandler 설정


DispatcherServlet의 매핑이 "/"로 지정하면 JSP를 제외한 모든 요청이 DispatcherServlet으로 가기 때문에 WAS가 제공하는 Default Servlet이 *.html, *.css같은 요청을 처리할 수 없게 된다. Default ServletHandler는 이런 요청들을 Default Servlet에게 전달해주는 Handler이다. Deafult ServletHandler를 활성화 하려면 다음과 같이 한다.

```java
@EnableWebMvc
@Configuration
public class DispatcherConfig implements WebMvcConfigurer {
  public void configureDefaultServletHandling(org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer configurer) {
    configurer.enable();
  }
}
```

**디폴트 서블릿은 정적 자원들과 디렉토리 목록(디렉토리 목록이 지원된다면)을 지원하는 서블릿이다.**URL에 매핑되는 Controller가 없을 경우, Default Servlet이 처리하게 하는 설정 추가해야 한다. 

Spring Boot 내장 톰캣에 대한 자세한 설정은 다른 문서를 참고하고 우선 다음과 같이 설정한다.  

- 포트 번호를 80번으로 변경한다.
- register-default-servlet: true 를 정의하여 Default Servlet을 활성화 한다.

**defaultServletName 오류**

@EnableWebMvc 어노테이션을 설정 후에 Spring Boot 2.4로 업그레드한 후 더이상 시작되지 않는다.  다음과 같은 오류가 있다

```java
enable to locate the default servlet for serving static content. Please set the 'defaultServletName' property explicitly.
```

Spring 2.4 릴리스 노트에 기술된 내용에는 DefaultServlet은 임베디드 서블릿 컨테이너에 더 이상 기본적으로 등록되지 않는다. applicaion.yml 파일에 다음과 같이 등록한다.

```java
server:
  servlet:
    register-default-servlet: true
```