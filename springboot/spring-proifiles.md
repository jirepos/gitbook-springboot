# Spring Proifiles 적용

Spring Boot를 사용하여 개발환경, 검증환경, 운영환경 마다 다른 설정값을 가지도록 application.yml 파일을 사용한다.

**Spring Boot 2.4 부터는 spring.profiles 가 deprecated 되었다.**

## profile 정의

application.yml 파일에서 프로파일의 구분은 '---'을 사용한다. 'common'이라는 프로파일을 하나 만들어 보자.

```yaml
---
spring:
  config:
    activate:
      on-profile: common
```

방금 만든 profile에 'user'라는 프러퍼티를 하나 정의한다. user 프러퍼티는 들여쓰기 하지 않는다.

```yaml
---
spring:
  config:
    activate:
      on-profile: common
user: SoGood      
```

localdb라는 프로파일을 하나 더 만든다. 지금까지 설정한 내용은 아래와 같이 보일 것이다.

```yaml
---
spring:
  config:
    activate:
      on-profile: common
user: SoGood      

---
spring:
  config:
    activate:
      on-profile: localdb
database: mysql 
```

이제 'spring.profiles.active=local' 과 같이 적용할 프로파일 그룹을 정의해야 한다. 프로파일 그룹은 다음과 같이 정의한다. 프로파일 그룹인 local은 'common' 프로파일과 'localdb' 프로파일을 포함한다. 'local' 프로파일 그룹은 local pc의 개발환경에 적용할 프로파일들을 그룹핑한다고 보면 된다.

```yaml
---
spring:
  profiles:
    group:
      local: common, localdb
```

이제 운영환경에 적용할 프로파일인 'proddb' 프로파일을 정의한다.

```yaml
---
spring:
  config:
    activate:
      on-profile: proddb
database: oracle 
```

운영환경 프로파일 그룹인 'prod'를 정의한다.

```yaml
---
spring:
  profiles:
    group:
      prod: common,proddb
```

전체는 다음과 같이 보일 것이다.

```yaml
server:
  servlet:
    register-default-servlet: true

spring:
  #spring.freemarker.template-loader-path=classpath:/templates 
  freemarker:
    suffix: .ftl
    template-loader-path: classpath:/templates 
  web:
    resources:
      static-locations:
        - classpath:/static
        - classpath:/public

# loggin section
logging:
  level:
     root: INFO 
     com:
        sogood: DEBUG

# Spring Profiles 
---
spring:
  profiles:
    group:
      local: common, localdb
---
spring:
  profiles:
    group:
      prod: common,proddb

---
spring:
  config:
    activate:
      on-profile: common
user: sogood


---
spring:
  config:
    activate:
      on-profile: localdb
database: mysql 

---
spring:
  config:
    activate:
      on-profile: proddb
database: oracle 
```

프로파일 설정 윗 부분은 전역 설정이다. 프로파일을 스프링이 알게하려면 여러가지 방법이 있는데 System.setProperty()를 일단 이용하자. "spring.profiles.active" 프러퍼티 명을 사용한다. 값에는 사용하려고 하는 프로파일 그룹명을 설정한다.

```java
package com.sogood;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SogoodApplication {
	public static void main(String[] args) {
		System.setProperty("spring.profiles.active", "local");
		SpringApplication.run(SogoodApplication.class, args);
	}
}///~
```

YAMLConfig.java 파일을 다음과 같이 작성한다.

```java
package com.sogood.core.config;


import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
//@ConfigurationProperties(prefix="app")
@ConfigurationProperties
public class YAMLConfig {

  private String database;
  private String user; 
  

  public String getUser() {
    return user;
  }

  public void setUser(String user) {
    this.user = user;
  }

  public String getDatabase() {
    return database;
  }

  public void setDatabase(String database) {
    this.database = database;
  }

}///~
```

DemoController.java의 test() 메소드를 다음과 같이 작성한다.

```java
package com.sogood.demo;

import com.sogood.core.config.YAMLConfig;
import com.sogood.core.config.YAMLConfig.Menu;
import com.sogood.core.log.Log;

import org.slf4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class DemoController {
	/** Logger */
	protected static @Log Logger logger;

	@Autowired
	private YAMLConfig yamlConfig;

	@GetMapping("/demo")
	public String test() {
		logger.debug(yamlConfig.getDatabase());
		logger.debug(yamlConfig.getUser());
		return "demo";
	}
}/// ~
```

브라우저에서 /demo를 요청한다. DEBUG CONSOLE에 다음과 같이 출력될 것이다.

```shell
2021-06-09 19:00:22.938 DEBUG 9512 --- [nio-8080-exec-1] com.sogood.demo.DemoController           : mysql
2021-06-09 19:00:22.939 DEBUG 9512 --- [nio-8080-exec-1] com.sogood.demo.DemoController           : sogood
```

이제 프로파일 그룹을 'prod'로 변경하자.

```java
public static void main(String[] args) {
		System.setProperty("spring.profiles.active", "prod");
		SpringApplication.run(SogoodApplication.class, args);
	}
```

브라우저에서 /demo를 다시 요청한다. DEBUG CONSOLE에 다음과 같이 출력될 것이다.

```shell
2021-06-09 19:02:49.030 DEBUG 9512 --- [nio-8080-exec-1] com.sogood.demo.DemoController           : oracle
2021-06-09 19:02:49.030 DEBUG 9512 --- [nio-8080-exec-1] com.sogood.demo.DemoController           : sogood
```

## 환경변수에 프로파일을 설정하는 방법

**programmatically**

```java
SpringApplication.setAdditionalProfiles("dev");
```

**Maven**

```xml
<plugins>
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
            <profiles>
                <profile>dev</profile>
            </profiles>
        </configuration>
    </plugin>
    ...
</plugins>
```
