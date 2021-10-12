# JWT

## JSON Web Token

쿠키-세션 기반의 인증 아키텍처가 현재의 모바일 우선의 웹앱환경에서 요구사항을 만족시키지 못해 나온 기술이다.

* JSON 포맷을 이용한 Web Token
* Claim based Token
* 두 개체에서 JSON 객체를 이용해 Self-contained 방식으로 정보를 안전한게 전달
* 회원 인증, 정보 전달에 주로 사용
* RFC 7519

### 의존성 추가

pom.xml 파일에 다음을 추가한다.

```xml
<dependency>
  <groupId>io.jsonwebtoken</groupId>
  <artifactId>jjwt-impl</artifactId>
  <version>0.10.7</version>
</dependency>
<dependency>
  <groupId>io.jsonwebtoken</groupId>
  <artifactId>jjwt-jackson</artifactId>
  <version>0.10.7</version>
</dependency>
```

### JWT 토큰 구성

세 파트로 나누어진다.

* Header
* Payload
* Signature

#### Header

JWT 웹 토큰의 헤더 정보로 토큰의 타입과 해시 암호화 알로리즘으로 구성되어 있다.

#### Payload

실제 토큰으로 사용하려는 데이터가 담기는 부분. 각 데이터를 Claim이라고 하며 다음과 같이 3가지 종류가 있다.

#### Signature

Header와 Payload의 데이터 무결성과 변조 방지를 위한 서명으로 Header + Payload 를 합친 후, Secret 키와 함께 Header의 해싱 알고리즘으로 인코딩한다.

Key를 생성하고 생성된 키로 Token을 생성하고 생성된 Token을 읽는다. 생성된 Key는 보관해야 하고 보관된 키는 노출되면 안된다.

### Key 생성

토큰을 생성할 때 사용할 키를 생성해야 한다. 키를 생성하고 Base64로 변환한다. 이 키는 다시 토큰을 인증할 때 사용하기 때문에 서버에 보관해야 한다.

```java
// JWS 쓰기
// 알고리즘 선택
SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256;
// 키 생성
Key key = Keys.secretKeyFor(signatureAlgorithm);
// 키 바이트로 변환
byte[] encodedKey = key.getEncoded();
// base64로 변환
    String encodedString = Base64.getEncoder().encodeToString(encodedKey);
```

### JWT 생성

Base64로 변환한된 키를 보관하고 있다가 Token을 생성할 때 사용해야한다. 키를 다시 byte 배열로 변환하여 hmacShaKeyfor()를 이용하여 Key 객체로 변환한다.

```java
// base64를 byte[]로 변환
byte[] decodedByte = Base64.getDecoder().decode(encodedString.getBytes());
// byte[]로 Key 생성
Key key2 = Keys.hmacShaKeyFor(decodedByte);
```

위에서 생성한 암호화 키를 사용하여 JWS를 생성한다. JWS는 JSWON Web Signatrue의 약자로 Private Key로 서명한 Token이라고 보면 된다.

```java
 String jws = Jwts.builder().setIssuer("me").setSubject("Bob").setAudience("you").claim("user", "Andy")
                .claim("password", "123456").setExpiration(expDate) // a java.util.Date
                .setId(UUID.randomUUID().toString())// just an example id
                .signWith(key2).compact();
        System.out.println(jws);
```

생성된 JWS는 다음과 같이 보인다.

```
eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJtZSIsInN1YiI6IkJvYiIsImF1ZCI6InlvdSIsInVzZXIiOiJBbmR5IiwicGFzc3dvcmQiOiIxMjM0NTYiLCJleHAiOjE1OTY1MTI4MzcsImp0aSI6IjQwNzgxN2RiLTdjMTAtNDQ4Yi1iMGI3LTFhNzc1NjdjNzE1ZCJ9.8VuE0nDf921SYLdr1zHYfED8tpJT9IwFFkvF-57WQLo
```

### JWS 읽기

생성된 토큰을 읽으려면 역시 이전에 생성g한 키를 사용한다. subject, issue, audience, id, expiration은 getBody() 메소드를 사용하여 바로 읽을 수 있다.

```java
        // JWS 읽기
        Jws<Claims> jws2 = Jwts.parser().setSigningKey(key2).parseClaimsJws(jws);
        jws2.getBody().getSubject();
        jws2.getBody().getIssuer();
        jws2.getBody().getAudience();
        jws2.getBody().getId();
        jws2.getBody().getExpiration();
```

