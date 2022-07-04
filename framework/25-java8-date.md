# Java8 Date

## DateTime 처리 기준 

* 날짜 시간에 대한 클래스 멤버 필드는 LocalDateTime을 사용한다. 
* DB에는 UTC 값을 저장한다. 
* 3.4.5 버전부터, MyBatis는 JSR-310(Date 와 Time API) 를 기본적으로 지원한다.
* DB에서 가져온 DateTime은 사용자의 시간대로 변환하여 클라이언트에 반환한다.
* LocalDateTime을 JSON으로 변환할 때는 '2022-01-18T11:10:43'의 형식으로 변환한다. 
* 클라이언트가 전송한 DateTime은 사용자의 시간대의 LocalDateTime이기 때문에 UTC로 변환한다. 
* 시간대는 사용자의 PC의 시간대가 아닌 사용자가 선택한 시간대를 사용한다.
* 시간대를 사용하려면 반드시 날짜가 있어야 한다. 시간만 가지고서는 시간대에 맞게 변환이 불가능하다. 

## Java Bean에서 LocalDateTime 필드 
클래스 필드에서 LocalDateTime 클래스를 타입으로 사용한다. 
```java
public class TestBean { 
	private LocalDateTime regDt; 
}
```

## JSON 포맷 설정 
LocalDateTime을 JSON으로 변환하기 위해 \@JsonFormat 어노테이션을 사용한다. 
```java
public class Testbean {
  @JsonFormat(shape=JsonFormat.Shape.STRING, pattern="yyyy-MM-dd'T'HH:mm:ss.SSS")
	private LocalDateTime regDt;
} 
```
LocalDateTime을 Serialize하려면 ObjectMapper의 JavaTimeModule를 활성화 해야 한다. 
```java
ObjectMapper mapper = new ObjectMapper();
// LocalDateTime을 Serialize하려면 JavaTimeModule을 활성화해야 한다.
mapper.registerModule(new JavaTimeModule());
```

ObjectMapper를 직접사용하지 않는 경우, 즉 SpringBoot에서 사용하는 경우에는 application.yml에 다음을 설정한다. 

```yaml
spring:
  jackson:
    serialization:
      WRITE_DATES_AS_TIMESTAMPS: false
```

### \@JsonFormat 사용하지 않고 처리 

Spring에서 java.util.Date와 java.time.LocalDateTime을 Serialize하면 다음과 같이 시리얼라이즈된다. 
```shell
 1642992708042   // java.util.Date
 [2022, 1, 24, 11, 51, 48, 48293600]  // java.time.LocalDate
```
Date type을 Serialize/deserialize 하려면 두가지를 고려해야 한다. 

첫번쩨는, JSON을 Object로 변환하거나 Object를 JSON으로 변환하는 것이고
두번째는, Request parameter나 PathVariable를 Bean으로 변환하는 것이다. 

#### extendMessageConverters() 구현 
Object를 JSON으로 변환하거나 반대의 경우에는 Jackson을 사용한다. Date 타입을 ISO 형식에 맞게 시리얼라이즈/디시리얼라이즈 하려면 Jackson2ObjectMapperBuilder를 사용한다.  WebMvcConfigurer 인터페이스의  extendMessageConverters()를 구현하면 Date type을 원하는 형태로 변환할 수 있다. 
```java
@EnableWebMvc
@Configuration
public class DispatcherConfig implements WebMvcConfigurer {
  @Override
  public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
    ObjectMapper objectMapper = Jackson2ObjectMapperBuilder // 스프링이 제공하는 클래스
            .json()
            // 다음 매서드는 유닉스 타임스태프로 출력하는 기능을 비활성화(ISO-8601 사용)
            .featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
            .build();
    /*
     * 미리 등록된 HttpMessageConverter에는 Jackson을 사용하는 것도 포함되어 있으므로,
     * 새로 생성한 HttpMessageConverter는 다음과 같이 인덱스 0에 위치(맨 앞)함
     */
    converters.add(0, new MappingJackson2HttpMessageConverter(objectMapper));
  }

}  
```

