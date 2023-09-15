---
title: Annotations
keywords: keyword1, keyword2
desc: This is a lightweight framework for permission validation and access control based on SpringBoot. It is suitable for lightweight and progressive projects.
date: 2023-09-10
---

## Annotation Parameters
```java
public @interface IsAuthor {
    // Interface permission name
    String value() default "";
    // Class of the authentication handler, default to scan if the value exists in permissions
    Class<? extends AutoAuthHandler>[] handler() default DefaultAuthHandler.class;
    // Class of the authentication handler chain
    Class<? extends AutoAuthHandlerChain>[] handlerChain() default DefaultAuthHandlerChain.class;
}
```
## Handler
You need to extend AutoAuthHandler, and if you are using other Spring beans, you can register them as a @Component (recommended to add the @Component annotation). If the validation fails, it will throw a PermissionsException exception.
```java
public class MyAutoAuthHandler extends AutoAuthHandler {

    /**
     * Check if the condition is met
     * @param permission Required permission name
     * @return Return true if it meets the condition (pass)
     */
    @Override
    public boolean isAuthor(HttpServletRequest request, String permission){
        // Return true to pass if validation succeeds, otherwise return false
        return true;
    }

}
```
If you do not use @Component, the first call to the handler will create a new instance with a parameterless constructor and cache it. Subsequent calls will retrieve the corresponding handler object from the cache. You can use simple-auth.func.handler-cache=true to disable the cache.

## HandlerChain
You need to extend AutoAuthHandlerChain and add handlers in the addChain method to combine multiple handlers for modularized calling. Similarly, you can choose to register it as a bean.
```java
public abstract class AutoAuthHandlerChain {
    public abstract void addChain();
}
```

Common methods:
`addLast(Class<? extends AutoAuthHandler> autoAuthHandler)`: Add a handler to the end, the parameter can also be the name of the handler bean.

`addFirst(Class<? extends AutoAuthHandler> auto)`: Add a handler to the beginning.