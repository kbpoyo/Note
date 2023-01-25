### 创建注解标志

**创建@MyLog注解，作为切点标志**

```java
@Target({ElementType.FIELD, ElementType.CONSTRUCTOR, ElementType.TYPE, ElementType.METHOD})  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
public @interface MyLog {  
  
    String module() default "";  //在哪个模块
    String operator() default "";  //执行什么操作
}
```

**对接口加上注解进行标记**

```java
@MyLog(module="文章", operator="获取文章列表")  
@PostMapping  
public Result listArticle(@RequestBody PageDto pageDto) {  
  
    //查询文章分页信息  
    Page pageArticle = articleService.listArticle(pageDto);  
  
    //对查询结果进行校验  
    if (pageArticle == null) {  
        return Result.fail(ResultEnum.Query_ERROR);  
    }  
  
    //前端不需要分页的其他信息，只需要文章数据  
    return Result.success(pageArticle.getRecords());  
  
}
```

### 创建AOP通知类

**导入AOP实现类包**
```xml
<dependency>  
    <groupId>org.aspectj</groupId>  
    <artifactId>aspectjweaver</artifactId>  
</dependency>
```

**创建通知类**
```java
@Aspect  
@Component  
@Slf4j  
public class LogAspect {  
  
    //@annotation注解方式标明切点, 指明注解，如果方法上加上该注解则对方法进行增强  
    @Pointcut("@annotation(com.kbpoyo.blog.common.aop.MyLog)")  
    public void pt(){};  
  
    @Around("pt()")  
    public Object advice(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {  
  
        //开始执行  
        long beginTime = System.currentTimeMillis();  
  
        //执行方法  
        Object result = proceedingJoinPoint.proceed();  
  
        //终止执行  
        long endTime = System.currentTimeMillis();  
  
        //保存日志  
        recordLog(proceedingJoinPoint, (endTime - beginTime));  
  
  
        return result;  
    }  
  
    private void recordLog(ProceedingJoinPoint joinPoint, long time) {  
        //获取方法签名  
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();  
        Method method = signature.getMethod();  
        MyLog myLog = method.getAnnotation(MyLog.class);  
        log.info("=====================log start================================");  
        log.info("module:{}",myLog.module());  
        log.info("operator:{}",myLog.operator());  

		//获取请求类型  
		HttpServletRequest request = HttpContextUtils.getHttpServletRequest();  
		String requestMethod = request.getMethod();  
		log.info("request type:{}", requestMethod);
  
        //请求的方法名  
        String className = joinPoint.getTarget().getClass().getName();//获取请求的controller类名  
        String methodName = signature.getName();  
        log.info("request method:{}",className + "." + methodName + "()");  
  
        //请求的参数  
        Object[] args = joinPoint.getArgs();  
        String params = JSONUtils.writeValueAsString(args);  
        log.info("params:{}",params);  
  
        //获取request 设置IP地址  
        log.info("ip:{}", IpUtils.getIpAddr());  
  
        log.info("excute time : {} ms",time);  
        log.info("=====================log end================================");  
    }  
  
}
```

**开启springboot 的 AOP功能**
```java
@SpringBootApplication  
@EnableAspectJAutoProxy  //添加该注解，开启AOP功能
@EnableTransactionManagement  
public class BlogApp {  
  
    public static void main(String[] args) {  
        SpringApplication.run(BlogApp.class, args);  
    }  
}
```


### 运行测试

**url请求**
>http://localhost:8888/articles

**日志记录结果**
![](http://imgs.kbpoyo.top/imgs/spring_boot_202208261511918.png)
