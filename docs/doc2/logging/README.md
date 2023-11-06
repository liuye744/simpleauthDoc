---
title: Logging
keywords: keyword1, keyword2
desc: This is a lightweight framework for permission validation and access control based on SpringBoot. It is suitable for lightweight and progressive projects.
date: 2023-09-10
class: heading_no_counter
---

## Enabling Authentication Module Logging
```properties
# StdOut: Log to the console
# jdk: Java's built-in logging system
simple-auth.log.log-impl=StdOut
```

After configuring, the console will output the following message when the project starts:
`Auth Logging initialized using class com.codingcube.simpleauth.logging.stdout.StdOutImpl adapter.`

## Explanation of Authentication Module Logging
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
- `uri`: The request's URI.
- `handlerName`: The full class name of the handler being used.
- `source`: The configuration source of the handler. In this example, it's "Dynamic Permission Configuration" (configured dynamically through a Provider), and the handler is part of the "MyHandlerChain."
- `Required permission`: The required permission name.
- `Permissions to carry`: The carried permission names.
- `Principal to carry`: The carried instance object.
- `Pass or not`: Whether it passed.

## Enabling Access Control Module Logging
```properties
# StdOut: Log to the console
# jdk: Java's built-in logging system
simple-auth.log.limit-log-impl=StdOut
# Whether to include the user's operation list in the log. Default is false.
simple-auth.log.show-opt-list=true
```

After configuring, the console will output the following message when the project starts:
`Limit Logging initialized using class com.codingcube.simpleauth.logging.stdout.StdOutImpl adapter.`

## Explanation of Access Control Module Logging
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
- `max-times`: Maximum number of requests.
- `time`: Recorded requests (including the current one).
- `seconds`: Recording time.
- `ban`: Time the request is banned after exceeding the maximum number of requests.
- `item`: The name of the item for access control. By default, it's the interface name if configured using annotations.
- `sign`: User identifier. By default, it's the user's IP. In this example, it shows as "0:0:0:0:0:0:0:1" because it's local access.
- `source`: The source of the limit's configuration (configured dynamically through a Provider).
- `judgeAfterReturn`: Whether to determine if the current request is recorded after returning. When set to true, you can determine if the request is recorded based on the return value.
- `effectiveStrategic`: The class used to determine if the request is recorded.
- `effective`: Whether it's recorded.
- `optionList`: The list of recorded requests. It's shown when configured in the SpringBoot properties file with `simple-auth.log.show-opt-list=true`.
- `Pass or not`: Whether the current request passed.

## Custom Logging Module
Implement the `Log` interface and add a constructor with a `String` parameter.
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
    // ... and so on
}
```

Configure your custom logging module in the properties file.
```properties
# Parameter is the full-qualified name of your custom Log class
simple-auth.log.log-impl=com.example.simpleauthtest.domain.MyLog
```