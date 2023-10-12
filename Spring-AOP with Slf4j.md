
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
흔히 lf4j