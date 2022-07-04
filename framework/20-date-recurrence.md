# 반복일정 


## 의존성 설정
gom.gooble.ical.jar를 사용한다. 

```shell
group: com.liferay
artifact: com.google.ical.jar 
```

com.google.ical은 2016년 이후에 업데이트가 없다. compile 시에 joda-time을 의존하고 있다. pom.xml에 다음을 설정한다. 

```xml
		<dependency>
			<groupId>joda-time</groupId>
			<artifactId>joda-time</artifactId>
			<version>2.9.9</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/com.liferay/com.google.ical -->
		<dependency>
			<groupId>com.liferay</groupId>
			<artifactId>com.google.ical</artifactId>
			<version>20110304</version>
		</dependency>
```


## 반복일정 생성 시작하기 
간단한 샘플을 작성해 보자.  먼저 RRule 객체를 생성해야 한다. 

**RRule 객체 생성** 
```java
String rrulestr = "RRULE:FREQ=WEEKLY;BYDAY=MO;INTERVAL=1;COUNT=20";  // 매주 월요일 , 20번 
RRule rrule = new RRule(rrulestr);
```
**반복일정 생성 시작일**
언제부터 반복일정을 생성할 것인지 DateTime 객체를 생성한다. 
```java
DateTime iterateFrom = new DateTime("2021-10-22T06:30:00.000+08:00");   // 10월 22일부터(22일 포함)
```
**반복 일정 생성**
DateTimeIteratorFactor.createDateTimeIterator()를 사용하여 반복 일정을 생성한다. 
```java
    DateTimeIterator it = DateTimeIteratorFactory.createDateTimeIterator(
       rrule.toIcal()  // rrule 
      ,iterateFrom     // 언제부터 반복일정을 생성할 것인지 
      ,iterateFrom.getZone() // 반복일정의 타임존 
      , true);
```
Iterator를 반복하며 생성된 일정을 List에 담는다. 
```java
      List<DateTime> occurrences = new ArrayList<>();
      while (it.hasNext()) {
        DateTime n = it.next().withZone(iterateFrom.getZone());
        occurrences.add(n); 
      }
```
생성된 반복 일정을 출력해 본다. 
```java
      occurrences.forEach( item -> {
        System.out.println(item.toString());
      });
```

결과는 다음과 같다.  22일이 월요일이어서 22일이 포함되었다. 
> 22일부터 이후로 20번이므로 21개가 총 생성된 반복 일정이 된다. 



```shell
2021-10-22T07:30:00.000+09:00
2021-10-25T07:30:00.000+09:00
2021-11-01T07:30:00.000+09:00
2021-11-08T07:30:00.000+09:00
2021-11-15T07:30:00.000+09:00
2021-11-22T07:30:00.000+09:00
2021-11-29T07:30:00.000+09:00
2021-12-06T07:30:00.000+09:00
2021-12-13T07:30:00.000+09:00
2021-12-20T07:30:00.000+09:00
2021-12-27T07:30:00.000+09:00
2022-01-03T07:30:00.000+09:00
2022-01-10T07:30:00.000+09:00
2022-01-17T07:30:00.000+09:00
2022-01-24T07:30:00.000+09:00
2022-01-31T07:30:00.000+09:00
2022-02-07T07:30:00.000+09:00
2022-02-14T07:30:00.000+09:00
2022-02-21T07:30:00.000+09:00
2022-02-28T07:30:00.000+09:00
2022-03-07T07:30:00.000+09:00
```

## DateTimeIteratorFactor

DateTimeIteratorFactor 클래스의 createDateTimeIterator() 메서드의 시그니처를 한번 살펴보자.  다음과 같이 사용한다. 
```java
 DateTimeIteratorFactor.createDateTimeIterator(rdata, start, tzid, strict);
```

**Parameters** 

|param | Description |
|------|------|
| rdata | RULE 객체(RULE, EXRULE, RDATE, and EXDATE lines.) |
| start | 반복의 첫번째 발생일 the first occurrence of the series. | 
| tzid  | the local timezone -- TZID 값이 없는 경우 시작일을 해석하기 위해서 used to interpret start and any dates in RDATE and EXDATE lines that don't have TZID params |
| strict | paring이 실패한 경우 ParseException을 던지려면 true로 설정, false는 무시 |


## TimeZone 설정 

### 시간 문자열 표시 
ISO 8601을 기준으로 한다.  자바객체를 문자열로 전환할 때 ISO 8601 형식을 사용한다. 

시간대 관련해서 자세한 사항은 Java의 시간대 관련한 문서를 참고하기로 하고 이 문서에서는 자세히 설명하지 않는다. 



## 필터링








```java
public List<DateTime> occurrencesBetween(DateTime start, DateTime to, DateTime scheduleStart) {
    try {
        // start iterating from a date before today, to avoid having the passed time
        // as the first occurrence
        //(start.isAfter(s.startDateTime()) ? start : s.startDateTime()).minusHours(s.rule().interval());
        DateTime iterateFrom = scheduleStart;
        // to allow first occurrence at current hour, advance to the hour before it
        DateTime firstOccurrence = (start.isAfter(scheduleStart) ? start : scheduleStart).minusMinutes(1);
        List<DateTime> occurrences = new ArrayList<>();
        DateTimeIterator it = DateTimeIteratorFactory.createDateTimeIterator(rrule.toIcal(), iterateFrom, iterateFrom.getZone(), true);
        // advance to the start of the interval
        it.advanceTo(firstOccurrence);
        // iterate until first date out of the interval
        while (it.hasNext()) {
            DateTime n = it.next().withZone(iterateFrom.getZone());
            if (n.isBefore(to)) {
                occurrences.add(n);
            } else {
                break;
            }
        }
        return occurrences;
    } catch (ParseException e) {
        throw new RuntimeException("Error parsing ical", e);
    }
}
```




## Rerferecnes
[Exampe](https://www.javatips.net/api/com.google.ical.compat.jodatime.datetimeiteratorfactory) 
[RRule 정리](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=burin&logNo=40198125788)
[RRule Generator for Web](https://www.textmagic.com/free-tools/rrule-generator)
[Joda Time 사용방법](https://kodejava.org/how-do-i-create-datetime-object-in-joda-time/)
[DateTimeIteratorFactory Class](https://github.com/gafter/google-rfc-2445/blob/master/src/com/google/ical/compat/jodatime/DateTimeIteratorFactory.java)





