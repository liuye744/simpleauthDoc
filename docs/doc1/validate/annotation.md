---
title: 快速上手-权限校验
keywords: keyword1, keyword2
desc: 这是一款基于SpringBoot的轻量化的权限校验和访问控制的框架。适用于轻量级以及渐进式的项目。
date: 2023-09-10
class: heading_no_counter
---
本节中将会介绍如何通过注解进行权限校验

## 注解
```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
public @interface SimpleValidate {
    //用于校验的类，类中用于校验的函数返回值必须为Boolean
    Class<?> value() default Object.class;
    //用于校验的函数名
    String[] methods() default {"validate"};
    //验证失败后的拒绝策略，默认策略抛出ValidateException
    Class<? extends ValidateRejectedStratagem> rejected() default DefaultValidateRejectedStratagem.class;
}
```
## 注解用法
value参数传入用于校验的类。
类中用于校验的函数返回值必须为`Boolean`，参数为需要校验的类。
例：`public Boolean fillUser(User user);`

```java
public class MyValidateObj {
    //验证用户年龄为1-99
    public Boolean fillUser(User user){
        final Integer age = user.getAge();
        if (age <= 1 || age >= 99){
            return false;
        }
        return true;
    }
}
```
在Controller中添加注解
```java
@RestController
public class MyController {
    @GetMapping("/say")
    @SimpleValidate(value = MyValidateObj.class, methods = {"fillUser"})
    public String say(User user){
        return user.getName();
    }
}
```
若多个函数需要用到同一个校验类可将校验类写在类注解中。当类的注解上指定了验证类
```java
@RestController
@SimpleValidate(value = MyValidateObj.class)
public class MyController {

    @GetMapping("/say")
    @SimpleValidate(methods = {"fillUser"})
    public String say(User user){
        System.out.println("say");
        return user.getName();
    }

    @GetMapping("/eat")
    @SimpleValidate(methods = {"partUser"})
    public String eat(User user){
        System.out.println("eat");
        return user.getName();
    }
}
```
## 注解用法2
在Controller中添加注解(指定的校验类可在函数或类上添加)
```java
@RestController
@SimpleValidate(MyValidateObj.class)
public class MyController {
    @GetMapping("/say")
    public String say(@SimpleValidate User user){
        return user.getName();
    }
}
```
当访问`/say`时，会执行MyValidateObj类的所有带有User参数的校验函数

## 拒绝策略
默认的拒绝策略为抛出`ValidateException`
```java
public class MyRejected implements ValidateRejectedStratagem {
    @Override
    public void doRejected(HttpServletRequest request, HttpServletResponse response, Object validationObj) {
    }
}
```
当验证类返回`false`时运行doRejected函数，
`validationObj`为验证的对象，如当验证User对象验证失败返回false后，doRejected中的validationObj为刚刚验证的User。

