# Spring 5 Recipe 노트



## 1. @Component대신 @Repository, @Service, @Controller를 사용해야하는 이유

- 단순히 사용자가 기능을 구분하기 쉬워 보이도록하는 것 뿐만 아니라
- 특정 용도에 맞는 추가 기능들이 있음
- ex) @Repository 의 경우 발생한 예외를 DataAccessException으로 Wrapping 후 throw

## 2. @Autowired

- 한 클래스 내에서 다른 @bean으로 생성된 인스턴스를 가져올 때 java method명으로 호출하면 해당 method가 실행되서 새로 instance를 만드는게 아니라 Spring Context에서 자동으로 맵핑시킴

```java
@Configuration
public class ExampleConfig {
	@Bean
	public ExampleA exampleA() {
		return new ExampleA();
	}
	
	@Bean
	public ExampleA exampleB() {
		return new ExampleB(exampleA());
	}
}
```

- array나 List에 @Autowired를 붙이는 경우 해당 타입의 빈들을 모두 찾아서 넣어줌
  - 위의 코드에서 ExampleA와 ExampleB가 ExampleParent를 상속받은 경우

```java
@Component
public class autowireExample {
  @Autowired
  private ExampleParent[] examples; // [ExampleA=(), ExampleB=()]
  @Autowired
  private List<ExampleParent> examples2; // [ExampleA=(), ExampleB=()]
}
```

- Map의 경우 key가 빈 이름으로 들어감

```java
@Component
public class autowireExample {
  @Autowired
  private Map<String, ExampleParent> examples; // {"exampleA": ExampleA=(), "exampleB": ExampleB=()}
}
```

- `@Autowired(required = false)` 해당 Bean 타입을 찾지 못하는 경우 Exception을 발생시키지 않고 넘어감

## 3. @Scope

- Scope 정리

| singleton     | IoC Container 당 bean instance 하나 생성                     |
| ------------- | ------------------------------------------------------------ |
| prototype     | Bean을 요청할 때마다 (ex. getBean이 불릴 때 마다) Bean instance를 생성 |
| request       | HTTP Request 마다 Bean Instance 생성 (webapplication-context에서만 해당) |
| session       | HTTP Session 당 Bean Instance 생성 (webapplication-context에서만 해당) |
| globalSession | Global HTTP Session 당 Bean Instance 생성 (portal-application-context에서만 해당) |

