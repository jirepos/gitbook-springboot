# 시작하기

Spring Boot를 사용하여 Web Application을 구축하는 과정을 튜터리얼로 정리한다. 이 튜터리얼은 Spring Boot를 빠르게 시작하는 방법을 설명하지만, 초보자를 위한 튜토리얼이 아니기 때문에 기초적인 내용은 설명하지 않는다.

## SpringBoot 프로젝트 구조

SpringBoot 의 프로젝트 구조는 Maven과 Gralde에서 공통으로 사용할 수 있는 구조로 만든다. 프로젝트는 Maven 프로젝트 형태로 만들어도 되고 Gradle 프로젝트로 만들어도 된다. 둘 다 Java의 표준 프로젝트 구조로 만든다.

```
src/main/java
src/main/resources
src/test/java
```

SpringBoot 프로젝트를 쉽게 만들 수 있도록 지원하는 사이트가 있다. 아래의 사이트로 가서 만드는 것을 추천한다. Spring Initializer를 통해 Spring 프로젝트를 만들었다고 가정한다.

[Spring initializr](https://start.spring.io)

SpringBoot에서 Image, CSS, JavaScript 와 같은 정적파일들은 src/main/resources 디렉토리 아래에 둔다.

```shell
📁project_root
  📁src
    📁main
      📁resources
         📁public
         📁static
         📁templates            
```

**templates**\
Thymleaf나 FreeMarker와 같은 View 파일들을 둔다. 브라우저에서 직접 호출이 할 수 없다.

**static**\
image, css , javascript 등을 둔다. 브라우저를 통해 직접 호출이 가능하다. static은 URL에 포함하지 않는다.

**public**\
공개된 html 파일들을 둔다. 브라우저를 통해 직접 호출이 가능하다.

SpringBoot에서는 자동적으로 정적 리소스들을 다음의 폴더에 두고 클래스패스에 추가한다.

* classpath:/public/
* classpath:/static/
* classpath:/resources/
* classpath:/META-INF/resources/

이 위치는 classpath에 있게 되고 모두가 그 위치를 사용할 수 있다.

### index.html

static 폴더 아래에 index.html을 만든다. Spring Boot를 시작하고 브라우저를 열고 다음의 URL을 입력한다.

* Spring Boot가 static 폴더에서부터 경로를 찾는다. 따라서 static/index.html과 같이 URL을 입력할 필요가 없다.
* index.html을 URL에 적을 필요는 없다. 디폴트로 index.html을 찾는다.

```
http://localhost:8080/
```

### SpringBoot 특징

**default package는 사용하지 않음**\
패키지 경로가 없는 클래스를 생성하지 말아야 한다. 스프링 부트는 @ComponentScan, @EntityScan 을 사용하는 스프링 부트에서는 특별한 문제가 될 수 있다.

**Main Application 위치**\
다른 클래스의 상위의 root package에 main application을 두어라. 예를들어, 'com.sogood.xxx'와 같이 'com.sogood' 도메인의 하위 패키지부터 패키지 구조를 만든다면 'com.sogood' 패키지 아래에 Appliation.java를 두어야 한다.

**Static Content**\
스프링 부트 웹 애플리케이션에서 HTML, CSS, JS 및 IMAGE 파일과 같은 정적 파일은 다음 클래스 경로 위치에서 즉시 제공 될 수 있다. 구성이 필요하지 않다.

```
📁src/main/resources
  📄application.properties
  📁static
  📁templates
```

위의 프로젝트 구조에서 resources 폴더를 보자. 이름에서 알 수 있듯이이 디렉토리는 모든 정적 리소스, 템플릿 및 속성 파일 전용이다. 기본적으로 스프링 부트는 클래스 경로의 다음 위치 중 하나에서 정적 컨텐츠를 제공한다.

```
📁project_root
   📁static
   📁public
   📁resources
   📁META-INF/resources
```

Spring은 기본적으로 다음 템플릿 엔진을 지원한다. 이 템플릿은 적절한 스프링 부트 스타터를 사용하여 활성화 할 수 있다.

* FreeMarker - spring-boot-starter-freemarker
* Groovy - spring-boot-starter-groovy
* Thymeleaf - spring-boot-starter-thymeleaf
* Mustache - spring-boot-starter-mustache

이러한 모든 템플릿 엔진은 /src/main/resources/templates 경로에서 템플릿 파일을 확인한다. 스프링 부트 애플리케이션에서 정적 컨텐츠를 제공하는 방법에 대한 자세한 내용은 Spring Boot Official Doc-정적 컨텐츠를 참조하라.

**pom.xml**

스프링 부트 프로젝트에서 스타터 세트를 추가하여 대부분의 모듈을 활성화 또는 비활성화 할 수 있다. 모든 Spring Boot 프로젝트는 일반적으로 spring.boot-starter-parent를 pom.xml의 부모로 사용한다.

```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>2.0.4.RELEASE</version>
</parent>
```

spring-boot-starter-parent는 여러 하위 프로젝트 및 모듈에 대해 다음 사항을 관리 할 수있게한다.

```
Configuration - Java Version and Other Properties
Dependency Management - Version of dependencies
Default Plugin Configuration
```

이 의존성에 대해서는 Spring Boot 버전 번호 만 지정해야한다. 추가 스타터를 가져 오면 버전 번호를 안전하게 생략 할 수 있다.
