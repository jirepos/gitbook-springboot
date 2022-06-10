# MyBatis 설정

com.sogood.core.jdbc 패키지를 생성한다. 

MyBatisConfig.java를 생성한다. 

```java
package com.sogood.core.jdbc;

public class MyBatisConfig {
}
```

application.yml 파일을 열고  다음을 추가한다. 

```java

spring:
  datasource:
    driver-class-name: org.mariadb.jdbc.Driver
    jdbc-url: jdbc:mariadb://127.0.0.1:3306/lattedb?autoReconnect=true&useUnicode=true&characterEncoding=utf8&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=UTC
    username: latte
    password: 1234
```

MyBatisConfig.java를 다음과 같이 수정한다. 

```java
package com.sogood.core.jdbc;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.sql.DataSource;
@Configuration
@EnableTransactionManagement
public class MyBatisConfig {
    @Bean(name="MainDataSource")
    @ConfigurationProperties(prefix="spring.datasource")
    public DataSource buildDataSource() {
        return DataSourceBuilder.create().build();
    }
}
```

**@MapperScan 적용**

@MapperScan annotation을 명시해 준 class는 basePackages로 지정한 곳에 존재하는 @Mapper로 명시된 interface를 스캔한다

```java
package com.sogood.core.jdbc;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.sql.DataSource;
@Configuration
@EnableTransactionManagement
@MapperScan(basePackages = "com.sogood")
public class MyBatisConfig {
    @Bean(name="MainDataSource")
    @ConfigurationProperties(prefix="spring.datasource")
    public DataSource buildDataSource() {
        return DataSourceBuilder.create().build();
    }
}
```

## Mapper.xml 설정

mybatis-spring을 사용하면 applicaion.yml에 mapper-locaions을 사용하여 위치를 설정할 수 있다. 

우리는 Java Configuration을 사용할 것이므로 다른 설정을 가져갈 것이다.  src/main/resources 디렉터리 안에 mapper디렉터리를 생성할 것이다. 

```java
📁src/main/resources
   📁mapper
```

applicatoin.yml 파일에 다음을 추가한다. 

```java
database-config:
  datasource:
    mapper-locations:
      - classpath:mapper/**/*.xml
```

