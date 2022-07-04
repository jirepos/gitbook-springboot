# Enum을 이용한 코드 관리 

## 코드 관리 원칙 

코드 관리 원칙은 다음과 같다. 

1. 코드는 서버에서 관리한다. 
2. 코드는 Enum으로 작성한다. 
3. 화면에서 사용할 코드는 Enum을 시리얼라이즈해서 화면으로 전송한다. 
4. 모든 Enum 은 동일한 방식으로 작성하도록 정의된 인터페이스를 구현한다. 
5. Java Bean의 코드를 다루는 필드는 Enum 클래스를 사용한다. 
6. MyBatis에서는 Custom Type Handler를 사용하여 Enum을 처리한다. 


이름 규칙

1. 인터페이스는 CodeEnumType.java로 정의한다. 
2. Enum 클래스는 Code를 뒤에 붙인다. 
   예) CommonCode.java 
3. 각 업무 모듈은 code 패키지를 만들고 그 안에 Enum 클래스를 생성한다.
4. TypeHandler는 EnumTypeHandler를 뒤에 붙인다. 
   예) MybatisStringEnumTypeHandler.java 



## 인터페이스 정의 
다음의 경로에 CodeTypeEnum을 생성한다. 
```
📂com
  📂sogood
    📂core 
      📂types 
         📄CodeEnumType.java
```

인터페이스는 다음과 같이 정의한다. 
```java
public interface CodeEnumType<T> { 
  /** 코드값 반환 */
  T getCode();
  /** 상수의 한글 이름 반환 */
  String getKorName();
  /** 코드값 설명  */
  String getDesc();
}
```

## Enum 정의 
Enum은 CodeEnumType 인터페이스를 구현해야 한다. 다음은 기본 구조이다. CodeEnumType 인터페이스에는 코드의 자료 타입을 정의해야 한다. 아래는 String을 설정했다. 
```java
@Getter
@AllArgsConstructor
public enum BlogPostCode implements CodeEnumType<String> {

  BLOG_POST_DELETE_YN_N("N", "삭제여부", "아니오"),
  BLOG_POST_DELETE_YN_Y("Y", "삭제여부", "예") ;


  private String code; 
  private String korName; 
  private String desc; 

  //  Serialize/Deseriaze하기 위해 어노테이션 사용하고 override한다. 
  @JsonValue
  @Override
  public String getCode() {
    return this.code;
  }


}///~

```

### 코드값으로 Enum 반환
코드 값으로 Enum을 반환할 find() 메서드를 정의한다. 모두 static으로 정의해야 한다. 
```java
  // EnumSet으로 상수 저장
  private static EnumSet<BlogPostCode> enumSet = null;
  static {
    enumSet = EnumSet.allOf(BlogPostCode.class);
  }

  public static BlogPostCode find(String code) {
    // 상수를 Stream을 사용해서 찾기
    return enumSet.stream().filter(constant -> constant.getCode().equals(code)).findFirst().orElseGet(() -> null);
  }
```


### JSON으로 시리얼라이즈하기
자바스크립트에서 코드를 사용하기 위해서 Enum에 정의된 상수를 JSON으로 변환해야 한다. 다음과 같이 getJSON()을 구현한다. 
```java
  public static String getJSON()  {
    // 클라이언트에 전체 상수 값을 내력 주기 위해
    try { 
      JSONArray arr = new JSONArray();
      for (BlogPostCode etype : BlogPostCode.values()) {
        JSONObject json = new JSONObject();
        json.put(etype.name(), etype.getCode());
        json.put("korName", etype.getKorName());
        json.put("desc", etype.getDesc());
        arr.put(json);
      }
      return arr.toString();
    }catch(Exception e) {
      throw new RuntimeException(e);
    }
  }// :
```
다음과 같이 출력된다. 
```shell
[
  {"korName":"삭제여부","USE_YN_Y":"Y","desc":"네"}
 ,{"USE_YN_N":"N","korName":"삭제여부","desc":"아니오"}
]
```


## Enum을 사용한 로직 처리 

### JavaBean에서 코드값 필드 정의 
Java Bean에서는 코드값은 Enum을 사용한다. 
```java
@Getter
@Setter 
public class PostBean {
  private postId; 
  private BlogPostCode delYn; 
}
```

### Enum 사용 필드 값 설정 
다음과 같이  Enum을 사용하여 값을 설정한다.
```java
PostBean bean = new PostBean();
bean.setPostId("111");
bean.setDelYn(BlogPostCode.DEL_YN_N);
```

### NULL 체크 
switch 구문 등에서 null인 경우에는 오류가 발생하므로 null 체크를 한다. 
```java
    if (board.boardType == null) {
      System.out.println("BoardType이 null이다.");
    }
```
### if 문 사용하기 
```java
    if (board.boardType == BoardTypeEnum.USE_YN_Y) {
      System.out.println("사용중입니다.");
    } else {
      System.out.println("사용중이지 않습니다.");
    }
```

### switch 구문 사용하기

```java
    // switch 구문을 사용
    switch (board.boardType) {
    case USE_YN_N:
      System.out.println("사용 중이다.");
      break;
    case USE_YN_Y:
      System.out.println("사용중이지 않다.");
      break;
    default:
      System.out.println("값이 없다.");
      break;
    }
```
## MyBatis에서 Enum 사용 
Enum 타입을 사용하기 위해서는 TypeHandler를 구현해야 한다. 타입 핸들러 정의하고 Enum에 타입 핸들러를 반환하는 메소드를 정의한다 마지막으로 SqlSessionFactory에 핸들러를 등록해야 한다. 

