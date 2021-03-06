# 다국어 처리

스프링 제공하는 [spring:message](spring:message) 커스텀 태그는 웹 요청과 관련된 언어 정보를 이용해서 알맞은 언어의 메시지를 출력한다. 웹 브라우저의 언어 설정을 한국어(ko_kr)로 했을 때와 영어(en_us)로 했을 때 [spring:message](spring:message) 커스텀 태그가 언어에 따라 알맞은 메시지를 출력해 주는 결과 화면을 보여준다.

Spring MVC는 LocaleResolver를 이용해서 웹 요청과 관련해서 Locale을 추출하고 이 Locale 객체를 이용해서 알맞은 언어의 메시지를 선택한다.

**LocaleResolver 종류**

| 타입                         | 설명                                                         |
| -------------------------- | ---------------------------------------------------------- |
| AcceptHeaderLocaleResolver | 웹 브라우저가 전송한 Accept-Language 헤더로부터 Locale을 선택               |
| CookieLocaleResolveer      | 쿠키를 이용해서 Locale 정보 구함. setLocale() 메소드는 쿠키에 Locale 정보를 저장함 |
| SessionLocaleResolver      | 세션으로부터 Locale 정보를 구함. setLocale() 메소드는 세션에 Locale 정보를 저장함  |
| FixedLocaleResolver        | 고정 Locale을 사용함. setLocale() 메소드가 없음                        |

**Locale 전략**\
사용자의 PC 환경의 Locale 설정에 따른 Locale을 사용하면 사용자가 다른 언어로 웹페이지를 보고 싶을 때 변경할 수 없다. 사용자가 자신의 PC의 Locale에 관계없이 언어 설정을 변경하려고 할 때 변경할 수 있어야 한다. 그래서 Cookie를 이용한 Locale 설정이 적합하다.

**LocaleResolver 생성** com.sogood.core.config 패키지의 DispatcherConfig.java 파일을 열고 다음을 추가한다.

```java
package com.sogood.core.config;

import java.util.Locale;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import org.springframework.stereotype.Controller;
import org.springframework.stereotype.Repository;
import org.springframework.stereotype.Service;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.servlet.i18n.CookieLocaleResolver;


@EnableWebMvc
@Configuration
public class DispatcherConfig implements WebMvcConfigurer {
  
  @Bean
  public CookieLocaleResolver localeResolver() {
    CookieLocaleResolver localeResolver = new CookieLocaleResolver();
    localeResolver.setDefaultLocale(Locale.KOREA); // 기본 한국어
    localeResolver.setCookieName("locale"); // 쿠키 이름 ; locale
    localeResolver.setCookieMaxAge(60 * 60); // 쿠키 살려둘 시간, -1로 하면 브라우져를 닫을 때 없어짐.
    localeResolver.setCookiePath("/"); // Path를 지정해 주면 해당하는 path와 그 하위 path에서만 참조
    return localeResolver;
  }//:
}///~
```

위와 같이 정의하면 Spring은 다국어 메시지파일을 읽을 때 Cookike의 'locale' 값을 읽어서 어떤 언어의 메시지를 가져올지를 결정한다. 그런데 이 시점에서 SpringBoot를 다시 시작하면 Bean 이름이 중복되었고 overriding이 안된다는 메시지가 출력된다. Spring Boot 2.1부터는 같은 이름으로 Bean을 생성하는 것이 디폴트로 disabled 되었다. 우연히 미리 정의된 bean이 중복되는 것을 막기 위해서 알려주는 것이다. 이것을 해결하려면 이름을 바꾸던가 applicaiton.yml에 spring.main.allow-bean-definition-overriding 속성을 true한다.

확인해 보니 SpringBoot는 디폴트로 AcceptHeaderLocaleResolver를 사용한다.

```yaml
spring:
  main:
    #spring.main.allow-bean-definition-overriding=true    
    allow-bean-definition-overriding: true    
```

