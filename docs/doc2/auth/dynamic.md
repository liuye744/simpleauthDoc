---
title: Dynamic Permission Validation
keywords: keyword1, keyword2
desc: This is a lightweight framework for permission validation and access control based on SpringBoot. It is suitable for lightweight and progressive projects.
date: 2023-09-10
class: heading_no_counter
---

## Enabling Dynamic Permission Validation
Add the `@EnableDynamicAuthRequest` annotation to your Application's startup class.

```java
@SpringBootApplication
@EnableDynamicAuthRequest
public class SimpleAuthTestApplication {
    public static void main(String[] args) {
        SpringApplication.run(SimpleAuthTestApplication.class, args);
    }
}
```

Alternatively, you can configure it in your application.properties file:

```properties
simple-auth.func.dynamic-auth=true
```

## Creating a Provider
```java
@Component
public class MyAuthItemProvider implements RequestAuthItemProvider {
    @Override
    public List<RequestAuthItem> getRequestAuthItem() {
        List<RequestAuthItem> list = new ArrayList<>();
        // When accessing the '/say' path, use MyHandler for handling, and carry the request permission 'visitor'.
        // Note: The actual request permission handling is done by MyHandler; this is just for carrying the permission.
        // Note: You can also pass a List of Strings as paths if the handling is the same for multiple paths.
        // Note: You can initialize RequestAuthItem through the database to dynamically adjust permission validation.
        list.add(new RequestAuthItem("/say", "visitor", MyHandler.class));
        list.add(new RequestAuthItem(MyHandlerChain.class, "/user/*", "vip"));
        return list;
    }
}
// Handler for the '/say' path
@Component
public class MyHandler extends AutoAuthHandler {
    @Override
    public boolean SimpleAuthor(HttpServletRequest request, String permission) {
        // Check if the 'name' parameter is the same as the permission. If it's the same, allow access.
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
// Handlers in MyHandlerChain
@Component
public class MyHandler1 extends AutoAuthHandler {
    @Override
    public boolean SimpleAuthor(HttpServletRequest request, String permission) {
        // Query the database to add user permissions. For demonstration purposes, we manually add 'visitor'.
        ArrayList<String> permissions = new ArrayList<>();
        permissions.add("visitor");
        setPermissions(permissions);
        // Return true to allow access
        return true;
    }
}
@Component
public class MyHandler2 extends AutoAuthHandler {
    @Override
    public boolean SimpleAuthor(HttpServletRequest request, String permission) {
        final ArrayList<String> permissions = getPermissions();
        return !permissions.contains(permission);
    }
}
```