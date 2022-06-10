# YAML 설정 사용

이 글에서는 Spring Boot에서 application.properties 대신에 application.yml 파일을 사용하여 어플리케이션을 설정하는 방법에 대해서 설명한다.

Springboot가 설정파일을 읽을 때 다음을 기준으로 한다.

* src/main/resources 폴더 아래의 application이 접두사로 붙은 설정 파일을 순서대로 읽는다.
* 특별한 경우가 아니면 application.yml, applicationXXX.yml 파일들을 자동으로 읽는다.
* @PropertySource는 스프링에서도 권장하지 않는 방법이다.
* YAML 파일은 @PropertySource 어노테이션을 사용해서 불러 올 수 없다.

freemarker view respovler를 사용하기 위해 src/main/resources 폴더 아래에 application.yml을 생성한다. applicaiton.properties 파일은 application.properties.org로 변경한다. 다음과 같이 입력한다.

```yaml
spring:
  #spring.freemarker.template-loader-path=classpath:/templates 
  freemarker:
    suffix: .ftl
    template-loader-path: classpath:/templates 
```

## 설정값 읽기 - 1

application.yml에 설정된 프러퍼티의 값을 읽어보자. 먼저 ApplicationContext를 주입할 수 있도록 설정한다.

```java
@Autowired
private ApplicationContext applicationContext; 
```

application.yml 파일에 다음과 같이 프러퍼티릂 설정한다.

```yaml
i18n-message-files: com/sogood/i18n/message, com/sogood/i18n/common
```

getEnvironment().getProperty("propertyName")으로 설정값을 읽을 수 있다.

```java
System.out.println(applicationContext.getEnvironment().getProperty("i18n-message-files")); 
```

## 설정값 읽기 -2

application.yml에 정의되어 있는 프러터티들은 ApplicationContext.getEnvironment().getProperty()를 사용하여 읽을 수 있지만 Spring Bean이 아닌 일반 POJO에서도 이 값을 읽어야 하는 경우가 있다.

### ApplicationContextHolder

ApplicationContext 인스턴스를 가지고 있는 클래스를 정의한다. @Component 애너테이션을 붙이고 AppicationContextAware를 구현한다. 스프링이 로드시에 ApplicationContext 인스턴스를 주입한다. 주의할 것은 ApplicationContext 참조변수는 static으로 정의해야 한다.

```java
package com.sogood.core.config.util;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

@Component 
public class ApplicationContextHolder implements ApplicationContextAware {
  private static ApplicationContext applicationContext;

  @Override
  public void setApplicationContext(org.springframework.context.ApplicationContext arg0) throws BeansException {
    applicationContext = arg0; 
  } 
  public static ApplicationContext geApplicationContext() {
    return applicationContext; 
  }
}///~
```

### PropertyUtil

이제 설정값을 읽을 수 있는 유틸리티 클래스를 하나 만든다.

```java
package com.sogood.core.config.util;

import org.springframework.context.ApplicationContext;

public class PropertyUtil {
  
  public static String getProperty(String propertyName) {
    return getProperty(propertyName, null);
  }
  public static String getProperty(String propertyName, String defaultValue) {
    ApplicationContext applicationContext = ApplicationContextHolder.geApplicationContext();
    String value = applicationContext.getEnvironment().getProperty(propertyName);
    return (value == null)? value : defaultValue; 
  }
}
```

getProperty() 메서드를 이용하여 설정값을 읽을 수 있다.

```java
String uploadDir = PropertyUtil.getProperty("upload-dir");
```
