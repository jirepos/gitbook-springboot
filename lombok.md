# Lombok 사용 


## Summary 

* @Builder 어노테이션을 사용하는 경우에 @NoArgsConstructor가 없으면 new Bean()으로 인스턴스 생성 안된다. 
* @Builder 어노테이션을 사용하는 경우에 @NoArgsConstructor가 있으면 @AllArgsConstructor도 선언이 되어야 한다. 
* 결국 다음과 같이 모두 선언해야 한다. 

```java
/** Lombok Builder를 테스트하기 위한 Bean */
@Getter
@Setter
// @AllArgsConstructor도 같이 선언되어 있어야 함
@Builder   // @NoargsConstructor가 선언된 경우에는 이 어노테이션을 사용하지 못함 
@NoArgsConstructor // @ 이것이 없으면 new LombokBean()으로 샏성 못함 
@AllArgsConstructor
public class LombokBean {
    private String name;
    private int age;
    private String address;
}
```






## Lombok Builder 
Lombok Builder를 사용하면 간단히  setter를 사용하여 객체의 필드에 값을 채울 수 있다. 


Getter,Setter, Builder 어노테이션을 적용한다.


```java
package com.test.java.lombok;

import lombok.Builder;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
@Builder
@RequiredArgsConstructor
public class LombokBean {
    private String name;
    private int age;
}
```

**Builder 사용**

builder를 사용하면 간단히 값을 설정할 수 있다. 마지막에 build() 메서드를 사용한다.
 
```java
package com.test.java.lombok;

import org.junit.jupiter.api.Test;

public class LombokTest {
    @Test
    public void builderTest() {
        LombokBean bean = LombokBean.builder()
                                  .name("홍길동")
                                  .age(20)
                                  .build();
        System.out.println(bean.getName());

    }
}
```



생성자에 Builder 어노테이션을 적용할 수 있다. 

```java
    @Builder
    public LombokTest(final String name, final int age) {
        this.name = name;
        this.age = age;
    }
```

@Builder.Default  어노테이션을 사용하여 기본값을 설정할 수 있다. 

```java
package com.test.java.lombok;

import lombok.Builder;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
@Builder
@RequiredArgsConstructor
public class LombokBean {
    private String name;
    @Builder.Default
    private int age = 25;
}
```