[MyBatisConfig.java](http://MyBatisConfig.java) 파일에 내부 static class를 다음과 같이 정의한다. 

```java
public class MyBatisConfig {

    static class MapperLocation {
        private String[] mapperLocations;

        public String[] getMapperLocations() {
            return mapperLocations;
        }

        public void setMapperLocations(String[] mapperLocations) {
            this.mapperLocations = mapperLocations;
        }
    }
}
```

MapperLocaion 클래스의 인스턴스를 생성하는 코드를 다음과 같이 정의한다.  ConfigurationProperties 어노테이션을 사용한다. 

```java
@Bean(name="mapperLocation")
    @ConfigurationProperties(prefix="database-config.datasource")
    public MapperLocation mapperLocation() {
        return new MapperLocation();
    }
```

## SqlSessionFactory 생성

다음과 같이 SqlSessionFactory 인스턴스를 생성하는 메서드를 MyBatisConfig.java에 정의한다. 

```java
@Primary
    @Bean(name = "sqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory(@Qualifier("dataSource") DataSource dataSource, ApplicationContext applicationContext) throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        //sqlSessionFactoryBean.setTypeAliasesPackage("com.test.api.entity, com.test.api.vo");
        MapperLocation mapperLocation = (MapperLocation)applicationContext.getBean("mapperLocation");
        Resource[] resources = SpringContextUtils.getResources(applicationContext, mapperLocation.getMapperLocations());
        sqlSessionFactoryBean.setMapperLocations(resources);
        //sqlSessionFactoryBean.setMapperLocations(applicationContext.getResources("classpath*:com/sogood/mapper/**.xml"));
        org.apache.ibatis.session.Configuration mybatisConfig = new org.apache.ibatis.session.Configuration();
        mybatisConfig.setMapUnderscoreToCamelCase(true);
        sqlSessionFactoryBean.setConfiguration(mybatisConfig);
        //mybatisConfig.setJdbcTypeForNull(JdbcType.NULL); // 좀 더 확인해 보고
        return sqlSessionFactoryBean.getObject();
    }//:
```

여러 개의 DB를 사용할 것을 가정하여  Primary 어노테이션을 사용한다. 

### SqlSessionTemplate 생성

이제 SqlSessionTemplate와 TransactionManager를 생성하기 위해서 다음과 같이 정의한다. 

```java

@Primary
  @Bean(name = "sqlSessionTemplate")
  public SqlSessionTemplate sqlSessionTemplate(@Qualifier("sqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
    return new SqlSessionTemplate(sqlSessionFactory);
  }

	@Primary
	@Bean(name = "transactionManager")
	public DataSourceTransactionManager transactionManager(@Autowired @Qualifier("dataSource") DataSource dataSource) {
		return new DataSourceTransactionManager(dataSource);
	}//:
```

 MapperScan 어노테이션에 sqlSessionFactoryRef 속성과 sqlSessionTemplateRef 속성에 값을 설정한다. 

```java
import javax.sql.DataSource;
@Configuration
@EnableTransactionManagement
@MapperScan(
        basePackages = "com.sogood",
        sqlSessionFactoryRef = "sqlSessionFactory",
        sqlSessionTemplateRef = "sqlSessionTemplate"
)
public class MyBatisConfig {
}
```

다음은 전체 코드이다. 

```java
package com.sogood.core.jdbc;

import com.sogood.core.util.SpringContextUtils;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.Resource;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.sql.DataSource;
@Configuration
@EnableTransactionManagement
@MapperScan(
        basePackages = "com.sogood",
        sqlSessionFactoryRef = "sqlSessionFactory",
        sqlSessionTemplateRef = "sqlSessionTemplate"
)
public class MyBatisConfig {
    @Bean(name="dataSource")
    @ConfigurationProperties(prefix="spring.datasource")
    public DataSource buildDataSource() {
        return DataSourceBuilder.create().build();
    }

    static class MapperLocation {
        private String[] mapperLocations;

        public String[] getMapperLocations() {
            return mapperLocations;
        }

        public void setMapperLocations(String[] mapperLocations) {
            this.mapperLocations = mapperLocations;
        }
    }

    @Bean(name="mapperLocation")
    @ConfigurationProperties(prefix="database-config.datasource")
    public MapperLocation mapperLocation() {
        return new MapperLocation();
    }
    @Primary
    @Bean(name = "sqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory(@Qualifier("dataSource") DataSource dataSource, ApplicationContext applicationContext) throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        //sqlSessionFactoryBean.setTypeAliasesPackage("com.test.api.entity, com.test.api.vo");
        MapperLocation mapperLocation = (MapperLocation)applicationContext.getBean("mapperLocation");
        Resource[] resources = SpringContextUtils.getResources(applicationContext, mapperLocation.getMapperLocations());
        sqlSessionFactoryBean.setMapperLocations(resources);
        //sqlSessionFactoryBean.setMapperLocations(applicationContext.getResources("classpath*:com/sogood/mapper/**.xml"));
        org.apache.ibatis.session.Configuration mybatisConfig = new org.apache.ibatis.session.Configuration();
        mybatisConfig.setMapUnderscoreToCamelCase(true);
        sqlSessionFactoryBean.setConfiguration(mybatisConfig);
        //mybatisConfig.setJdbcTypeForNull(JdbcType.NULL); // 좀 더 확인해 보고
        return sqlSessionFactoryBean.getObject();
    }//:

    @Primary
    @Bean(name = "sqlSessionTemplate")
    public SqlSessionTemplate sqlSessionTemplate(@Qualifier("sqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

    @Primary
    @Bean(name = "transactionManager")
    public DataSourceTransactionManager transactionManager(@Autowired @Qualifier("dataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }//:

}
```

## MyBatis Test

### mapper.xml 생성

src/main/resources/mapper/demo   DemoMyBatisMapper.xml을 생성한다. 

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.sogood.demo.mapper.DemoMyBatisMapper">

    <!-- create table demo_mybatis_employee ( user_id varchar(20), user_name varchar(20) ) -->
  <select id="selectAll" resultType="com.sogood.demo.vo.DemoMyBatisBean">
    SELECT USER_NAME,
           USER_ID
    FROM DEMO_MYBATIS_EMPLOYEE
  </select>
  
</mapper>
```

### Java Bean 생성

`com.sogood.demo.vo 패키지에 DemoMyBatisBean을 생성한다.` 

```java
package com.sogood.demo.vo;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class DemoMyBatisBean {
    private String userId;
    private String userName;
}
```

### Mapper Interface 생성

`com.sogood.demo.mapper 패키지에 DemoMyBatisMapper 인터페이스를 작성한다.` 

```java
package com.sogood.demo.mapper;

import com.sogood.demo.vo.DemoMyBatisBean;
import org.apache.ibatis.annotations.Mapper;

import java.util.List;

@Mapper
public interface DemoMyBatisMapper {
    List<DemoMyBatisBean> selectAll();
}
```

### Service Class 생성

`com.sogood.demo.service` 패키지에 DemoMyBatisService 클래스를 생성한다. 

```java
package com.sogood.demo.service;

import com.sogood.demo.mapper.DemoMyBatisMapper;
import com.sogood.demo.vo.DemoMyBatisBean;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class DemoMyBatisService {
    @Autowired
    private DemoMyBatisMapper demoMyBatisMapper;
    public List<DemoMyBatisBean> selectAll() {
        return this.demoMyBatisMapper.selectAll();
    }
}
```

### Controller Class 생성

`com.sogood.demo.controller` 패키지에 MyBatisTestController 클래스를 생성한다. 

```java
package com.sogood.demo.controller;

import com.sogood.demo.service.DemoMyBatisService;
import com.sogood.demo.vo.DemoMyBatisBean;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.List;

@RestController
@RequestMapping("/demo/mybatis")
public class MyBatisTestController {

    @Autowired
    private DemoMyBatisService demoMybatisService;

    @GetMapping("/select-all-employees")
    public ResponseEntity<List<DemoMyBatisBean> > selectAllEmployees(HttpServletRequest request, HttpServletResponse response) {
        List<DemoMyBatisBean> list = this.demoMybatisService.selectAll();
        HttpHeaders headers = new HttpHeaders();
        return new ResponseEntity<List<DemoMyBatisBean> >(list, headers,  HttpStatus.OK);
    }//:

}///~
```

### JavaScript 생성

test 코드를 생성하여 테스트 해본다. 

```java
<script>
    window.addEventListener('load', () => {

      // select-all-employees 테스트 
      let btnSelectAll = document.querySelector("#btnSelectAll");
      btnSelectAll.addEventListener('click', () => {
        let options = {
          method: 'GET', 
          url:'/demo/mybatis/select-all-employees' 
        }
        ajax(options)
          .then((response) => {
            console.log(response);
          })
          .catch(error => {
          })
      }) //:

    })// window.load 
  </script>
```