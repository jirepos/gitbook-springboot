# Logging 처리

이 튜터리얼에서는 Spring Boot에서 Logging을 처리하는 방법에 대해서 설명한다.

Spring Boot에서는 기본적으로 Logback이 선택되어 있어 pom.xml 파일에 의존성을 주입할 필요는 없다.

```
📁src/main/java
  📁com
    📁sogood
      📁core
        📁log
          📄 Log.java
          📄 LogInjector.java
```

## 로깅처리를 위한 Annotation 정의

src/main/java 소스 폴더에 com.sogood.core.log 패키지를 생성한다. 그 안에 Log.java 파일을 생성한다. 다음과 같이 코드를 작성한다.

```java
package com.sogood.core.log;

import static java.lang.annotation.ElementType.FIELD;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

import java.lang.annotation.Documented;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;

/** 
 * Logger를 삽입하기 위한 어노테이션이다. 
 */
@Retention(RUNTIME)
@Target(FIELD)
@Documented
public @interface Log {

}
```

## 로거 주입을 위한 Component 생성

SPring Bean 파일에 Logger를 주입하기 위한 LoggerInjector.java 파일을 com.sogood.core.log 패키지에 생성하고 다음과 같이 작성한다.

```java
package com.sogood.core.log;


import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.stereotype.Component;
import org.springframework.util.ReflectionUtils;

import java.lang.reflect.Field;

/**
 * Log 어노테이션이 적용된 클래스 Logger 객체를 주입한다. 
 */
@Component
public class LogInjector implements BeanPostProcessor {

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    @Override
    public Object postProcessBeforeInitialization(final Object bean, String name) throws BeansException {
    	//System.out.println("※ beforeInitialization:" + name);
        ReflectionUtils.doWithFields(bean.getClass(), new ReflectionUtils.FieldCallback() {
            public void doWith(Field field) throws IllegalArgumentException, IllegalAccessException {
                // make the field accessible if defined private
                ReflectionUtils.makeAccessible(field);
                if (field.getAnnotation(Log.class) != null) {
                    Logger log = LoggerFactory.getLogger(bean.getClass());
                    field.set(bean, log);
                }
            }
        });
        return bean;
    }
}
```

## 클래스에 Logger 멤버 선언

Controller,Service, DAO와 같은 Spring Bean에 Logger 변수를 선언해야 한다. com.sogood.demo 패키지 아래에 DemoController.java가 있을 것이다. 없으면 생성하고 다음과 같이 코드를 작성한다.

```java
package com.sogood.demo;

import com.sogood.core.log.Log;

import org.slf4j.Logger;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class DemoController {
	/** Logger */
	protected static @Log Logger logger;

	@GetMapping("/demo")
	public String test() {
		return "demo";
	}
}/// ~
```

Spring Boot 어플리케이션이 시작되면서 자동적으로 @Log 어노테이션을 사용한 멤버에 Logger를 주입할 것이다. 그럼 제대로 로깅이 되는지 확인하기 위해서 다음과 같이 test() 메소드에 코드를 추가한다.

```java
@GetMapping("/demo")
public String test() {
	logger.debug(">>>> DEBUG");
	return "demo";
}
```

## Log Level 조절

마지막으로 com.sogood 패키지의 로그 레벨을 설정해야 한다. applicaion.yaml 파일을 열고 다음을 추가한다.

```yaml
# loggin section
logging:
  level:
     root: INFO 
     com:
        sogood: DEBUG
```

이제 브라우저에서 /demo URL을 호출해 보자.

```shell
http://localhost:8080/demo
```

다음과 같이 콘솔에 DEBUG라고 찍일 것이다.

```shell
2021-06-08 14:54:23.651 DEBUG 23556 --- [nio-8080-exec-5] com.sogood.demo.DemoController           : >>>> DEBUG
```

## Logback 설정

아래 설정은 프로젝트 루트 아래에 logs 폴덜를 만들고 sogood_app.log 파일을 만든다. max-size보다 sogood_app.log 파일이 크면 pattern.rolling-file-name 에서정의한 형식으로 파일을 생성한다.

```yaml
logging:
  file:
    path: logs/sogood_app.log
    pattern:
      console: "%d %-5level %logger : %msg%n"
      #file:  "%d %-5level [%thread] %logger : %msg%n"
      rolling-file-name: sogood-app-%d{yyyy-MM-dd-HH-mm}.%i.log
    max-size: 500MB
    max-history: 10
  level:
    root: ERROR
    org.springframework.web: ERROR
    com.sogood: DEBUG 
```

**loging.file.max-size** 로그파일의 최대 크기. **logging.file.max-history**\
최대 몇일 동안 로그를 저장할 것인지 설정.

**loggin.file.total-size-cap** 모든 로그파일의 최대 크기.

### 로그 형식

```shell
%d{HH:mm} %-5level %logger{36} - %msg%n
```

**%d{HH:mm}** 로그가 출력되는 시간이 출력된다. 중괄호{ } 안은 이 시간의 포맷이다.

**%-5level** 로그 레벨을 5의 고정폭 값으로 출력하라는 것을 의미한다.

* ERROR 에러
* WARN 주의
* INFO 정보
* DEBUG 디버그
* TRACE 경로추적

패키지를 기준으로 로그레벨을 설정하려면 다음과 같이 한다.

```yaml
  level:
    root: INFO
    spring.framework: INFO
    com.sogood: DEBUG
```

**%logger{length}** logger의 이름을 축약해서 출력한다. 중괄호{ } 안에는 length이다. 최대 자릿수를 의미한다.

**%thread** 현재 Thread name

**%msg** %message의 alias로, 사용자가 출력한 메시지가 출력된다. %n은 줄바꿈이다.

### Appender

Appender는 어디에 어떤 포맷으로 로그를 남길 것인지 정할 수 있는 방법을 제공한다. 대표적으로 ConsoleAppender, FileAppender, RollingFileAppender 등이 있다.

ConsoleAppender는 콘솔에 로그를 어떤 포맷으로 출력할지 설정할 때 사용하고 기본적으로 기록되는 파일명은 file 요소에 입력되어 있는 access.log이다. FileAppender는 파일에 로그를 어떤 포맷으로 출력할지 설정할 때 사용한다.

**파일 이름 패턴**\
%d{} 내부의 dateTime 패턴 안에 '/' 또는 '' 기호는 디렉터리 구분자로 인식한다. 이를 통해 시간으로 원하는 디렉터리 구조를 구성할 수 있다.

```
/var/log/%d{yyyy/MM}/soogood.%d{yyyy-MM-dd}.%i.log
```

**%i** 같은 기준의 파일이 생성될 때 파일이름이 중복되지 않게 서수를 붙인다.

* **/app.%d** - default. %d는 yyyy-MM-dd. 매일 자정에 새로운 로그파일을 rollover한다.
* **/app/%d{yyyy/MM}/app.log** 매월 새로운 디렉터리에 app.log 파일을 만든다.
* **/app/%d{yyyy-MM-dd_HH-mm}** 매 분 새로운 로그 파일로 rollover한다.
* **/app/app.%d.gz** 매일 새로운 로그 파일로 rollover하고 이전 로그파일을 GZIP으로 압축한다.