CookieLocaleResolver의 쿠키 설정 관련 프로퍼티

| 프로퍼티         | 설 명                   |
| ------------ | --------------------- |
| cookieName   | 사용할 쿠키 이름             |
| cookieDomain | 쿠키 도메인                |
| cookiePath   | 쿠키 경로, 기본값은 "/"이다.    |
| cookieMaxAge | 쿠키 유효 시간              |
| cookieSecure | 보안 쿠키 여부, 기본값은 false. |

## 다국어 메시지 작성

### properties 파일을 이용하는 방법

다국어 메시지는 src/main/resources 소스폴더에 properties 파일로 작성한다. 이 폴더에 com.sogood.i8n 패키지를 생성한다.

```
📁src/main/resources
   📁com
     📁sogood
        📁i18n
```

다국어 메시지 파일의 파일명의 형식은 다음과 같이 작성해야 한다.

```
파일명_[locale].properties
```

local이 없이 파일명을 작성할 수도 있다. 예를들어 다음과 같다.

```
파일명.properties
```

위와 같이 locale이 붙지 않은 파일은 default 메시지 파일이다. 사용자가 선택한 locale이 이 없을 경우 디폴트를 선택한다. 한국어, 일본어, 영어를 지원한다면 다음과 같이 파일들을 생성해야 한다.

```
📁src/main/resources
   📁com
     📁sogood
        📁i18n
          📄 message_ko.properties
          📄 message_en.properties
          📄 message_ja.properties
          📄 message.properties
```

다국어 파일에 hello라는 프러퍼티를 하나 추가한다.

```properties
# message_ko.properties
hello=안녕
```

```properties
# message_en.properties
hello=Hello
```

이제 hello 메시지를 출력해보자. MessageSource를 필드 멤버로 getMessage() 메소드를 사용하여 메시지를 가져올 수 있다. MessageSource의 자세한 사용방법은 설명하지 않는다. 아래 코드에서는 Locale.getDefault()를 사용했지만 실제 코드를 작성할 때에는 RequestContextUtils를 사용하여 LocaleResolver로부터 Locale을 구해야 한다.

```java
// DemoController.java
package com.sogood.demo;
// ... 생략 
import org.springframework.context.MessageSource;

@Controller
public class DemoController {
  // ... 생략
	@Autowired 
	private MessageSource  messageSource; 

	@GetMapping("/demo")
	public String test(HttpServletRequest request) {
		logger.debug(messageSource.getMessage("hello", null, Locale.getDefault()));
		return "demo";
	}
}/// ~
```

### YAML 파일 사용

다국어 메시지는 기본으로 properties 파일을 사용하지만 그룹핑하기 편하고 사람이 읽기 편한 YAML 파일로 메지지 파일을 만드는 것이 좋다.

src/main/resources 소스 폴더 아래의 com.sogood.i18n 패키지에 다음과 같이 yaml 파일을 생성한다.

```
📁src/main/resources
   📁com
     📁sogood
        📁i18n
          📄 message_ko.yml
          📄 message_en.yml
          📄 message_ja.yml
          📄 message.yml
          📄 common_ko.yml
          📄 common_en.yml
          📄 common_ja.yml
          📄 common.yml          
```

언어별로 다르게 다국어 메시지를 만든다.

```yaml
# common_ko.yml
common: 컴온
```

```yaml
# common_en.yml
common: common
```

```yaml
# message_ko.yml
hello: 안녕
```

```yaml
# message_en.yml
hello: Hello
```

pom.xml 파일을 열고 의존성을 추가한다.

```xml
<dependency>
	<groupId>net.rakugakibox.util</groupId>
	<artifactId>yaml-resource-bundle</artifactId>
	<version>1.2</version>
</dependency>
```

com.sogood.core.util 패키지를 생성하고 그 안에 YmlMessageSource.java 파일을 생성한 다음에 다음과 같이 코드를 작성한다.

