---
title: 日志
keywords: keyword1, keyword2
desc: 这是一款基于SpringBoot的轻量化的权限校验和访问控制的框架。适用于轻量级以及渐进式的项目。
date: 2023-09-10
class: heading_no_counter
---

## 权限校验模块日志
### 开启鉴权模块日志
```properties
#StdOut：输出到控制台
#jdk：java自带的注解
simple-auth.log.log-impl=StdOut
```

配置完成项目启动后控制台将会输出：
`Auth Logging initialized using class com.codingcube.simpleauth.logging.stdout.StdOutImpl adapter.`
### 鉴权模块日志详解
```
SimpleAuth auth => 
	uri: /say
	handlerName: com.example.simpleauthtest.handler.MyHandler
	source: Dynamic Permission Configuration handlerChain com.example.simpleauthtest.handler.MyHandlerChain
	Required permission: vip
	Permissions to carry: [visitor]
	Principal to carry: null
	Pass or not: true
SimpleAuth auth => 
	uri: /say
	handlerName: com.example.simpleauthtest.handler.MyHandler2
	source: Dynamic Permission Configuration handlerChain com.example.simpleauthtest.handler.MyHandlerChain
	Required permission: vip
	Permissions to carry: [visitor]
	Principal to carry: null
	Pass or not: true
```
* `uri`：请求的URI
* `handlerName`：处理的Handler的全限定名
* `source`：Handler的配置方式，本例中是Dynamic Permission Configuration(通过Provider动态配置) 且本Handler在MyHandlerChain中
* `Required permission`：需要的权限名
* `Permissions to carry`：携带的权限名
* `Principal to carry`：携带的实例对象
* `Pass or not`：是否通过了

## 访问控制模块日志
### 开启访问控制模块日志
```properties
#StdOut：输出到控制台
#jdk：java自带的注解
simple-auth.log.limit-log-impl=StdOut
#输出日志时是否带有用户的操作列表。默认为false
simple-auth.log.show-opt-list=true
```
配置完成项目启动后控制台将会输出：
`Limit Logging initialized using class com.codingcube.simpleauth.logging.stdout.StdOutImpl adapter.`

### 访问控制模块日志详解
```properties
SimpleAuth limit => 
	max-times: 2
	time: 1
	seconds: 60
	ban: 0
	item: /say
	signStrategic: com.codingcube.simpleauth.auth.strategic.DefaultSignStrategic
	sign: 0:0:0:0:0:0:0:1
	source: dynamic limit
	judgeAfterReturn: false
	effectiveStrategic: com.codingcube.simpleauth.limit.strategic.DefaultEffectiveStrategic
	effective: true
	optionList: [Mon Sep 11 01:13:12 CST 2023]
	Pass or not: true
```
* `max-times`：最大请求次数
* `time`：记录的请求次数(包括本次)
* `seconds`：记录的时间
* `ban`：请求超过最大请求次数后被禁止的时间
* `item`：进行访问控制的item名，若为注解配置默认为接口名
* `sign`：用户标志。默认为用户IP，这里是本机访问所以显示为0:0:0:0:0:0:0:1
* `source`：Limit的配置来源(这里是通过Provider动态配置)
* `judgeAfterReturn`：是否是在返回后判断本次请求是否被记录。为true可以通过返回值进行判断本次请求是否被记录
* `effectiveStrategic`：判断本次请求是否被记录的类
* `effective`：是否被记录了
* `optionList`：被记录的请求列表，需要在SpringBoot的配置文件中配置`simple-auth.log.show-opt-list=true`才会显示
* `Pass or not`：本次请求是否通过

## 参数校验模块日志
### 开启参数校验模块日志
```properties
#StdOut：输出到控制台
#jdk：java自带的注解
simple-auth.log.validated-log-impl=StdOut
```

### 参数校验模块日志详解
```properties
SimpleAuth validate => 
	validateObj: class com.codingcube.simpleauthtest.cache.MyValidate
	validateTarget: class java.lang.String
	pass or not: false
```
`validateObj`：校验类
`validateTarget`：校验目标类型
`pass or not`：本次请求是否通过

## 自定义日志模块
实现Log接口，并添加带有一个String参数的构造函数
```java
import com.codingcube.simpleauth.logging.Log;
public class MyLog implements Log {
    public MyLog(String clazz) {
        //TODO
    }

    @Override
    public void debug(String s) {
        //TODO
    }
    
    @Override
    public void error(String s, Throwable e) {
        //TODO
    }

    @Override
    public void warn(String s) {
        //TODO
    }
	//...略
}
```
配置自己的日志模块
```properties
#参数为自定义Log的全限定名
simple-auth.log.log-impl=com.example.simpleauthtest.domain.MyLog
```