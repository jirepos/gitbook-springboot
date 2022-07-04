# MyBatis SQL Builder 사용 




## 간단한 샘플 
```java
  public static String buildGetUsersByName(final String name) {
    return new SQL(){{
      SELECT("*");
      FROM("users");
      if (name != null) {
        WHERE("name like #{value} || '%'");
      }
      ORDER_BY("id");
    }}.toString();
  }
```
결과
``` SQL 
SELECT *
FROM users
WHERE (name like #{value} || '%')
ORDER BY id
```



## INSERT/UPDATE/DELTE


### DELETE 
```java
  /** DELETE */
  public static String deleteSQL() {
    return new SQL(){{
      DELETE_FROM("PERSON");
      WHERE("ID = #{id}");
    }}.toString();
  }
```
```sql
DELETE FROM PERSON
WHERE (ID = #{id})
```


### INSERT 
```java

  public static String insertPersonSql() {
    return new SQL() {{
      INSERT_INTO("PERSON");
      VALUES("ID, FIRST_NAME", "#{id}, #{firstName}");
      VALUES("LAST_NAME", "#{lastName}");
    }}.toString();
  }
```
```sql
INSERT INTO PERSON
 (ID, FIRST_NAME, LAST_NAME)
VALUES (#{id}, #{firstName}, #{lastName})
```


### INSERT (2) 
```java
  public static String insertSQL() {
    return new SQL() {{
      INSERT_INTO("PERSON");
      INTO_COLUMNS("ID, FIRST_NAME");
      INTO_VALUES("#{id}, #{firstName}");
    }}.toString();
  }
```
```sql
INSERT INTO PERSON
 (ID, FIRST_NAME)
VALUES (#{id}, #{firstName})
```

### UPDATE
```java
  public static String updatePersonSql() {
    return new SQL() {{
      UPDATE("PERSON");
      SET("FIRST_NAME = #{firstName}");
      WHERE("ID = #{id}");
    }}.toString();
  }
```
```sql
UPDATE PERSON
SET FIRST_NAME = #{firstName}
WHERE (ID = #{id})

```


## SELECT 
### FROM 절 두 개의 테이블 
두 개의 FROM() 메소드를 사용하여 두 개의 테이블을 나열할 수 있다. 
```java
  public static String twoFromsSQL() {
    return new SQL() {
      {
        SELECT("SELECT *");
        FROM("USERS A");
        FROM("DEPT B");
      }
    }.toString();
  }
```
FROM 절에 두 개의 테이블이 나열된 것을 볼 수 있다. 
```sql
SELECT SELECT *
FROM USERS A, DEPT B
```

### FROM 절에 두 개의 테이블 (2) 
```java
  public static String twoFromsSQL2() {
    return new SQL() {
      {
        SELECT("SELECT *");
        FROM("USERS A, DEPT B");
      }
    }.toString();
  }
```
```sql
SELECT SELECT *
FROM USERS A, DEPT B
```

### JOIN 
#### INNER JOIN (1)
```java
  public static String innerJoinSQL() {
    return new SQL() {
      {
        SELECT("SELECT *");
        FROM("USERS A");
        INNER_JOIN("DEPT B on B.deptId = A.deptId");
        WHERE("A.userId = #{userId}");
      }
    }.toString();
  }
```
WHERE 조건이 괄호로 둘러 쌓인 것을 확인할 수 있다. 
```sql
SELECT SELECT *
FROM USERS A
INNER JOIN DEPT B on B.deptId = A.deptId
WHERE (A.userId = #{userId})
```


#### INNER JOIN (2)
두 개의 INNERJ_JOIN() 메소드를 사용하여 두 개의 테이블을 JOIN한다. 
```java
  public static String innerJoinSQLTwo() {
    return new SQL() {
      {
        SELECT("SELECT *");
        FROM("USERS A");
        INNER_JOIN("DEPT B on B.deptId = A.deptId");
        INNER_JOIN("DEPT_DETAIL C on C.deptID = C.deptId");
        WHERE("A.userId = #{userId}");
        WHERE("A.userStat = '1'");
      }
    }.toString();
  }
```

두 개의 WHERE() 메소드를 사용했는데 괄호로 둘어 쌓여 있고 두 개의 조건은 AND로 연결되어 있다. 

