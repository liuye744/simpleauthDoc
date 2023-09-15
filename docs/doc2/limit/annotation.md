---
title: Annotations
keywords: keyword1, keyword2
desc: This is a lightweight framework for permission validation and access control based on Spring Boot. Suitable for lightweight and progressive projects.
date: 2023-09-10
---


## Annotation Parameters
```java
public @interface IsLimit {
    // Maximum number of requests
    int value() default 100;
    // Record the time of the request
    int seconds() default 300;
    // Time period during which access is denied after exceeding the limit
    int ban() default 0;
    // User identifier generation strategy
    Class<? extends SignStrategic> signStrategic() default DefaultSignStrategic.class;
    // Interface identifier, each interface corresponds to multiple user request records
    String item() default "";
    // Strategy to verify whether this request is recorded
    Class<? extends EffectiveStrategic> effectiveStrategic() default DefaultEffectiveStrategic.class;
    // Whether to check if the request is recorded after the request returns
    boolean judgeAfterReturn() default true;
}

```

## User Identifier Generation Strategy
```java
@Component
public class MySignStrategic extends SignStrategic {
    @Override
    public String sign(HttpServletRequest request, SimpleJoinPoint joinPoint) {
        return "Sign"; // Return the user identifier
    }
}

```
Create your own class that extends SignStrategic and return the user identifier. If necessary, you can register this class as a Bean.

## Request Record Verification Strategy
```java
@Component
public class MyEffectiveStrategic extends EffectiveStrategic {
    @Override
    public Boolean effective(HttpServletRequest request, SimpleJoinPoint joinPoint, Object result) {
        return true;
    }
}
```
When the judgeAfterReturn parameter is set to true, this method will run after the interface call. You can determine whether the request is recorded based on the result. 

If judgeAfterReturn is set to false, it will run before the interface call, and result will be null.
## Explanation of 'ban'
When ban is equal to 0, it will record all operations within the specified time, and expired records will be deleted when the next operation occurs.

When ban is not equal to 0, requests exceeding the maximum number will be recorded in the ban list. The ban will be lifted after a certain period, and all request records will be deleted.