# html과 같은 정적 파일 서비스

Spring Boot에서 html,CSS, javascript 파일과 같은 정적파일들을 Conroller를 거치지 않고 서비스하기 위해서는 src/main/resources 파일 아래에 폴더를 만든다. 일단, static으로 폴더를 만든다.

```
src/main/resources
  /static 
```

static 폴더 아래에 test.html을 생성한다. WebMvcConfigurer 인터페이스를 구현한 클래스에 addResourceHandlers() 메소드를 구현한다.

addResouresHandler() 메소드에 URL에서 사용할 path를 설정하고 addResourceLocations() 메소드에 클래스 패스를 적용한다.

```java
public class DispatcherConfig implements WebMvcConfigurer  {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/**").addResourceLocations("classpath:/static/");
    }
}
```

브라우저를 열고 http://localhost:8080/static/test.html을 호출한다.

## application.properties에서 설정하는 방법

```yaml
  mvc:
    statc-path-pattern: /static/**
  resources:
    static-locations: classpath:/static/    
```
