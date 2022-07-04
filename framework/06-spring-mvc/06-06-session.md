# Session 처리 


## 세션 처리

세션 처리는 SessionUtils 클래스를 이용한다. 

### 세션 타임아웃

기본적으로 한 시간을 유효기간으로 설정한다.  세션을 생성할 때 timeout을 주고 싶으면 다음과 같이 한다. 

```java
HttpSession session = SessionUtils.getSession(request, timeout);
```

### 세션 무효화

세션을 제거하고 싶으면 invalidate()를 사용한다. 

```java
SessionUtils.invalidate(request);
```

정리할 것 

## 세션에 저장하는 정보

세션에 담는 정보는 다음과 같다. 

- 로그인 사용자 정보

아직 다른 것들은 담고 있지 않다. 

## 로그인 사용자 정보 객체

OrgLoginUserVo 클래스는 OrgUserVo 클래스를 상속 받는다. 

```java
package com.sogood.biz.org.vo;

import java.util.List;

import com.sogood.biz.blog.vo.BlogVo;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class OrgLoginUserVo extends OrgUserVo {
  protected List<BlogVo> blogs;
}
```

```java
package com.sogood.biz.org.vo;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class OrgUserVo {
  /** the user's ID */
  protected String userId; 
  /** Password */
  protected String password; 
  /** The user's name */
  protected String name; 
  /** The user's email */
  protected String email;
}
```

로그인 할 때 사용자 정보를 세션에 담는다. 

```java
public OrgLoginUserVo login(OrgUserVo vo) {
    OrgLoginUserVo loginUser = new OrgLoginUserVo();
    loginUser.setEmail(user.getEmail());
    loginUser.setUserId(user.getUserId());
    loginUser.setName(user.getName());
    loginUser.setBlogs(this.blogService.selectUserBlogs(user.getUserId()));
    SessionUtils.setSignedUser(loginUser);
}
```