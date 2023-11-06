---
title: SimpleAuth拦截器
keywords: keyword1, keyword2
desc: 这是一款基于SpringBoot的轻量化的权限校验和访问控制的框架。适用于轻量级以及渐进式的项目。
date: 2023-09-10
class: heading_no_counter
---

## 配置SimpleAuth拦截器

```java
//继承SimpleAuthWebConfig，可以传入Class类或Bean名
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