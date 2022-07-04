# 컨텍스트 분리


# Spring Boot에서의 Context 분리

**Spring Boot에서는 root web application context와 servlet web application context에 대한 구분이 없다**. 디폴트로 single web application context를 생성할 것이다. Embedded container 안에서 돌아가는 Spring Boot Web application들은 설계에 의해 어떠한 WebApplicationInitializer를 실행하지 않는다. 선택된 배포 전략에 따라 SpringBootInitializer 또는 ServletContextInitializer에서 같은 로직을 작성할 수는 있다. 그러나, servlets, filters, listeners 이 기사에서 보여진 것과같은 것들에 대해서는, 사실 Spring Boot 가 자동적으로 모든 서블릿 관련된 Bean을 컨테이너에 등록한다.

따라서 Spring Boot Web Application에서는 아래의 내용은 의미가 없다. Spring Boot에서는 @SpringBootApplication 어노테이션을 적용하여 Application의 진입점을 만든다. @SpringBootApplication 어노테이션을 적용할 Main Class는 패키지 루트에 생성해야 한다. 예를들어 com.sogood.common, com.sogood.biz 와 같은 형태로 패키지 구조를 만든다면 메인 클래스는 com.sogood 패키지 아래에 두어야 한다.

```java
📁src/main/java
  📁com
    📁sogood
      📄 SogoodApplicaion.java
      📁core
        📁log
```

@SpringBootApplication이 적용된 SogoodApplicaion.java가 있는 패키지가 basePackage가 되기 때문에 @ComponentScan을 적용할 필요가 없다. 간단히 SogoodApplicaion.java를 다음과 같이 작성한다.

```java
package com.sogood;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SogoodApplication {
	public static void main(String[] args) {
		SpringApplication.run(SogoodApplication.class, args);
	}
}///~
```