# HttpServletRequest 사용 

Spring에서 파라미터로 request, reponse를 전달하지 않고 ReqeustContextHolder를 통해서 구할 수 있다. 

```jsx
public String requestContextHolderTest() {
        // Request 구하는 방법
        ServletRequestAttributes attr = (ServletRequestAttributes) RequestContextHolder.currentRequestAttributes();
        HttpServletRequest request = (HttpServletRequest)attr.getRequest();
        log.debug(request.getRequestURI());
        // Response 구하는 방법 
        HttpServletResponse response =  attr.getResponse();
        response.setHeader("X-Addtion-Header", "Hello");

        return "Success";
    }//:
```