```sql
SELECT SELECT *
FROM USERS A
INNER JOIN DEPT B on B.deptId = A.deptId
INNER JOIN DEPT_DETAIL C on C.deptID = C.deptId
WHERE (A.userId = #{userId} AND A.userStat = '1')
```

#### Left/Right Join 
```java
    /** LEFT JOIN, RIGHT JOIN*/
    public static String leftRightJoinSQL() {
      return new SQL() {
        {
          SELECT("SELECT *");
          FROM("USERS A");
          INNER_JOIN("DEPT B on B.deptId = A.deptId");
          LEFT_OUTER_JOIN("DEPT_DETAIL C on C.deptID = C.deptId");
          RIGHT_OUTER_JOIN("DEPT_LOG D on D.deptId = B.deptId");
          WHERE("A.userId = #{userId}");
          WHERE("A.userStat = '1'");
        }
      }.toString()
```
```sql
SELECT SELECT *
FROM USERS A
INNER JOIN DEPT B on B.deptId = A.deptId
LEFT OUTER JOIN DEPT_DETAIL C on C.deptID = C.deptId
RIGHT OUTER JOIN DEPT_LOG D on D.deptId = B.deptId
WHERE (A.userId = #{userId} AND A.userStat = '1')


```



### OR 사용 
WHERE 조건을 OR()를 사용하여 분리할 수 있다. 
```java

  /** OR() 사용 */
  public static String orSQL() {
    return new SQL() {
      {
        SELECT("SELECT *");
        FROM("USERS A");
        INNER_JOIN("DEPT B on B.deptId = A.deptId");
        WHERE("A.userId = #{userId}");
        OR();
        WHERE("A.userId = #{admind}");
      }
    }.toString();
  }
```
OR 이후의 조건이 OR로 연결된다. 
```sql
SELECT SELECT *
FROM USERS A
INNER JOIN DEPT B on B.deptId = A.deptId
WHERE (A.userId = #{userId})
OR (A.userId = #{admind})
```

### AND 사용 
```java
    /** AND() 사용 */
    public static String andSQL() {
      return new SQL() {
        {
          SELECT("SELECT *");
          FROM("USERS A");
          INNER_JOIN("DEPT B on B.deptId = A.deptId");
          WHERE("A.userId = #{userId}");
          WHERE("A.flag = #{flag}");
          AND();
          WHERE("B.flag = #{bFlag}");
          WHERE("B.stat = #{bStat}");
        }
      }.toString();
    }
```
처음 두 개의 WHERE가 ( AND ) 로 묶이고, AND 로 연결된 다음에 AND 이후의 WHERE가 ( AND ) 묶인다. 

```sql
SELECT SELECT *
FROM USERS A
INNER JOIN DEPT B on B.deptId = A.deptId
WHERE (A.userId = #{userId} AND A.flag = #{flag})
AND (B.flag = #{bFlag} AND B.stat = #{bStat})
```    

### OR로 묶기
WERHE()를 두 개 사용하면 이 두개는 AND로 연결되기 때문에 두 개의 조건을 OR로 묶고 싶으면 하나의 WHERE()를 사용한다.
 ```java

    /** AND()와 OR() 혼용 사용 */
    public static String andOrSQL2() {
      return new SQL() {
        {
          SELECT("SELECT *");
          FROM("USERS A");
          INNER_JOIN("DEPT B on B.deptId = A.deptId");
          WHERE("A.userId = #{userId}");
          WHERE("A.flag = #{flag}");
          AND();
          WHERE("B.flag = #{bFlag}  OR B.stat = #{bStat}"); // 이렇게 하나의 WHERE() 안에서 사용
          OR();
          WHERE("B.delYn = #{delYn}");
          WHERE("B.regDt >= #{regDt}");        }
      }.toString();
    }
 ```
 ```sql
 SELECT SELECT *
FROM USERS A
INNER JOIN DEPT B on B.deptId = A.deptId
WHERE (A.userId = #{userId} AND A.flag = #{flag})
AND (B.flag = #{bFlag}  OR B.stat = #{bStat})
OR (B.delYn = #{delYn} AND B.regDt >= #{regDt})
 ```

### ORDER BY 

