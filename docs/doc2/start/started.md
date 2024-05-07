---
title: Quick Start - Permission Verification
keywords: keyword1, keyword2
desc: This is a lightweight permission verification and access control framework based on SpringBoot. It is suitable for lightweight and progressive projects.
date: 2023-09-10
class: heading_no_counter
---
In this section, we will introduce how to perform permission verification through annotations. The first case is to verify whether the request parameters contain a parameter with the key '114514'.

## Step 1: Add Maven Dependency
```xml
<dependency>
    <groupId>io.github.liuye744</groupId>
    <artifactId>simpleAuth-spring-boot-starter</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
```
## Step 2: Create Handler
Create a class that extends `AutoAuthHandler` and override the `isAuthor` function.

```java
public class KeyAutoAuthHandler extends AutoAuthHandler {
   @Override
   public boolean isAuthor(HttpServletRequest request, String permission) {
        // Verify if the request parameter carries a parameter with the key '114514'.
        // More complex operations can also be performed here.
       final String key = request.getParameter("key");
       // Return true if the verification is successful, false if the verification fails, which will throw a PermissionsException.
       return "114514".equals(key);
   }
}
```
## Step 3: Add Annotations
Next, add the `@SimpleAuth` annotation to the Controller or its functions. If added to a class, the Handler will be executed before all methods in the class.
```java
@Controller
@SimpleAuth(handler = KeyAutoAuthHandler.class)
public class MyController {
}
```
If added to a method, the Handler will be executed before the method.
```java
@RestController
public class MyController {

    @SimpleAuth(handler = AddPermissionKeyHandler.class)
    @GetMapping("say")
    public String say(){
        return "Hello World";
    }
}
```
Note: If you have multiple `AutoAuthHandler`, you can write the annotation like this:
```java
@SimpleAuth(handler = { KeyAutoAuthHandler1.class, KeyAutoAuthHandler2.class })
```
The handler parameter can also be the Bean name of the Handler. These classes will perform permission checks in sequence. Alternatively, create a class that inherits `AutoAuthHandlerChain` and add all Handlers to this class.
```java
public class MyHandlerChain extends AutoAuthHandlerChain {
   @Override
   public void addChain() {
       this.addLast(KeyAutoAuthHandler1.class)
        .addLast(KeyAutoAuthHandler2.class);
   }
}
// Use @SimpleAuth(handlerChain = MyHandlerChain.class) when adding the annotation.
```

## Other Cases
### Use Case 1: Role-Based Permission Verification
```java
@RestController
// Add annotation to the class
@SimpleAuth(authentication = AddPermissionKeyHandler.class)
public class MyController {
   @SimpleAuth("visitor")
   @GetMapping("say")
   public String say(){
       return "Hello World";
   }
   
   @SimpleAuth("vip")
   @GetMapping("eat")
   public String eat(){
       return "eat";
   }
}
public class AddPermissionKeyHandler extends AutoAuthHandler {
   @Override
   public boolean isAuthor(HttpServletRequest request, String permission) {
       ArrayList<String> permissions = new ArrayList<>();
       // Or query the database to add a role key for the current request
       permissions.add("visitor");
       this.setPermissions(request,permissions);
       // If the query is successful, allow it to pass
       return true;
   }
}
```
When requesting `/say`, since the `@SimpleAuth` annotation is added to the MyController class, the `SimpleAuth` function in `AddPermisonKeyHandler` will run first. In this function, the string `visitor` is added to the user’s permissions, so it will pass the verification and access normally. When requesting `/eat`, since “vip” is not in the permission list, the request will fail, throwing a `PermissionsException` exception, which can be handled by global exception handling to complete the permission verification.
### Use Case 2: Passing Instance Objects
```java
// Instance used
public class User {
    String name;
    public User(String name) {this.name = name;}
    public String getName() {return name;}
}
// Interface
@RestController
public class MyController {
    @SimpleAuth(handler = {MyFirstHandler.class, MySecondHandler.class})
    @GetMapping("/say")
    public String say(){
        return "Hello World";
    }
}

// First Handler
public class MyFirstHandler extends AutoAuthHandler {
    @Override
    public boolean isAuthor(HttpServletRequest request, String permission) {
        final String name = request.getParameter("name");
        final User user = new User(name);
        // Pass the instance object
        setPrincipal(user);
        // Allow to pass
        return true;
    }
}
// Second Handler
public class MySecondHandler extends AutoAuthHandler {
    @Override
    public boolean is(HttpServletRequest request, String permission) {
        // Get the instance object and verify if name equals CodingCube
        final User user = getPrincipal();
        return "CodingCube".equals(user.getName());
    }
}
```
When accessing `http://localhost:8080/say?name=CodingCube`, it will pass. If the parameter name is other than CodingCube, a `PermissionsException` exception will be thrown.