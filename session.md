# 세션처리

로그인한 사용자의 정보를 쿠키에 담는 것은 위험하기 때문에 보통 세션에 저장을 한다. 세션은 사용자 인증에서 매우 중요한 역할을 하기 때문에 각자 세션을 처리하는 것이 아니라 공통화가 필요하다. 세션을 관리하기 위해서 SessionUtils 클래스를 com.sogood.core.web.session 패키지에 생성할 것이다.

## 세션 생성

세션을 생성할 때 세션이 언제까지 유효할지 결정해야 한다.특별한 설정이 없으면 기본 세션 타임아웃을 적용할 필요가 있다. 이 부분은 나중에 설정값으로 바꿀 것이다.

```java
/**
 * 0(zero) 또는 음수인 경우에는 session이 timeout되지 않음 
 */
public static final int MAX_INACTIVE_INTERVAL = ( 60 * 60);
```

HttpServletRequest 객체로 부터 세션을 구하기 위해서 getSession() 메소드를 정의한다. 이 메소드는 디폴트 세션 타임아웃 값을 사용한다. 실제 세션을 생성하는 것은 같은 이름의 overloading된 메소드이다. 바로 아래에 나온다.

```java
public static HttpSession getSession(HttpServletRequest request) {
	return getSession(request, MAX_INACTIVE_INTERVAL);
}//:
```

사용자가 로그인을 하지 않았더라도 최초에 사이트에 접속하면 세션이 생성된다. Requset 객체로 부터 session을 구하는데 없으면 새로 생성하도록 하기 위해서 getSesion(true)로 작성한다. setMaxInactiveInterval() 메소드를 사용하여 세션 만료 시간을 설정할 수 있다.

```java
 public static HttpSession getSession(HttpServletRequest request, int essionTimeout) {
	// true : 세션이 없다면 생성한다. 
	HttpSession session = request.getSession(true);
	session.setMaxInactiveInterval(sessionTimeout);
	return session;
}//:
```

## 세션 아이디 구하기

세션은 생성될 때 아이디가 부여된다. getId()를 통해 세션의 아이드를 구할 수 있다.

```java
public static String getSessionID(HttpServletRequest request) {
	HttpSession sess = getSession(request);
	return sess.getId();
}
```

## 세션 무효화하기

세션을 무효화할 수 있는데 현재 세션에 붙어 있는 사용자만 세션을 무효화할 수 있다.

```java
public static void invalidate(HttpServletRequest request) {
	getSession(request).invalidate();
}//:
```

## 세션에 객체 저장

setAttribute()를 사용하여 세션에 객체를 저장할 수 있다. 메모리 문제가 발생할 수 있으므로 너무 큰 객체를 저장하지 않도록 한다.

```java
public static void setObject(HttpServletRequest request, String key, Object data) 
	HttpSession session = getSession(request);
	synchronized (session) {
		session.setAttribute(key, data);
	}
}//:
```

저장할 때 사용한 키를 이용하여 객체를 꺼내올 수 있다.

```java
public static Object getObject(HttpServletRequest request, String key) {
	Object obj = null;
	HttpSession session = getSession(request);
	synchronized (session) {
		obj = session.getAttribute(key);
	}
	return obj;
}//:
```

## 스레드 객체

ThreadLocal을 사용하면 하나의 스레드에서 공유할 수 있는 객체를 생성할 수 있다. 쓰레드가 끝나면 객체는 사라진다. 클라이언트가 서버에 요청을 하면 하나의 스레드가 생성된다. 사용자에게 무엇인가를 응답할 때까지 Controller에서 시작하여 Service, DAO까지 여러 클래스의 메소드를 호춯한다. 이때 스레드내에서 각 객체 간에 파라미터로 전달하지 않아도 공유할 수 있는 객체를 만들려면 ThreadLocal을 이용한다. 이 프로젝트에서는 이것을 이용하여 HttpServletRequest와 HttpServletResponse 객체를 담을 것이다.

```java
package com.sogood.core.web.session;

import java.util.Map;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class HttpServletHolder {

  	/** ThreadLocal에 저장할 HttpServletRequest의 키 */
	public static final String THREAD_LOCAL_HTTP_REQUEST_KEY = "THREAD_LOCAL_HTTP_REQUEST_KEY";
	/** ThreadLocal에 저장할 HttpServletResponse의 키 */
	public static final String THREAD_LOCAL_HTTP_RESPONSE_KEY = "THREAD_LOCAL_HTTP_RESPONSE_KEY";

  private static ThreadLocal<Map<String, Object>> threadLocal = new ThreadLocal<Map<String, Object>>();
	
	/** Map을 설정한다.*/
	public static void setValue(Map<String, Object> m) {
		threadLocal.set(m);
	}//:
	/** Map을 반환한다.*/
	public static Map<String,Object> getValue() {
		return threadLocal.get();
	}//:
	
	/** HttpServletRequest 를 반환한다. */
	public static HttpServletRequest getServletRequest() {
		Map<String, Object> map = threadLocal.get();
		return (HttpServletRequest)map.get(THREAD_LOCAL_HTTP_REQUEST_KEY);
	}//:
	
	/** HttpServletResponse를 반환한다.*/
	public static HttpServletResponse getServletResponse() {
		Map<String, Object> map = threadLocal.get();
		return (HttpServletResponse)map.get(THREAD_LOCAL_HTTP_RESPONSE_KEY);
	}//: 
}
```

ThreadLocal에 HttpServletRequest와 HttpServletResponse 객체를 설정하는 것은 Interceptor에서 할 것이다. 이 글에서는 설명하지 않는다.

## 사용자 정보 저장

사용자의 정보를 세션에 저장하는 메소드를 만든다. 이제는 HttpServletRequest와 HttpServletResponse 객체의 참조를 파라미터로 넘기지 않아도 된다.

```java
public static void setSignedUser(Object signedUser) {
	HttpServletRequest request = HttpServletHolder.getServletRequest();
	setObject(request, SESSION_SIGNIN_USER_KEY, signedUser );
}//:
```

## 사용자 정보 가져오기

```java
public static Object getSignedUser() {
	HttpServletRequest request = HttpServletHolder.getServletRequest();
	return getObject(request, SESSION_SIGNIN_USER_KEY);
}//:
```

## 로그인 여부 확인

세션에 사용자 정보 객체가 있는지 확인하여 로그인 여부를 판단할 수 있다.

```java
public static boolean isSigned() {
	HttpServletRequest request = HttpServletHolder.getServletRequest();
	Object o = getObject(request, SESSION_SIGNIN_USER_KEY);
	return (o == null) ? false : true; 
}//:
```
