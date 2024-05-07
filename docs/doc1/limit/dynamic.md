---
title: 动态访问控制
keywords: keyword1, keyword2
desc: 这是一款基于SpringBoot的轻量化的权限校验和访问控制的框架。适用于轻量级以及渐进式的项目。
date: 2023-09-10
class: heading_no_counter
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
或者在配置文件中配置
```properties
simple-auth.func.dynamic-limit=true
```

## 创建LimitItemProvider

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
`Class<? extends SignStrategic> itemStrategic`：Item的生成策略，默认为uri
`Class<? extends SignStrategic> signStrategic`：用户标识的生成策略，默认为用户IP
`Class<? extends EffectiveStrategic> effectiveStrategic`：判断请求是否被记录的策略，默认为true
`Class<? extends TokenLimit> tokenLimit`：限流算法，默认为CompleteLimit，即所有的请求都准确记录

