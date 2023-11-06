---
title: Dynamic Access Control
keywords: keyword1, keyword2
desc: This is a lightweight framework for permission validation and access control based on SpringBoot. It is suitable for lightweight and progressive projects.
date: 2023-09-10
class: heading_no_counter
---

## Enabling Dynamic Access Control
Use the `@EnableDynamicLimit` annotation to enable dynamic access control.

```java
@SpringBootApplication
@EnableDynamicLimit
public class SimpleAuthApplication {
    public static void main(String[] args) {
        SpringApplication.run(SimpleAuthApplication.class, args);
    }
}
```

Alternatively, you can enable it with the `simple-auth.func.dynamic-limit=true` configuration.

## Creating a Provider
```java
@Component
public class MyLimitItemProvider implements RequestLimitItemProvider {
    @Override
    public List<RequestLimitItem> getRequestLimitItem() {
        List<RequestLimitItem> list = new ArrayList<>();
        // Match the path '/say', allow only 2 requests within 60 seconds, and ban for 10 seconds if exceeded.
        list.add(new RequestLimitItem("/say", 2, 60, 10));
        return list;
    }
}
```
The parameters for `RequestLimitItem` are as follows:
- `List<String> path`: The matched path.
- `Integer times`: The maximum number of requests.
- `Integer seconds`: The recording time.
- `Integer ban`: The ban time.
- `Class<? extends SignStrategic> itemStrategic`: The item generation strategy, default is uri.
- `Class<? extends SignStrategic> signStrategic`: The user identification generation strategy, default is user IP.
- `Class<? extends EffectiveStrategic> effectiveStrategic`: The strategy to determine if the request is recorded, default is true.
- `Class<? extends TokenLimit> tokenLimit`: The rate limiting algorithm, default is CompleteLimit, which records all requests accurately.