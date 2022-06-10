# SpringBoot 로깅 
## Default Logging
아무것도 설정하지 않은 상태에서의 로깅 

```shell
.   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.5.0)

2021-10-27 19:04:02.216  INFO 2736 --- [  restartedMain] com.sogood.SogoodApplication             : Starting SogoodApplication using Java 12.0.2 on hyeon with PID 2736 (G:\github\sogood-core\target\classes started by latte in g:\github\sogood-core)
2021-10-27 19:04:02.221  INFO 2736 --- [  restartedMain] com.sogood.SogoodApplication             : The following profiles are active: custom-properties.yml,prod,common,proddb,sqllog
2021-10-27 19:04:02.287  INFO 2736 --- [  restartedMain] o.s.b.devtools.restart.ChangeableUrls    : The Class-Path manifest attribute in d:\mavenrepository\com\oracle\database\jdbc\ojdbc8\21.1.0.0\ojdbc8-21.1.0.0.jar referenced one or more files that do not exist: file:/d:/mavenrepository/com/oracle/database/jdbc/ojdbc8/21.1.0.0/oraclepki.jar
2021-10-27 19:04:02.287  INFO 2736 --- [  restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : Devtools property defaults active! Set 'spring.devtools.add-properties' to 'false' to disable
2021-10-27 19:04:02.288  INFO 2736 --- [  restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : For additional web related logging consider setting the 'logging.level.web' property to 'DEBUG'
2021-10-27 19:04:03.143  INFO 2736 --- [  restartedMain] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data JDBC repositories in DEFAULT mode.
2021-10-27 19:04:03.177  INFO 2736 --- [  restartedMain] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 29 ms. Found 0 JDBC repository interfaces.
2021-10-27 19:04:03.328  WARN 2736 --- [  restartedMain] o.m.s.mapper.ClassPathMapperScanner      : Cannot use both: sqlSessionTemplate and sqlSessionFactory together. sqlSessionFactory is ignored.
4
2021-10-27 19:04:03.329  WARN 2736 --- [  restartedMain] o.m.s.mapper.ClassPathMapperScanner      : Cannot use both: sqlSessionTemplate and sqlSessionFactory together. sqlSessionFactory is ignored.
2021-10-27 19:04:04.098  INFO 2736 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 80 (http)
2021-10-27 19:04:04.117  INFO 2736 --- [  restartedMain] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2021-10-27 19:04:04.118  INFO 2736 --- [  restartedMain] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.46]
2021-10-27 19:04:04.254  INFO 2736 --- [  restartedMain] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationCo
```


그럼 info level에서 로깅을 해 보자. 

## Slf4j Annotation 사용 
### 의존성 설치 

pom.xml에 의존성을 추가한다.

```xml
<dependency>
  <groupId>org.projectlombok</groupId>
	<artifactId>lombok</artifactId>
	<version>1.18.20</version>
	<scope>provided</scope>
</dependency>
```

```java
@Slf4j
@Controller 
public class YourController { 
   public ResponseEntity<User> login(HttpServletRequest request, HttpServletResponse response) {
       log.debug("debug 모드");
       log.info("info 모드");
   }
}
```


실행하면 디폴트 상태에서는 info 레벨의 로그만 출력이된다. 



## 로거(Logger) 정의 

패키지 별로 로거를 만들어서 출력을 상세히 정의해보자. 

로그를 콘솔에 출력하면서 파일로 저장하기 위해서 file appender를 정의했다. logs/app 디렉터리 내에 로그파일이 날짜별로 생성되도록 했다.


### application.yml
```yaml
logging:
  file:
    # path는 디렉터리 이름 
    path: logs/app
    pattern:
      console: "%d %-5level %logger : %msg%n"
      #file:  "%d %-5level [%thread] %logger : %msg%n"
      rolling-file-name: sogood-app-%d{yyyy-MM-dd-HH-mm}.%i.log
    max-size: 100MB
    max-history: 10
  level:
    root: ERROR
```

SpringBoot를 시작하면 다음과 같이 출력된다. 

