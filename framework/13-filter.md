# Filter 적용



## 필터 클래스 정의

Filter 인터페이스를 구현한다.  @Order 어노테이션을 적용하여 순서를 정의한다. 

```jsx
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

import lombok.extern.slf4j.Slf4j;

@Order(1)
@Slf4j
public class AllRequestFilter implements Filter {

  //private Logger logger = LoggerFactory.getLogger(this.getClass());
  
  @Override
  public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    HttpServletRequest req = (HttpServletRequest) request;
    chain.doFilter(request, response);
  }
}
```

## 필터 등록

필터를 적용하려면 클래스를 정의하고 @Configuration 어노테이션을 적용한다.   작성한 필터를 @Bean 어노테이션을 사용하여 등록한다. 

```jsx
package com.sogood.core.web.filter;

import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FilterConfig {
  
  @Bean
  public FilterRegistrationBean<AllRequestFilter> allRequestFilter(){
    FilterRegistrationBean<AllRequestFilter> registrationBean = new FilterRegistrationBean<>();
    registrationBean.setFilter(new AllRequestFilter());
    registrationBean.addUrlPatterns("/*");
    return registrationBean;    
  }
}
```