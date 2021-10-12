# Cookie

서버에서 Cookie를 다룰 때에는 CookieUtils 클래스를 사용한다.

## CookieUtils.java

### 모든 쿠키 가져오기

모든 쿠키를 가져오려면 readAllCookie() 메서드를 사용한다.

```java
public static Cookie[] readAllCookie(HttpServletRequest request) {
	Cookie[] cookies = request.getCookies();
	return cookies;
}//:
```

### 특정 쿠키의 값 구하기

특정 쿠키의 값을 구하려면 getCookieValue() 메서드를 사용한다.

```java
public static String getCookieValue(HttpServletRequest request, String cookieName) {
		
	Cookie[] cookies = readAllCookie(request);
		
	if(cookies != null) {
		for(Cookie c : cookies) {
			if(c.getName().equals(cookieName) ) {
				return c.getValue();
			}
		}
	}
	return null; 
}//:
```

### 쿠키 추가

서버에서 쿠키를 추가하려면 addCookie() 메서드를 이용한다.

```java
public static void addCookie(HttpServletResponse response, String cookieName, String cookieValue) {
	Cookie cookie = new Cookie(cookieName, cookieValue);
	cookie.setPath("/");
	response.addCookie(cookie);
}//:
```

쿠키의 유효기간을 지정하려면 다음 메서드를 사용한다.

```java
public static void addCookie(HttpServletResponse response, String cookieName, String cookieValue, int expiry) {
	Cookie cookie = new Cookie(cookieName, cookieValue);
		
	// 범위를 지정하지 않으면 브라우저에서 쿠키를 설정하는 데 사용 된 경로에 대해서만 쿠키가 서버로 전송됩니다.
	cookie.setPath("/");
	cookie.setMaxAge(expiry);
	response.addCookie(cookie);
}//:
```

### 쿠키 삭제

쿠키를 삭제하려면 deleteCookie() 메서드를 사용한다.

```java
public static void deleteCookie(HttpServletResponse response, String cookieName) {
	Cookie cookie = new Cookie(cookieName, null);
	cookie.setMaxAge(0);
	cookie.setPath("/");
	response.addCookie(cookie);
}//:
```
