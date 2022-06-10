# 요청과 응답처리

## 요청과 응답 객체

클라이언트가 서버에 요청할 때 image, css, javascript, html과 같은 일반적인 정적인 자원을 요청할 때에는 정적폴더에서 직접 다운로드 받을 수 있게 한다. 그러나 word,excel과 같은 정보를 담고 있는 파일들은 이런 방법으로 다운로드 받지 못하도록 해야 한다. 파일 다운로드에 대해서는 다른 글에서 다루기로 하겠다.

클라이언트에서 서비스를 요청할 때에는 원칙적으로 JSON 형태로 전송한다. 예를들어 다음과 같다.

```
POST http://localhost:8080/echo HTTP/1.1
Content-Length: 73
Accept: application/json
Content-Type: application/json
Cookie: username=Latte

{"token":null,"payload":{"userId":"latte","userEmail":"latte@gmail.com"}}
```

JSON 형식으로 요청 데이터를 보내고 받을 때에 설정해야할 중요한 Header 값이 두 가지가 있다.

**Accept**\
Accept 헤더는 서버로 부터 응답을 받을 때 클라이언트가 인식을 할 수 있는 미디어 타입을 정의한다. 즉, 클라이언트는 이런 타입을 응답으로 되돌려 줘야 처리할 수 있다는 의미다.

클라이언트가 Accept 헤더에 application/json을 설정하여 요청할 때 서버에서 오류가 발생하면 이 값을 기준으로 응답을 JSON으로 반환한다. 이 값이 없으면 일반적인 HTTP 오류를 반환한다.

**Content-Type**\
Content-Type 헤더는 클라이언트와 서버가 모두 사용할 수 있다. 클라이언트는 이 헤더 값을 참고하여 서버로부터 응답이 JSON인지 아닌지 판단하여 처리한다. 기타 다른 타입들도 적절한 MIME TYPE을 설정해야 한다.

### 요청 객체

클라이언트가 서버에 무엇인가를 처리해 달라고 요청할 때에는 JSON 형식으로 서버에 보내야 한다. com.sogood.core.web.message 패키지를 생성한다. 그 안에 JSONReqMessage를 다음과 같이 작성한다. 'token' 필드는 JWT를 이용한 인증을 처리할 때 token을 주고 받기 위해 사용할 것이다.

```java
package com.sogood.core.web.message;

public class JSONReqMessage<T> {

  /** JWT Token */
  private String token;
  /** 요청 데이터 */
  private T payload;
  
  public String getToken() {
    return token;
  }
  public void setToken(String token) {
    this.token = token;
  }
  public T getPayload() {
    return payload;
  }
  public void setPayload(T payload) {
    this.payload = payload;
  }
}
```

> 실제 데이터인 payload는 무엇이든 담을 수 있도록 Java Generic을 사용한다. 나중에 Swagger를 이용하여 API 문서화를 진행할 것인데 현재로서는 Swagger이 Generic을 처리할 수 있는지는 확실하지 않다. 추후 내용을 보완할 예정이다.

### 응답 객체

응답 객쳬 또한 클라이언트가 전송하는 실제 데이터는 Java Generic을 사용한다.

```java
package com.sogood.core.web.message;

import org.springframework.http.HttpStatus;

public class JSONResMessage<T> {
  /** 응답코드: 정상인 경우에는 default: 200. Business Exception은 9000 번대 사용 */
  private String code;
  /** 에러가 있는 경우에는 에러 메시지. default는 success. */
  private String message;
  /** 응답 데이터  */
  private T payload;
  /** JWT Token */
  private String token;
  public JSONResMessage() {
    this.code = String.valueOf(HttpStatus.OK);
    this.message = "OK"; 
  }

  public String getCode() {
    return code;
  }
  public void setCode(String code) {
    this.code = code;
  }
  public String getMessage() {
    return message;
  }
  public void setMessage(String message) {
    this.message = message;
  }
  public T getPayload() {
    return payload;
  }
  public void setPayload(T payload) {
    this.payload = payload;
  }
  public String getToken() {
    return token;
  }
  public void setToken(String token) {
    this.token = token;
  } 
}
```

## 요청과 응답을 JSON으로 처리하기

