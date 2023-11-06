---
title: SimpleAuth Interceptor
keywords: keyword1, keyword2
desc: This is a lightweight framework for permission validation and access control based on SpringBoot. It is suitable for lightweight and progressive projects.
date: 2023-09-10
class: heading_no_counter
---

## Configuring the SimpleAuth Interceptor

```java
// Inherit SimpleAuthWebConfig and can pass a Class or Bean name
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
```