---
title: 动态访问控制
keywords: keyword1, keyword2
desc: 这是一款基于SpringBoot的轻量化的权限校验和访问控制的框架。适用于轻量级以及渐进式的项目。
date: 2023-09-10
---


## 开启动态访问控制
使用`@EnableDynamicLimit`注解开启动态访问控制
```java
@SpringBootApplication
@EnableDynamicLimit
public class SimpleAuthApplication {
    public static void main(String[] args) {
        SpringApplication.run(SimpleAuthApplication.class, args);
    }
}
```
或使用`simple-auth.func.dynamic-limit=true`配置开启

## 创建Provider

```java
@Component
public class MyLimitItemProvider implements RequestLimitItemProvider {
    @Override
    public List<RequestLimitItem> getRequestLimitItem() {
        List<RequestLimitItem> list = new ArrayList<>();
        //匹配路径`/say`, 60秒只能访问2次，超过则被禁止10s
        list.add(new RequestLimitItem("/say",2,60,10));
        return list;
    }
}
```
RequestLimitItem 的参数如下
`List<String> path`:：匹配的路径
`Integer times`：最大请求次数
`Integer seconds`：记录时间
`Integer ban`：禁止时间
`Class<? extends SignStrategic> itemStrategic`：Item的生成策略
`Class<? extends SignStrategic> signStrategic`：用户标识的生成策略
`Class<? extends EffectiveStrategic> effectiveStrategic`：判断请求是否被记录的策略