```java
package com.sogood.core.util;

import java.util.Locale;
import java.util.MissingResourceException;
import java.util.ResourceBundle;

import org.springframework.context.support.ResourceBundleMessageSource;

import net.rakugakibox.util.YamlResourceBundle;

public class YamlMessageSource extends ResourceBundleMessageSource {
  @Override
  protected ResourceBundle doGetBundle(String basename, Locale locale) throws MissingResourceException {
    return ResourceBundle.getBundle(basename, locale, YamlResourceBundle.Control.INSTANCE);
  }
}/// ~
```

com.sogood.core.config 패키지의 AppConfig.java를 열고 messageSource() 메소드를 다음과 같이 변경한다.

```java
package com.sogood.core.config;

import com.sogood.core.util.YamlMessageSource;

import org.springframework.context.MessageSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

  // @Bean
  // public ResourceBundleMessageSource messageSource() {
  // ResourceBundleMessageSource messageSource = new
  // ResourceBundleMessageSource();
  // messageSource.setDefaultEncoding("UTF-8");
  // messageSource.setBasename("com/sogood/i18n/message");
  // return messageSource;
  // }//:

  @Bean
  public MessageSource messageSource() {
    YamlMessageSource messageSource = new YamlMessageSource();
    messageSource.setDefaultEncoding("UTF-8");
    String[] basenames = new String[] {
      "com/sogood/i18n/message",
      "com/sogood/i18n/common"
    };
    messageSource.setBasenames(basenames);
    return messageSource;
  }// :

}/// ~
```

위 코드에서 이전과는 다르게 setBasename() 이 아닌 setBasenames()를 사용했다. 프로젝트의 규모가 큰 경우에 하나의 메시지 파일로 정의하는 것이 어렵기 때문에 려러개의 파일로 분리할 수 있다.

DemoController.java를 열고 다음과 같이 코드를 수정한다.

```java
// DemoController.java
package com.sogood.demo;
// ... 생략 
import org.springframework.context.MessageSource;

@Controller
public class DemoController {
  // ... 생략
	@Autowired 
	private MessageSource  messageSource; 

	@GetMapping("/demo")
	public String test(HttpServletRequest request) {
		logger.debug(messageSource.getMessage("common", null, Locale.getDefault()));
		return "demo";
	}
}/// ~
```

프로젝트가 진행되면 처음 설계할 때와 달리 메시지 파일을 추가할 필요가 있을 것이다. 이런 경우를 대비해서 basenames의 값을 하드코딩하지 않는 방법이 필요하다.

application.yml파일을 열고 다음과 같이 basenames를 추가한다.

```yaml
i18n-message-files: com/sogood/i18n/message, com/sogood/i18n/common
```

위의 방법은 보기 좋지 않으므로 아래와 같이 YAML에서 지원하는 포맷으로 바꾸려고 했지만 오류가 나서 일단 제외했다. 원인파악을 하는 중이다.

```yaml
i18n-messages:
  - com/sogood/i18n/message
  - com/sogood/i18n/common
```

위에서 작성한 messageSource() 메소드를 '@Value'를 사용하여 다음과 같이 수정한다.

```java
  @Bean
  //public MessageSource messageSource(@Value("${spring.msg}") String[] basenames) {
  public MessageSource messageSource( @Value("${i18n-message-files}") List<String> basenames) {
    YamlMessageSource messageSource = new YamlMessageSource();
    messageSource.setDefaultEncoding("UTF-8");
    // String[] basenames = new String[] {
    //   "com/sogood/i18n/message",
    //   "com/sogood/i18n/common"
    // };
   String[] messageList = new String[basenames.size()];
    messageList = basenames.toArray(messageList);
    messageSource.setBasenames(messageList);
    return messageSource;
  }// 
```

이제는 다국어 파일을 여러 개 설정하여 사용할 수 있다.
