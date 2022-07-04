# Interceptor 적용



@Component를 사용하여 Interceptor를 등록하는 방법은 확장성이 좀 떨어지는 부분이 있어서 interceptor를 조금 변형해서 등록하는 방법으로 적용한다. 

## Interceptor 클래스 정의

HandlerInterceptor를 구현한다. 

```jsx
package com.sogood.core.web.interceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.HandlerInterceptor;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class HttpServletHolderInterceptor implements HandlerInterceptor  {
  @Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		return true; 
	}//:
}
```

# interceptor-setting.json 정의

src/main/resources 아래에 interceptor-setting.json 파일을 다음과 같이 정의한다.  includes에 포함할 패턴을 정의하고 excludes에 인터셉터가 작동하지 않을 패턴을 정의한다. 

```jsx
[
  {
    "name": "com.sogood.core.web.interceptor.HttpServletHolderInterceptor",
    "description": "",
    "includes": [ "/**"],
    "excludes": []
  }
]
```

 

## Interceptor 등록

WebMvcConfigurer를 구현한 DispatcherConfig에 인터셉터를 등록하기 위해서 addInterceptors()를 다음과 같이 구현한다. 

```jsx
/** src/main/resource 아래의 interceptor-setting.json을 읽어서 interceptor를 등록한다.  */
  @Override
  public void addInterceptors(org.springframework.web.servlet.config.annotation.InterceptorRegistry registry) {
    
    String pathAndFile = "interceptor-setting.json";  // 기본 인터셉터 설정 
    String jsonString = IoUtils.readFileClasspathToString(pathAndFile, "utf-8");

    //String jsonString = new IoUtils().readFileClasspathToString2(pathAndFile, "utf-8");
    //ResourceBundleMessageSource source = new ResourceBundleMessageSource();
    List<InterceptorSetting> interceptorList =JsonUtils.toList(jsonString, InterceptorSetting.class);
    if(interceptorList.size() == 0) return; 

    ClassLoader classLoader = this.getClass().getClassLoader();
    for(InterceptorSetting setting: interceptorList) {
      try {
        Class<?> aClass = classLoader.loadClass(setting.getName().trim()); // class name  
        // Creates a interceptor
        AutowireCapableBeanFactory factory = this.applicationContext.getAutowireCapableBeanFactory();
        BeanDefinitionRegistry beanRegistry = (BeanDefinitionRegistry) factory;
        GenericBeanDefinition beanDefinition = new GenericBeanDefinition();
        beanDefinition.setBeanClass(aClass);
        beanDefinition.setAutowireCandidate(true);
        beanRegistry.registerBeanDefinition(aClass.getSimpleName(), beanDefinition);
        factory.autowireBeanProperties(this, AutowireCapableBeanFactory.AUTOWIRE_BY_TYPE, false);
        // InterceptorRegistration regis = registry
        // .addInterceptor((HandlerInterceptor) aClass.newInstance());
        InterceptorRegistration regis = registry.addInterceptor((HandlerInterceptor) this.applicationContext.getBean(aClass.getSimpleName()));
        if(setting.getIncludes() != null && setting.getIncludes().length > 0) {
          regis.addPathPatterns(setting.getIncludes());
        }
        if(setting.getExcludes() != null && setting.getExcludes().length > 0) {
          regis.excludePathPatterns(setting.getExcludes());
        }
        // if(logger != null) {
        //   // @WebMvcTest로 테스트할 때에는 logger가 null임
        //   logger.info("📝 " + setting.getName() + " has been registered.");
        // }
      }catch (Exception e) {
        throw new RuntimeException(e);
      }

    }// for
  }//:
```

### InterceptorSetting.java 구현

인터셉터를 정의한 JSON 파일을 Java Bean으로 변경하기 위한 InterceptorSetting.java를 다음과 같이 구현한다. 

```jsx
package com.sogood.core.web.interceptor;

/** 인터셉터 설정 */
public class InterceptorSetting {
  /** interceptor 이름 */
  private String name;
  /** 인터셉터 설명 */
  private String description;
  /** 인터셉터가 처리할 패턴 */
  private String[] includes;
  /** 인터셉터가 처리하지 않을 패턴 */
  private String[] excludes;

  
  public String getName() {
    return name;
  }
  public void setName(String name) {
    this.name = name;
  }
  public String getDescription() {
    return description;
  }
  public void setDescription(String description) {
    this.description = description;
  }
  public String[] getIncludes() {
    return includes;
  }
  public void setIncludes(String[] includes) {
    this.includes = includes;
  }
  public String[] getExcludes() {
    return excludes;
  }
  public void setExcludes(String[] excludes) {
    this.excludes = excludes;
  } 
  
  
}
```