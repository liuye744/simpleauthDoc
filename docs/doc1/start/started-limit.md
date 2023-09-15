---
title: 快速上手-访问控制
keywords: keyword1, keyword2
desc: 这是一款基于SpringBoot的轻量化的权限校验和访问控制的框架。适用于轻量级以及渐进式的项目。
date: 2023-09-10
---
本节中将会介绍如何通过注解限制接口访问


## 第一步：添加Maven依赖

```xml
<dependency>
    <groupId>io.github.liuye744</groupId>
    <artifactId>simpleAuth-spring-boot-starter</artifactId>
    <version>1.1.0.RELEASE</version>
</dependency>
```

## 第二步：为Controller添加注解

可以为整个Controller添加，也可以为Controller中单个方法添加。访问超过限制则会抛出`AccessIsRestrictedException`

### 用例1：规定时间内限制访问次数

```java
@RestController
public class MyController {
    @GetMapping("say")
    //10分钟内只允许访问5次，超过之后将会被禁止10分钟
    @IsLimit(value = 5, seconds = 600, ban = 600)
    public String say(){
        return "Hello World";
    }
}
```
### 用例2：根据返回值确实是否记录此次操作
当返回“success”时才记录操作，返回其他内容时不记录操作，不限制访问
```java
@RestController
public class MyController {
    @GetMapping("say")
    @IsLimit(effectiveStrategic = MyEffectiveStrategic.class)
    public String say(String str){
        if (str.length()>3 && str.length()<12){
            return "success";
        }else {
            return "fail";
        }
    }
}

public class MyEffectiveStrategic extends EffectiveStrategic {
    @Override
    public Boolean effective(HttpServletRequest request, ProceedingJoinPoint joinPoint, Object result) {
        String myResult = (String)result;
        //返回true则记录，false不记录
        return "success".equals(myResult);
    }
}
```

### 用例3：同一个接口不同的参数操作记录分别计算
传递的参数不同访问限制不同(例如想要规定时间内每个资源只能点赞N次)

```java
@RestController
public class MyController {
    @GetMapping("say")
    @IsLimit(signStrategic = MySignStrategic.class)
    public String say(String str){
        return "Hello World";
    }
}
public class MySignStrategic extends SignStrategic {
    @Override
    public String sign(HttpServletRequest request, ProceedingJoinPoint joinPoint) {
        final Object[] args = joinPoint.getArgs();
        final Signature signature = joinPoint.getSignature();
        //将参数拼接到用户sign中，保证每个用户传递不同的参数标志不相同
        StringBuilder sb = new StringBuilder();
        sb.append(signature);
        for (Object arg : args) {
            sb.append(arg.toString());
        }
        System.out.println(sb);
        return sb.toString();
    }
}
```
### 用例4：自定义访问控制

```java
//全局访问控制
@Component
public class MyInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        final String addr = request.getRemoteAddr();
        //以用户的地址作为标志，每5分钟(300s)只允许访问2次，超过之后被禁止10分钟
        //addRecord方法调用后可以访问则返回true,禁止访问返回false
        return LimitInfoUtil.addRecord("GLOBAL_ACCESS_CONTROL", addr, 2, 300, 600);
    }
}
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyInterceptor()).addPathPatterns("/*");
    }
}
```