```java
    @Test 
    public void testAndOrSQL2() {
      System.out.println(andOrSQL2());
    }
    /** ORDER BY 사용 */
    public static String orderBySQL() {
      return new SQL() {
        {
          SELECT("SELECT *");
          FROM("USERS A");
          WHERE("A.userId = #{userId}");
          ORDER_BY("A.userId");
          ORDER_BY("A.regDt desc");
        }
      }.toString();
    }
```
```sql
SELECT SELECT *
FROM USERS A
WHERE (A.userId = #{userId})
ORDER BY A.userId, A.regDt desc
```


### GROUP BY 사용
```java
    public static String groupBySQL() {
      return new SQL() {
        {
          SELECT("SELECT *");
          FROM("USERS A");
          WHERE("A.userId = #{userId}");
          GROUP_BY("A.userId");
          GROUP_BY("A.user_age");
        }
      }.toString();
    }
```
```sql
SELECT SELECT *
FROM USERS A
WHERE (A.userId = #{userId})
GROUP BY A.userId, A.user_age
```



### HAVING 사용 
```java
    public static String havingSQL() {
      return new SQL() {
        {
          SELECT("SELECT *");
          FROM("USERS A");
          WHERE("A.userId = #{userId}");
          GROUP_BY("A.userId");
          HAVING("count(A.userId) > 1");
        }
      }.toString();
    }
```
```sql
SELECT SELECT *
FROM USERS A
WHERE (A.userId = #{userId})
GROUP BY A.userId
HAVING (count(A.userId) > 1)
```


### Limit 사용 
```java
    /** Limit  사용 */
    public static String limitSQL() {
      return new SQL() {
        {
          SELECT("SELECT *");
          FROM("USERS");
          WHERE("A.userId = #{userId}");
          LIMIT("5, 10");
        }
      }.toString();
    }    
```
```sql
SELECT SELECT *
FROM USERS
WHERE (A.userId = #{userId}) LIMIT 5, 10
```


## SQL 재사용 하기 
 Mapper.xml을 사용하면 공통으로 사용할 SQL을 \<sql\> 로 분리하여 include를 사용하여 포함할 수 있는데, SQLBuilder를 사용하면 메소드로 분리해야 한다. 

```java
   public static String commonWhere() {
     return "A.NAME = #{name}";
  }
```
다음과 같이 메소드를 호출할 수 있다. 
```java
  @Test 
  public void testCommonWhere() {
    String sql = new SQL() {
      {
        SELECT("*");
        FROM("PERSON A");
        FROM("DEPT B");
        WHERE("A.ID = B.ID");
        WHERE(commonWhere());
        ORDER_BY(commonOrderBy());
      }
    }.toString();
    System.out.println(sql);
  }//:
```
```sql
SELECT *
FROM PERSON A, DEPT B
WHERE (A.ID = B.ID AND A.NAME = #{name})
ORDER BY  A.USER_NAME
```
다른 방법으로는 SQL 인스턴스를 넘겨서 작성할 수 있다. 
```java
  public static void commonOrderBy2(SQL sqlObject){ 
    sqlObject.ORDER_BY("A.USER_NAME");
  }
```
```java
@Test 
  public void testCommonWhere2() {
    String sql = new SQL() {
      {
        SELECT("*");
        FROM("PERSON A");
        FROM("DEPT B");
        WHERE("A.ID = B.ID");
        WHERE(commonWhere());
        commonOrderBy2(this);
      }
    }.toString();
    System.out.println(sql);
  }//:
```

```sql
SELECT *
FROM PERSON A, DEPT B
WHERE (A.ID = B.ID AND A.NAME = #{name})
ORDER BY A.USER_NAME

```


## INLINE VIEW 사용 
INLINE VIEW를 생성할 SQL을 다음과 같이 작성한다. 
```java
  public String inlineSQL() {
    return new SQL() {
      {
        SELECT("*");
        FROM("EMPLOYEE");
        WHERE("EMP_ID = #{empId}");
      }
    }.toString();
  }
  
```

다음과 같이 FROM 절에 inlineSQL()를 호출하여 SQL을 삽입한다. 
```java

  @Test 
  public void testInlineView() {
    String sql = new SQL() {
      {
        SELECT("A.*");
        FROM( "(" + inlineSQL() + ") A" );
        WHERE("A.FALG = 'D'");
      }
    }.toString();
    System.out.println(sql);
  }//:

```
```sql
SELECT A.*
FROM (SELECT *
FROM EMPLOYEE
WHERE (EMP_ID = #{empId})) A
WHERE (A.FALG = 'D')
```


