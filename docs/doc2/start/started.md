---
title: Quick Start - Permission Validation
keywords: keyword1, keyword2
desc: This is a lightweight framework for permission validation and access control based on Spring Boot. Suitable for lightweight and progressive projects.
date: 2023-09-10
---

In this section, we will learn how to perform permission validation using annotations.

## Step 1: Add Maven Dependency
```xml
<dependency>
    <groupId>io.github.liuye744</groupId>
    <artifactId>simpleAuth-spring-boot-starter</artifactId>
    <version>1.0.0.RELEASE</version>
</dependency>
```
## Step 2: Add Annotations

### Use Case 1: Validating Parameters through Request

Create a class that extends `AutoAuthHandler` and override the `isAuthor` method.

```java
public class KeyAutoAuthHandler extends AutoAuthHandler {
   @Override
   public boolean isAuthor(HttpServletRequest request, String permission) {
        // Verify if the request parameters contain a key parameter with the value "114514".
        // You can perform more complex operations as needed.
       final String key = request.getParameter("key");
       // Return true if the validation is successful, or false to throw a PermissionsException for validation failure.
       if ("114514".equals(key)){
           return true;
       }
       return false;
   }
}
```
Then, add the @IsAuthor annotation to the Controller class or its methods.
```java
@Controller
@IsAuthor(handler = KeyAutoAuthHandler.class)
public class MyController {
}
```
Note: If you have multiple AutoAuthHandler instances, you can use the annotation as follows:
```
@IsAuth(handler = { KeyAutoAuthHandler1.class, KeyAutoAuthHandler2.class })
```
The handler parameter can also specify the bean names of the handlers. These classes will execute permission checks in the specified order. Alternatively, you can create a class that extends AutoAuthHandlerChain and add all handlers to that class.
```java
public class MyHandlerChain extends AutoAuthHandlerChain {
   @Override
   public void addChain() {
       this.addLast(KeyAutoAuthHandler1.class)
        .addLast(KeyAutoAuthHandler2.class);
   }
}
// When adding the annotation, use @IsAuthor(handlerChain = MyHandlerChain.class)
```
### Use Case 2: Role-Based Permission Validation

```java
@RestController
// Add the annotation to the class
@IsAuthor(authentication = AddPermissionKeyHandler.class)
public class MyController {
   @IsAuthor("visitor")
   @GetMapping("say")
   public String say(){
       return "Hello World";
   }
   
   @IsAuthor("vip")
   @GetMapping("eat")
   public String eat(){
       return "eat";
   }
}

public class AddPermissionKeyHandler extends AutoAuthHandler {
   @Override
   public boolean isAuthor(HttpServletRequest request, String permission) {
       ArrayList<String> permissions = new ArrayList<>();
       // Alternatively, you can query a database to add the role key for the current request.
       permissions.add("visitor");
       this.setPermissions(request, permissions);
       // Verification successful, allow access.
       return true;
   }
}
```
When a request is made to /say, the AddPermissionKeyHandler class is executed first due to the @IsAuthor annotation added to the MyController class. In this method, the string "visitor" is added to the user's permissions, allowing the request to pass and access the endpoint.

When a request is made to /eat, the request fails because the permission list does not contain "vip," resulting in a PermissionsException exception. You can handle permission validation through global exception handling.
### Use Case 3: Passing Instance Objects

```java
// Class for the instance used
public class User {
    String name;
    public User(String name) {this.name = name;}
    public String getName() {return name;}
}

// Controller
@RestController
public class MyController {
    @IsAuthor(handler = {MyFirstHandler.class, MySecondHandler.class})
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
        // Allow access
        return true;
    }
}

// Second Handler
public class MySecondHandler extends AutoAuthHandler {
    @Override
    public boolean isAuthor(HttpServletRequest request, String permission) {
        // Retrieve the instance object and verify if the name is "CodingCube"
        final User user = getPrincipal();
        return "CodingCube".equals(user.getName());
    }
}
```
When accessing http://localhost:8080/say?name=CodingCube, the request is allowed to pass. If the name parameter is set to something other than "CodingCube," a PermissionsException exception is thrown.