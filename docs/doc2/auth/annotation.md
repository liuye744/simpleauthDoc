---
title: Annotations
keywords: keyword1, keyword2
desc: This is a lightweight framework for permission validation and access control based on SpringBoot. It is suitable for lightweight and progressive projects.
date: 2023-09-10
class: heading_no_counter
---

## Annotation Parameters
```java
public @interface SimpleAuth {
    // Interface permission name
    String value() default "";
    // AuthHandler's Class, by default, scans whether there is an interface corresponding to 'value' in 'permissions'
    Class<? extends AutoAuthHandler>[] handler() default DefaultAuthHandler.class;
    // AuthHandlerChain's Class
    Class<? extends AutoAuthHandlerChain>[] handlerChain() default DefaultAuthHandlerChain.class;
    // Rejection strategy, the default is DefaultAuthRejectedStratagem, which throws a PermissionsException(“lack of permissions”) exception 
    Class<? extends AuthRejectedStratagem> rejected() default NullTarget.class;
}
```

## Handler
You need to extend AutoAuthHandler, and if you use other SpringBeans, you can register them as Beans (recommended to add the @Component annotation). If the validation fails, it will throw a `PermissionsException` exception.
```java
public class MyAutoAuthHandler extends MyAutoAuthHandler {

    /**
     * Check if the conditions are met
     * @param permission The required permission name
     * @return Return true to allow (pass)
     */
    @Override
    public boolean isAuthor(HttpServletRequest request, String permission){
        // Return true for validation pass, false for failure
        return true;
    }

}
If you do not use @Component, the first call to the Handler will create a new instance with a parameterless constructor and cache it. Subsequent calls will query the corresponding Handler object in the cache. You can use the `simple-auth.func.handler-cache=true` configuration to disable the cache.
```

## HandlerChain
You need to extend AutoAuthHandlerChain, and in the `addChain` function, add Handlers to combine multiple Handlers together for modularized calling. Similarly, you can choose to register them as Beans.
```java
public abstract class AutoAuthHandlerChain {
    public abstract void addChain();
}
```

Commonly used functions
`addLast(Class<? extends AutoAuthHandler> autoAuthHandler)`: Add the Handler to the end, the parameter can also be the Bean name of the Handler.
`addFirst(Class<? extends AutoAuthHandler> auto)`: Add the Handler to the beginning.

## AuthRejectedStratagem
By default, it throws a `PermissionsException("lack of permissions")` exception. It is generally not recommended to modify this.
The default rejection strategy is as follows:
```java
public class DefaultAuthRejectedStratagem implements AuthRejectedStratagem {
    @Override
    public void doRejected(HttpServletRequest request, HttpServletResponse response, LogAuthFormat authFormat) {
        throw new PermissionsException("lack of permissions");
    }
}
```
You can create your own class that extends the `AuthRejectedStratagem` interface and override the `doRejected` method. To use it, you can configure it through annotations:
```java
// Newly created rejection strategy
public class MyAuthRejectedStratagem implements AuthRejectedStratagem {
    @Override
    public void doRejected(HttpServletRequest request, HttpServletResponse response, LogAuthFormat authFormat) {
        throw new PermissionsException("this is my AuthRejectedStratagem");
    }
}
// Controller
@GetMapping("/checkNameWhetherCodingCube2")
@SimpleAuth(handler = MyHandler.class, rejected = MyAuthRejectedStratagem.class)
public String checkNameWhetherCodingCube(String name){
    return "Hello World";
}
```