클라이언트에서 서버로 요청할 때에는 View 화면 요청을 제외하곤 기본적으로 JSON 형태로 서버로 전송한다. JSON 형식으로 전송된 데이터는 @ReqeustBody 어노테이션을 사용하여 객체로 변환한다. 클라이언트에게 응답 하는 데이터도 기본적으로 @ResponeBody 어노테이션을 사용하여 JSON 형식으로 반환한다.

클라이언트의 요청을 처리하는 Controller의 메소드는 다음과 같은 형식으로 작성한다.

```java
  @PostMapping("/echo")
  public @ResponseBody JSONResMessage<DemoBean> echo(HttpServletRequest request, HttpServletResponse response, @RequestBody JSONReqMessage<DemoBean> payload) {
    JSONResMessage<DemoBean> resMessage = new JSONResMessage<DemoBean>();
    resMessage.setPayload(payload.getPayload());
    return resMessage; 
  }//:
```

@RequestMapping은 사용하지 않는다. GET 메소드 호출인지 POST 메소드 호출인지 명확하게 하기 위해서 @PostMapping 또는 @GetMapping을 사용하여 메소드를 명확하게 해야 한다.

### 요청데이터 작성

JSON 형식으로 서버에 요청할 때에는 객체 리터럴을 사용하여 객체를 정의한다. 객체에는 token과 payload 필드가 있어야 한다.

```javascript
var jsonReqMessage = { 
  token: null, 
  payload : {
    userId: "iinet",
    userEmail: "iinet@gmail.com" 
  }
}
```

이것을 실제 데이터를 전송할 때에는 JSON.stringify()을 사용하여 문자열로 변환해야 한다.

```javascript
var jsonReqStr = JSON.stringify(jsonReqMessage);
```

실제 요청 데이터를 작성하는 핵심부분은 이것이 전부다. 다만, 어떤 라이브러리를 사용하여 서버에 요청할지 아직 결정이 되지 않았기 때문에 Header 값 설정 등의 다른 설정에 대해서는 설명하지 않는다.

### 응답 데이터 작성

응답데이터는 @ResponseBody 어노테이션을 사용한다. 메소드는 다음과 같이 작성한다.

```java
  @PostMapping("/echo")
  public @ResponseBody JSONResMessage<클래스> echo(
          HttpServletRequest request
        , HttpServletResponse response
        , @RequestBody JSONReqMessage<클래스> reqMessage) {
    return 객체; 
  }//:
```

응답 데이터는 JSONResMessage 클래스를 인스턴흐화 해서 반환한다. 이 클래스는 Generic을 사용하므로 실제 데이터인 payload의 타입을 설정해야 한다.

```java
JSONResMessage<DemoBean> resMessage = new JSONResMessage<DemoBean>();
```

이 메소드의 전체 코드는 다음과 같이 작성될 수 있을 것이다.

```java
  @PostMapping("/echo")
  public @ResponseBody JSONResMessage<DemoBean> echo(
          HttpServletRequest request
        , HttpServletResponse response
        , @RequestBody JSONReqMessage<DemoBean> reqMessage) {
          JSONResMessage<DemoBean> resMessage = new JSONResMessage<DemoBean>();
          DemoBean demoBean = service.getDemoBean();
          resMessage.setPayload(demoBean);
    return resMessage; 
  }//:
```

위 코드에서 try-catch로 감싸는 예외처리 구문이 없다는 것을 눈치 챘을 것이다. 예외처리는 @ExceptionHandler 어노테이션을 사용하여 공통으로 처리하기 때문에 컨트롤러의 메소드에서는 try-catch 블록을 사용하지 않는다.

## ResponseEntity 사용

ResponseEntity 객체로 반환해주게 되면, HTTP 정보들이 JSON으로 변환되어 프론트엔드 단으로 전달된다. ResponseEntity는 JSON 또는 xml 형식으로 변환이 가능한데, default는 JSON이다.

```java
  @PostMapping( value="/addPost")
  public ResponseEntity<BlogPostVo> addPost(HttpServletRequest requets, HttpServletResponse response, @RequestBody BlogPostVo vo) throws IOException {
    log.debug(vo.getContent());
    vo.setUserId("latte");
    vo = blogPostService.addPost(vo);
    return  new ResponseEntity<BlogPostVo>(vo, HttpStatusCode.OK);
  }//:
```