```shell
.   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.5.0)
 ```
 root 로거의 레벨을 info로 바꾸면 좀 더 다양한 정보가 출력된다. 
 ```shell
 2021-10-27 17:33:38.915  INFO 6824 --- [  restartedMain] o.s.b.devtools.restart.ChangeableUrls    : The Class-Path manifest attribute in d:\mavenrepository\com\oracle\database\jdbc\ojdbc8\21.1.0.0\ojdbc8-21.1.0.0.jar referenced one or more files that do not exist: file:/d:/mavenrepository/com/oracle/database/jdbc/ojdbc8/21.1.0.0/oraclepki.jar
2021-10-27 17:33:38.915  INFO 6824 --- [  restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : Devtools property defaults active! Set 'spring.devtools.add-properties' to 'false' to disable
2021-10-27 17:33:38.915  INFO 6824 --- [  restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : For additional web related logging consider setting the 'logging.level.web' property to 'DEBUG'
2021-10-27 17:33:40.213  INFO 6824 --- [  restartedMain] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data JDBC repositories in DEFAULT mode.
2021-10-27 17:33:40.271  INFO 6824 --- [  restartedMain] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 43 ms. Found 0 JDBC repository interfaces.
5
2021-10-27 17:33:40.460  WARN 6824 --- [  restartedMain] o.m.s.mapper.ClassPathMapperScanner      : Cannot use both: sqlSessionTemplate and sqlSessionFactory together. sqlSessionFactory is ignored.
2021-10-27 17:33:41.342  INFO 6824 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 80 (http)
2021-10-27 17:33:41.357  INFO 6824 --- [  restartedMain] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2021-10-27 17:33:41.357  INFO 6824 --- [  restartedMain] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.46]
2021-10-27 17:33:41.547  INFO 6824 --- [  restartedMain] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2021-10-27 17:33:41.547  INFO 6824 --- [  restartedMain] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 2632 ms
2021-10-27 17:33:43.209  INFO 6824 --- [  restartedMain] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2021-10-27 17:33:43.464  INFO 6824 --- [  restartedMain] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2021-10-27 17:33:43.811  INFO 6824 --- [  restartedMain] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2021-10-27 17:33:43.975  INFO 6824 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 80 (http) with context path ''
2021-10-27 17:33:44.010  INFO 6824 --- [  restartedMain] com.sogood.SogoodApplication             : Started SogoodApplication in 6.002 seconds (JVM running for 7.003)
2021-10-27 17:33:44.017  INFO 6824 --- [  restartedMain] o.s.b.a.ApplicationAvailabilityBean      : Application availability state LivenessState changed to CORRECT
2021-10-27 17:33:44.021  INFO 6824 --- [  restartedMain] o.s.b.a.ApplicationAvailabilityBean      : Application availability state ReadinessState changed to ACCEPTING_TRAFFIC
```
root 로거는 원래대로 ERROR로 설정하고 org.springframework 로거의 레벨을 INFO로 설정한다. 동일한 결과를 얻을 수 있다.

```yaml
#spring.resources.static-locations
# loggin section
logging:
  file:
    # path는 디렉터리 이름 
    path: logs/app
    pattern:
      console: "%d %-5level %logger : %msg%n"
      #file:  "%d %-5level [%thread] %logger : %msg%n"
      rolling-file-name: sogood-app-%d{yyyy-MM-dd-HH-mm}.%i.log
    max-size: 100MB
    max-history: 10
  level:
    root: ERROR
    org.springframework: INFO
```

이제 응용 패키지의 로거를 정의해 보자. 

com.sogood 패키지의 로그 레벨을 INFO로 설정했다.

```yaml
logging:
  file:
    # path는 디렉터리 이름 
    path: logs/app
    pattern:
      console: "%d %-5level %logger : %msg%n"
      #file:  "%d %-5level [%thread] %logger : %msg%n"
      rolling-file-name: sogood-app-%d{yyyy-MM-dd-HH-mm}.%i.log
    max-size: 100MB
    max-history: 10
  level:
    root: ERROR
    org.springframework: ERROR
    com.sogood: INFO
```
log.info() 로 출력한 메시지만 로그에 출력될 것이다. 

