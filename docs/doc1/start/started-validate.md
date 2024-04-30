---
title: 快速上手-访问控制
keywords: keyword1, keyword2
desc: 这是一款基于SpringBoot的轻量化的权限校验和访问控制的框架。适用于轻量级以及渐进式的项目。
date: 2023-09-10
class: heading_no_counter
---
本节中将会介绍如何通过注解校验参数，首个案例为验证用户昵称与年龄。


## 第一步：添加Maven依赖

```xml
<dependency>
    <groupId>io.github.liuye744</groupId>
    <artifactId>simpleAuth-spring-boot-starter</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
```

## 第二步：创建验证类与验证方法
方法返回值必须为Boolean，有且只有一个需要校验的对象作为参数。
```java
//用到的实例对象
public class User {
    String name;
    Integer age;
    String phone;
    //省略getter setter等
}

//要求验证年龄(age)在1-99，昵称(name)的长度为5-16
public class MyValidateObj {
    public Boolean fillUser(User user){
        final Integer age = user.getAge();
        if (age <= 1 || age >= 99){
            return false;
        }
        final int nameLength = user.getName().length();
        if (nameLength <= 5 || nameLength >= 16){
            return false;
        }
        return true;
    }
}
```

## 第三步：为Controller添加注解
在Controller中的单个函数上添加。验证失败则会抛出`ValidateException`。

```java
//Controller对象，其中methods为MyValidateObj中的函数名
@RestController
public class MyController {
    @GetMapping("/say")
    @SimpleValidate(value = MyValidateObj.class, methods = {"fillUser"})
    public String say(User user){
        System.out.println("Controller "+user);
        return user.getName();
    }
}
```

## 其他案例
### 用例1：同一个Controller中多个函数进行相同的校验
`/say`需要验证name和age字段
`/eat`仅需要验证phone字段是否为合法手机号

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
//验证类
public class MyValidateObj {
    public Boolean fillUser(User user){
        final Integer age = user.getAge();
        if (age <= 1 || age >=99){
            return false;
        }
        final int nameLength = user.getName().length();
        if (nameLength <= 5 || nameLength >= 16){
            return false;
        }
        return true;
    }

    public Boolean partUser(User user){
        final String phone = user.getPhone();
        //判断是否为空，为空则验证失败返回false
        if (phone == null){
            return false;
        }
        //正则校验手机号码是否合法
        Pattern pattern = Pattern.compile("^(13[0-9]|14[01456879]|15[0-35-9]|16[2567]|17[0-8]|18[0-9]|19[0-35-9])\\d{8}$");
        return pattern.matcher(phone).matches();
    }
}
```
### 用例2：运用工具类简化校验过程
此部分实现的功能与用例1相同
```java
public class MyValidateObj {
    public Boolean fillUser(User user){
        //notFalse函数中传入多个boolean值，只要存在false则返回false
        //lengthRange函数是验证字符串长度，是否在5-16之间(包含5和16)
        //range函数验证该数字是否在1-99之间(包含1和99)
        return SVU.notFalse(
            SVU.lengthRange( user.getName(), 5, 16),
            SVU.range( user.getAge(), 1, 99)
        );
    }

    public Boolean partUser(User user){
        //正则校验phone符合正则表达式
        //Regex类提供了大量的常用正则表达式字符串常量
        return SVU.pattern(user.getPhone(), Regex.CHINESE_MOBIL_PHONE_NUMBER);
    }
}
```

### 用例3：校验失败时抛出不同的异常
此部分实现的功能与上一例相同
```java
//校验失败之后会抛出ValidateException,输入的提示信息会携带在异常对象的message中
public class MyValidateObj {
    public Boolean fillUser(User user){
        return SVU.notFalse(
            SVU.lengthRange("昵称长度需要在5-16之间", user.getName(), 5, 16),
            SVU.range("年龄需要在1-99之间", user.getAge(), 1, 99)
        );
    }

    public Boolean partUser(User user){
        return SVU.pattern("手机号格式不合法", user.getPhone(), Regex.CHINESE_MOBIL_PHONE_NUMBER);
    }
}
//建议在Advice中捕获ValidateException并处理
@ControllerAdvice
public class MyExceptionHandler {
    @ExceptionHandler(value = ValidateException.class)
    @ResponseBody
    public String exceptionHandler(ValidateException e){
        return e.getMessage();
    }
}
```
如上的代码当访问`http://localhost:8080/say?name=CodingCube`时会返回`年龄需要在1-99之间`