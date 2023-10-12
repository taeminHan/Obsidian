
## AOP(Aspect-Oriented Programming)



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

## 

![spring-AOP_aspect](https://velog.velcdn.com/images/kai6666/post/819d1819-7322-4804-a730-a3a9a235a5c3/image.png)


- Aspect
	- 공
- JoinPoint
- Advice
- PointCut
- 

```xml
<dependency>
	<groupId>org.springframework.boot</groupId> 
	<artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```


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