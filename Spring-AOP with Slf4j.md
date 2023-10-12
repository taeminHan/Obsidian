
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
