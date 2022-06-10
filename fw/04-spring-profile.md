# Profile 적용

Spring Boot를 사용하여 개발환경, 검증환경, 운영환경 마다 다른 설정값을 가지도록 application.yml 파일을 사용한다.

**Spring Boot 2.4 부터는 spring.profiles 가 deprecated 되었다.**

- 여러 개의 profile을 만들고 group을 생성하여 그룹에 프로파일들을 포함한다.
- 프로파일은 '—-'을 사용하여 구분한다.
- spring.config.activate.on-profile 속성을 사용하여 프로파일을 정의한다.
- 프로파일의 속성은 띄어 쓰기 없이 시작한다.

```java
# production db
---
sring:
  config:
    activate:
      on-profile: production-db
database-config:
  datasource:
    mapper-locations:
      - classpath:mapper/**/*.xml
    jdbc-url: jdbc:mariadb://127.0.0.1:3306/lattedb?autoReconnect=true&useUnicode=true&characterEncoding=utf8&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=UTC
    #driver-class-name: com.mysql.cj.jdbc.Driver
    driver-class-name: org.mariadb.jdbc.Driver
    username: latte
    password: 1234
```

정의한 프로파일은 group을 생성하고 group에 포함한다. 

```java
---
spring:
  profiles:
    group:
      production: production-db
```

local 환경과 production 환경을 구분하려면 production-db 프로파일과 local-db 프로파일을 만들고 production 그룹과 local그룹을 만든다. 

```java
# Spring Profiles

---
spring:
  profiles:
    group:
      production: production-db
      local: local-db

# production db
---
sring:
  config:
    activate:
      on-profile: production-db
database-config:
  datasource:
    mapper-locations:
      - classpath:mapper/**/*.xml
    jdbc-url: jdbc:mariadb://127.0.0.1:3306/lattedb?autoReconnect=true&useUnicode=true&characterEncoding=utf8&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=UTC
    #driver-class-name: com.mysql.cj.jdbc.Driver
    driver-class-name: org.mariadb.jdbc.Driver
    username: latte
    password: 1234

# local db
---
sring:
  config:
    activate:
      on-profile: local-db
database-config:
  datasource:
    mapper-locations:
      - classpath:mapper/**/*.xml
    jdbc-url: jdbc:mariadb://127.0.0.1:3306/lattedb?autoReconnect=true&useUnicode=true&characterEncoding=utf8&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=UTC
    #driver-class-name: com.mysql.cj.jdbc.Driver
    driver-class-name: org.mariadb.jdbc.Driver
    username: latte
    password: 1234
```

SpringBoot를 실행할 때 다음과 같이 profile 환경 변수를 추가한다.  하나 이상일 때 콤마로 구분한다. 

```java
-Dspring.profiles.active=local, sqllog
```