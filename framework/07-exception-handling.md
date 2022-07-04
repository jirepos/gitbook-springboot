# 예외 처리 

서버에 AJAX 요청을 하는 경우에는 RestController를 사용하여 데이터를 응답으로 반환한다.  서버에 요청하는 API는 브라우저 내장 객체인 fetch() 함수를 사용한다.  서버는 정상적인 응답일 때 HttpStatus 코드 200을 반환하고 오류인 경우에는 표준 Status Code를 사용하고 별도의 에러 객체를 JSON으로 변환하여 반환한다. 

예를 들어 Bad Request 오류인 경우는 다음과 같이 서버가 응답한다. 

**※ 메시지 데이터 스키마는 예를 든 것이며 정의가 필요하다.** 

```jsx
HTTP/1.1 400 Bad Request
Content-Type: application/json;charset=utf-8
Date: Tue, 02 Jul 2013 01:48:19 GMT
 …
 
{
     "status" :  400,
     "error":  "요청이 잘못되었습니다.",
     "code": "ERR-1200",
     "message": "입력 값이 비었습니다."
}
```

## 서버에서 에러 처리

AJAX 호출은 RestController가 처리하도록 개발하고 공통 예외처리는 [@RestControllerAdvice 어노테이션을 사용하여 처리한다.  에러 처리 핸들러 메서드는 ResponseEntity를 사용한다.](https://www.notion.so/RestControllerAdvice-ResponseEntity-3254a0bab0394743a1b30425818c192b) 

- RestControllerAdvice를 사용
- CustomException 공통 클래스를 정의하여 에러처리에 사용

```jsx
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler({CustomException.class  })
    protected ResponseEntity<ErrorResponse>  handleCustomException(CustomException e) {
        return 
    }//:
}
```

### 어떤 에러를 처리할 것인가.

서버에 요청했는데 서버에서 오류가 발생하면 HttpStatus 값을 숫자로 반환해야 한다. 

```jsx
HTTP/1.1 404 404
Date: Wed, 03 Nov 2021 06:49:08 GMT
Server: Apache
```

StatusCode는 표준 값을 사용한다.  

**400 Bad Request** 

- 클라이언트가 잘못된 요청을 하는 경우의 처리 시스템 오류를 제외하고 대부분 사용자에게 정보를 전달하여 처리를 하도록 가이드 하는 경우 이 상태 값을 사용한다.

**401 Unauthorized** 

- 사용자가 로그인을 하지 않는 경우에 사용한다.

**403 Forbidden**

- 로그인은 되었으나 권한이 없는 자원에 접근 시 사용한다.

**404 Not Found**

- 요청한 자원이나 서비스가 없는 경우에 사용한다.

**404  Method Not Allowed**

- POST, GET  메소드를 잘못 호출하는 경우
- 개발자에게 필요한 정보이므로 알려주는 것이 좋지만 보안상 위험하다.

**406 Not Accepted** 

- 서버에서 요청을 허용하지 않는 경우 사용한다.
- 컨트롤러 메서드 정의가 잘못된 경우에 발생한다.
- 개발자에게 필요한 정보이므로 알려주는 것이 좋지만 보안상 위험하다.

**409 Conflict**

- 값이 중복되거나 이미 있는 파일보다 오래된 파일을 업로드할 때 발생한다.

**500 Internal Server Error**

- SQL 오류나 NPE 같은 Java 로직 오류가 발생 시 사용한다.

**502 Bad Gateway** 

- 서버 쪽 네트워크 문제인 경우이므로 사용자에게 적절한 안내 문구를 표시한다.

### 에러 코드 관리

에러 코드는 Enum을 사용하여 관리한다.  에러 코드는 ErrorCodeEnumType 인터페이스를 구현한다. 
```java
package com.sogood.core.types;

import org.springframework.http.HttpStatus;

/** 모든 Error Code를 처리할 Enum 클래스에서 구현할 인터페이스이다. */
public interface ErrorCodeEnumType {
    /**
     * HttpStatus를 반환한다.
     * @return HttpStatus
     */
    HttpStatus status();

    /**
     * 에러 메시지를 반환한다.
     * @return 상세 메시지
     */
    String detailMessage();

    /**
     * Enum의 상수명을 반환한다.
     * @return
     *      상수명
     */
    String errorName();
}//:

```

- framework에서 사용하는 에러 코드와  각 모듈에서 사용하는 에러 코드를 나누어 관리한다.

```jsx
package com.sogood.core.error;

import com.sogood.core.types.ErrorCodeEnumType;
import lombok.AllArgsConstructor;
import lombok.Getter;
import org.springframework.http.HttpStatus;

/** 전역에서 사용할 에러 코드 */
@Getter
@AllArgsConstructor
public enum ErrorCode implements ErrorCodeEnumType {
    //  400 BAD REQUSET
    /** 정의되지 않은 유형의 오류 */
    OTHER_ERROR(HttpStatus.BAD_REQUEST, "정의되지 않는 유형의 오류입니다."),
    USER_NOT_FOUND(HttpStatus.BAD_REQUEST, "존재하지 않는 사용자입니다."),

    // 401 Unauthorized
    /** 로그인  하지 않은 사용자 */
    UNAUTHORIZED_USER(HttpStatus.UNAUTHORIZED, "로그인 하지 않은 사용자입니다. 로그인을 먼저 해야 합니다."),

    // 500 Internal Server Error
    INTERNAL_SERVER_ERROR(HttpStatus.INTERNAL_SERVER_ERROR, "시스템 오류입니다.");

    private final HttpStatus httpStatus;
    private final String detail;

    public HttpStatus status() {
        return httpStatus;
    }
    public String detailMessage() {
        return detail;
    }
    public String errorName() {
        return this.toString();
    }
}///~
```

### 에러 던지기

Service 클래스에서 다음과 같이 에러를 throw 한다. 

```jsx
public User login(User user) {
  ..... 생략 
	if(!user.getPassword().equals(vo.getPassword())) {
	   throw new CustomException(ErrorCode.UNAUTHORIZED_USER);
	}
}
```

OTHER_ERROR를 던졌을 때의 응답 

OTHER_ENUM은 다음과 같이 정의되어 있다. 

```jsx
OTHER_ERROR(HttpStatus.BAD_REQUEST, "정의되지 않는 유형의 오류입니다."),
```

```jsx
HTTP/1.1 400
Vary: Origin
Vary: Access-Control-Request-Method
Vary: Access-Control-Request-Headers
Content-Type: application/json
Transfer-Encoding: chunked
Date: Wed, 03 Nov 2021 10:06:05 GMT
Connection: close

{
	"timestamp": [
		2021,
		11,
		3,
		19,
		6,
		5,
		696143000
	],
	"status": 400,
	"error": "BAD_REQUEST",
	"code": "OTHER_ERROR",
	"message": "정의되지 않는 유형의 오류입니다."
}
```

## 클라이언트에서 에러 처리

브라우저 내장 객체인 fetch() 함수를 사용하여  API를 호출한다. 

### fetch() 기본 사용방법

```jsx
fetch(url, { /* options */})
.then(response => {
    // 서버 응답이 200인경우 
    
    // response.status >= 200 && response.status <= 299
    if(response.ok) {
        // 서버가 json을 되돌리는 경우 
        return res.json(); 
    }else {
       // 서버에서 오류코드를 받은 경우 여기서 오류를 던져야 함 
       throw new Error("Error");
    }
})
.then(response => { 
     // res.json() 이후에 이 부분이 실행된다.
}) 
.catch(error => {
     // 서버가 다운된 경우에 이곳을 탄다 
     // 클라이언트에서 발생한 오류인 경우, TypeError인 경우 
     if(error instanceof TypeError) {
        // do something
     }
     // throw new Error로 던진 오류 처리 
     // Error는 사용자 Error를 만들어 던져야 한다. 
})
```

[https://bcp0109.tistory.com/303](https://bcp0109.tistory.com/303)