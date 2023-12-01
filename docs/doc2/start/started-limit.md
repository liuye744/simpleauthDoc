---
title: Quick Start - Access Control
keywords: keyword1, keyword2
desc: This is a lightweight framework for permission validation and access control based on SpringBoot. It is suitable for lightweight and progressive projects.
date: 2023-09-10
class: heading_no_counter
---

In this section, we will explain how to restrict access to interfaces using annotations.

## Step 1: Add Maven Dependency
```xml
<dependency>
    <groupId>io.github.liuye744</groupId>
    <artifactId>simpleAuth-spring-boot-starter</artifactId>
    <version>1.4.3.RELEASE</version>
</dependency>
```

## Step 2: Add Annotations

You can add the annotations to the entire Controller or individual functions within the Controller. Exceeding the access limits will result in an `AccessIsRestrictedException`.

### Use Case 1: Limiting Access Count Within a Specified Time

```java
@RestController
public class MyController {
    @GetMapping("say")
    // Allow only 5 accesses within 10 minutes, and restrict access for 10 minutes if the limit is exceeded.
    @SimpleLimit(value = 5, seconds = 600, ban = 600)
    public String say(){
        return "Hello World";
    }
}
```

### Use Case 2: Determining Whether to Record an Operation Based on the Return Value

Record an operation only when the return value is "success," and do not record when the return value is different. There are no access restrictions in this case.

```java
@RestController
public class MyController {
    @GetMapping("say")
    @SimpleLimit(effectiveStrategic = MyEffectiveStrategic.class)
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
        // Return true to record the operation, false to not record it
        return "success".equals(myResult);
    }
}
```

### Use Case 3: Recording Different Operations Separately with Different Parameters

Different parameters lead to different access restrictions. For example, you may want to limit the number of likes for each resource within a specific time frame.

```java
@RestController
public class MyController {
    @GetMapping("say")
    @SimpleLimit(signStrategic = MySignStrategic.class)
    public String say(String str){
        return "Hello World";
    }
}
public class MySignStrategic extends SignStrategic {
    @Override
    public String sign(HttpServletRequest request, ProceedingJoinPoint joinPoint) {
        final Object[] args = joinPoint.getArgs();
        final Signature signature = joinPoint.getSignature();
        // Concatenate the parameters with the user's sign to ensure a different parameter flag for each user
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
You can also use the built-in `DiffParameterSign` strategy to achieve the same effect.

```java
@RestController
public class MyController {
    @GetMapping("say")
    @SimpleLimit(signStrategic = DiffParameterSign.class)
    public String say(String str){
        return "Hello World";
    }
}
```

### Use Case 4: Custom Access Control

Global access control can be implemented using an `HandlerInterceptor`. In this example, access is restricted to 2 times every 5 minutes (300 seconds) for a user based on their IP address. If a user exceeds the limit, access is restricted for 10 minutes (600 seconds).

```java
// Global access control
@Component
public class MyInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        final String addr = request.getRemoteAddr();
        // Allow only 2 accesses every 5 minutes (300s) for each user, and restrict access for 10 minutes (600s) if the limit is exceeded.
        // Return true if `addRecord` allows access, and false if access is restricted.
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
```