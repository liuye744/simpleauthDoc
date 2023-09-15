---
title: SimpleAuth Interceptor
keywords: keyword1, keyword2
desc: This is a lightweight framework for permission validation and access control based on Spring Boot. Suitable for lightweight and progressive projects.
date: 2023-09-10
---

## Configuring the SimpleAuth Interceptor

```java
// Extend SimpleAuthWebConfig and provide a Class or Bean name
@Component
public class MySimpleAuthConfig extends SimpleAuthWebConfig {
    @Override
    public void addAuthHandlers() {
        addAuthHandler(MyHandler.class).addPathPatterns("/say");
        addAuthHandlerChain(MyHandlerChain.class)
                .addPathPatterns("/user/*")
                .excludePathPatterns("/user/vip/*");
    }
}
