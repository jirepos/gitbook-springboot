# MongoDB 사용하기 
### 의존성 설치 

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

## application.yml
```yaml
spring:
  data:
    mongodb:
      host: localhost
      port: 27017
      database: chat
```      

또는 
```yaml
spring:
  data:
    mongodb:
      uri: mongodb://localhost:27017/tutorial
```

또는 
```yaml
mongodb:
    connection-string: mongodb://localhost:27017/chat
```
MongoDB Connection String의 형식은 다음과 같다. 
```shell
mongodb://[username:password@]host1[:port1][,...hostN[:portN]][/[defaultauthdb][?options]] 
```



## Configuration 
```java
package com.sogood.core.config.jdbc;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.mongodb.MongoDatabaseFactory;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.SimpleMongoClientDatabaseFactory;

@Configuration
public class MongoConfig {

    @Value("${mongodb.connection-string}")
    private String connectionString;

    @Bean
    public MongoDatabaseFactory mongoDatabaseFactory() {
        return new SimpleMongoClientDatabaseFactory(connectionString);
    }

    @Bean
    public MongoTemplate mongoTemplate() {
        return new MongoTemplate(mongoDatabaseFactory());
    }
}
```



## Collection 객체
우선, 도큐먼트 스키마를 작성해주세요. @Document가 몽고DB에서는 collection이 될거에요.몽고 DB에서 collection명은 보통 복수형으로 표기합니다. 

예제에서는 단수로 사용했다. 

```java
package com.sogood.demo.vo;

import lombok.Getter;
import lombok.Setter;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection="user")
@Getter
@Setter
public class MongoUser {
    @Id
    private String id;
    private String name;
    private String email;
}///~
```



## Service Class 
```java
package com.sogood.demo.service;

import com.sogood.demo.vo.MongoUser;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.stereotype.Service;

import java.util.List;


@Service
public class MongoService {
    @Autowired
    private MongoTemplate mongoTemplate;

    public List<MongoUser> selectUsers() {
        return mongoTemplate.findAll(MongoUser.class);
    }//:

}////~

```
## Controller class 
```java
package com.sogood.demo.controller;

import com.sogood.demo.service.MongoService;
import com.sogood.demo.vo.MongoUser;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.List;

@Controller
@RequestMapping("/demo/mongo")
public class MongoController {

    @Autowired
    private MongoService mongoService;
    @GetMapping("/select-users")
    public ResponseEntity<List<MongoUser>> selectUsers(HttpServletRequest request, HttpServletResponse response) {
        List<MongoUser> list = mongoService.selectUsers();
        HttpHeaders headers = new HttpHeaders();
        return new ResponseEntity<List<MongoUser> >(list, headers,  HttpStatus.OK);
    }//:
}///~
```

반환값은 다음과 같다. 
```json
[{"id":"61dba2fdfc9301357c16fa67","name":"Hong gildong","email":"latte@gmail.com"},{"id":"61dba93889357c931b800f51","name":"홍길동2","email":"latte2@gmail.com"},{"id":"61dbaf99fc9301357c16fb93","name":"이미연","email":" lee@gmail.com"},{"id":"61dbbc2efc9301357c16fcbf","name":"이미숙","email":"misook@gmail.com"}]
```



## MongoTemplate 
## find 

```java
    public MongoUser findOne() {
        Criteria criteria = new Criteria("name");
        criteria.is("고민시");
        Query query = new Query(criteria);
        return mongoTemplate.findOne(query, MongoUser.class, "user");
    }//:
```



### insert 

insert()를 사용하여 Document를 추가한다. 마지막 인자에 Collection 이름을 입력한다. 
```java
    /**
     * MongoDB에 Document 추가
     */
    public MongoUser insertUser() {
        MongoUser user = new MongoUser();
        user.setEmail("bbb@gmail.com");
        user.setName("고민시");
        // 마지막 인자는 Collection Name
        return mongoTemplate.insert(user, "user");
        // {"id":"61dbe9ab66d3af6d7a8ca3fe","name":"고민시","email":"bbb@gmail.com"}
    }
```

insert 시에 컬렉션 이름을 주지 않으려면 책체에 @Document 어노테이션을 붙이고 collection 속성에 "user"를 설정한다. 


