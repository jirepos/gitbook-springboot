# JSON 처리

pom.xml 파일에 다음을 추가한다. Spring Boot에서는 기본적으로 Jackson이 의존성에 추가 되었기 때문에 추가할 필요는 없다.

```xml
<!-- Jackson -->
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>
</dependency>
```

Spring이 객체를 어떻게 JSONd로 반환하는지 알아보기 위해 간단한 자바 클래스를 다음과 같이 작성한다.

```java
package com.sogood.demo;

import java.util.Date;
import java.util.List;


public class DemoBean {
  private String userId;
  private String userName;
  private int age; 
  private Date orderDate; 
  private String[] addrs;
  private List<String> favors;
}
```

## Controller에서 JSON 반환

간단히 DemoBean을 생성하고 반환하는 메소드를 Controller에 작성한다.

```java
// JSON 테스트 
@GetMapping(value = "/getDemoBean", produces = ResMediaType.JSON)
public @ResponseBody DemoBean getDemoBean(HttpServletRequest request, HttpServletResponse response) 
	return new DemoBean();
}// 
```

브라우저에서 서버로 호출해 보니 다음과 같이 결과가 반환된다.

```json
{
	"userId": null,
	"userName": null,
	"age": 0,
	"orderDate": null,
	"addrs": null,
	"favors": null
}
```

결과를 보면 숫자는 자바가 인스턴스화할 때 자동ㅇ으로 0이 설정되기 때문에 0이 설정이 되었지만 나머지는 모두 null로 설정이 되었다.

이번에는 인스턴스를 생성하고 값을 설정한 다음에 반환해 보자.

```java
// JSON 테스트 
@GetMapping(value = "/getDemoBean", produces = ResMediaType.JSON)
public @ResponseBody DemoBean getDemoBean(HttpServletRequest request, HttpServletResponse response) 
	DemoBean bean = new DemoBean();
	String[] addrs = new String[] { "11", "22", "33"};
	bean.setAddrs(addrs);
	bean.setFavors(Arrays.asList(addrs));
	bean.setAge(10);
	bean.setOrderDate(new Date());
	return bean; 
}// :
```

브라우저에서 다시 호출해 보면 다음과 같이 결과가 반환된다.

```json
{
	"userId": null,
	"userName": null,
	"age": 10,
	"orderDate": 1623989740467,
	"addrs": [
		"11",
		"22",
		"33"
	],
	"favors": [
		"11",
		"22",
		"33"
	]
}
```

List나 Array는 차이가 없다. Date는 Long값으로 반환된다.

> 커스텀할까도 생각해 보았지만 특별히 하지 않는게 좋을 것 같다.

## ObjectMapper 사용

### 객체를 JSON으로 변환

```java
public void testJson() throws Exception  {
    DemoBean bean = new DemoBean();
		String[] addrs = new String[] { "11", "22", "33"};
		bean.setAddrs(addrs);
		bean.setFavors(Arrays.asList(addrs));
		bean.setAge(10);
		bean.setOrderDate(new Date());
    ObjectMapper objectMapper = new ObjectMapper();
    String json = objectMapper.writeValueAsString(bean); 
  }
```

### JSON 문자열을 객체로 변환

#### 단일 객체로 변환

```java
public void testJson() throws Exception  {

    String json = "...."; // JSON 문자열은 생략
    DemoBean convBean = objectMapper.readValue(json, DemoBean.class);
    System.out.println(convBean.getOrderDate().toString());
  }
```

이전에는 Deserializer를 할 때 JSON에 필드가 없으면 객체를 변환하면서 오류가 발생했는데 지금 버전에서는 오류가 발생하지 않는다.

```java
  public void testJson2() throws Exception  {
    String json = "{ \"userId\": \"latte\" }";
    ObjectMapper objectMapper = new ObjectMapper();
    objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false); // 없는 필드로 인한 오류 무시, 현재 버전에서는 오류가 발생하지 않음 
    DemoBean convBean = objectMapper.readValue(json, DemoBean.class);
    System.out.println(convBean.getUserId());
    System.out.println("OK");
  }
```

#### List로 변환

```java
  @Test
  public void testJsonList() throws Exception  {
    String json1 = "{ \"userId\": \"latte\" }";
    String json2 = "{ \"userId\": \"kim\" }";
    String json = "[" + json1 + "," + json2 + "]";
    ObjectMapper objectMapper = new ObjectMapper();
    List<DemoBean> list = objectMapper.readValue(json, new TypeReference<List<DemoBean>>(){});
    list.forEach(s -> System.out.println(s.getUserId()));
    System.out.println("OK");
  }
```

#### Map으로 변환

```java
  @Test
  public void testJsonMap() throws Exception  {
    String json = "{ \"userId\": \"latte\" }";
    ObjectMapper objectMapper = new ObjectMapper();
    Map<String, Object> map = objectMapper.readValue(json, new TypeReference<Map<String, Object>>(){});
    System.out.println(map.get("userId"));
    System.out.println("OK");
  }
```

## 프로젝트에서 JSON 처리

### JsonObjectMapper.java 생성하기

객체를 JSON으로 변환하거나 반대로 객체로 변화할 때 ObjectMapper를 사용한다. 하지만 후에 ObjetMapper에 커스텀이 필요한 경우가 있을 수도 있어어 ObjectMapper를 상속 받은 JsonObjectMapper.java를 생성하고 이 클래스를 사용한다.

```java
package com.sogood.core.json;

import com.fasterxml.jackson.databind.ObjectMapper;

public class JsonObjectMapper extends ObjectMapper  {
  
}///~
```

프로젝트 진행 중에 JSON으로 변환할 일이 있으면 JSON 처리를 통일성 있게 하기 위해 JsonUtils.java를 이용한다.

## JSONUtils.java 사용

프로젝트에서 ObjectMapper를 직접 사용하지 않고 JSONUtils.java를 사용한다.

### 객체를 JSON 문자열로 변환

toJson() 메서드를 이용한다.

```java
DemoBean bean = new DemoBean();
String[] addrs = new String[] { "11", "22", "33"};
bean.setAddrs(addrs);
bean.setFavors(Arrays.asList(addrs));
bean.setAge(10);
bean.setOrderDate(new Date());
  
String json = JsonUtils.toJSON(bean);
System.out.println(json);
```

### JSON 문자열을 객체로 변환

toObject() 메서드를 이용한다.

```java
String json = "{\"userId\":null,\"userName\":null,\"age\":10,\"orderDate\":1624246540691,\"addrs\":[\"11\",\"22\",\"33\"],\"favors\":[\"11\",\"22\",\"33\"]}";
DemoBean bean = (DemoBean)JsonUtils.toObject(json, DemoBean.class);
System.out.println(bean.getAge());
```

### JSON 문자열을 List로 변환

toList() 메서드를 이용한다.

```java
List<DemoBean> list = new ArrayList<>();
DemoBean bean = new DemoBean();
bean.setUserId("latte");
list.add(bean);
 
String json = JsonUtils.toJSON(list);
System.out.println(json);
List<DemoBean> list2 = JsonUtils.toList(json, DemoBean.class);
list2.forEach(s -> System.out.println(s.getUserId()));
```