다시 com.sogood 로거의 레벨을 DEBUG로 설정해 보자. 



```yaml
  level:
    root: ERROR
    org.springframework: ERROR
    com.sogood: DEBUG
```



서버에 다시 요청하면 다음과 같이 SQL 로깅이 출력된다.  이것은 원하는 결과가 아니다. 

```shell
2021-10-28 09:28:34.082 DEBUG 14012 --- [p-nio-80-exec-1] c.s.b.o.mapper.OrgUserMapper.selectUser  : ==>  Preparing: SELECT USER_ID, password, name, email FROM org_user WHERE email = ?
2021-10-28 09:28:34.115 DEBUG 14012 --- [p-nio-80-exec-1] c.s.b.o.mapper.OrgUserMapper.selectUser  : ==> Parameters: aah@gmail.com(String)

2021-10-28 09:28:34.144 DEBUG 14012 --- [p-nio-80-exec-1] c.s.b.o.mapper.OrgUserMapper.selectUser  : <==      Total: 0

```



위 로그에 출력된 SQL Logging은 com.sogood 패키지의 로그레벨이 DEBUG로 설정되어 있어서 나온다. 이것을 막으려면 에러 레벨을 INFO로 올리면 나오지 않는다. 

```yaml
level:
    root: ERROR
    org.springframework: ERROR
    com.sogood: DEBUG
```

그러나 코드에 debug로 출력하는 다른 로그들도 나오지 않아서 이렇게 처리하기는 어렵다. 다른 방법으로는 패키지 경로를 세밀하게 나누는 방법이 있는데 설정이 복잡해서 하지 않는 게 좋다. 그래서 그냥 두기로 했다. 



## SQL Logging하기 

실행되는 쿼리를 로그에 출력하기 위해서  p6spy 라이브러리를 사용할 것이다. 

### 의존성 추가

pom.xml 파일에 의존성을 추가한다.  스프링 부트에서 테스트를 하는 것이기 때문에 스프링 부트용 의존성을 추가한다.


```xml
<!-- https://mvnrepository.com/artifact/com.github.gavlyukovskiy/p6spy-spring-boot-starter -->
		<dependency>
				<groupId>com.github.gavlyukovskiy</groupId>
				<artifactId>p6spy-spring-boot-starter</artifactId>
				<version>1.7.1</version>
		</dependency>
```

서버에 서비스를 요청하면 아무것도 출력이 안된다. applicaion.yml 파일에 설정을 추가해야 한다. 

### 설정 추가

applicaion.yml 파일을 연다.  다음을 추가한다.


```yaml
decorator:
  datasource:
    p6spy:
      enable-logging: true
      logging: slf4j
      tracing:
        include-parameter-values: true
      multiline: true 
```

쿼리를 여러 줄로 출력하도록 multilne=true로 설정했다. logger는 slf4j를 사용하도록 했다. 

SQL Logging을 하기 위해서는 한가지 더 설정을 해야 한다.  applicaion.yml 파일을 연다. p6spy 로거를 추가하고 로그 레벨을 DEBUG로 설정한다.


```yaml
logging:
  file:
    # path는 디렉터리 이름 
    path: logs/app
    pattern:
      console: "%d %-5level %logger : %msg%n"
      #file:  "%d %-5level [%thread] %logger : %msg%n"
      rolling-file-name: sogood-app-%d{yyyy-MM-dd-HH-mm}.%i.log
    max-size: 100MB
    max-history: 10
  level:
    root: ERROR
    org.springframework: ERROR
    com.sogood: DEBUG
    p6spy: DEBUG  # SQL Logging을 하기 위해 설정해야 한다.
```

이제 다음과 같이 출력이 될 것이다. 

