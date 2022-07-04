# 다국어 처리

src/main/resources 폴더 아래에 com/latteonterrace/app/i18n 폴더를 생성한다. 그 아래 다국어 리소스 파일(\*.properties)을 디폴트 파일과 언어별 파일을 생성한다. 디폴트 다국어 파일은 로케일을 붙이지 않는다. 다국어 파일명을 다음과 같은 형식으로 작성한다.

아래는 default 파일명의 형식이다.

```
messages.properties
```

아래는 Locale 별 파일 이름 형식이다.

```
messages_[locale].properties
```

예를들어 영어 리소스 파일은 다음과 같이 이름을 작성한다.

```
messages_en.properties
```

src/main/resources 폴더에 i18n.json 파일을 생성한다. 다음과 같이 읽어 들여야 할 리소스 파일을 정의한다. 로케일과 확장자는 표시하지 않는다. 배열이므로 여러개의 리소스 파일을 지정할 수 있다.

```json
[
    "com/latteonterrace/app/i18n/messages"
]
```

다국어 메시지를 가져오려면 MEssageSource를 사용한다. 클래스 변수로 MessagerSource를 설정한다. @Autowired 어노테이션을 사용한다.

```java
	@Autowired
  private MessageSource messageSource; 
```

getMessage() 메소드를 사용하여 메시지를 읽어온다.

````java
	String value = messageSource.getMessage("latte.user.name", null, new Locale("ko"));
```    System.out.println("MessageSource -> latte.user.name:" + value);
    
````
