---
title: 快速上手-权限校验
keywords: keyword1, keyword2
desc: 这是一款基于SpringBoot的轻量化的权限校验和访问控制的框架。适用于轻量级以及渐进式的项目。
date: 2023-09-10
---
本节中将会介绍如何通过注解进行权限校验

## 第一步：添加maven依赖
```xml
<dependency>
    <groupId>io.github.liuye744</groupId>
    <artifactId>simpleAuth-spring-boot-starter</artifactId>
    <version>1.4.0.RELEASE</version>
</dependency>
```

## 第二步：添加注解

### 用例1：通过request验证参数

创建一个类继承AutoAuthHandler，并重写SimpleAuthor方法

```java
public class KeyAutoAuthHandler extends AutoAuthHandler {
   @Override
   public boolean SimpleAuthor(HttpServletRequest request, String permission) {
        //验证请求参数中是否携带key为114514的参数. 
        //当然也可以进行更复杂的操作
       final String key = request.getParameter("key");
       //返回true则表示验证成功，返回false表示验证失败将会抛出PermissionsException
       if ("114514".equals(key)){
           return true;
       }
       return false;
   }
}
```

然后将`@SimpleAuthor `注释添加到 Controller 或其中的方法。

```java
@Controller
@SimpleAuthor(handler = KeyAutoAuthHandler.class)
public class MyController {
}
```

注: 如果你有多个` AutoAuthHandler`，你可以像这样写注释:
```java
@SimpleAuth (handler = { KeyAutoAuthHandler1.class，KeyAutoAuthHandler2.class })
```
handler参数也可以写 Handler 的Bean 名称。这些类将按顺序执行权限检查。或者创建 继承`AutoAuthHandlerChain` 的类，并将所有 Handler 添加到该类中。

```java
public class MyHandlerChain extends AutoAuthHandlerChain {
   @Override
   public void addChain() {
       this.addLast(KeyAutoAuthHandler1.class)
        .addLast(KeyAutoAuthHandler2.class);
   }
}
//添加注解时使用 @SimpleAuthor(handlerChain = MyHandlerChain.class)
```

### 用例2：基于角色的权限校验

```java
@RestController
//添加注解到类
@SimpleAuthor(authentication = AddPermissionKeyHandler.class)
public class MyController {
   @SimpleAuthor("visitor")
   @GetMapping("say")
   public String say(){
       return "Hello World";
   }
   
   @SimpleAuthor("vip")
   @GetMapping("eat")
   public String eat(){
       return "eat";
   }
}
public class AddPermissionKeyHandler extends AutoAuthHandler {
   @Override
   public boolean SimpleAuthor(HttpServletRequest request, String permission) {
       ArrayList<String> permissions = new ArrayList<>();
       //或者查询数据库为当前请求添加角色key
       permissions.add("visitor");
       this.setPermissions(request,permissions);
       //查询成功，放行
       return true;
   }
}
```
当请求`/say`时，由于注释`@IsAutor` 被添加到 MyController类上，`AddPermisonKeyHandler` 中的 `SimpleAuthor` 方法将首先运行。在这个方法中，字符串`visitor`被添加到用户的权限中，因此它将会验证通过并正常访问。
当请求`/eat`时，由于权限列表中没有“vip”则会请求失败，抛出`PermissionsException`异常，可以通过全局异常处理完成权限校验

### 用例3：传递实例对象
```java
//用到的实例
public class User {
    String name;
    public User(String name) {this.name = name;}
    public String getName() {return name;}
}
//接口
@RestController
public class MyController {
    @SimpleAuthor(handler = {MyFirstHandler.class, MySecondHandler.class})
    @GetMapping("/say")
    public String say(){
        return "Hello World";
    }
}

//第一个Handler
public class MyFirstHandler extends AutoAuthHandler {
    @Override
    public boolean SimpleAuthor(HttpServletRequest request, String permission) {
        final String name = request.getParameter("name");
        final User user = new User(name);
        //传递实例对象
        setPrincipal(user);
        //放行
        return true;
    }
}
//第二个Handler
public class MySecondHandler extends AutoAuthHandler {
    @Override
    public boolean SimpleAuthor(HttpServletRequest request, String permission) {
        //获取实例对象,并验证name是否等于CodingCube
        final User user = getPrincipal();
        return "CodingCube".equals(user.getName());
    }
}
```
当访问 `http://localhost:8080/say?name=CodingCube` 时可以通过，参数name为其他时抛出`PermissionsException`异常