
## AOP(Aspect-Oriented Programming)

> `컴퓨팅`에서 관점 지향 프로그래밍(aspect-oriented programming, AOP)은 횡단 관심사(cross-cutting concern)의 분리를 허용함으로써 모듈성을 증가시키는 것이 목적인 프로그래밍 패러다임이다. 
> 출처:  [위키백과](https://ko.wikipedia.org/wiki/%EA%B4%80%EC%A0%90_%EC%A7%80%ED%96%A5_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D)


#### 기존 방식

```java
//...
private final Logger logger = LoggerFactory.getLogger(this.getClass());
//...
logger.debug("debug anything");

logger.info("START, method A")

//...
} catch(Excetion e){
	logger.error("ERROR, exception {}", e);
}
```
흔히 slf4j 사용하여 로그 출력을 작성 할 때는 습관적으로 위와 같이 작성하게 된다. 크게 문제 될 부분은 없지만 method의 시작, 종료, 오류 등 모든 영역에서 중복되는 코드를 작성하여 반복적인 코드작성을 진행해야 한다.

하지만 공통 관심사항(cross-cutting concerns)와 핵심 비즈니스 로직의 코드와 분리하여 코드의 간결성을 높이기 위하여 Spring-AOP를 이용하여 공통 관심 사항인 로그 출력을 비즈니스 로직과 분리하기로 한다.

Spring AOP는 런타임 시점에 적용하는 방식을 사용한다. 컴파일 시점과 클래스 로딩 시점에 적용하기 위해서는 별도의 컴파일러와 클래스로더를 써야 하는데, 이것을 정하고 사용하고 유지보수 하는 과정이 매우 어렵고 복잡하기 때문이다.

## AOP 적용하기

![spring-AOP_aspect](https://velog.velcdn.com/images/kai6666/post/819d1819-7322-4804-a730-a3a9a235a5c3/image.png)


- Aspect
	- 공통 기능을 의미
	- 관심사 등을 모듈화 한 것.(주로 부가기능을 모듈화 함)
- JoinPoint
	- Advice가 적용 될 위치, 메서드 진입 지점, 생성자 호출 시점 등 다양한 시점에 적용 가능

- Advice
	- 어떤 일을 해야 할 지에 대한 것, 부가 기능의 구현체
	- Aspect를 언제 적용할지 정의

|Type(Annotaion)|설명|
|------|---|
|@Before|JoinPoint 실행 이전에 실행, 일반적으로 retrun Type은 void|
|@AfterReturning|JoinPoint 완료 후 실행|
|@AfterThrowing|특정 위치에서 메서드가 예외 처리하는 경우|
|@After|JoinPoint의 동작과 관계없이 실행|
|@Around|메서드 호출 전후에 수행, 가장 강력한 Advice이므로 주의해야함|

- PointCut
	- JointPoint의 상세한 스펙을 정의한 것.
	- '특정 method의 진입 시점에 호출할 것'과 같이 더욱 구체적으로 Advice가 실행될 지점을 정할 수 있음

PointCut은 `PCD`(Pointcut Designator) 이라 하는 특정 지정자를 선언함으로써 시작한다.

```java
@Pointcut("within(…)")
```

|PCD|설명|
|------|---|
|execution|기본적인 PCD|
|within|패키지와 클래스를 제한하는 PCD|

`within`은 `excution`과 비슷하지만, method가 아니라 특정 타입에 속한 method를 PointCut으로 설정할 때 사용한다. 또한 @Controller, @Service, @Repository 처럼 어노테이션을 사용해서 AOP를 사용할 수 있다.

`@within`은 클래스에 특정 어노테이션이 지정되어 있으면, 해당 클래스의 모든 메서드에 Advice를 적용한다.
```java
// @RestController가 선언되어 있는 클래스의 모든 메서드에 적용
@Pointcut("within(@org.springframework.web.bind.annotation.RestController *)")
```


|Wildcard|설명|
|---|---|
|&#42;|기본적으로 임의의 문자열을 의미한다. 패키지를 표현할 때는 임의의 패키지 1개 계층을 의미한다. 메서드의 매개변수를 표현할 때는 임의의 인수 1개를 의미한다.|
|..|패키지를 표현할 때는 임의의 패키지 0개 이상 계층을 의미한다. 메서드의 매개변수를 표현할 때는 임의의 인수 0개 이상을 의미한다.|
|+|클래스명 뒤에 붙여 쓰며, 해당 클래스와 해당 클래스의 서브클래스, 혹은 구현 클래스 모두를 의미한다.|




### Dependency
```xml
<dependency>
	<groupId>org.springframework.boot</groupId> 
	<artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

### Result code
```java
@Component  
@Aspect  
@Slf4j  
public class ControllerLogAspect {  
  
    // @RestController 어노테이션이 적용된 *.class의 메서드를 대상  
    @Pointcut("within(@org.springframework.web.bind.annotation.RestController *)")  
    public void controllerPointCut(){  
    }  
  
    // 메서드 시작 전 실행  
    @Before("controllerPointCut()")  
    public void beforeControllerLogging(JoinPoint joinPoint) {  
       MethodSignature methodSig = (MethodSignature) joinPoint.getSignature();  
       Object[] args = joinPoint.getArgs();  
       String[] parameterNames = methodSig.getParameterNames();  
  
       StringJoiner logMessage = new StringJoiner(", ");  
  
       for (int i = 0; i < args.length; i++) {  
          String paramName = parameterNames[i];  
          Object paramValue = args[i];  
          if (paramValue instanceof RequestData) {  
             paramValue = ((RequestData<?>) paramValue).getContents();  
          }  
          logMessage.add(paramName + "=" + paramValue);  
       }  
  
       log.info("START, {}", joinPoint.getSignature().toShortString());  
       log.info("Parameters: {}", logMessage);  
    }  
    // 메서드 종료 후  
    @AfterReturning(pointcut ="controllerPointCut()")  
    public void afterReturningControllerLogging(JoinPoint joinPoint) {  
       log.info("END, {}", joinPoint.getSignature().toShortString());  
    }  
    // 메서드 Exception 발생 시  
    @AfterThrowing(pointcut ="controllerPointCut()", throwing = "e")  
    public void afterThrowingControllerLogging(JoinPoint joinPoint, HandyException e) {  
       log.error("Occurred ERROR in {}", joinPoint.getSignature().toShortString());  
       log.error("{}", e.getMessage());  
    }  
}
```


Spring AOP와 slf4j를 적용함으로써 HandyOne의 SpringBoot 코드에서 많은 log 출력코드를 제거하여 많은 