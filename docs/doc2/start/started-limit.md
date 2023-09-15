---
title: Quick Start - Access Control
keywords: keyword1, keyword2
desc: This is a lightweight framework for permission validation and access control based on Spring Boot. Suitable for lightweight and progressive projects.
date: 2023-09-10
---

In this section, we will learn how to restrict access to endpoints using annotations.

## Step 1: Add Maven Dependency

```xml
<dependency>
    <groupId>io.github.liuye744</groupId>
    <artifactId>simpleAuth-spring-boot-starter</artifactId>
    <version>1.1.0.RELEASE</version>
</dependency>
```
## Step 2: Add Annotations to Controller
You can add annotations to the entire Controller or to individual methods within the Controller. If access exceeds the limit, an AccessIsRestrictedException will be thrown.

### Use Case 1: Limiting Access Count Within a Specified Time Period

```java
@RestController
public class MyController {
    @GetMapping("say")
    // Allow only 5 accesses within 10 minutes, and restrict access for 10 minutes if exceeded
    @IsLimit(value = 5, seconds = 600, ban = 600)
    public String say(){
        return "Hello World";
    }
}
```

### Use Case 2: Determining Whether to Record the Operation Based on the Return Value

In this example, we want to record the operation only when the endpoint returns "success." If it returns anything else, we don't want to record the operation, and there are no access restrictions.

```java
@RestController
public class MyController {
    @GetMapping("say")
    @IsLimit(effectiveStrategic = MyEffectiveStrategic.class)
    public String say(String str){
        if (str.length() > 3 && str.length() < 12){
            return "success";
        } else {
            return "fail";
        }
    }
}

public class MyEffectiveStrategic extends EffectiveStrategic {
    @Override
    public Boolean effective(HttpServletRequest request, ProceedingJoinPoint joinPoint, Object result) {
        String myResult = (String)result;
        // Return true to record the operation, or false to skip recording
        return "success".equals(myResult);
    }
}
```

### Use Case 3: Recording Operations with Different Access Limits Based on Different Parameters

In this example, we want to apply different access limits to the same endpoint based on different parameters. For instance, we want to limit the number of "likes" for each resource within a specified time frame.

```java
@RestController
public class MyController {
    @GetMapping("say")
    @IsLimit(signStrategic = MySignStrategic.class)
    public String say(String str){
        return "Hello World";
    }
}

public class MySignStrategic extends SignStrategic {
    @Override
    public String sign(HttpServletRequest request, ProceedingJoinPoint joinPoint) {
        final Object[] args = joinPoint.getArgs();
        final Signature signature = joinPoint.getSignature();
        
        // Concatenate the parameters into the user's sign to ensure a unique sign for different parameter values
        StringBuilder sb = new StringBuilder();
        sb.append(signature);
        for (Object arg : args) {
            sb.append(arg.toString());
        }
        
        System.out.println(sb);
        return sb.toString();
    }
}
```
Alternatively, you can use the predefined DiffParameterSign strategy to achieve the same effect.
```java
@RestController
public class MyController {
    @GetMapping("say")
    @IsLimit(signStrategic = DiffParameterSign.class)
    public String say(String str){
        return "Hello World";
    }
}
```
### Use Case 4: Custom Access Control

```java
// Global access control
@Component
public class MyInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        final String addr = request.getRemoteAddr();
        // Use the user's address as an identifier, allowing only 2 requests every 5 minutes (300s),
        // and blocking access for 10 minutes if exceeded.
        // The `addRecord` method returns true if access is allowed, and false if access is prohibited.
        return LimitInfoUtil.addRecord("GLOBAL_ACCESS_CONTROL", addr, 2, 300, 600);
    }
}

@Configuration
public class InterceptorConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyInterceptor()).addPathPatterns("/*");
    }
}
