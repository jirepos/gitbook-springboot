# YAML 사용하기

Spring Boot 설정 파일은 src/main/resources 폴더에 둔다. .properties 파일로 생성해도 되고 .yml 파일로 생성해도 된다. 이 글에서는 YAML 파일로 작성된 설정파일을 사용하는 것으로 가정한다. 다음은 application.yml 파일의 내용이다.

```yaml
# custom properties
db: localhost
version: 1.0 
```

## 기본 사용방법

spring.freemarker.sufix의 값을 읽어보도록 하자. com.sogood.core.config 패키지에 YAMLConfig.java 파일을 생성하고 다음과 같이 코드를 작성한다.

```java
package com.sogood.core.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties
public class YAMLConfig {
  private String db;
  private String version;
  
  public String getDb() {
    return db;
  }
  public void setDb(String db) {
    this.db = db;
  }
  public String getVersion() {
    return version;
  }
  public void setVersion(String version) {
    this.version = version;
  } 
}///~
```

com.sogood.demo 패키지의 DemoController.java에 YAMLConfig 클래스를 멤버 변수로 정의한다. test() 메소드에서 YAML에 설정된 db 프러퍼티의 값을 출력한다.

```java
package com.sogood.demo;

import com.sogood.core.config.YAMLConfig;
import com.sogood.core.log.Log;

import org.slf4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class DemoController {

	@Autowired
	private YAMLConfig yamlConfig;

	@GetMapping("/demo")
	public String test() {
		logger.debug(yamlConfig.getDb());
		return "demo";
	}
}/// ~
```

## prefix 사용

yaml 설정 파일의 프러퍼티가 계층 구조일 때는 prefix 속성을 사용할 수 있다. yaml에 spring.freemarker.suffix를 사용해보자.

```yaml
spring:
 freemarker:
   suffix: .ftl
   template-loader-path: classpath:/templates 
```

YAMLConfig.java를 다음과 같이 작성한다. prefix 속성을 사용하여 'spring.freemarker'를 설정한다. 'template-loader-path' 프러퍼티를 Camel Notation 표기 방식으로 멤버 변수를 선언한 것을 볼 수있다.

```java
package com.sogood.core.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix="spring.freemarker")
//@ConfigurationProperties
public class YAMLConfig {

  private String suffix; 
  private String templateLoaderPath;

  public String getSuffix() {
    return suffix;
  }
  public void setSuffix(String suffix) {
    this.suffix = suffix;
  }
  public String getTemplateLoaderPath() {
    return templateLoaderPath;
  }
  public void setTemplateLoaderPath(String templateLoaderPath) {
    this.templateLoaderPath = templateLoaderPath;
  } 
  
}///~
```

DemoController.java를 다음과 같이 작성한다.

```java
package com.sogood.demo;

import com.sogood.core.config.YAMLConfig;
import com.sogood.core.log.Log;

import org.slf4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class DemoController {
	/** Logger */
	protected static @Log Logger logger;

	@Autowired
	private DemoService demoService; 

	@Autowired
	private YAMLConfig yamlConfig;

	@GetMapping("/demo")
	public String test() {
		logger.debug(yamlConfig.getTemplateLoaderPath());
		return "demo";
	}
}/// ~
```

## 내부 클래스를 사용하는 방법

application.properties을 다음과 같이 작성한다. app 하위의 menus는 배열이고 title, name, path를 가진 class로 표현될 수 있다.

```yaml
# custom properties
app:
  menus:
    - title: Home
      name: Home
      path: /
    - title: Login
      name: Login
      path: /login
  compiler:
    timeout: 5
    output-folder: /temp/
  error: /error/
```

Menu 내부 클래스를 YAMLConfig.java 내부에 정의한다. 다음과 같이 YAMLConfig.java의 코드를 작성한다.

```java
package com.sogood.core.config;

import java.util.ArrayList;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix="app")
//@ConfigurationProperties
public class YAMLConfig {

  private String error;
  public String getError() {
    return error;
  }

  public void setError(String error) {
    this.error = error;
  }

  public List<Menu> getMenus() {
    return menus;
  }

  public void setMenus(List<Menu> menus) {
    this.menus = menus;
  }

  private List<Menu> menus = new ArrayList<>();

  public static class Menu {
      private String name;
      private String path;
      private String title;
      public String getName() {
        return name;
      }
      public void setName(String name) {
        this.name = name;
      }
      public String getPath() {
        return path;
      }
      public void setPath(String path) {
        this.path = path;
      }
      public String getTitle() {
        return title;
      }
      public void setTitle(String title) {
        this.title = title;
      }
  }
}///~
```

DemoController.java를 다음과 같이 작성한다.

```java
package com.sogood.demo;

import com.sogood.core.config.YAMLConfig;
import com.sogood.core.config.YAMLConfig.Menu;
import com.sogood.core.log.Log;

import org.slf4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class DemoController {
	/** Logger */
	protected static @Log Logger logger;

	@Autowired
	private YAMLConfig yamlConfig;

	@GetMapping("/demo")
	public String test() {
		for(Menu menu : yamlConfig.getMenus()) {
			logger.debug(menu.getName());
		}
		return "demo";
	}
}/// ~
```
