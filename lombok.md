# Lombok 사용 


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




