# 문서화

## Spring RESTful 서비스 테스트

이 섹션에서는 Spring REST API를 테스트하는 방법을 알아본다.

### 의존성

아래의 의존성이 추가되어 있다면 별다른 의존성 추가는 없다.

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
</dependencies>
```

### import

MockMvcRequestBuilders의 메서드 들을 static으로 임포트한다.

```java
import static org.hamcrest.Matchers.containsString;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
```

### MockMvc 주입

MockMvc class를 멤버 변수로 주입한다.

```java
package com.sogood.demo;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.restdocs.AutoConfigureRestDocs;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.web.servlet.MockMvc;

import static org.hamcrest.Matchers.containsString;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@WebMvcTest(DemoRestController.class)
@AutoConfigureRestDocs
public class DemoRestControllerTest {
  
  @Autowired
	private MockMvc mockMvc; 

  @Test 
  public void testGetMember() throws Exception { 
  }
}
```

### @WebMvcTest

* MVC 테스를 위한 어노테이션
* 웹에서 테스트하기 어려운 컨트롤러를 테스트하기에 적합
* @SpringBootTest 어노테이션보다 가볍게 테스트 가능

@SpringBootTest의 경우 일반적인 테스트로 slicing을 전혀 사용하지 않기 때문에 전체 응용 프로그램 컨텍스트를 시작한다. 그렇기 때문에 전체 응용 프로그램을 로드하여 모든 bean을 주입하기 때문에 속도가 느리다. @WebMvcTest의 경우는 Controller layer를 테스트하고 모의 객체를 사용하기 때문에 나머지 필요한 bean을 직접 세팅해줘야 한다. 그렇기 때문에 가볍고 빠르게 테스트 가능하다.

### MockMvc 메서드

MockMvc 클래스를 사용하여 API를 테스트할 수 있다. perform()을 통하여 요청을 보내고 andExpect()를 통하여 결과를 점검하고 andDo()를 통하여 응답 결과를 출력할 수 있다.

```java
@WebMvcTest(DemoRestController.class)
@ActiveProfiles("prod")
public class DemoRestControllerTest {

  @Autowired
  private MockMvc mockMvc; 

  @Test
  void testGetMember() throws Exception { 
    this.mockMvc
         .perform(get("/rest/member/1").accept(MediaType.APPLICATION_JSON))
         .andExpect(status().isOk())
         .andDo(print());
  }
}///~
```

### perform()

요청을 전송하는 역할을 한다. 요청을 보낼 때에는 get(), post(), put(), delete() 등의 함수를 사용한다. get() 메서드에 URL 정보를 전달하고 이 메서드가 반환하는 객체를 통해서 요청 정보들을 설정한다.

**accept()**

get(), post(),delete() 등의 메서드 함수는 accept(MediaType.APPLICAION_JSON)과 같이 클라이언트가 받아들일 수 있는 Media Type을 지정할 수 있다.

```java
// DemoRestController.java
@GetMapping("/member/{id}")
public DemoMemberBean getMember(@PathVariable Integer id) {
  DemoMemberBean bean = new DemoMemberBean();
  bean.setId("latte");
  bean.setTitle("Manager");
  bean.setAuthor("Latte");
  return bean; 
}//:
```

```java
// DemoRestcontrollerTest.java
@Test
void testGetMember() throws Exception { 
  this.mockMvc
       .perform(get("/rest/member/1").accept(MediaType.APPLICATION_JSON))
       .andExpect(status().isOk())
       .andDo(print());
}
```

get("/rest/member/1")과 같이 URL을 지정하여 GET 요청을 할 수 있다. andExpect(status().isOk())를 사용하여 응답이 정상적인지 확인할 수 있다. 결과는 andDo(print())를 사용하여 출력한다.

**header()**

header()를 사용하여 HEADER 정보를 설정할 수 있다.

```java
@GetMapping( value="/member2/{id}", headers = "X-UserType=EMPLOYEE")
public DemoMemberBean getMember2(@PathVariable Integer id) {
  DemoMemberBean bean = new DemoMemberBean();
  bean.setId("id");
  bean.setTitle("title");
  bean.setAuthor("Latte");
  return bean; 
}//:
```

```java
@Test
void testGetMember2() throws Exception { 
  this.mockMvc
       .perform(get("/rest/member2/1")
             .accept(MediaType.APPLICATION_JSON)
             .header("X-UserType", "EMPLOYEE"))
       .andExpect(status().isOk())
       .andDo(print());
}
```

#### headers()를 이용한 여러개의 HEADER 설정

```java

  void testGetHeaders() throws Exception { 
    HttpHeaders httpHeaders = new HttpHeaders();
    httpHeaders.set("X-UserType", "EMPLOYEE");
    httpHeaders.set("X-DeptType", "DEPT");

    this.mockMvc
         .perform(get("/rest/getHeaders")
               .accept(MediaType.APPLICATION_JSON)
               .headers(httpHeaders))
         .andExpect(status().isOk())
         .andDo(print());
  }
