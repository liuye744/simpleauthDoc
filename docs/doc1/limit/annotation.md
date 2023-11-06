---
title: 注解
keywords: keyword1, keyword2
desc: 这是一款基于SpringBoot的轻量化的权限校验和访问控制的框架。适用于轻量级以及渐进式的项目。
date: 2023-09-10
---


## 注解参数
```java
public @interface SimpleLimit {
    //最大请求次数
    int value() default 100;
    //记录请求的时间
    int seconds() default 300;
    //超过限制后被禁止访问的时间
    int ban() default 0;
    //用户标识的生成策略
    Class<? extends SignStrategic> signStrategic() default DefaultSignStrategic.class;
    //接口的标识，每个接口对应多个用户请求记录表
    String item() default "";
    //验证本次请求是否被记录的策略
    Class<? extends EffectiveStrategic> effectiveStrategic() default DefaultEffectiveStrategic.class;
    //是否在请求返回后判断请求是否被记录
    boolean judgeAfterReturn() default true;
    //限流的算法
    Class<? extends TokenLimit> tokenLimit() default CompleteLimit.class;
}
```

## 用户标识生成策略
```java
@Component
public class MySignStrategic extends SignStrategic {
    @Override
    public String sign(HttpServletRequest request, SimpleJoinPoint joinPoint) {
        return "Sign";//返回用户标志
    }
}
```
创建自己的类继承SignStrategic，返回用户标志。若有需要可以将此类注册为Bean。

## 验证请求记录策略
```java
@Component
public class MyEffectiveStrategic extends EffectiveStrategic {
    @Override
    public Boolean effective(HttpServletRequest request, SimpleJoinPoint joinPoint, Object result) {
        return true;
    }
}
```
当`judgeAfterReturn`参数为true时，将会在接口调用之后运行此函数，可以根据result判断此次请求是否被记录。
若`judgeAfterReturn`为false，将在接口调用之前运行，此时`result`为null。

## ban详解
当ban等于0时，将会记录规定时间内的所有操作，下次操作发生时删除逾期记录。
当ban不为0时，请求超过最大次数后将会被记录到禁止列表，禁止时间满足取消禁止之后将删除所有请求的记录。

## 自定义tokenLimit算法
默认的算法是将所有的操作都精确的记录，类为`CompleteLimit`,当然也可以使用令牌桶算法`TokenBucket`。
指定全局的默认限流算法可以添加配置`simple-auth.func.limit-plan=tokenbucket`
或者自己实现`TokenLimit`接口
```java
public interface TokenLimit {
    /**
     * 尝试获取，返回是否可以成功操作
     */
    boolean tryAcquire();

    /**
     * 初始化*
     * @param limit 限流的次数
     * @param seconds 限制的单位时间
     */
    void init(Integer limit, Integer seconds);

    /**
     * init *
     * @param capacity 限流的容量(限流的次数)
     * @param fillRate 添加的速率
     */
    void init(int capacity, double fillRate);

    /**
     * 移除最后一次操作记录
     */
    void removeFirst();

    /**
     * 还可以请求的数量
     */
    int size();

    /**
     * 已记录的已请求的数量
     */
    int optSize();

    /**
     * 最大的请求数量
     */
    int maxOptSize();

    /**
     * 更新同步信息
     */
    void sync();

    /**
     * 获取同步锁
     */
    Object getSyncMutex();
}
```
实现之后可以通过配置全局指定
`simple-auth.func.limit-plan={类的全限定名}`
或通过注解添加到具体的Controller