Payload(JWT Claim Set)을 읽으려면 jws2.getBody().get(key)를 사용한다. key는 JWS에 클레임을 설정하면서 사용한 키를 말한다.

```java
        // key를 사용하여 claim 값을 구함
        String user = (String) jws2.getBody().get("user");
        System.out.println(user);
        String password = (String) jws2.getBody().get("password");
        System.out.println(password);
```

### JWT 전체 테스트 코드

```java
  /** Token 쓰기, 읽기 테스트 케이스이다. */
    @Test
    public void testToken() throws Exception {
        // JWS 쓰기
        // 알고리즘 선택
        SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256;
        // 키 생성
        Key key = Keys.secretKeyFor(signatureAlgorithm);
        // 키 바이트로 변환
        byte[] encodedKey = key.getEncoded();
        // base64로 변환
        String encodedString = Base64.getEncoder().encodeToString(encodedKey);
        // base64를 byte[]로 변환
        byte[] decodedByte = Base64.getDecoder().decode(encodedString.getBytes());
        // byte[]로 Key 생성
        Key key2 = Keys.hmacShaKeyFor(decodedByte);

        // 현재시간
        Date now = new Date();
        // 현재 날짜 시간의 timestamp
        long nowmills = now.getTime();
        // 유효기간 : 하루
        long ttlMillis = 60 * 60 * 24;
        // 만료 날짜,시간 설정을 위한 timestamp
        long expiration = nowmills + ttlMillis;
        // 만료 날짜,시간 Date 생성
        Date expDate = new Date(expiration);
        // issuer 발행자 이름
        // audience 구독자 이름 
        // expiration 만료기간 millisceond 사용 ex) 하루  60 * 60 * 24 * 1000
        // claim 토큰에 담을 정보, 값은 암호화 해서 넣어야 한다. 
        // jws생성
        String jws = Jwts.builder().setIssuer("me").setSubject("Bob").setAudience("you").claim("user", "Andy")
                .claim("password", "123456").setExpiration(expDate) // a java.util.Date
                .setId(UUID.randomUUID().toString())// just an example id
                .signWith(key2).compact();
        System.out.println(jws);

        // JWS 읽기
        Jws<Claims> jws2 = Jwts.parser().setSigningKey(key2).parseClaimsJws(jws);
        jws2.getBody().getSubject();
        jws2.getBody().getIssuer();
        jws2.getBody().getAudience();
        jws2.getBody().getId();
        jws2.getBody().getExpiration();
        // key를 사용하여 claim 값을 구함
        String user = (String) jws2.getBody().get("user");
        System.out.println(user);
        String password = (String) jws2.getBody().get("password");
        System.out.println(password);
        // 만료기간 구함
        Date rexpdate = jws2.getBody().getExpiration();
        System.out.println("Expiration:" + rexpdate);

        // 현재시간
        Date now2 = new Date();
        long now2Mills = now2.getTime();
        long rexpMills = rexpdate.getTime();
        // 토큰이 만료되지 않았는지 체크
        if (rexpMills >= now2Mills) {
            System.out.println("유효한 토큰");
        } else {
            System.out.println("유효하지 않은 토큰");
        }
    }// :
```

## JSONTokenUtils.java

프로젝트에서 JWT를 사용하기 위해서 JWT를 쉽게 사용할 수 있는 JSONTokenUtils.java를 생성했다. 사용법을 알아보자.

### Key 생성

createKey() 메소드를 이용해서 Token을 생성할 키를 생성한다.

```java
String tokenKey = JSONTokenUtils.createKey();
System.out.println(tokenKey);
```

### Token 생성

Token을 생성할 때에는 Token의 주제, 발생자, 청중 및 토큰의 유효기간 등이 필요하다. 그리고 Token에 담을 실제 정보가 필요하다. 토큰에 담을 정보는 객체의 값을 JSON 문자열로 변환하여 보관했다가 나중에 그 값을 다시 객체로 변환하여 사용할 수도 있다. 토큰에 담을 값을 클레임(claim)이라고 부르는데 이것은 Map의로 작성한다.