이렇게 하면, JSON을 Java Bean 객체로 변환할 수 있다. 
```java
    @GetMapping("/get-date")
    public ResponseEntity<DemoBean> getJson() {
        DemoBean bean = new DemoBean();
        bean.setUserName("Hong");
        bean.setAge(10);
        bean.setOrderDate(Date.from(Instant.now()));
        bean.setRegDt(LocalDateTime.now());
        return new ResponseEntity<DemoBean>(bean, HttpStatus.OK);
    }//:
```
JSON 응답결과를 보면 날짜 형식이 java.util.Date와 java.time.LocalDateTime이 조금 다른데  java.time.LocalDateTime의 경우 ISO 형식으로 봔한 되는 것을 볼 수 있다. 
```java
{
    "userId": null,
    "userName": "Hong",
    "age": 10,
    "orderDate": "2022-01-24T03:03:14.925+00:00",  // java.util.Date 
    "regDt": "2022-01-24T12:03:14.9294194",  // java.time.LocalDateTime
    "addrs": null,
    "favors": null
}
```
JSON을 Java Bean의 Object로 변환하려면 ISO 형식에 맞게 문자열을 만들어서 보내야 한다. 
```json
        let options = {
          method: 'POST',
          url:'/demo/date/put-date',
          body: {
            userId: "a123",
            userName: "홍길동",
            age: 30,
            regDt: "2022-01-21T18:17:32.7784342" // ISO 형식
          }
```          


```java
    @PostMapping("/put-date")
    public ResponseEntity<DemoBean> getJson(@RequestBody DemoBean bean) {
        return new ResponseEntity<DemoBean>(bean, HttpStatus.OK);
    }//:
```


#### addFormatters() 구현 

일단, Conrolller의 메서드를 살펴보자. 

```java
    @GetMapping("/use-path/{regDt}")
    public ResponseEntity<DemoBean> getJsonPathVariable(@PathVariable("regDt") LocalDateTime regDt) {
        DemoBean bean = new DemoBean();
        bean.setRegDt(regDt);
        return new ResponseEntity<DemoBean>(bean, HttpStatus.OK);
    }//:
```    
아래 예시에서처럼 \@PathVariable이나 Reqeuest Paraemter를 LocalDateTime으로 변환하려면 WebConfigurer의 addFormatters() 메서드를 구현해야 한다. 
```java
@EnableWebMvc
@Configuration
public class DispatcherConfig implements WebMvcConfigurer {
  @Override
  public void addFormatters(FormatterRegistry registry) {
    DateTimeFormatterRegistrar dateTimeFormatterRegistrar = new DateTimeFormatterRegistrar();
//    dateTimeFormatterRegistrar.setUseIsoFormat(true); // ISO 포맷 사용시, 그게 아니면 각자 명시적 설정
    dateTimeFormatterRegistrar.setDateFormatter(DateTimeFormatter.ISO_DATE);
    dateTimeFormatterRegistrar.setDateTimeFormatter(DateTimeFormatter.ISO_DATE_TIME);
    dateTimeFormatterRegistrar.registerFormatters(registry);
  }
}  
```

아래 코드와 같이 \@PathVariable이나 Request Parameter를 객체로 변환하려면 별도의 포맷터를 구현해야 한다. 차후에 필요한 경우에 구현하기로 하고 그렇다는 것만 기억하자. 

```java
    @GetMapping("/use-path/{regDt}")
    public ResponseEntity<Person> getJsonPathVariable(@PathVariable("regDt") Person person) {
        return new ResponseEntity<Person>(person, HttpStatus.OK);
    }//:
```
JSON을 보낼 때에는 다음과 같이 한다. 
```jsx
        let  regDt = "2022-01-21T18:17:32.7784342";

        let options = {
          method: 'GET',
          url:'/demo/date/use-path/' + regDt
        }
```


## 시간대 저장 
사용자가 시간대를 선택할 수 있도록 시간대를 DB에 저장한다. 시간대는 ZoneId와 Offset을 구분하여 저장한다. ZoneIdBean을 사용한다. 
```java
public class ZoneIdBean {
    /** ZoneId 문자열 */
    private String zoneId;
    /** Offset 값. 예) +09:00 */
    private String offset;
}///~
```

