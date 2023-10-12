
## AOP(Aspect-Oriented Programming)

```java

//...
private final Logger logger = LoggerFactory.getLogger(this.getClass());
//...
logger.debug("debug anything");

logger.info("START, method A")

//...
catch(Excetion e){
	logger.error("ERROR, exception {}", e);
}
```