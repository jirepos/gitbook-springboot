# 서버에 요청하기


### 응답 유형 설정

클라이언트에서 JSON 타입으로 응답을 받고 싶으면 Accept 헤더의 값을 application.json으로 설정한다. 

```java
headers: [
   'Accept': 'application/json', // 클라이언트가 이해 가능한 컨텐츠 타입이 무엇인지
]
```

서버는 Accept 헤더의 값을 확인하여 Message 객체로 응답한다. 

### 전송 데이터 타입 설정

클라이언트가 서버로 데이터를 전송할 때 어떤 타입의 데이터를 보내는지 Content-Type 헤더에 설정해야 한다. 

```java
headers: {
      'Accept': 'application/json',    // 클라이언트가 이해 가능한 컨텐츠 타입이 무엇인지 
      'Content-Type': 'application/json', // 서버에 어떤 형식의 데이터를 보내는지 알려줌
      // 'Content-Type': 'application/x-www-form-urlencoded',
 },
```

서버에서 응답할 때 타입이 JSON이면 마찬가지로 Content-Type 헤더의 값을 JSON으로 설정해야 한다.  서버는 응답할 데이터의 타입을 Content-Type 헤더에 설정해야 한다.  아래는 설정 가능한 값이다. 바이너리 파일의 경우 application/octet-stream으로 설정한다. 

```java
application/json
text/xml
text/plain
image/png
application/octet-stream
application/x-www-form-urlencoded
```

Ajax 요청인 경우에는 HttpStatus 값으로 Fetch 함수에서 처리하기 때문에 Content-Type을 굳이 설정하지 않아도 된다. 

## 요청 및 응답 객체

요청 및 응답 Payload에 데이터 이외의 Token 값 등을 전송할 필요가 있을 때에는 `com.sogood.core.web.message` 패키지의 Message class를 사용한다. 

```java
package com.sogood.core.web.message;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class Message<T> {
    private String code;
    private String error;
    private String jwtToken;
    private T payload;
}
```

code, error 프러퍼티는 현재는 사용하지 않는다.  예비를 위해서 남겨 두었다. 

> 타 시스템과의 인터페이스를 할 때를 고려한 구조는 아직 아니다. 좀 더 검토 후에 정의할 생각이다.  또한 예외처리에 대한 표준도 검토해야 한다. 구글이나 마이크로소프트의 Open API를 검토할 필요가 있다.
> 

> 마이크로소프트의 Graph API의 오류 처리에 대한 내용: "Graph API 오류에는 HTTP 상태 코드, 메시지 및 값으로 구성된 특정 형식의 본문이 있습니다. 오류 처리 논리에서 오류 본문의 속성을 사용합니다."
> 

```java
HTTP/1.1 400 Bad Request
 Content-Type: application/json;odata=minimalmetadata;charset=utf-8
 request-id: ddca4a7e-02b1-4899-ace1-19860901f2fc
 Date: Tue, 02 Jul 2013 01:48:19 GMT
 …
 
 {
     "odata.error" : {
         "code" : "Request_BadRequest",
         "message" : {
             "lang" : "en",
             "value" : "A value is required for property 'mailNickname' of resource 'Group'."
         },
     "values" : null
    }
 }
```