## SubQuery 사용 
Sub Query 도 SQL 인스턴스를 생성하여 사용할 수 있다. 
```java
 @Test 
  public void testSubQuery(){ 
    String sql = new SQL() {
      {
        SELECT("A.*");
        FROM("EMPLOYEE A");
        WHERE("A.FLAG = 'Y'");
        WHERE("EXISTS ("
            + new SQL() {
              {
                SELECT("1");
                FROM("STAT B");
                WHERE("B.ID = A.ID");
              }
            }.toString()
            +
        ")");
      }
    }.toString();

    System.out.println(sql);

  }//:
```
```sql
SELECT A.*
FROM EMPLOYEE A
WHERE (A.FLAG = 'Y' AND EXISTS (SELECT 1
FROM STAT B
WHERE (B.ID = A.ID)))
```

## Scalar Query 
Scalar Query도 Inline View 또는 Sub Query와 같은 방식으로 작성할 수 있다. 





## 다른 스타일 
SQL을 생성하면서 코드 블럭에서 SQL을 작성하지 않고 다음과 같이 작성할 수도 있다. 
```java
  @Test
  public void testAnotherStyle() {
    SQL sb = new SQL();
    sb.SELECT("*");
    sb.FROM("EMPLOYEE");
    sb.WHERE("EMP_ID = #{empId}");
    String sql = sb.toString();
    System.out.println(sql);
  }
```

```sql
SELECT *
FROM EMPLOYEE
WHERE (EMP_ID = #{empId})
```


## Mapper 사용하기 
Mapper 인터페이스를 생성하고 @Mapper 어노테이션을 붙인다. 
```java
@Mapper 
public interface EmployeeMapper {
}
```
갇단한 쿼리는 SQL 인스턴스르 사용하지 않아도 된다. 
```java
@Mapper
public interface EmployeeMapper  {

  @Select("SELECT * FROM EMPLOYEE")
  List<Employee> selectAll();

  @Select("SELECT * FROM EMPLOYEE WHERE EMP_ID = #{empId"})
  Employee selectEmployee(@Param("empId") String empId);

  @Delete("DELETE FROM EMPLOYEE WHERE EMP_ID = #{empId}")
  int deleteEmployee(String empId); 

}
```

SQL 인스턴스를 사용하여 쿼리를 빌드하는 경우에는 SelectProvider를 사용한다. 
```java
@SelectProvider(type = UserSqlBuilder.class, method = "buildGetUsersByName")
List<User> getUsersByName(String name);
```
```java
class UserSqlBuilder {
  public static String buildGetUsersByName(final String name) {
    return new SQL(){{
      SELECT("*");
      FROM("users");
      if (name != null) {
        WHERE("name like #{value} || '%'");
      }
      ORDER_BY("id");
    }}.toString();
  }
}
```
SelectProvider 말고 InsertProvider, UpdateProvider, DeleteProvider가 있다. 

```java
  @InsertProvider(type = UserSqlBuilder.class, method = "createUser")
  void createUser(User user);

  @UpdateProvider(type = UserSqlBuilder.class, method = "updateUser")
  void updateUser(User user);

  @DeleteProvider(type = UserSqlBuilder.class, method = "deleteUser")
  void deleteUser(int id);
  ```

## 결과 매핑 

다음 예제는 @Results애노테이션의 id속성을 명시해서 결과매핑을 명명하는 방법을 보여준다.
```java
@Results(id = "userResult", value = {
  @Result(property = "id", column = "uid", id = true),
  @Result(property = "firstName", column = "first_name"),
  @Result(property = "lastName", column = "last_name")
})
@Select("select * from users where id = #{id}")
User getUserById(Integer id);

@Results(id = "companyResults")
@ConstructorArgs({
  @Arg(column = "cid", javaType = Integer.class, id = true),
  @Arg(column = "name", javaType = String.class)
})
@Select("select * from company where id = #{id}")
Company getCompanyById(Integer id);
```

## Reference
[SQL Builder 클래스](https://mybatis.org/mybatis-3/ko/statement-builders.html)
[MyBatis Java API](https://mybatis.org/mybatis-3/ko/java-api.html)