```shell
2021-10-28 12:08:32.751  INFO 12396 --- [p-nio-80-exec-1] p6spy                                    : #1635390512750 | took 1ms | statement | connection 1| url jdbc:mariadb://address=(host=127.0.0.1)(port=3306)(type=primary)/lattedb?user=latte&password=1234&useUnicode=true&useJDBCCompliantTimezoneShift=true&serverTimezone=UTC&useLegacyDatetimeCode=false&characterEncoding=utf8&autoReconnect=true
SELECT USER_ID, password, name, email
     FROM org_user
     WHERE email = ?
SELECT USER_ID, password, name, email
     FROM org_user
     WHERE email = '111';
2021-10-28 12:08:32.788 DEBUG 12396 --- [p-nio-80-exec-1] c.s.b.o.mapper.OrgUserMapper.selectUser  : <==      Total: 0
```

### P6spy 포맷팅 커스텀하기

로그를 보면 email = ? 와 같이 prepared statement가 출력되는 것을 볼 수 있다. 이것이 출력되지 않도록 커스텀해보자. 

먼저 application.yml에 show-prepard-sql 사용자 프러퍼티를 하나 정의하자  false로 설정하면 prepared statement는 출력되지 않을 것이다.


```yaml
# For SQL Logging
decorator:
  datasource:
    p6spy:
      enable-logging: true
      logging: slf4j
      tracing:
        include-parameter-values: true
      multiline: true 
      #appender: com.p6spy.engine.spy.appender.StdoutLogger
  show-prepard-sql: false
  package-name: com.sogood 
```


처음 p6spy가 초기화될 때 쿼리를 포매팅하는 객체를 지정하는데 `Default` 객체가 `MultiLineFormat`이다.

`private boolean multiline = true`이며

`private String logFormat = null`이다

위 조건으로 인해 `CustomLogMessageFormat`이 아닌 `MultiLineFormat``으로 타고 들어간다.

이후 `MultiLineFormat`의 포맷을 보면

원하는 포맷으로 확장하기 위해서 포매터를 직접 구현하여 지정해주면 된다.


```java
@Configuration
public class P6spyConfig {
    
    @PostConstruct
    public void setLogMessageFormat() {
        P6SpyOptions.getActiveInstance().setLogMessageFormat(P6spyPrettySqlFormatter.class.getName());
    }
    
}
```


설정 클래스를 생성하여 새로운 `LogFormatter`를 지정해준 후 구현에 들어간다.

```java
public class P6spyPrettySqlFormatter implements MessageFormattingStrategy {
    
    @Override
    public String formatMessage(final int connectionId, final String now, final long elapsed, final String category, final String prepared, final String sql, final String url) {
        return null;
    }
    
}
```


`MessageFormattingStrategy`을 구현한다.

이름 그대로 메시지 포매팅 전략이다.

기본적으로 `SingleLineFormat`, `CustomLineFormat`, `MultiLineFormat`이 구현돼있으며

`CustomLineFormat`은 이름 때문에 약간 헷갈리는데 사용자가 커스터마이징 할 포매터가 아니고,

`SingleLineFormat`을 약간 더 손본 포매터다. 그러므로 이 녀석을 쓰면 안 되고 직접 구현해야 한다.

아래는 `CustomLineFormat`의 전체 코드이다. 참고 바람.

```java
package com.sogood.core.log;

import java.text.MessageFormat;
import java.util.Arrays;
import java.util.Stack;
import java.util.stream.Stream;

import com.p6spy.engine.spy.appender.MessageFormattingStrategy;
import com.sogood.core.config.util.PropertyUtil;

public class P6spyPrettySqlFormatter implements MessageFormattingStrategy {

  private static final String NEW_LINE = System.lineSeparator();
  /** StackTrace에서 출력할 패키지 이름 */
  private String packageName = null; 
  /** PreparedStatement를 출력할지 여부. 디폴터는 False */
  private Boolean showPreparedSql = Boolean.FALSE; 

  public P6spyPrettySqlFormatter() {
    this.packageName = PropertyUtil.getProperty("decorator.package-name", "com.sogood");
    this.showPreparedSql =  Boolean.parseBoolean(PropertyUtil.getProperty("decorator.show-prepared-sql", "false"));
  }

