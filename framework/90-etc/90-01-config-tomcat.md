# 내장 톰캣 설정

## application.yml

다음과 같이 설정한다. 

```java
# server properties
server:
  port: 80
  tomcat:
    threads:
      max: 200
  #connection-timeout: 5s deprecated
  max-http-header-size: 8KB
  servlet:
    register-default-servlet: true
  #ssl:
    #enabled: false
    #protocol: TLS 
    #key-store-password: my_password
    #key-store-type: keystore_type
    #key-store: keystore-path
    #key-alias: tomcat
  error:
    whitelabel:
      # spring boot provides a standard error web page 
      enabled: true   
    #  server.error.path를 customize할 수 있다. 
    #path: /user-error
```

**server.port**  

서버의 포트 번호. default=8080. 

**server.servlet.register-default-servlet**

DefaultServlet는 기본적으로  false 이다. 응용 프로그램에 필요한 경우 server.servlet.register-default-servlet을 true 설정하여 사용한다.