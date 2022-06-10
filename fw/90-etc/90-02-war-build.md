# SpringBoot WAR 빌드하고 Tomcat 띄우기


## 환경

- SpringBoot 2.5
- Tomcat 8.5

## 변경 사항

pom.xml 파일 수정

```jsx
<packaging>war</packaging>

<properties>
		<maven.test.skip>true</maven.test.skip>
	</properties>
```

[SogoodApplicaion.java](http://SogoodApplicaion.java) 수정

```jsx
@SpringBootApplication
/**
 * WAR로 배포하기 위해 SpringBootServletInitializer를 상속 받는다.
 */
public class SogoodApplication extends SpringBootServletInitializer {
    public static void main(String[] args) {
        //System.setProperty("spring.profiles.active", "prod"); // 프로파일 설정
        SpringApplication.run(SogoodApplication.class, args);
    }

	/** WAR로 배포하기 위해 */
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(SogoodApplication.class);
    }
}///~
```

VSCode에서 jvm.options 파일 수정

```jsx
-Dspring.profiles.active=prod
```

## 문제 발생

Filter에서 init() 오류가 발생하는데 아직 해결 방법 모름