DB에 저장할 데이터를 구하기 위해서 DateUtils.java의 getTimeZoneIdsWithZoneIdBean() 메서드를 사용한다. 

```java
    @Test
    public void testZonIds() {
        List<ZoneIdBean> list = DateUtils.getTimeZoneIdsWithZoneIdBean();
        list.forEach( item -> {
            System.out.format("%s (%s) \n", item.getZoneId(), item.getOffset() );
         });
    }
```
결과 

```shell
Asia/Aden (+03:00) 
America/Cuiaba (-03:00) 
Etc/GMT+9 (-09:00) 
Etc/GMT+8 (-08:00) 
... 생략 ....
```


## 현재 시간 구하기 
현재 시간은 항상 UTC를 기준으로 구한다.  DateTimeUtils.java의 nowUTC() 메서드를 사용한다. 
```java
    @Test
    public void testNowUTC() {
        LocalDateTime ldt = DateUtils.nowUTC();
        System.out.println(ldt);  //2022-01-17T06:36:30.618465300
    }//:
```


결과 

```shell
2022-01-19T00:37:53.151740400
```

DB에 저장할 때는 UTC 기준 시간대의 LocalDateTime을 그대로 저장한다. 


## 시간대 변경하기
DB에는 UTC 기준으로 시간값이 저장되어 있다. DB에서 데이터를 조회한 다음에 클라이언트에 보내려면 사용자의 시간대로 변환해야 한다. DateUtils.java의 changeUTCToAnotherZone() 메소드를 사용한다. 

사용자가 선택한 시간대가 Asia/Seoul이라면 다음과 같이 작성한다. 
```java
// UTC를 서울 시간대로 변경
LocalDateTime d2 = DateUtils.changeUTCToAnotherZone(bean.getRegDate(), ZoneId.of("Asia/Seoul")); 
bean.setRegDate(d2);  // 사용자 시간대로 다시 값을 설정 
```

반대로, 사용자가 선택한 시간을 DB에 저장하려면 시간대를 UTC로 변경하고 UTC 시간대의 LocalDateTime을 구해야 한다.  DateUtils.java의 changeZoneToUTC() 메서드를 사용한다.  이 메서드의 두번째 인자는 사용자의 시간대 ZoneId이다. 

```java
LocalDateTime d4 = DateUtils.changeZoneToUTC(d2, ZoneId.of("Asia/Seoul"));
```


## DB에서 검색 
DB에는 UTC 기준으로 시간 값이 들어 있다. 사용자는 검색을 위해 날짜를 선택하면 DB에 저장된 UTC 기준으로 변환하여 조건식을 만들어야 한다. 예를 들어, 사용자가 선택한 시간대가 2022-01-01이면 사용자의 시간대 기준으로 조건은 다음과 같다. 

```shell
2022-01-01 00:00:00 부터 2022-01-01 23:59:59 까지 
```

밀리세컨이나 나노세컨이 계산이 어려우니까 계산을 쉽게 하자면  다음과 같이 할 수 있다. 

```shell
날짜 >= 2022-01-01 00:00:00 AND 날짜 < 2022-01-02 00:00:00 까지 
```

먼저 선택한 날짜를 0시 0분을 기준으로 LocalDateTime으로 변환한다. 그리고 변환된 날짜에 1일을 더한다.  그러면 from ~ to 조건의 LocalDateTime을 만들 수 있다. 
```java
        String dateStr = "2022-01-01";
        LocalDate ld = LocalDate.parse(dateStr, DateTimeFormatter.ofPattern("yyyy-MM-dd" ));
        LocalTime lt = LocalTime.of(00, 00);
        LocalDateTime localFrom = LocalDateTime.of(ld,lt);
        // 이것을 UTC로 변경
        LocalDateTime from = DateUtils.changeZoneToUTC(localFrom, ZoneId.of("Asia/Seoul"));
        System.out.println(from);  // 2021-12-31T15:00
        LocalDateTime to = from.plusDays(1);
        System.out.println(to);  // 2022-01-01T15:00
```