  @Override
  /**
   * @param connectionId  커넥션 아이디 
   * @param now 현재시간. 밀리세컨드로 표시 
   * @param elapsed 쿼리 수행후 경과시간 
   * @param category 오퍼레이션의 카테고리
   * @param prepared Prerpared SQL statement
   * @param sql 실행된 SQL 
   * @param url database URL 
   */
  public String formatMessage(final int connectionId, final String now, final long elapsed, final String category, final String prepared, final String sql, final String url) {

      StringBuilder callStackBuilder = getStackBuilder();
      StringBuilder sb = new StringBuilder()
              .append("\n\tConnection ID: ").append(connectionId)
              .append("\n\ttime ").append(now).append(", ").append("category ").append(category).append(", ")
              .append("\n\tExecution Time: ").append(elapsed).append(" ms\n")
              .append("\n\tCall Stack (number 1 is entry point): ").append(callStackBuilder)
              .append("\n\t");
      if (showPreparedSql) {
          sb.append(prepared)
                  .append("\n----------------------------------------------------------------------------------------------------")
                  .append("\n\t");
      }
      sb.append("\n").append(sql);

      return sb.toString();
  }//:

  /**
   * 
   * @return 
   */
  private StringBuilder getStackBuilder() {
    /*
    Stack Trace란? 
    응용 프로그램(Application)이 시작된 시점부터 프로그램 내에서 현재 실행 위치까지의 메서드 호출 목록
    예외가 발생했을 때까지 프로그램의 위치와 진행정도를 나타내기 위해 예외가 발생하면 JVM에 의해 자동으로 생성
    가장 최근의 메서드 호출이 목록에 맨 위에 있음
    
    StackTrace는 java.lang.StackTraceElement 클래스 배열로 캡슐화됨(JDK 1.4 ~)
     - java.lang.StackTraceElement 클래스 배열 : Throwable.getStackTrace()로 반환된 StackTrace 요소 배열
       > 각 요소는 단일 스택 프레임
       > 스택의 상단에 있는 것을 제외한 모든 스택 프레임은 메서드 호출
       > 스택 상단 프레임은 스택 추적이 생성된 실행 지점 - 일반적으로 throwable이 생성된 지점

    java.lang.StackTraceElement 클래스 : StackTrace 요소
      - 지정된 실행 지점을 나타내는 스택 추적 요소 생성

    */

    final Stack<String> callStack = new Stack<>();
    Stream<StackTraceElement> stream = Arrays.stream(new Throwable().getStackTrace());
    stream.map(element -> element.toString())
          .filter(elementStr -> elementStr.startsWith(packageName) && !elementStr.contains("P6spyPrettySqlFormatter") && !elementStr.contains("doFilter"))
          .forEach(callStack::push);

    int order = 1;
    final StringBuilder callStackBuilder = new StringBuilder();
    while (!callStack.empty()) {
        callStackBuilder.append(MessageFormat.format("{0}\t\t{1}. {2}", NEW_LINE, order++, callStack.pop()));
    }
    return callStackBuilder;

  }//:

}///~
```

이제 SQL이 다음과 같이 출력될 것이다. 

```shell
2021-10-28 13:59:26.409 DEBUG 9604 --- [p-nio-80-exec-1] c.s.b.o.mapper.OrgUserMapper.selectUser  : ==> Parameters: 111(String)
2021-10-28 13:59:26.415  INFO 9604 --- [p-nio-80-exec-1] p6spy                                    : 
	Connection ID: 1
	time 1635397166412, category statement, 
	Execution Time: 1 ms

	Call Stack (number 1 is entry point): 
		1. com.sogood.biz.login.controller.LoginController.login(LoginController.java:33)
		2. com.sogood.biz.login.service.LoginService.login(LoginService.java:25)
		3. com.sogood.biz.org.service.OrgUserService.selectUser(OrgUserService.java:20)
	
SELECT USER_ID, password, name, email
     FROM org_user
     WHERE email = '111'
2021-10-28 13:59:26.464 DEBUG 9604 --- [p-nio-80-exec-1] c.s.b.o.mapper.OrgUserMapper.selectUser  : <==      Total: 0
```

아래 URL에서 커스텀 관련한 내용을 참고할 수 있다. 

[https://github.com/shirohoo/p6spy-custom-formatter](https://github.com/shirohoo/p6spy-custom-formatter)

