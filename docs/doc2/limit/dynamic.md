---
title: Dynamic Access Control

keywords: keyword1, keyword2

desc: This is a lightweight framework for permission validation and access control based on Spring Boot. Suitable for lightweight and progressive projects.

date: 2023-09-10

---

## Enabling Dynamic Access Control
To enable dynamic access control, use the `@EnableDynamicLimit` annotation:

```java
@SpringBootApplication
@EnableDynamicLimit
public class SimpleAuthApplication {
    public static void main(String[] args) {
        SpringApplication.run(SimpleAuthApplication.class, args);
    }
}
```
Alternatively, you can enable it using configuration:
```java
simple-auth.func.dynamic-limit=true
```

## Creating a Provider

```java
@Component
public class MyLimitItemProvider implements RequestLimitItemProvider {
    @Override
    public List<RequestLimitItem> getRequestLimitItem() {
        List<RequestLimitItem> list = new ArrayList<>();
        // Matching path '/say', allowed 2 requests within 60 seconds, and a ban for 10 seconds if exceeded.
        list.add(new RequestLimitItem("/say", 2, 60, 10));
        return list;
    }
}
```
The parameters for RequestLimitItem are as follows:

`List<String> path`: The matching paths.

`Integer times`: Maximum number of requests.

`Integer seconds`: Recording time.

`Integer ban`: Ban time.

`Class<? extends SignStrategic> itemStrategic`: Item generation strategy.

`Class<? extends SignStrategic> signStrategic`: User identifier generation strategy.

`Class<? extends EffectiveStrategic> effectiveStrategic`: Strategy to determine whether the request is recorded.