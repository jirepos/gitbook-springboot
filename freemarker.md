# Freemarker 사용

이 튜터리얼에서는 Spring Boot에서 Freemarker Template Engine을 사용하는 방법을 설명한다.

## 의존성 추가

freemarker를 사용하기 위해서 pom.xml에 freemarker에 대한 의존성을 추가해야 한다.

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-freemarker</artifactId>
 </dependency>
```

ViewResolver가 확장자를 자동으로 추가하기 때문에 hello.ftl을 모두 작성할 필요는 없다. hello만 입력한다.

## application.properties 설정

SpringBoot가 Freemarker 파일을 처리할 수 있도록 application.properties 파일에 FreeMarker 설정을 추가한다.

```properties
# freemarker 
spring.freemarker.template-loader-path=classpath:/templates 
spring.freemarker.suffix=.ftl
```

SpringBoot에서 Freemarker를 사용하여 View를 작성하는 방법을 알아보자.

## hello.ftl 생성

src/main/resources 소스 폴더의 templates 폴더 아래에 hello.ftl 파일을 생성하고 다음과 같이 작성한다.

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Home page</title>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
    </head>
    <body>
        <h2>hello</h2>
    </body>
</html>
```

## HelloController.java 생성

HelloContoller를 생성하고 GetMapping을 사용하여 URL을 매핑한다.

```java
package com.sogood.demo;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller 
public class HelloController {
  @GetMapping("/hello")
	public String test() {
		return "hello";
	}
}///~
```

## URL 호출

SpringBoot의 내장 톰캣의 포트를 변경하지 않았기 때문에 8080 포트 번호를 붙여 URL을 호출해야 한다.

```shell
http://localhost:8080/hello
```

브라우저에서 hello 인사말을 볼 수 있을 것이다.