```java
 String subject = "test";
 String issuer = "latteonterrace@gmail.com";
 String audience = "all";
 long expiration = 60 * 60 *  1000; // 하루 
 Map<String, String> claims = new HashMap<String, String>();
 String userJson = "{ \"userId\": \"latteonterrace@gmail.com\" }";
 claims.put("useIinfo", userJson);
```

이제 createJws() 메소드를 사용하여 Token을 생성한다.

```java
String jws = JSONTokenUtils.createJws(encodedKey, subject, issuer, audience, expiration, claims);
System.out.println(jws);
```

### Token 읽기

JWS를 읽으려면 JWS를 생성할 때 사용했던 키가 필요하다. 토큰을 읽으면 Jws\<Claims>가 봔환된다.

```java
// Token 읽기
Jws<Claims> jwstoken = JSONTokenUtils.readToken(encodedKey, jws);
```

#### token 주제 읽기

getSubject() 메소드를 이용하여 token의 주제(subject)를 읽는다.

```java
String returnedSubject = JSONTokenUtils.getSubject(jwstoken);
System.out.println(returnedSubject);
```

#### Audience 읽기

getAudience() 메소들 이용하여 token의 청중(Audience)를 읽는다.

```java
String returnedAudience = JSONTokenUtils.getAudience(jwstoken);
System.out.println(returnedAudience);
```

#### ID 읽기

토큰을 생성할 때 ID를 부여하는데 ID를 읽는다.

```java
String returnedId = JSONTokenUtils.getId(jwstoken);
System.out.println(returnedId);
```

#### Issuer 읽기

getIssuer() 메소드를 사용하여 발행자를 읽을 수 있다.

```java
String returnedIssuer = JSONTokenUtils.getIssuer(jwstoken);
System.out.println(returnedIssuer);
```

#### 만료 일시 읽기

getExpiration() 메소드를 이용하여 토큰의 만료 일시를 읽을 수 있다.

```java
Date date = JSONTokenUtils.getExpiration(jwstoken); 
System.out.println(date.toString());
```

#### claim 읽기

claim을 생성할 때 사용한 키를 이용하여 클레임을 읽을 수 있다.

```java
String returnedClaim = JSONTokenUtils.getClaim(jwstoken, "userInfo");
System.out.println(returnedClaim);
```

### 만료 여부 체크

Token이 만료되었는지를 체크하려면 몇줄의 코드를 작성해야 한다. 간단히 체크할 수 있도록 isExpired() 메소드를 제공한다.

```java
// 만료 여부 체크 
if(!JSONTokenUtils.isExpired(date)) {
  System.out.println("유효한 토큰.");
}else { 
  System.out.println("만료된 토큰");
}
```

## JSONWebToken.java

JSONTokenUtils.java를 직접 사용할 수도 있지만 더 간단하게 사용하려면 JSONWebToken.java를 사용한다.

```java
package com.sogood.core.token;

import java.util.Date;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jws;


public class JSONWebToken {
  
  private Jws<Claims> claimsJws;
  
  /**
   * 생성자
   * @param encodedKey JSON 생성 키
   * @param jws JWS 
   */
  public JSONWebToken(String encodedKey, String jws) {
    this.claimsJws = JSONTokenUtils.readToken(encodedKey, jws);
  }

  /** Issuer를 구한다.  */
  public String getIssuer(){ 
   return JSONTokenUtils.getIssuer(claimsJws); 
  }
  /** Audience를 구한다. */
  public String getAudience() {
    return JSONTokenUtils.getAudience(claimsJws);
  }
  /** Token 아이디를 구한다.  */
  public String getId() {
    return JSONTokenUtils.getId(claimsJws);
  }
  /** Subject를 구한다.  */
  public String getSubject() {
    return JSONTokenUtils.getSubject(claimsJws);
  }
  /** 만기일을 구한다.  */
  public Date getExpiration(){ 
    return JSONTokenUtils.getExpiration(claimsJws);
  }
  /** 토큰이 유효한지 체크한다.  */
  public boolean isValid() {
    return !JSONTokenUtils.isExpired(this.getExpiration());
  }
  /**
   * 클레임을 구한다. 
   * @param key Cliaim의 키 
   * @return
   *  클레임
   */
  public String getCliam(String key) {
    return JSONTokenUtils.getClaim(claimsJws, key);
  }
}///~
```
