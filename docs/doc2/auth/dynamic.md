---
title: Dynamic Permission Validation
keywords: keyword1, keyword2
desc: This is a lightweight framework for permission validation and access control based on Spring Boot. Suitable for lightweight and progressive projects.
date: 2023-09-10
---

## Enabling Dynamic Permission Validation
To enable dynamic permission validation, add the `@EnableDynamicAuthRequest` annotation to your Application's startup class.

```java
@SpringBootApplication
@EnableDynamicAuthRequest
public class SimpleAuthTestApplication {
    public static void main(String[] args) {
        SpringApplication.run(SimpleAuthTestApplication.class, args);
    }
}
```
Alternatively, you can configure it in your properties file.
```java
simple-auth.func.dynamic-auth=true
```
## Creating a Provider
```
@Component
public class MyAuthItemProvider implements RequestAuthItemProvider {
    @Override
    public List<RequestAuthItem> getRequestAuthItem() {
        List<RequestAuthItem> list = new ArrayList<>();
        // When accessing the '/say' path, it will be handled by MyHandler with the 'visitor' permission.
        // Note: The specific handling of the request permission is delegated to MyHandler; here, we are only specifying it.
        // Note: The path in RequestAuthItem can also accept a List<String> to represent multiple paths with the same handling.
        // Note: You can initialize RequestAuthItem from a database to dynamically adjust permission validation.
        list.add(new RequestAuthItem("/say", "visitor", MyHandler.class));
        list.add(new RequestAuthItem(MyHandlerChain.class, "/user/*", "vip"));
        return list;
    }
}

// Handler for the '/say' path
@Component
public class MyHandler extends AutoAuthHandler {
    @Override
    public boolean isAuthor(HttpServletRequest request, String permission) {
        // Check if the 'name' parameter is the same as the 'permission'. If they match, allow access.
        final String name = request.getParameter("name");
        return name.equals(permission);
    }
}

// HandlerChain for the '/user/*' path
@Component
public class MyHandlerChain extends AutoAuthHandlerChain {
    @Override
    public void addChain() {
        addLast(MyHandler1.class).addLast(MyHandler2.class);
    }
}

// Handler within MyHandlerChain
@Component
public class MyHandler1 extends AutoAuthHandler {
    @Override
    public boolean isAuthor(HttpServletRequest request, String permission) {
        // Query the database to add user permissions. For demonstration purposes, we are manually adding "visitor."
        ArrayList<String> permissions = new ArrayList<>();
        permissions.add("visitor");
        setPermissions(permissions);
        // Return true to allow access directly.
        return true;
    }
}

@Component
public class MyHandler2 extends AutoAuthHandler {
    @Override
    public boolean isAuthor(HttpServletRequest request, String permission) {
        final ArrayList<String> permissions = getPermissions();
        return !permissions.contains(permission);
    }
}
```