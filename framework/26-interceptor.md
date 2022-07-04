# Interceptor 사용 방법

src/main/resources에 interceptor-mapping.json 파일을 생성한다. 다음과 같이 등록하려는 interceptor들을 등록한다.

```json
[
     {
       "name":"com.latteonterrace.framework.web.interceptor.ThreadLocalInterceptor",
       "description": "",
       "addPathPattern": [ "/**" ],
       "excludePathPattern": []
     },
     {
       "name":"com.latteonterrace.framework.web.interceptor.TestInterceptor",
       "description": "",
       "addPathPattern": [ "/**" ],
       "excludePathPattern": []
     }
]   
```

설정파일의 속성은 다음과 같다.

**name**\
인터셉터 클래스. 전체 패키지 경로를 입력한다.

**description**\
설명을 작성한다.

**addPathPattern**\
인터셉터에서 처리할 패턴을 등록한다. 배열로 등록한다.

**excludePathPattern**\
인터셉터에서 처리하지 않을 패턴을 등록한다. 배열로 등록한다.

## Interceptor 클래스 작성

HandlerInterceptor 인터페이스를 상속하여 인터셉터를 구현한다. 인터셉터 작성방법은 여기에서 설명하지 않는다.

```java
package com.latteonterrace.framework.web.interceptor;

import java.util.Locale;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.LocaleResolver;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.support.RequestContextUtils;

import com.latteonterrace.framework.base.BaseBean;

//@InterceptorAnnotation(addPathPattern = { "/**" }, excludePathPattern = {})
public class TestInterceptor extends BaseBean implements HandlerInterceptor  {
	
	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)	throws Exception {
		LocaleResolver localeResolver = RequestContextUtils.getLocaleResolver(request);
		Locale locale =  RequestContextUtils.getLocale(request);
		System.out.println("Language="  +locale.getLanguage());
		return true;
	}//:~

	@Override
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			ModelAndView modelAndView) throws Exception {
	}

	@Override
	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
			throws Exception {
	}
	
	
}////~
```
