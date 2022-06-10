# DispatcherServlet

Spring MVC를 사용하면 클라이언트에서 들어오는 모든 요청을 DispatcherServlet이 처리한다.

1. HandlerMapping 객체에게 컨트롤러 객체를 요청한다.
2. HandlerMapping은 요청 경로를 이용해서 이를 처리할 컨트롤러를 찾아서 DispatcherServlet에게 리턴한다.
3. DispatcherServlet은 HandlerAdapter에게 처리를 위임한다.
4. HandlerAdapter는 컨트롤러의 알맞은 메소드를 호출해서 요청을 처리하고 반환받은 결과를 ModelAndView에 담아서 DispatcherServlet에게 반환한다.
5. DispatcherServlet는 ViewResolver를 이용해서 결과를 보여줄 뷰를 검색한다. ViewResolver 객체는 ModelAndView 객쳉 담긴 뷰 이름을 이용해서 View 객체를 찾거나 생성해서 리턴한다.
6. DispatcherServlet ViewResolver가 반환한 View 객체에게 결과 생성을 요청한다.

```java
void doDispatch() {
  HandlerExecutionChain mappedHandler = getHandler();  // 현재 요청에 맞는 핸들러를 구한다. 
  HandlerAdapter ha = getHandlerAdapter() // 핸들러를 실행할 HandlerAdapter를 구한다. 
  // 적용할 interceptor가 있다면, preHandle 적용
  if (!mappedHandler.applyPreHandle(processedRequest, response)) {
         return;
  }
  // 핸들러(Controller)의 메소드 실행 
  ModelAndView mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
  //View 에 대한 설정이 없다면, RequestToViewNameTranslator 전략 빈에 의해 view name을 설정한다.
  applyDefaultViewName(processedRequest, mv);
  //interceptor postHandle 실행
  mappedHandler.applyPostHandle(processedRequest, response, mv);
  //핸들러 실행 결과 ModelAndView를 적절한 response 형태로 처리한다.
   processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
}
```

**HandlerMapping**\
요청이 들어오면 DispatcherServlet이 하는 첫번째 하는 일은 요청을 처리할 Controller를 찾는 일이다. Controller는 @Controller 어노테이션을 붙인 클래스이다. 요청을 처리할 컨트롤러는 HandlerMapping 객체의 getHandler() 메소드를 통해서 구한다.

```java
public interface HandlerMapping{
	HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;
}
```

HandlerMapping은 HTTP 요청 정보를 이용해서 컨트롤러를 찾아주는 역할을 하는데 BeanNameUrlHandlerMapping, ControllerNameHandlerMapping 등 여러가지 HandlerMapping 들이 존재한다.

DefaultAnnotationHandlerMapping 클래스는 @RequestMapping이라는 어노테이션을 이용해 매핑하는 방식이다. Controller와 Controller의 메소드를 작성할 때 @RequestMapping, @GetMapping, @PostMapping 어노테이션을 사용할 수 있다.

들어오는 요청을 HandlerMapping에서 처리할 컨트롤러를 찾지 못할 경우에는 특정한 컨트롤러가 처리할 수 있도록 지정할 수 있다. defaultHandler 속성에 Controller의 참조를 설정한다.

**HandlerAdapter**\
HandlerAdapter는 HandlerMapping을 통해 찾은 컨트롤러를 직접 실행하는 기능을 담당한다. DispatcherServlet는 HandlerAdapter.handle() 메소드를 통해 Controller를 실행한다. HandlerAdapter는 HandlerMapping과 마찬가지로 다양한 HandlerAdapter 들이 있다. handle() 메소드는 ModelAndView를 반환한다.

```java
public interface HandlerAdapter{
	ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;
}
```

**HandlerInterceptor**\
핸들러 매핑은 요청정보를 통해 컨트롤러를 찾아주는 기능 외에 인터셉터를 적용해주는 기능 또한 제공한다. 핸뜰러 인터셉터는 DispatcherServlet이 컨트롤러를 호출하기 전과 후에 요청, 응답을 가공할 수 있다. 핸들러 매핑은 DispatcherServlet으로 부터 매핑 작업을 요청받으면 등록된 핸들러 인터셉터들을 순서대로 수행하고 컨트롤러를 호출한다. 등록된 핸들러 인터셉터가 없다면 컨트롤러를 바로 호출한다.
