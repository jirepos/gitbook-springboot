# 에러 코드 관리 


Error 코드는 Enum으로 작성한다. 

## 프레임워크 에러 코드  관리

로그인을 제외하고 비즈니스와 관련 없는 에러 코드는 com.sogood.core.error 패키지의 ErrorCode.[java](http://ErrorCode.java)   클래스에서 관리한다. 

```java
package com.sogood.core.error;

import lombok.AllArgsConstructor;
import lombok.Getter;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.GetMapping;

/** 전역에서 사용할 에러 코드 */
@Getter
@AllArgsConstructor
public enum ErrorCode {
    //  400 BAD REQUSET
    /** 정의되지 않은 유형의 오류 */
    OTHER_ERROR(HttpStatus.BAD_REQUEST, "정의되지 않는 유형의 오류입니다."),
    USER_NOT_FOUND(HttpStatus.BAD_REQUEST, "존재하지 않는 사용자입니다."),

    // 401 Unauthorized
    /** 로그인  하지 않은 사용자 */
    UNAUTHORIZED_USER(HttpStatus.UNAUTHORIZED, "로그인 하지 않은 사용자입니다. 로그인을 먼저 해야 합니다.");

    private final HttpStatus httpStatus;
    private final String detail;
}///~
```

## 비지니스 에러 코드 관리

비지니스 로직을 처리할 때 사용할 에러 코드는 com.sogood.biz.common.error 패키지의 CommonError.java에서 관리한다. 

```java
package com.sogood.biz.common.error;

import lombok.AllArgsConstructor;
import lombok.Getter;
import org.springframework.http.HttpStatus;

@Getter
@AllArgsConstructor
public enum CommonErrorCode {
    // Blog Section
    // 에러코드, 다국어 메시지 키
    BLOG_NOT_FOUND(HttpStatus.BAD_REQUEST, "Blog Not Found"),
    PARAM_INSUFFICIENT(HttpStatus.BAD_REQUEST, "Parameter[s] is/are insufficient."),
    INDEX_ERROR(HttpStatus.BAD_REQUEST, "Lucene Indexing faield.");

    private final HttpStatus httpStatus;
    private final String detail;
}///~
```


