---
title: 快速上手-权限校验
keywords: keyword1, keyword2
desc: 这是一款基于SpringBoot的轻量化的权限校验和访问控制的框架。适用于轻量级以及渐进式的项目。
date: 2023-09-10
---
本节中将会介绍如何通过注解进行权限校验

## 注解
```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
public @interface SimpleValidate {
    //用于校验的类，类中用于校验的方法返回值必须为Boolean
    Class<?> value() default Object.class;
    //用于校验的方法名
    String[] methods() default {"validate"};
    //验证失败后的拒绝策略，默认策略抛出ValidateException
    Class<? extends ValidateRejectedStratagem> rejected() default DefaultValidateRejectedStratagem.class;
}
```
## 注解用法