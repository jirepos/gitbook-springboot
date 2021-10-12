# Filter

Filter를 적용하는 방법에 대해 알아보자. Spring Boot에서 servlet에서 제공하는 Filter interface를 구현하여 사용할 수 있다. Filter를 작성할 때에는 필터체인에서 실행될 순서를 지정하기 위해 @Order 어노테이션을 붙인다. 지정할 값은 -105보다 큰 값을 지정해야 한다. 왜냐면 스프링이 사용하는 RequestFilter의 Order 값은 -105이다. 우리는 그냥 1부터 지정한다.

```java
package com.sogood.core.web.filter;

import java.io.IOException;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.core.annotation.Order;

@Order(1)
public class AllRequestFilter implements Filter {

  private Logger logger = LoggerFactory.getLogger(this.getClass());
  
  @Override
  public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    HttpServletRequest req = (HttpServletRequest) request;
    //HttpServletResponse res = (HttpServletResponse) response;
    logger.debug(">>>>> doFilter:" + req.getRequestURI());
    chain.doFilter(request, response);
  }
}
```

이제 스프링에 Filter를 등록해야 한다. FilterRegistrationBean을 이용하여 등록한다. addUrlPatterns() 메서드를 이용하여 필터가 처리할 URL Pattern을 설정할 수 있다.

```java
@Bean
public FilterRegistrationBean<AllRequestFilter> loggingFilter(){
  FilterRegistrationBean<AllRequestFilter> registrationBean = new lterRegistrationBean<>();
  registrationBean.setFilter(new AllRequestFilter());
  registrationBean.addUrlPatterns("/*");
  return registrationBean;    
}
```

> 주의: 필터를 정의할 때 @Component 어노테이션을 사용하지 말아야 한다.

아직 구체적으로 어떤 필터를 지정해야할지 정하지 않았다. 추후 보완할 예정이다.
