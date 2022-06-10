# JDBC

이 글의 내용 중에서 상당한 부분은 [mybatis-spring-boot-autoconfigure](https://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/) 문서를 참고했다.

## 의존성 주입

Spring JDBC를 사용하기 위해서 다음의 의존성을 추가한다.

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
```

MyBatis 사용을 위해서 다음의 의존성을 추가한다.

```xml
<dependency>
	<groupId>org.mybatis.spring.boot</groupId>
	<artifactId>mybatis-spring-boot-starter</artifactId>
	<version>2.2.0</version>
</dependency>
```

JDBC 드라이버를 추가한다. MariaDB와 MySQL Driver가 분리되어 있다는 것에 주의한다.

```xml
<dependency>
	<groupId>com.microsoft.sqlserver</groupId>
	<artifactId>mssql-jdbc</artifactId>
	<scope>runtime</scope>
</dependency>
<dependency>
	<groupId>com.oracle.database.jdbc</groupId>
	<artifactId>ojdbc8</artifactId>
	<scope>runtime</scope>
</dependency>
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<scope>runtime</scope>
</dependency>
<dependency>
	<groupId>org.mariadb.jdbc</groupId>
	<artifactId>mariadb-java-client</artifactId>
	<scope>runtime</scope>
</dependency>
```

이미 알다시피, Spring과 MyBatis를 사용하려면 적어도 하나의 SqlSessionFactory와 적어도 하나의 Mapper Interface를 필요로한다. MyBatis-Spring-Boot-Starter는

* 존재하는 DataSource를 자동 감지한다.
* SqlSessionFactoryBean을 사용하여 입력으로 DataSource를 전달하여 SqlSessionFactory의 인스턴스를 생성하고 등록할 것이다.
* SqlSessionFactory에서 구한 SqlSessionTemplate의 인스턴스를 생성하고 등록할 것이다.
* mapper들을 자동 스캔할 것이고, SqlSesionTemplate에 그것들을 연결할 것이고, Bean 들에 그것들이 주입 될 수 있도록 Spring context에 등록할 것이다.

다음의 Mapper를 가졌다고 가정해 보자.

```java
@Mapper
public interface CityMapper {
  @Select("SELECT * FROM CITY WHERE state = #{state}")
  City findByState(@Param("state") String state);
}
```

보통의 스프링 부트 어플리케이션을 단지 생성하고 다음과 같이 Mapper가 주입될 수 있도록 한다. (Spring 4.3+ 에서도 가능하다.)

```java
@SpringBootApplication
public class SampleMybatisApplication implements CommandLineRunner {
  private final CityMapper cityMapper;
  public SampleMybatisApplication(CityMapper cityMapper) {
    this.cityMapper = cityMapper;
  }
  public static void main(String[] args) {
    SpringApplication.run(SampleMybatisApplication.class, args);
  }
  @Override
  public void run(String... args) throws Exception {
    System.out.println(this.cityMapper.findByState("CA"));
  }
}
```

## Advanced scanning

MyBatis-Spring-Boot-Starter는 디폴트로 @Mapper 어노테이션으로 마크된 mapper들을 찾을 것이다. 커스텀 어노테이션과 scanning을 위한 marker interface를 정의하고 싶을 수도 있다. 그렇다면, @MapperScan을 사용해야 한다.

## Using an SqlSession

SqlSessionTemplate의 인스턴스는 생성되고 Spring Context에 추가된다. 그래서 다음과 같이 bean 들에 주입될 수 있도록 설정하여 MyBatis API를 사용할 수 있다. (Spring 4.3+ 에서 가능)

```java
@Component
public class CityDao {
  private final SqlSession sqlSession;

  public CityDao(SqlSession sqlSession) {
    this.sqlSession = sqlSession;
  }
  public City selectCityById(long id) {
    return this.sqlSession.selectOne("selectCityById", id);
  }
}
```

## Configuration

다른 Spring Boot application에서 처럼, MyBatis-Spring-Boot-Application 설정 파라미터들은 appliation.properties(or applicaion.yml) 내부에 저장된다.

MyBatis는 mybatis 접두사를 사용한다.

| Property                                | Description                                                                                                                                                                                                                                |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| config-location                         | MyBatis xml config file의 위치.                                                                                                                                                                                                               |
| check-config-location                   | Indicates whether perform presence check of the MyBatis xml config file.                                                                                                                                                                   |
| mapper-locations                        | Mapper xml config file의 위치.                                                                                                                                                                                                                |
| type-aliases-package                    | Packages to search for type aliases. (Package delimiters are “,; \t\n”)                                                                                                                                                                    |
| type-aliases-super-type                 | The super class for filtering type alias. If this not specifies, the MyBatis deal as type alias all classes that searched from type-aliases-package.                                                                                       |
| type-handlers-package                   | Packages to search for type handlers. (Package delimiters are “,; \t\n”)                                                                                                                                                                   |
| executor-type                           | Executor type: SIMPLE, REUSE, BATCH                                                                                                                                                                                                        |
| default-scripting-language-driver       | The default scripting language driver class. This feature requires to use together with mybatis-spring 2.0.2+.                                                                                                                             |
| configuration-properties                | Externalized properties for MyBatis configuration. Specified properties can be used as placeholder on MyBatis config file and Mapper file. For detail see the MyBatis reference page.                                                      |
| lazy-initialization                     | Whether enable lazy initialization of mapper bean. Set true to enable lazy initialization. This feature requires to use together with mybatis-spring 2.0.2+.                                                                               |
| mapper-default-scope                    | Default scope for mapper bean that scanned by auto-configure. This feature requires to use together with mybatis-spring 2.0.6+.                                                                                                            |
| configuration.\*                        | Property keys for Configuration bean provided by MyBatis Core. About available nested properties see the MyBatis reference page. NOTE: This property cannot be used at the same time with the config-location.                             |
| scripting-language-driver.thymeleaf.\*  | Property keys for ThymeleafLanguageDriverConfig bean provided by MyBatis Thymeleaf. About available nested properties see the MyBatis Thymeleaf reference page.                                                                            |
| scripting-language-driver.freemarker.\* | Properties keys for FreeMarkerLanguageDriverConfig bean provided by MyBatis FreeMarker. About available nested properties see the MyBatis FreeMarker reference page. This feature requires to use together with mybatis-freemarker 1.2.0+. |
| scripting-language-driver.velocity.\*   | Properties keys for VelocityLanguageDriverConfig bean provided by MyBatis Velocity. About available nested properties see the MyBatis Velocity reference page. This feature requires to use together with mybatis-velocity 2.1.0+.         |

예를 들면:

```properties
# application.properties
mybatis.type-aliases-package=com.example.domain.model
mybatis.type-handlers-package=com.example.typehandler
mybatis.configuration.map-underscore-to-camel-case=true
mybatis.configuration.default-fetch-size=100
mybatis.configuration.default-statement-timeout=30
...
```

```yaml
# application.yml
mybatis:
    type-aliases-package: com.example.domain.model
    type-handlers-package: com.example.typehandler
    configuration:
        map-underscore-to-camel-case: true
        default-fetch-size: 100
        default-statement-timeout: 30
...
```

## 설정하기

### 커넥션 풀 설정

Spring boot에서 커넥션 풀을 설정하는 방법을 알아본다. SpringBoot에서 아래와 같은 dependency를 추가해 주면 hikari DBCP가 default로 사용된다.

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
```

datasource 설정은 다음을 참고하여 application.yml에 설정한다. Spring Boot 2 부터는 Java Config에서 dataSource 빈을 생성하지 않아도 예약된 키에 프러퍼티 설정을 하면 자동적으로 jpa 설정과 dataSource 빈을 생성해 준다. DB.연결에 관한 예약된 key는 spring.datasource이고 dirver-class-name, url, username, password 속성을 반드시 생성해야 한다.

```yaml
spring:
  datasource: 
    dirver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://xxx.xxx.xxx.xxx:3306/mdb?autoReconnect=true&useUnicode=true&characterEncoding=utf8&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=UTC
    username: xxxx
    password: xxx
    hikari:
      connection-timeout: 20000  # maximum number of milliseconds that a client will wait for a connection
      connection-test-query: SELECT 1 
      minimum-idle: 10       # minimum number of idle connections maintained by HikariCP in a connection pool 
      maximum-pool-size: 10  # maximum pool size
      idle-timeout: 10000    # maximum idle time for connection
      max-lifetime: 10000     # maximum idle time for connection
      auto-commit: true      # default auto-commit behavior.        
```

#### Hikari 설정

**필수 설정**

**datasource-class-name(dataSourceClassName)**\
JDBC driver에 의해 제공되는 DataSource 클래스의 이름이다. 이 클래스의 이름을 얻기 위해서 특정 JDBC driver에 대한 문서를 참조한다. 또는 아래의 테이블을 참조한다. 구식의 DriverManager-based DJBC driver 설정을 위해 jdbcUrl을 사용한다면 이 속성이 필요하지 않다. 기본값은 'None'이다.

여기에 JDBC DataSource 클래스들의 목록이 있다.

| Database      | Driver       | DataSource class                                   |
| ------------- | ------------ | -------------------------------------------------- |
| Apache Derby  | Derby        | org.apache.derby.jdbc.ClientDataSource             |
| Firebird      | Jaybird      | org.firebirdsql.ds.FBSimpleDataSource              |
| H2            | H2           | org.h2.jdbcx.JdbcDataSource                        |
| HSQLDB        | HSQLDB       | org.hsqldb.jdbc.JDBCDataSource                     |
| IBM DB2       | IBM JCC      | com.ibm.db2.jcc.DB2SimpleDataSource                |
| IBM Informix  | IBM Informix | com.informix.jdbcx.IfxDataSource                   |
| MS SQL Server | Microsoft    | com.microsoft.sqlserver.jdbc.SQLServerDataSource   |
| MySQL         | Connector/J  | com.mysql.jdbc.jdbc2.optional.MysqlDataSource      |
| MariaDB       | MariaDB      | org.mariadb.jdbc.MariaDbDataSource                 |
| Oracle        | Oracle       | oracle.jdbc.pool.OracleDataSource                  |
| OrientDB      | OrientDB     | com.orientechnologies.orient.jdbc.OrientDataSource |
| PostgreSQL    | pgjdbc-ng    | com.impossibl.postgres.jdbc.PGDataSource           |
| PostgreSQL    | PostgreSQL   | org.postgresql.ds.PGSimpleDataSource               |
| SAP MaxDB     | SAP          | com.sap.dbtech.jdbc.DriverSapDB                    |
| SQLite        | xerial       | org.sqlite.SQLiteDataSource                        |
| SyBase        | jConnect     | com.sybase.jdbc4.jdbc.SybDataSource                |

**jdbc-url(jdbcUrl)**\
이 속성은 HikariCP가 DriverManager-based 설정을 사용하게 한다. DataSource-based 설정이 다양한 이유로 더 뛰어나다고 느끼지만 실제로 많은 배포의 경우에 중요한 차이는 거의 없다. 구식의 driver들과 이 속성을 사용할 때, driverClassName 속성의 설정이 필요할 수도 있다. 그러나 그것 없이 시작해 본다.

**username**\
사용자 아이디. **password**\
패스워드.

**자주 사용하는 설정**

**auto-commit**\
기본값은 true이다. pool에서 반환된 커넥션들의 자동 커밋 동작을 결정한다.

**connection-timeout**\
커넥션 연결을 얼마나 기다릴지를 millisecond로 설정한다. 기본값은 30000(30초)

**idle-timeout**\
pool에서 얼마나 idle 상태로 있을지 설정한다. 기본값은 600000(10분). 이 설정은 maximum-pool-size보다 적게 설정되었을 때에만 적용된다. idle connection들은 mimium-idle 커넥션들에 도달하면 제거되지 않을 것이다.

**keep-alive-time**\
이 속성은 HikariCP가 커넥션을 유지하기 위해 얼마자 자주 시도할지를 설정한다. 기본값은 0이다.

**max-life-time**\
pool에서 커넥션의 최대 수명을 설정한다. 사용중인 커넥션은 제거되지 않는다. 이 값을 설정하라고 Hikari 공식사이트에서 강력하게 말하고 있다. 최소값은 30000(30 초). 기본값은 1800000(30분).

**connection-test-query**\
커넥션이 유효한지 체크하기 위한 쿼리를 설정한다.

**minimum-idle**\
커넥션을 유지하기 해 idle connection의 최소 수를 설정한다.

**maximum-pool-size**\
커넥션의 최대 수를 설저한다.

더 많은 것을 알고 싶으면 [Spring Boot DataSource Configuration](https://howtodoinjava.com/spring-boot2/datasource-configuration/)을 참고해도 좋을 것이다.

### MyBatis 설정하기

application.yml에 MyBatis 설정을 추가한다. mapper-locaions에는 매퍼의 위치를 작성한다. mapper-locations는 배열이어서 여러 경로를 설정할 수 있다. map-underscore-tocamel-case를 true로 설정한다. 보통 DB의 컬럼은 USER_NAME과 같이 underscore를 사용하는데 Java에서는 userName과 같이 필드명을 설정한다. 이것을 자동으로 매핑하도록 하려면 이 설정을 사용한다.

```yaml
# MyBatis
mybatis:
  # Mapper.xml 파일의 위치 
  mapper-locations: com/sogood/mapper/**/*.xml
  configuration:
    # Model Propery camelcase 설정 
    map-underscore-to-camel-case: true 
    # 패키지명을 생략할 수 있게 alias 설정
    #type-aliases-package: com.sogood.sample.model
```

### Mapper.xml 작성

mapper xml은 src/main/resources 소스 폴더 아래에 생성한다. com.sogood.mapper 폴더를 생성하고 그 아래에 업무별로 폴더를 구조화해서 mapper.xml을 위치시킨다. 테스트를 위해 com/sogood/mapper아래에 DemoEmployeeMapper.xml 파일을 만든다. 관례상 Mapper를 파일명 뒤에 붙인다. 다음과 같이 작성한다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.sogood.demo.DemoEmployeeMapper">

  <select id="selectAll" resultType="com.sogood.demo.DemoEmployeeBean">
    SELECT USER_NAME,
           USER_ID
    FROM EMPLOYEE
  </select>
</mapper>
```

Mapper를 작성할 때 기억해야할 중요한 점은:

* namespace의 값은 mapper xml과 대응할 Interface 이름을 패키지 경로와 함께 작성한다. 패키지 경로와 이름이 다르면 Interface와 매핑 시킬 수 없다는 것에 주의한다.
* select 요소의 id인 selectAll은 interface의 method 이름과 동일해야 한다.
* resultType은 select 구문에 나열된 컬럼이 매핑될 Java Bean을 패키지 명과 함께 설정한다. Alias를 쓸 수도 있지만 너무 많은 Bean이 있을 경우 혼란이 있을 수 있으니 사용하지 않는다.

## 테스트해보기

### JavaBean 생성

com/sogood/demo 패키지 아래에 DemoEmployeeBean.java 파일을 생성하고 다음과 같이 작성한다.

```java
package com.sogood.demo;

import java.util.Date;
import java.util.List;

public class DemoBean {
package com.sogood.demo;

public class DemoEmployeeBean {
  private String userId;
  private String userName;
    // ... getter/setter 생략 
}
```

### DemoEmployeeMapper 인터페이스 작성

com/sogood/demo 패키지 아래에 DemoEmployeeMapper.java를 생성하고 다음과 같이 작성한다. Maper xml에서 select 요소의 ID로 설정한 것과 동일하게 메서드의 이름을 작성한다.

```java
package com.sogood.demo;

import java.util.List;

import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface DemoEmployeeMapper {
  List<DemoEmployeeBean> selectAll();
}
```

### DataSource 사용하기

앞에서 한 것만으로 기본적인 설정이 모두 완료된 것이므로 이제 사용할 수 있다. 먼저 DataSource를 한 번 사용해 보자.DemoController.java에 DataSource가 주입될 수 있도록 멤버 변수로 선언한다.

```java
@Autowired
private DataSource dataSource; 
```

컨트롤러의 임의의 메서드에 아래와 같이 작성한다. DataSource로부터 Connection을 구하고 그 정보를 출력해 본다.

```java
	@GetMapping(value = "/getText", produces = ResMediaType.TEXT)
	public @ResponseBody String getText(HttpServletRequest request, HttpServletResponse response) throws Exception {
		System.out.println(">>> Connectin Test");
		Connection connection = dataSource.getConnection();
		System.out.println(">>>" + dataSource.getClass());
		System.out.println(">>>" + connection.getMetaData().getURL());
		return "한글반환입니다.";
	}// :
```

### Mapper 사용하기

Mapper가 주입될 수 있도록 멤버 변수로 Mapper Interface를 선언한다.

```java
@Autowired
private DemoEmployeeMapper demoMapper; 
```

Mapper의 selectAll() 메서드를 호출하고 그 결과를 출력하는 코드는 다음과 같다.

```java
@GetMapping(value = "/getText", produces = ResMediaType.TEXT)
public @ResponseBody String getText(HttpServletRequest request, HttpServletResponse response) throws Exception {
	List<DemoEmployeeBean> list = demoMapper.selectAll();
	for(DemoEmployeeBean bean : list) {
		System.out.println(">>" + bean.getUserName());
	}
	return "한글반환입니다.";
}// :
```

## Multiple Data Source

실제 운영환경에서는 여러 개의 DataSource에 연결해야할 필요가 있을 수 있다. 이런 경우에는 Java Configuration으로 설정해야 한다.
