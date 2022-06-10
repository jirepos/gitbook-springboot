# Springboot Embdded Tomcat

스프링부트 웹 애플리케이션은 임베디드 웹 서버를 포함하고 있다.

> 스프링부트 2.5.4 기준으로 9.0.52버전의 Tomcat을 내장하고 있다.


* sprinb-boot-starter: 기본적으로 tomcat을 사용한다. (spring-boot-starter-tomcat) 하지만 spring-boot-starter-jetty 혹은 spring-boot-starter-undertow을 대신 사용할 수도 있다.
* spring-boot-starter-webflux: Reactive 스택 애플리케이션은 netty를 기본으로 사용한다. (spring-boot-starter-reactor-netty) 하지만 tomcat, jetty, undertow를 사용할 수도 있다.

만약에 기본 HTTP 서버가 아닌 다른 서버를 사용하고자 한다면 기본 dependecy를 제외시키고, 사용하고자 하는 HTTP 서버의 dependency를 추가해야 한다. 예를 들어 아래와 같이 jetty를 사용하고자 하는 경우 tomcat을 의존성에서 제외시키고, jetty를 포함한다.

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<exclusions>
		<!-- Exclude the Tomcat dependency -->
		<exclusion>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
		</exclusion>
	</exclusions>
</dependency>
```

## Disabling the Web Server

Embedded 웹서버를 사용하고 싶지 않다면, 아래 설정을 추가하면 된다

```properties
spring.main.web-application-type: none
```

## Configure the Web Server

웹 서버를 커스터마이징 하려면 application.properties에 새로운 속성을 추가하면 된다. server.\* 설정이나 server.tomcat.\* 설정들을 통해서 웹서버를 커스터마이징할 수 있다. 하지만 커스터마이징 하고자 하는 부분이 설정으로 지원되지 않는 경우, WebServerFactoryCustomizer 클래스를 이용해야 한다. WebServerFactoryCustomizer에서는 Server Factory에 접근할 수 있다. 예를 들어 아래는 Tomcat을 커스터마이징 하는 WebServerFactoryCustomizer이다.

```java
@Component
public class MyTomcatWebServerCustomizer
		implements WebServerFactoryCustomizer<TomcatServletWebServerFactory> {

	@Override
	public void customize(TomcatServletWebServerFactory factory) {
		// customize the factory here
	}
}
```

## defaultServletName 오류

Spring Boot 2.4로 업그레이드한 후 더이상 시작되지 않는다. 다음과 같은 오류가 있다.

```shell
nable to locate the default servlet for serving static content. Please set the 'defaultServletName' property explicitly.
```

으로 봄 부팅 2.4 릴리스 노트에 기술 의 DefaultServlet임베디드 서블릿 컨테이너에 의해 제공은 더 이상 기본적으로 등록되지 않습니다. 응용 프로그램에 필요한 경우으로 설정 server.servlet.register-default-servlet하여 활성화 할 수 있습니다 true.

또는 WebServerFactoryCustomizerBean을 사용하여 프로그래밍 방식으로 구성 할 수 있습니다

```java
@Bean
WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> enableDefaultServlet() {
    return (factory) -> factory.setRegisterDefaultServlet(true);
}
```



## Server 설정
```yaml
# application.yml (아래의 값은 default)
server:
  tomcat:
    threads:
      max: 200        # 생성할 수 있는 thread의 총 개수
      min-spare: 10   # 항상 활성화 되어있는(idle) thread의 개수
    max-connections: 8192 # 수립가능한 connection의 총 개수
    accept-count: 100     # 작업큐의 사이즈
    connection-timeout: 20000  # timeout 판단 기준 시간, 20초
  port: 8080 # 서버를 띄울 포트번호
```



### 스레드풀
Thread Pool은 프로그램 실행에 필요한 Thread들을 미리 생성해놓는다는 개념이다. Thread는 cpu의 자원을 이용하여 코드를 실행하는 하나의 단위 이다. 

모든 요청에 대해 스레드를 생성하고 소멸하는 것은 OS와 JVM에 많은 부담이 되기 때문에 미리 Thread를 생성해 놓는다. 


* 스레드 풀은 미리 생성해 놓는다. 
* 요청이 들어 오면 Queue에 넣는다. 
* 유휴 상태의 스레드가 있으면 큐에서 꺼내 스레드에 작업을 할당하여 처리한다. 
* 유휴 상태의 스레드가 없으면 요청은 큐에서 대기한다. 
* 유휴 상태의 스레드가 없고 큐가 꽉 차면 새로운 스레드를 생성한다. 
* 스레드 최대 사이즈에 도달하고 큐도 꽉 차면 connection-refused 오류를 반환한다. 
* 작업 큐가 비어 있고 core size 이상의 스레드가 생성되어 있다면 스레드를 destroy 한다. 


![](.gitbook/assets/2022-01-11-09-08-35.png)



* 스레드는 많으면 너무 많은 스레드가 cpu의 자원을 두고 경합하게 되므로 처리속도가 느려질 수 있다. 
* 적으면 cpu자원을 최적으로 활용하지 못하여 처리속도가 느려질 수 있다. 
* 스레드는 적절한 수로 유지되는 것이 가장 좋다. 


**필드** 
* accept-count 작업큐의 사이즈 
* min-spare 디폴트는 10(core size를 10으로 설정) 


















