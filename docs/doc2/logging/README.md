---
title: Logging
keywords: keyword1, keyword2
desc: This is a lightweight framework for permission validation and access control based on Spring Boot. Suitable for lightweight and progressive projects.
date: 2023-09-10
---

## Enabling Authentication Module Logging
```properties
# StdOut: Output to the console
# jdk: Java's built-in logging
simple-auth.log.log-impl=StdOut
```
Once configured, upon project startup, the console will output:
`Auth Logging initialized using class com.codingcube.simpleauth.logging.stdout.StdOutImpl adapter.`
## Authentication Module Logging Explanation
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
`uri`: The request's URI.
`handlerName`: The fully qualified name of the handling Handler.
`source`: Handler configuration method. In this example, it's "Dynamic Permission Configuration" (configured dynamically via a Provider) and this Handler is within the "MyHandlerChain."
`Required permission`: The required permission name.
`Permissions to carry`: The carried permission names.
`Principal to carry`: The carried instance object.
`Pass or not`: Whether the request passed.

## Enabling Access Control Module Logging
```properties
# StdOut: Output to the console
# jdk: Java's built-in logging
simple-auth.log.limit-log-impl=StdOut
# Whether to include the user's operation list when logging. Default is false.
simple-auth.log.show-opt-list=true
```
Once configured, upon project startup, the console will output:
`Limit Logging initialized using class com.codingcube.simpleauth.logging.stdout.StdOutImpl adapter.`
## Access Control Module Logging Explanation
```
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
`max-times`: Maximum number of requests.

`time`: Number of recorded requests (including the current one).

`seconds`: Recording time.

`ban`: Time for which the request is banned after exceeding the maximum number of requests.

`item`: Name of the item for access control. Defaults to the interface name if configured via annotation.

`sign`: User identifier. Defaults to the user's IP address. In this case, it's showing as 0:0:0:0:0:0:0:1, indicating localhost.

`source`: Configuration source for Limit (configured dynamically via a Provider).

`judgeAfterReturn`: Whether to determine if the current request is recorded after it returns. If true, you can use the return value to determine if the request is recorded.

`effectiveStrategic`: Class used to determine if the current request is recorded.

`effective`: Whether the request is recorded.

`optionList`: List of recorded requests. Requires simple-auth.log.show-opt-list=true to be configured in the Spring Boot configuration file.

`Pass or not`: Whether the current request passed.
## Custom Logging Module
Implement the Log interface and add a constructor with a String parameter.
```java
import com.codingcube.simpleauth.logging.Log;

public class MyLog implements Log {
    public MyLog(String clazz) {
        // TODO: Implement your custom logging logic here.
    }

    @Override
    public void debug(String s) {
        // TODO: Implement debug log logic.
    }
    
    @Override
    public void error(String s, Throwable e) {
        // TODO: Implement error log logic.
    }

    @Override
    public void warn(String s) {
        // TODO: Implement warning log logic.
    }
	// ... and so on for other log levels.
}
```
Configure your custom logging module.
```java
# Provide the fully qualified name of your custom Log implementation.
simple-auth.log.log-impl=com.example.simpleauthtest.domain.MyLog
```