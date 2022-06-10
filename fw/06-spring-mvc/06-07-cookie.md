# Cookie 처리 

서버에서 쿠키를 사용하려면 CookieUtils을 사용한다. 

## Cookie  추가

addCookie를 사용한다. 

```java
@GetMapping("/add-cookie")
  public ResponseEntity<String> addCookie(HttpServletRequest request, HttpServletResponse response) {

    // Cookie를 추가한다. 브라우저에서 확인할 수 있다.
    CookieUtils.addCookie(response, "testCookie", "hello");

    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.TEXT_PLAIN);

    return new ResponseEntity<String>("success", headers,  HttpStatus.OK);
  }//:
```

## Cookie 값 읽기

getCookieValue() 를 사용한다. 

```java
@GetMapping("/get-cookie-value")
  public ResponseEntity<String> getCookieValue(HttpServletRequest request, HttpServletResponse response) {
    // Cookie를 추가한다. 브라우저에서 확인할 수 있다.
    String cookieVal = CookieUtils.getCookieValue(request, "testCookie");
    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.TEXT_PLAIN);  // MediaType을 설정해야 한다.

    return new ResponseEntity<String>(cookieVal, headers,  HttpStatus.OK);
  }//:
```

## 모든 쿠키값 읽기

```java
@GetMapping("/read-all-cookie")
  public ResponseEntity<String> readAllCookie(HttpServletRequest request, HttpServletResponse response) {
    // 모든 쿠키 값을 읽는다
    Cookie[] cookies = CookieUtils.readAllCookie(request);
    if(cookies != null) {
      for(Cookie cki : cookies) {
        System.out.println(cki.getName() );
      }
    }

    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.TEXT_PLAIN);  // MediaType을 설정해야 한다.

    return new ResponseEntity<String>("success", headers,  HttpStatus.OK);
  }//:
```

## 쿠키 삭제

```java
/** 쿠키 삭제 */
  @GetMapping("/delete-cookie/{cookieName}")
  public ResponseEntity<String> deleteCookie(HttpServletRequest request, HttpServletResponse response, @PathVariable String cookieName) {
    // 모든 쿠키 값을 읽는다
    CookieUtils.deleteCookie(response, cookieName);
    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.TEXT_PLAIN);  // MediaType을 설정해야 한다.
    return new ResponseEntity<String>("success", headers,  HttpStatus.OK);
  }//:
```