```

```java
  @GetMapping( value="/getHeaders")
  public Set<String> getHeaders(@RequestHeader  HttpHeaders httpHeaders) {
    return httpHeaders.keySet();
  }//:
```

#### Parameter 전달

```java
@Test
void testGetName() throws Exception { 
  this.mockMvc
       .perform(get("/rest/getName")
             .accept(MediaType.APPLICATION_JSON)
             .param("name", "Latte"))
       .andExpect(status().isOk())
       .andDo(print());
}
```

```java
@GetMapping( value="/getName")
public String getName(@RequestParam("name") String name) {
  return name; 
}//:
```

#### 여러개의 파라미터 전달

```java
@Test
void testGetNames() throws Exception { 
   MultiValueMap<String, String> reaParams = new LinkedMultiValueMap<>();
   reaParams.add("name", "Latte");
   reaParams.add("from", "Korea");
  this.mockMvc
       .perform(get("/rest/getParams")
             .accept(MediaType.APPLICATION_JSON)
             .params(reaParams))
       .andExpect(status().isOk())
       .andDo(print());
}
```

```java
@GetMapping( value="/getParams")
 public Set<String>  getName(@RequestParam Map<String, String> param) {
   return param.keySet();
}//:
```

#### 변수를 사용하여 호출

```java

@Test
public void getEmployeeByIdAPI() throws Exception 
{
  mvc.perform( MockMvcRequestBuilders
      .get("/employees/{id}", 1)
      .accept(MediaType.APPLICATION_JSON))
      .andDo(print())
      .andExpect(status().isOk())
      .andExpect(print());
}
```

#### HTTP POST

```java
@PostMapping(value = "/employees")
public ResponseEntity<EmployeeVO> addEmployee (@Valid @RequestBody EmployeeVO employee)
{
    //code
    return new ResponseEntity<EmployeeVO>(employee, HttpStatus.CREATED);
}
```

```java
@Autowired
private MockMvc mvc;
 
@Test
public void createEmployeeAPI() throws Exception 
{
  mvc.perform( MockMvcRequestBuilders
      .post("/employees")
      .content(asJsonString(new EmployeeVO(null, "firstName4", "lastName4", "email4@mail.com")))
      .contentType(MediaType.APPLICATION_JSON)
      .accept(MediaType.APPLICATION_JSON))
      .andExpect(status().isCreated())
      .andExpect(MockMvcResultMatchers.jsonPath("$.employeeId").exists());
}
 
