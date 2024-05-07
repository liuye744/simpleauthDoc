---
title: Quick Start - Access Control
keywords: keyword1, keyword2
desc: This is a lightweight permission verification and access control framework based on SpringBoot. It is suitable for lightweight and progressive projects.
date: 2023-09-10
class: heading_no_counter
---

This section will introduce how to restrict interface access through annotations and learn related parameters for more precise control.

## Step 1: Add Maven Dependency

```xml
<dependency>
    <groupId>io.github.liuye744</groupId>
    <artifactId>simpleAuth-spring-boot-starter</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
```

## Step 2: Add Annotations to Controller

Annotations can be added to the entire Controller or to individual functions within the Controller. If the access exceeds the limit, an `AccessIsRestrictedException` will be thrown.

```java
@RestController
public class MyController {
    @GetMapping("say")
    // Allow only 5 accesses within 10 minutes, after which access will be forbidden for 10 minutes
    @SimpleLimit(value = 5, seconds = 600, ban = 600)
    public String say(){
        return "Hello World";
    }
}
```
## Other Examples

### Use Case 1: Determine Whether to Record the Operation Based on the Return Value
Record the operation only when "success" is returned; do not record the operation when other content is returned, and do not restrict access.

```java
@RestController
public class MyController {
    @GetMapping("say")
    @SimpleLimit(effectiveStrategic = MyEffectiveStrategic.class)
    public String say(String str){
        if (str.length()>3 && str.length()<12){
            return "success";
        }else {
            return "fail";
        }
    }
}

public class MyEffectiveStrategic extends EffectiveStrategic {
    @Override
    public Boolean effective(HttpServletRequest request, ProceedingJoinPoint joinPoint, Object result) {
        String myResult = (String)result;
        // Return true to record, false not to record
        return "success".equals(myResult);
    }
}
```
### Use Case 2: Different Operation Records for the Same Interface with Different Parameters
Different access restrictions for different parameters passed (for example, wanting to restrict each resource to be liked only N times within a specified time period).

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
        // Append the parameters to the user's sign to ensure that each user has a different sign for different parameters
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
Or you can use the predefined `DiffParameterSign` strategy to achieve the same effect.
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

### Use Case 3: Custom Access Control

```java
// Global access control
@Component
public class MyInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        final String addr = request.getRemoteAddr();
        // Use the user's address as the identifier, allowing only 2 accesses every 5 minutes (300s), after which access is forbidden for 10 minutes
        // The addRecord function returns true if access is allowed after the call, and false if access is forbidden
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