간단히 하기 위해서 DateUtils.java의 toUTCLocalDateTime() 메서드를 사용한다. 

```java
    @Test
    public void testMakeCondition2() {
        String dateStr = "2022-01-01";
        LocalDateTime from = DateUtils.toUTCLocalDateTime(dateStr, "yyyy-MM-dd"
                , 0,0,0, ZoneId.of("Asia/Seoul"));
        LocalDateTime to = from.plusDays(1);
        System.out.format("%s ~ %s \n", from, to);
				// 2021-12-31T15:00 ~ 2022-01-01T15:00 
    }//:
```

쿼리에서 직접 사용하는 경우 다음과 같이 할 수 있다. 
```sql
select * from blog_post
where reg_dt >= str_to_date('20220119:040000', '%Y%m%d:%H%i%S')
and reg_dt < str_to_date('20220120:040000', '%Y%m%d:%H%i%S')
```




## DateUtils.java 
앞에서 설명하지 않은 주요 메서드들을 살펴본다. 








## Classes

### LocalTime/LocalDate/LocalDateTime

시간대(Zone Offset/Zone Region)에 대한 정보가 전혀 없는 API이다.

### ZoneOffset

UTC 기준으로 시간(Time Offset)을 나타낸 것이라고 보면 된다. 우리나라는 KST를 사용하는데 KST는 UTC보다 9시간이 빠르므로 UTC +09:00으로 표기한다. ZoneOffset은 ZoneId의 자식 클래스이다.

* Instant는 글로벌 타임 라인 (UTC)에서 순간적인 지점이며 시간대와 관련이 없다.
* LocalDate 및 LocalDateTime에는 표준 시간대 개념이 없지만 now()을 (를) 호출하면 올바른 시간을 얻을 수 있다.
* OffsetDateTime에는 시간대가 있지만 일광 절약 시간제를 지원하지 않습니다.
* ZonedDateTime에는 표준 시간대 지원이 있습니다.





### ZoneRegion

Time Zone을 나타낸 것이라고 보면 된다. KST는 타임존의 이름이고 이를 나타내는 ZoneRegion은 Asia/Seoul이다. ZoneRegion은 ZoneId의 자식 클래스이다.

### ZoneRules

ZoneOffset의 UTC +09:00과 ZoneRegion의 Asia/Seoul을 보면 전혀 차이가 없다. 그럼 ZoneOffset과 ZoneRegion은 왜 따로 분리돼있는 걸까? 좀 더 지역에 특화된, 지명 등등을 넣어서 그 의미를 살리고자 분리가 되거나 한 걸까? 이 차이는 DST(Daylight saving time, 서머타임)와 같은 Time Transition Rule을 포함하느냐, 포함하지 않느냐로 갈린다. ZoneOffset은 Time Transition Rule을 포함하지 않는 ZoneRules를 가진다. ZoneRegion은 Time Transition Rule을 포함할 수도, 포함하지 않을 수도 있다.

### OffsetDateTime

LocalDateTime + ZoneOffset에 대한 정보까지 포함한 API이다. 이러한 경우는 축구 경기 생중계 등등에 적합하다.

### ZonedDateTime

OffsetDateTime + ZoneRegion에 대한 정보까지 포함한 API이다. UTC +09:00의 Time Offset을 가지는 Time Zone도 여러가지이다.

* Asia/Seoul
* Asia/Tokyo
* 등등

하지만 시간을 나타내는데 있어서 Asia/Seoul을 쓰던 Asia/Tokyo를 쓰던 큰 차이점이 없다. OffsetDateTime과의 차이점은 DST(Daylight Saving Time)와 같은 Time Transition Rule을 포함하는 ZoneRegion을 갖고 있는 ZoneRules의 유무이다. 독일 등등에서 사용하는 CET(겨울), CEST(여름)는 서머타임을 사용하지 않는 나라에 사는 나 같은 경우에는 굉장히 생소하다. 그래서 어떤 때는 CET를 사용해야하고, 어떤 때는 CEST를 사용해야할지 매우 애매하고 계산하기도 까다롭다. 자바에서는 이 두 Time Zone을 하나의 Time Zone(CET)로 통일하고 Time Transition Rule을 가지는 ZoneRules를 통해 알아서 내부적으로 계산해준다.