public static String asJsonString(final Object obj) {
    try {
        return new ObjectMapper().writeValueAsString(obj);
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
}
```

#### HTTP PUT

```java
@PutMapping(value = "/employees/{id}")
public ResponseEntity<EmployeeVO> updateEmployee (@PathVariable("id") int id, @Valid @RequestBody EmployeeVO employee)
{
    //code
    return new ResponseEntity<EmployeeVO>(emp, HttpStatus.OK);
}
```

```java
@Test
public void updateEmployeeAPI() throws Exception 
{
  mvc.perform( MockMvcRequestBuilders
      .put("/employees/{id}", 2)
      .content(asJsonString(new EmployeeVO(2, "firstName2", "lastName2", "email2@mail.com")))
      .contentType(MediaType.APPLICATION_JSON)
      .accept(MediaType.APPLICATION_JSON))
      .andExpect(status().isOk())
      .andExpect(MockMvcResultMatchers.jsonPath("$.firstName").value("firstName2"))
      .andExpect(MockMvcResultMatchers.jsonPath("$.lastName").value("lastName2"))
      .andExpect(MockMvcResultMatchers.jsonPath("$.email").value("email2@mail.com"));
}
```

#### HTTP DELETE

```java
@DeleteMapping(value = "/employees/{id}")
public ResponseEntity<HttpStatus> removeEmployee (@PathVariable("id") int id)
{
    //code
    return new ResponseEntity<HttpStatus>(HttpSt
}
```

```java
@Test
public void deleteEmployeeAPI() throws Exception 
{
  mvc.perform( MockMvcRequestBuilders.delete("/employees/{id}", 1) )
        .andExpect(status().isAccepted());
}
```

### 응답값 검증(andExpect())

**andExpect()** 응답을 검증한다.

* 상태코드
  * 메서드 이름: 상태코드
  * isOk(): 200
  * isNotFound(): 404
  * isMethodNotAllowed(): 405
  * isInternalServerError(): 500
  * is(int status): status 상태코드
* 뷰(view())
  * 리턴하는 뷰 이름을 검증
  * view().name("blog") 리턴하는 뷰 이름이 blog인가?
* 리다이렉트(redirect())
  * 리다이렉트 응답을 검증
  * redirectUrll("/blog") blog로 리다이렉트 되었는가?
* 모델 정보(model())
  * 컨트롤러에서 저장한 모델들의 정보 검증
* 응답검증( content())
  * 응답에 대한 검증

```java
@Test
public void getEmployeeByIdAPI() throws Exception 
{
  mvc.perform( MockMvcRequestBuilders
      .get("/employees/{id}", 1)
      .accept(MediaType.APPLICATION_JSON))
      .andDo(print())
      .andExpect(status().isOk())
      .andExpect(MockMvcResultMatchers.jsonPath("$.employeeId").value(1));
}
```

JsonPath에 대한 자세한 사용방법은 [다른 문서](https://advenoh.tistory.com/28)를 참고한다.

```java
@Autowired
private MockMvc mvc;
 
@Test
public void getAllEmployeesAPI() throws Exception 
{
  mvc.perform( MockMvcRequestBuilders
      .get("/employees")
      .accept(MediaType.APPLICATION_JSON))
      .andDo(print())
      .andExpect(status().isOk())
      .andExpect(MockMvcResultMatchers.jsonPath("$.employees").exists())
      .andExpect(MockMvcResultMatchers.jsonPath("$.employees[*].employeeId").isNotEmpty());
}
 

@Test
public void getEmployeeByIdAPI() throws Exception 
{
  mvc.perform( MockMvcRequestBuilders
      .get("/employees/{id}", 1)
      .accept(MediaType.APPLICATION_JSON))
      .andDo(print())
      .andExpect(status().isOk())
      .andExpect(MockMvcResultMatchers.jsonPath("$.employeeId").value(1));
}
```

### 응답값 출력(andDo())

andDo() 메소드를 도큐먼트에서 찾아보면 “Perform a general action.” 이라는 아주 간단한 설명만 나온다. 이 메소드는 인수에 실행 결과를 처리할 수 있는 ResultHandler를 지정해서 사용하는데, 이 ResultHandler는 log 메소드와 print 메소드를 지원하고 있다. log() 실행결과를 디버깅 레벨에서 로그로 출력하고 print() 실행결과를 임의의 출력대상에 출력 출력대상을 지정하지 않으면 System.out 으로 출력한다.

## Spring REST Docs

지금까지 스프링 테스트 프레임워크를 사용하여 RESTful 서비스들을 테스트하는 방법을 살펴보았다. 이 섹션에서는 문서화하는 방법을 알아본다.

Sping REST Docs는 Spring MVC's test framework로 쓰여진 테스트들에 의해서 생성된 snippets를 사용한다. 즉, 테스트 프레임워크를 사용하여 테스트를 하면 target/generated-snippets에 snippet 문서들이 만들어 지는데 이 문서들은 확장자가 .adoc이다. adoc는 AsciiDoc라는 문서 포맷이다. 이 포맷은 Markdown과 비슷한 문서 포맷이라고 이해하면 된다. 그러면 Maven 빌드할 때 asciidoctor-maven-plugin이 sourceDirectory의 경로의 AsciiDoc 파일들을 찾아서 outputDirectory에 html 문서들을 생성한다.

### 요구사항

Spring REST Docs는 최소한 다음을 필요로 한다.

* Java 8
* SPring Framework 5.0.2 or later

### Build Configuration

pom.xml 파일을 열고 의존성과 플러그인 들을 추가한다.

```xml
	<properties>
		<java.version>11</java.version>
		<project-version>2.0.5.RELEASE</project-version>
		<asciidoctor-plugin.version>1.5.6</asciidoctor-plugin.version>
		<snippetsDirectory>${project.build.directory}/generated-snippets</snippetsDirectory>
	</properties>
  <dependencies>
		<dependency>
			<groupId>org.springframework.restdocs</groupId>
			<artifactId>spring-restdocs-mockmvc</artifactId>
			<version>${project-version}</version>
			<scope>test</scope>
		</dependency>
  </dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.asciidoctor</groupId>
				<artifactId>asciidoctor-maven-plugin</artifactId>
				<version>${asciidoctor-plugin.version}</version>
				<executions>
					<execution>
						<id>generate-docs</id>
						<phase>package</phase>
						<goals>
							<goal>process-asciidoc</goal>
						</goals>
						<configuration>
							<backend>html</backend>
							<doctype>book</doctype>
							<attributes>
								<snippets>${snippetsDirectory}</snippets>
							</attributes>
							<sourceDirectory>src/docs/asciidocs</sourceDirectory>
							<outputDirectory>target/generated-docs</outputDirectory>
						</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
```

* test scope로 spring-restdocs-mockmvc 의존성을 추가한다.
* Asciidoctor plugin을 추가한다.
* 문서가 패키지에 포함되도록 prepare-package를 사용한다.
* spring-restdocs-asciidoctor를 Asciidoctor plugin의 의존성으로 추가한다. 이것은 target/generated-snippets에 대한 포인트로 .adoc 파일들에서 사용되도록 snippets 속성들을 설정할 것이다.

### Packaging the Documetation

프로젝트의 jar 파일에 생산된 문서를 패키징하고 싶을 수도 있다. 예를 들어 Spring Boot에 의해 제공되는 static content에 넣고 싶을 수도 있다. 이것을 하기 위해 project build를 설정해야 한다.

```xml
	<build>
		<plugins>
			<plugin>
				<artifactId>maven-resources-plugin</artifactId>
				<version>2.7</version>
				<executions>
					<execution>
						<id>copy-resources</id>
						<phase>prepare-package</phase>
						<goals>
							<goal>copy-resources</goal>
						</goals>
						<configuration>
							<outputDirectory>
							${project.build.outputDirectory}/static/docs
						</outputDirectory>
							<resources>
								<resource>
									<directory>
									${project.build.directory}/generated-docs
								</directory>
								</resource>
							</resources>
						</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
```

* Asciidoctor plugin 이후에 resource plugin이 선언되어야 한다.
* static/docs 디렉터리로 생산된 문서들이 복사된다.

### Generating Documentation Snippets

Spring REST Docs는 Spring MVC's test framework를 사용한다.

#### Setting up Your JUnit 5 Tests

JUnit 5를 사용할 때 제일먼저 할일은 test class에 RestDocumentationExtension을 적용하는 것이다.

```java
@ExtendWith(RestDocumentationExtension.class)
public class JUnit5ExampleTests {
```

전형적인 스프링 애플리케인션을 테스트할 때는 SpringExtenssion을 적용해야 한다.

```java
@ExtendWith({RestDocumentationExtension.class, SpringExtension.class})
public class JUnit5ExampleTests {
```

RestDocumentationExtension은 build 툴에 의해 자동적으로 output directory가 구성된다.

| Build tool | Output directory          |
| ---------- | ------------------------- |
| Maven      | target/generated-snippets |
| Gradle     | build/generated-snippets  |

다음으로 MockMvc, WebTestClient, RESTAssured를 구성하기 위해서 @BeforeEach를 제공해야 한다.

```java
private MockMvc mockMvc;

@BeforeEach
public void setUp(WebApplicationContext webApplicationContext,
		RestDocumentationContextProvider restDocumentation) {
	this.mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext)
			.apply(documentationConfiguration(restDocumentation)) 
			.build();
}
```

MockMvcRestDocumentationConfigurer에 의해 MockMVc 인스턴스가 구성된다. org.springframework.restdocs.mockmvc.MockMvcRestDocumentation의 documentationConfiguration() 메서드로부터 이 클래스의 인스턴스를 얻을 수 있다.

구성자는 합리적인 기본값을 적용하고 구성을 사용자 지정하기위한 API도 제공한다. 자세한 정보는 [설정하기](https://docs.spring.io/spring-restdocs/docs/current/reference/html5/#configuration)를 참고한다.

실제 테스트 케이스를 실행하고 Snippets를 생성하기 위해서는 다음과 같이 애너테이션이 적용되어야 한다.

```java
@ActiveProfiles("prod")
@WebMvcTest(DemoRestController.class)
//@AutoConfigureMockMvc
@AutoConfigureRestDocs
@ExtendWith({ RestDocumentationExtension.class, SpringExtension.class })
// @SpringBootTest(classes = SogoodApplication.class)
public class DemoRestControllerTest {
}
```

**@ActiveProfiles** 어떤 프로파일 환경에서 테스트할지 설정한다. **@WebMvcTest** 어떤 컨트롤러를 테스트할지를 설정한다. @AutoConfigureMockMvc를 사용할 수도 있다. @AutoConfigureMockMvc를 사용하면 @Service, @Respository가 붙은 객체들도 모두 메모리에 올린다. 간단하게 테스트하기 위해서는 @WebMvcTest를 사용해야 한다.

@WebMvcTest는 @SpringBootTest와 같이 사용할 수는 없다. 왜냐하면 각자 서로의 MockMvc를 목킹하기 때문에 충돌이 발생하기 때문이다. @SpringBootTest(webEnvironment=WebEnvironment.MOCK) 설정으로 모킹한 객체를 의존성 주입받으려면 @AutoConfigureMockMvc를 클래스 위에 추가 해야한다.

**구글링을 하다보면 @AutoConfigureRestDocs를 사용하는 예제가 나오는데 실제로는 타입에 이 애너테이션을 붙이지 않아도 적상적으로 동작했다. **[**Spring REST Docs**](https://docs.spring.io/spring-restdocs/docs/current/reference/html5/#introduction)** 문서에서도 이 애너테이션을 사용하지는 않는다.**

**@SpringBootTest**는 @ExtendedWith(SpringExtension.class)를 가지고 있다.정리하자면, 최근 스프링 부트는 JUnit 5를 사용하기 때문에 더이상 JUnit 4에서 제공하던 @RunWith를 쓸 필요가 없고 (쓰고 싶으면 쓸 수는 있지만), @ExtendWith를 사용해야 하지만, 이미 스프링 부트가 제공하는 모든 테스트용 애노테이션에 메타 애노테이션으로 적용되어 있기 때문에 @ExtendWith(SpringExtension.class)를 생략할 수 있다.

#### Invoking the RESTful Service

testing framework를 구헝했으니까, RESTful 서비스를 호출하기 위해서 그것을 사용할 수 있고 요청과 응답을 문서화할 수 있다.

```java
this.mockMvc.perform(get("/").accept(MediaType.APPLICATION_JSON))  // ①
	.andExpect(status().isOk())   // ②
	.andDo(document("index"));  // ③
```

* ① 서비스의 루트(/)를 호출하고 application/json 응답을 요구한다.
* ② 서비스가 기대되는 응답을 생산할 것을 검증한다.
* ③ 서비스 호출을 문서화하고, index라는 이름으로 snippets를 쓴다. Snippets는 RestDocumentationResultHandler에 의해서 쓰여진다. org.springframework.restdocs.mockmvc.MockMvcRestDocumentation의 정적 document 메서드를 이 클래스의 인스턴스로부터 얻을 수 있다.

디폴트로, 여섯개의 snippets가 쓰여진다.

* /index/curl-request.adoc
* /index/http-request.adoc
* /index/http-response.adoc
* /index/httpie-request.adoc
* /index/request-body.adoc
* /index/response-body.adoc

#### Using the Snippets

생성된 snippet 들을 쓰기 전에, .adoc 소스 파일을 생성해야 한다. 이름은 무엇이던 관계 없고 .adoc 확장자만 있으면 된다. 결과로 생성된 HTML file은 같은 이름을 가지지만 .html 확장자를 가진다. 소스 파일들과 결과로 만들어진 HTML 파일들의 위치는 Maven 또는 Gradle을 사용하는 것에 따라 다르다.

| Build tool | Source files              | Generated files               |
| ---------- | ------------------------- | ----------------------------- |
| Maven      | src/main/asciidoc/\*.adoc | target/generated-docs/\*.html |
| Gradle     | src/docs/asciidoc/\*.adoc | build/asciidoc/html5/\*.html  |

생성된 snippets들을 include macro를 사용하여 수동으로 생성된 assciidoc에 포함할 수 있다. spring-restdocs-asciidoctor에 의해 자동적으로 설정된 snippets 속성을 사용할 수 있다.

```shell
include::{snippets}/index/curl-request.adoc[]
```

### Documenting Your API

#### 서비스 구분하기

andDo(document("index/index"))와 같이 코드를 작성하면 target/generated-snippets 디렉터리에 index/index 디렉터리가 생성되고 그 안애 snippets 파일들이 생성된다.

```java
@Test
public void testRoot() throws Exception { 
      this.mockMvc.perform(get("/rest/index").accept(MediaType.APPLICATION_JSON)) 
     .andExpect(status().isOk()) 
          //.andDo(print());
     .andDo(document("index/index")); 
}
```

#### 응답결과 문서화 하기

andDo(document("member/get"))과 같이 코드를 작성하면 target/generated-snippets 디렉터리 아래에 member/get 디렉터리가 생성되고 그 안에 snippets 파일들이 생성된다. responseFields()를 사용하여 응답 결과를 문서화 한다.

```java
@Test
void testGetMember() throws Exception {
      this.mockMvc.perform(get("/rest/member/1").accept(MediaType.APPLICATION_JSON)).ect(status().isOk())
                  .andDo(print())
                  .andDo(document("member/get", responseFields( 
                               fieldWithPath("id").description("사용자 아이디")
                             , fieldWithPath("title").description("직위")
                             , fieldWithPath("author").description("저자")
                         )));
}
```

snippets/member/get 디렉터리에 response_fields.adoc가 생성된다.

```adoc
|===
|Path|Type|Description

|`+id+`
|`+String+`
|사용자 아이디

|`+title+`
|`+String+`
|직위

|`+author+`
|`+String+`
|저자

|===
```

#### Hypermedia

Spring REST Docs는 hypermedia 기반의 링크들을 문서화하는 것을 지원한다.

```java
this.mockMvc.perform(get("/").accept(MediaType.APPLICATION_JSON))
	.andExpect(status().isOk())
	.andDo(document("index", links(   // (1)
			linkWithRel("alpha").description("Link to the alpha resource"),  // (2) 
			linkWithRel("bravo").description("Link to the bravo resource"))));  // (3)
```

* (1) REST docs가 응답의 링크들을 설명하는 snippet을 생성하기 위해 구성된다. org.springframework.restdocs.hypermedia.HypermediaDocumentation의 static 메서드를 사용한다.
* (2) 'rel'이 'alpha'인 링크를 기대한다. rg.springframework.restdocs.hypermedia.HypermediaDocumentation의 linkWithRel 정적 메서드를 사용한다.
* (3) rel이 bravo인 링크를 기대한다.

그 결과는 links.adoc라는 이름의 snippet이다. 그것은 리소스의 링크들을 포함하는 테이블을 포함한다.

## AsciiDocs

AsciiDocs 문서는 Makrdown과 비슷한 문서라고 생각하면 된다. 자세한 사항은 [여기](https://marketplace.visualstudio.com/items?itemName=asciidoctor.asciidoctor-vscode)를 참고하고 [공식사이트](https://docs.asciidoctor.org/asciidoc/latest/)를 참고한다. VSCode에서는 [AsciiDocs Extension](https://marketplace.visualstudio.com/items?itemName=asciidoctor.asciidoctor-vscode)를 설치하면 미리보기도 지원한다.

최종적으로 생성된 snippet 파일들을 HTML로 변환해야 한다. 이와 관련하여 pom.xml에 설정을 위에서 했었다. asciidoctor-maven-plugin에 sourceDirectory를 src/docs/asciidocs로 설정했었다.

```xml
<plugin>
	<groupId>org.asciidoctor</groupId>
	<artifactId>asciidoctor-maven-plugin</artifactId>
	<version>${asciidoctor-plugin.version}</version>
	<executions>
		<execution>
			<id>generate-docs</id>
			<phase>package</phase>
			<goals>
				<goal>process-asciidoc</goal>
			</goals>
			<configuration>
				<backend>html</backend>
				<doctype>book</doctype>
				<attributes>
					<snippets>${snippetsDirectory}</snippets>
				</attributes>
				<sourceDirectory>src/docs/asciidocs</sourceDirectory>
				<outputDirectory>target/generated-docs</outputDirectory>
			</configuration>
		</execution>
	</executions>
</plugin>
```

src/docs/asciidocs 디렉터리를 생성하고 그 안에 index.adoc 파일을 생성한다. 이름은 아무거나 관계 없다. 다음과 같이 입력한다.

```adoc
ifndef::snippets[]
:snippets: ../../../target/generated-snippets
endif::[]
:doctype: book
:icons: font
:source-highlighter: highlightjs
:toc: left
:toclevels: 2
:sectlinks:


[[resources]]
= Resources

[[resources-index]]
== Index API

[[resources-index-index]]
=== Index Demo Page

include::{snippets}/index/index/curl-request.adoc[]
include::{snippets}/index/index/http-request.adoc[]
include::{snippets}/index/index/http-response.adoc[]
include::{snippets}/index/index/httpie-request.adoc[]
include::{snippets}/index/index/request-body.adoc[]
include::{snippets}/index/index/response-body.adoc[]

[[resources-member]]
== Member API

[[resources-member-get]]
=== 멤버 조회

include::{snippets}/member/get/curl-request.adoc[]
include::{snippets}/member/get/http-request.adoc[]
include::{snippets}/member/get/http-response.adoc[]
include::{snippets}/member/get/httpie-request.adoc[]
include::{snippets}/member/get/request-body.adoc[]
include::{snippets}/member/get/response-body.adoc[]
include::{snippets}/member/get/response-fields.adoc[]
```

snippets 파일들을 읽어서 HTML을 생성하기 때문에 다음과 같이 snippets 경로를 설정해야 한다.

```adoc
ifndef::snippets[]
:snippets: ../../../target/generated-snippets
endif::[]
```

Maven으로 빌드하면 다음과 같이 목차가 생성된다.

```
Table of Contents
  Index API
     Index Demo Page
  Member API
    멤버 조회
```

목차를 만들기 위해서 다음과 같은 형식으로 작성해야 한다.

```adoc
[[resources]]
= Resources

[[resources-index]]
== Index API

[[resources-index-index]]
=== Index Demo Page
```

### 문서 속성

콜론(:)으로 시작하는 줄은 속성이다. 빌트인 속성들이 존재하는데 doctype도 그 중 하나이다. 속성은 AsciiDoc 문서를 처리하는 프로세스의 동작을 설정하거나 정보를 전달하기 위해서 사용한다.

#### doctype

```asciidoc
:doctype: book
```

doctype 속성은 출력될 문서의 형식을 설정한다. article, book, inline, manpage 중 하나를 설정할 수 있다.

#### toc

```asciidoc
:toc: left
:toclevels: 2
```

toc는 table of contesnts를 켜거나 위치를 결정한다. 빈 값일 수는 있는데 이럴때 디폴트는 auto이다. auto, left, right 중 하나를 선택한다.

toclevel은 표시할 최대 레벨을 표시한다. 1\~5까지 선택할 수 있다. 기본값은 2이다.

#### icons

```asciidoc
:icons: font 
```

경고 등을 표시하기 위해서 image나 font를 사용할 수 있다. 디폴트는 image이고 font,image 중 하나를 선택해야 한다.

#### source-highlighter

```asciidoc
:source-highlighter: highlightjs
```

source code highlighter를 선택할 수 있다. highlight.js, paygments, rouge 중 하나를 선택한다.

#### sectlinks

```asciidoc
:sectlinks:
```

Section title link 들을 활성화 한다. 값은 없다.

#### 사용자 속성

사용자 속성을 정의할 수 있다.

```asciidoc
:name: Latte
```

사용자 속성은 curly brace로 사용할 수 있다.

```asciidoc
This is {name}
```

### 타이틀

타이틀은 '='를 사용한다. '=' 는 가장 큰 타이틀이고 작은 타이틀은 레벨에 따라 '=='와 같이 문자를 반복한다.

```asciidoc
= Document Title (Level 0)

== Level 1 Section Title

=== Level 2 Section Title

==== Level 3 Section Title

===== Level 4 Section Title

====== Level 5 Section Title

== Another Level 1 Section Title
```

### links

간단히 링크를 매크로로 만들 수 있다.

```asciidoc
https://asciidoctor.org[]
```

다음과 같이 표시된다. 클릭하면 URL로 이동한다.

```
https://asciidoctor.org
```

링크를 문자열로 변환하고 싶으면 다음과 같이 작성한다.

```asciidoc
Ask questions on the https://discuss.asciidoctor.org/[*mailing list*].
```

다음과 같이 표시된다. mailing list는 볼드체로 표시된다.

```html
Ask questions on the mailing list.
```

### ID

직각 괄호를 사용하여 문서의 어느 곳에서나 ID를 만들 수 있다. 다음은 블록요소인 paragraph에 ID를 부여한다.

```asciidoc
[[notice]]
This paragraph gets a lot of attention.
```

다음은 inline-anchor를 정의한다.

```asciidoc
[[bookmark-a]]Inline anchors make arbitrary content referenceable.
```

inline anchor를 사용하여 section의 아이디를 부여할 수 있다.

```asciidoc
=== Version 4.9 [[version-4_9]]
```
