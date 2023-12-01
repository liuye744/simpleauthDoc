---
title: Quick Start - Permission Validation
keywords: keyword1, keyword2
desc: This is a lightweight framework for permission validation and access control based on SpringBoot. It is suitable for lightweight and progressive projects.
date: 2023-09-10
class: heading_no_counter
---

In this section, we will explain how to perform permission validation using annotations.

## Step 1: Add Maven Dependency
```xml
<dependency>
    <groupId>io.github.liuye744</groupId>
    <artifactId>simpleAuth-spring-boot-starter</artifactId>
    <version>1.4.3.RELEASE</version>
</dependency>
```

## Step 2: Add Annotations

### Use Case 1: Validating Parameters through Request

Create a class that extends `AutoAuthHandler` and override the `SimpleAuthor` function.

```java
public class KeyAutoAuthHandler extends AutoAuthHandler {
   @Override
   public boolean SimpleAuthor(HttpServletRequest request, String permission) {
        // Validate whether the request parameters include a key with the value "114514".
       final String key = request.getParameter("key");
       // Return true if the validation succeeds, and false if it fails, which will throw a PermissionsException.
       if ("114514".equals(key)){
           return true;
       }
       return false;
   }
}
```

Then, add the `@SimpleAuthor` annotation to your Controller or its methods.

```java
@Controller
@SimpleAuthor(handler = KeyAutoAuthHandler.class)
public class MyController {
}
```

Note: If you have multiple `AutoAuthHandler`s, you can use the annotation as follows:

```java
@SimpleAuthor(handler = {KeyAutoAuthHandler1.class, KeyAutoAuthHandler2.class})
```

You can also create a class that extends `AutoAuthHandlerChain` and add all the handlers to it.

```java
public class MyHandlerChain extends AutoAuthHandlerChain {
   @Override
   public void addChain() {
       this.addLast(KeyAutoAuthHandler1.class)
        .addLast(KeyAutoAuthHandler2.class);
   }
}
```

And then, add the annotation as `@SimpleAuthor(handlerChain = MyHandlerChain.class)`.

### Use Case 2: Role-Based Permission Validation

```java
@RestController
// Add the annotation to the class
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
       // Or query the database to add the role key for the current request
       permissions.add("visitor");
       this.setPermissions(request,permissions);
       // Verification successful, allow access
       return true;
   }
}
```

When a request is made to `/say`, the `SimpleAuthor` function in `AddPermissionKeyHandler` will run first because the `@IsAuthor` annotation is added to the `MyController` class. In this function, the string "visitor" is added to the user's permissions, allowing access.

When a request is made to `/eat`, it will fail because "vip" is not in the list of permissions, and a `PermissionsException` will be thrown. You can handle permission validation using a global exception handler.

### Use Case 3: Passing Instance Objects

```java
// User instance
public class User {
    String name;
    public User(String name) {this.name = name;}
    public String getName() {return name;}
}
// Controller
@RestController
public class MyController {
    @SimpleAuthor(handler = {MyFirstHandler.class, MySecondHandler.class})
    @GetMapping("/say")
    public String say(){
        return "Hello World";
    }
}

// First Handler
public class MyFirstHandler extends AutoAuthHandler {
    @Override
    public boolean SimpleAuthor(HttpServletRequest request, String permission) {
        final String name = request.getParameter("name");
        final User user = new User(name);
        // Pass the instance object
        setPrincipal(user);
        // Allow access
        return true;
    }
}

// Second Handler
public class MySecondHandler extends AutoAuthHandler {
    @Override
    public boolean SimpleAuthor(HttpServletRequest request, String permission) {
        // Get the instance object and verify if the name is "CodingCube"
        final User user = getPrincipal();
        return "CodingCube".equals(user.getName());
    }
}
```

When accessing `http://localhost:8080/say?name=CodingCube`, the request is allowed to pass. If the `name` parameter is different, a `PermissionsException` will be thrown.