### Instant

어느 순간을 나타내는 클래스이다. Unix Timestamp를 구할 때 사용한다. UTC 기준이기 때문에 글로벌한 서비스에서도 매우 적합할 것이다.

## JRE 및 DST 가변성

첫째, 전 세계 DST 영역이 매우 자주 변경 되며이를 조정하는 중앙 기관이 없다는 사실을 이해하는 것이 매우 중요 합니다.

국가 또는 경우에 따라 도시에서 신청 또는 취소 여부와 방법을 결정할 수 있습니다.

변경 사항이 발생할 때마다 [**IANA 시간대**](https://www.iana.org/time-zones) 데이터베이스에 기록되고 업데이트는 JRE 의 향후 릴리스 에서 롤아웃됩니다 .

기다릴 수없는 경우 Java SE 다운로드 페이지 에서 사용할 수있는 Java Time Zone Updater Tool 이라는 공식 Oracle 도구를 통해 새 DST 설정이 포함 된 수정 된 시간대 데이터를 JRE로 강제 적용 할 수 있습니다 .

> 라이선스가 없으면 Java Time Zone Updater Tool은 사용할 수 없음. \*\* 더 확인해 보아야 함 \*\*

## 올바른 TZDB 시간대 아이디

Java에서 DST를 처리하는 올바른 방법 은 "Europe/Rome"와 같은 특정 TZDB Timezone ID로 시간대를 인스턴스화하는 것 입니다.

## 현재의 로컬 시간을 구한다.

```java
	@Test
	public void testCurrentDateTime() {
		LocalDateTime localDateTime  = LocalDateTime.now();
		System.out.println(localDateTime);
		// 2019-09-25T15:50:43.265
	}
```

## 모든 ZoneID를 출력

```java
	/**
	 * 모든 ZoneId를 출력한다.
	 */
	@Test
	public void testPrintAllZoneIds() {
		Set<String> allZoneIds = ZoneId.getAvailableZoneIds();
		allZoneIds.forEach(id -> System.out.println(id));
  }
```

> java.time.ZonedDateTime 클래스는 영역 세부 정보로 datetime 객체를 처리하는 데 도움이됩니다. 가장 중요한 것은 java.time.ZonedDateTime 클래스가 DST를 완전히 인식 하고 있으므로 어깨에서 부담을 덜어줍니다.






```shell
// 로컬 시간을 의미하는 ISO 8601 문자열
2021-10-06T15:00:00.000
// UTC(GMT) 시간을 의미하는 ISO 8601 문자열
2021-10-06T06:00:00.000Z

// 로컬 시간을 의미하면서 UTC(GMT) 대비 +09:00 임을 의미하는 ISO 8601 문자열
2021-10-06T15:00:00.000+09:00
```
- `2021-10-06T15:00:00.000`은 **ISO 8601**의 기본 형식이다. 해당 시간이 로컬 시간 임을 의미한다.
- `2021-10-06T06:00:00.000Z`와 같이 뒤에 `Z` 식별자를 추가하면 해당 시간이 **UTC(GMT)** 시간 임을 의미한다.
- `2021-10-06T15:00:00.000+09:00`와 같이 뒤에 **Z** 대신 `+HH:mm` 식별자를 추가하면 해당 시간이 로컬 시간이면서 **UTC(GMT)**와 **09:00** 만큼 차이가 남을 의미한다. 이 형식의 장점은 인간이 손쉽게 추가적인 계산 없이 로컬 시간을 인지하면서 추가적으로 타임존 정보까지 제공하기 때문에 가장 인간친화적이라고 할 수 있다.




## References

[날짜와 시간 API with Java8](https://wickso.me/java/java-8-date-time/)\
[(Java8) 날짜와 시간 API](https://perfectacle.github.io/2018/09/26/java8-date-time/) [Java에서 일광 절약 시간을 처리하는 방법](http://www.programmergirl.com/handle-day-light-savings-java/)