* 타입핸들러 정의
* 타입핸들러 Enum 클래스에 등록
* SqlSessionFactory에 핸들러 등록 


### 핸들러 만들기 
MyBatis에서 제공하는 TypeHandler를 구현한 제네릭 클래스를 정의한다. 아래 핸들러는 String 타입을 처리하기 위한 핸드러이다. Integer 타입을 처리하려면 클래스를 하나 더 생성하여 구현한다. CodeENumType\<String\>을 타입 파라미터에 포함한 것에 주의해야 한다. 

```java

import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.EnumSet;

import com.sogood.core.types.CodeEnumType;

import org.apache.ibatis.type.JdbcType;
import org.apache.ibatis.type.TypeHandler;

public class MyBatisEnumStringTypeHandler<E extends Enum<E> & CodeEnumType<String> > implements TypeHandler<E> {

  /** Enum Constant 목록 */
  private EnumSet<E> enumSet;

  public MyBatisEnumStringTypeHandler(Class<E> type) {
    this.enumSet = EnumSet.allOf(type);
  }

  /**
   * {@inheritDoc}
   */
  @Override
  public void setParameter(PreparedStatement ps, int i, E parameter, JdbcType jdbcType) throws SQLException {

    ps.setString(i, parameter.getCode());
  }

  /**
   * {@inheritDoc}
   */
  @Override
  public E getResult(ResultSet rs, String columnName) throws SQLException {

    String code = rs.getString(columnName);
    return getEnum(code);
  }

  /**
   * {@inheritDoc}
   */
  @Override
  public E getResult(ResultSet rs, int columnIndex) throws SQLException {

    String code = rs.getString(columnIndex);
    return getEnum(code);
  }

  /**
   * {@inheritDoc}
   */
  @Override
  public E getResult(CallableStatement cs, int columnIndex) throws SQLException {

    String code = cs.getString(columnIndex);
    return getEnum(code);
  }
  private E getEnum(String code) {
    return this.enumSet.stream().filter(constant -> constant.getCode().equals(code)).findFirst().orElseGet(() -> null);
  }
}
```

### Enum 클래스에 TypeHandler 정의하기 
생성한 Enum 클래스에 TypeHnalder 클래스를 Inner class로 다음과 같이 정의한다. 
```java
  @MappedTypes(BlogPostCode.class)
  public static class TypeHandler extends MyBatisEnumStringTypeHandler<BlogPostCode> {
    public TypeHandler() {
      super(BlogPostCode.class);
    }
  }
```

Enum 클래스의 전체 코드는 다음과 같다. 
```java
package com.sogood.biz.blog.code;

import java.util.EnumSet;

import com.fasterxml.jackson.annotation.JsonValue;
import com.sogood.core.mybatis.MyBatisEnumStringTypeHandler;
import com.sogood.core.types.CodeEnumType;

import org.apache.ibatis.type.MappedTypes;
import org.json.JSONArray;
import org.json.JSONObject;

import lombok.AllArgsConstructor;
import lombok.Getter;

@Getter
@AllArgsConstructor
public enum BlogPostCode implements CodeEnumType<String> {

  BLOG_POST_DELETE_YN_N("N", "삭제여부", "아니오"), BLOG_POST_DELETE_YN_Y("Y", "삭제여부", "예");

  private String code;
  private String korName;
  private String desc;

  // Serialize/Deseriaze하기 위해 어노테이션 사용하고 override한다.
  @JsonValue
  @Override
  public String getCode() {
    return this.code;
  }

  // EnumSet으로 상수 저장
  private static EnumSet<BlogPostCode> enumSet = null;
  static {
    enumSet = EnumSet.allOf(BlogPostCode.class);
  }

  public static BlogPostCode find(String code) {
    // 상수를 Stream을 사용해서 찾기
    return enumSet.stream().filter(constant -> constant.getCode().equals(code)).findFirst().orElseGet(() -> null);
  }
  
  public static String getJSON()  {
    // 클라이언트에 전체 상수 값을 내력 주기 위해
    try { 
      JSONArray arr = new JSONArray();
      for (BlogPostCode etype : BlogPostCode.values()) {
        JSONObject json = new JSONObject();
        json.put(etype.name(), etype.getCode());
        json.put("korName", etype.getKorName());
        json.put("desc", etype.getDesc());
        arr.put(json);
      }
      return arr.toString();
    }catch(Exception e) {
      throw new RuntimeException(e);
    }
  }// :

  @MappedTypes(BlogPostCode.class)
  public static class TypeHandler extends MyBatisEnumStringTypeHandler<BlogPostCode> {
    public TypeHandler() {
      super(BlogPostCode.class);
    }
  }
}/// ~

```

### TypeHandler 등록하기 
마지막으로 Enum에 정의된 타입 핸들러를 SqlSessionFactory에 등록해야 한다. 자바 패키지 간의 역의존성 문제가 발생하므로 설정파일을 이용하여 처리하도록 한다. 
```java
  @Primary
  @Bean(name = "sqlSessionFactory")
  public SqlSessionFactory sqlSessionFactory(@Qualifier("dataSource") DataSource dataSource, ApplicationContext applicationContext) throws Exception {
         SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
         sqlSessionFactoryBean.setDataSource(dataSource);
         // 생략 
         sqlSessionFactoryBean.setTypeHandlers(new TypeHandler[] {
           new BlogPostCode.TypeHandler()  // 의존성 문제가 생기므로 설정파일을 이용하는 방법으로 변경해야 한다.
         });
         return sqlSessionFactoryBean.getObject();
  }//:
```





