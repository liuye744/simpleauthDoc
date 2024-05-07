---
title: Annotations
keywords: keyword1, keyword2
desc: This is a lightweight framework for permission validation and access control based on SpringBoot. It is suitable for lightweight and progressive projects.
date: 2023-09-10
class: heading_no_counter
---

## Annotation Parameters
```java
public @interface SimpleLimit {
    // Maximum number of requests
    int value() default 100;
    // Record the time of requests
    int seconds() default 300;
    // Time after which access is prohibited after exceeding the limit
    int ban() default 0;
    // User identification generation strategy
    Class<? extends SignStrategic> signStrategic() default DefaultSignStrategic.class;
    // Interface identifier, each interface corresponds to multiple user request records
    String item() default "";
    // Strategy to verify whether this request is recorded
    Class<? extends EffectiveStrategic> effectiveStrategic() default DefaultEffectiveStrategic.class;
    // Whether to check if the request is recorded after the response
    boolean judgeAfterReturn() default true;
    // Rate limiting algorithm
    Class<? extends TokenLimit> tokenLimit() default CompleteLimit.class;
    // The default rejection strategy throws an exception 
    Class<? extends RejectedStratagem> rejected() default NullTarget.class;
}
```

## User Identification Generation Strategy
```java
@Component
public class MySignStrategic extends SignStrategic {
    @Override
    public String sign(HttpServletRequest request, SimpleJoinPoint joinPoint) {
        return "Sign"; // Return the user identifier
    }
}
```
Create your own class that extends SignStrategic and return the user identifier. You can register this class as a Bean if needed.

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
When the `judgeAfterReturn` parameter is true, this function will run after the interface call, and you can determine whether the request is recorded based on the `result`.
If `judgeAfterReturn` is false, it will run before the interface call, and `result` will be null.

## Explanation of 'ban'
When 'ban' is set to 0, all actions within the specified time will be recorded, and expired records will be deleted when the next action occurs.
When 'ban' is not 0, requests exceeding the maximum limit will be recorded in the ban list, and all request records will be deleted after the ban period ends.

## Custom Token Limit Algorithm
The default algorithm records all operations precisely and is represented by `CompleteLimit`. You can also use the token bucket algorithm `TokenBucket`.
You can specify the default rate limiting algorithm globally by adding the configuration `simple-auth.func.limit-plan=tokenbucket`, or you can implement the `TokenLimit` interface yourself.
```java
public interface TokenLimit {
    /**
     * Try to acquire, return whether the operation can be successful.
     */
    boolean tryAcquire();

    /**
     * Initialize*
     * @param limit The limit of the number of times to limit
     * @param seconds The unit time of restriction
     */
    void init(Integer limit, Integer seconds);

    /**
     * init *
     * @param capacity The capacity of the limit (the number of times to limit)
     * @param fillRate The rate of addition
     */
    void init(int capacity, double fillRate);

    /**
     * Remove the last operation record
     */
    void removeFirst();

    /**
     * The number of requests that can still be made
     */
    int size();

    /**
     * The number of recorded requested operations
     */
    int optSize();

    /**
     * The maximum number of requests
     */
    int maxOptSize();

    /**
     * Update synchronization information
     */
    void sync();

    /**
     * Get synchronization lock
     */
    Object getSyncMutex();
}
```
After implementing the interface, you can specify it globally through configuration: `simple-auth.func.limit-plan={full-qualified class name}`, or you can add it to specific Controllers using annotations.