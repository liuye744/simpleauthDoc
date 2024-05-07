---
title: Logging
keywords: keyword1, keyword2
desc: This is a lightweight permission verification and access control framework based on SpringBoot. It is suitable for lightweight and progressive projects.
date: 2023-09-10
class: heading_no_counter
---

## Permission Verification Module Logging
### Enable Authentication Module Logging
```properties
# StdOut: Output to the console
# jdk: Annotations provided by Java itself
simple-auth.log.log-impl=StdOut
```
After the configuration is complete and the project is started, the console will output: `Auth Logging initialized using class com.codingcube.simpleauth.logging.stdout.StdOutImpl adapter.`

### Authentication Module Logging Explanation
```java
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
* `uri`: The requested URI
* `handlerName`: The fully qualified name of the handler processing the request
* `source`: The configuration method of the handler, in this case, it is Dynamic Permission Configuration (configured dynamically through a Provider) and the handler is in MyHandlerChain
* `Required permission`: The name of the required permission
* `Permissions to carry`: The name of the permissions being carried
* `Principal to carry`: The instance object being carried
* `Pass or not`: Whether the request passed the authentication

## Access Control Module Logging
### Enable Access Control Module Logging
```properties
# StdOut: Output to the console
# jdk: Annotations provided by Java itself
simple-auth.log.limit-log-impl=StdOut
# Whether to include the user's operation list in the log output. Default is false
simple-auth.log.show-opt-list=true
```
After the configuration is complete and the project is started, the console will output: `Limit Logging initialized using class com.codingcube.simpleauth.logging.stdout.StdOutImpl adapter.`
### Access Control Module Logging Explanation
```java
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
* `max-times`: The maximum number of requests allowed
* `time`: The number of recorded requests (including the current one)
* `seconds`: The duration for which the requests are recorded
* `ban`: The time period during which access is forbidden after exceeding the maximum number of requests
* `item`: The name of the item subject to access control; if configured via annotation, it defaults to the interface name
* `sign`: The user identifier. By default, it is the user’s IP address; in this case, it shows `0:0:0:0:0:0:0:1` because it’s a local access
* `source`: The source of the Limit configuration (configured dynamically via a Provider in this case)
* `judgeAfterReturn`: Whether the decision to record the request is made after the response is returned. If set to `true`, the decision to record the request can be based on the return value
* `effectiveStrategic`: The class that determines whether the current request should be recorded
* * `effective`: Whether the request was recorded
* `optionList`: The list of recorded requests. This is displayed only when `simple-auth.log.show-opt-list=true` is set in the SpringBoot configuration file
* `Pass or not`: Whether the current request passed the access control check

## Parameter Validation Module Logging
### Enable Parameter Validation Module Logging
```properties
# StdOut: Output to the console
# jdk: Annotations provided by Java itself
simple-auth.log.validated-log-impl=StdOut
```
### Parameter Validation Module Logging Explanation
```java
SimpleAuth validate => 
	validateObj: class com.codingcube.simpleauthtest.cache.MyValidate
	validateTarget: class java.lang.String
	pass or not: false
```
* `validateObj`: The validation class
* `validateTarget`: The type of the target being validated
* `pass or not`: Whether the current request passed the validation

## Custom Logging Module
Implement the Log interface, and add a constructor with a single String parameter.
```java
import com.codingcube.simpleauth.logging.Log;
public class MyLog implements Log {
    public MyLog(String clazz) {
        // TODO
    }

    @Override
    public void debug(String s) {
        // TODO
    }
    
    @Override
    public void error(String s, Throwable e) {
        // TODO
    }

    @Override
    public void warn(String s) {
        // TODO
    }
    // ... other methods
}
```
Configure your custom logging module.
```java
# The parameter is the fully qualified name of the custom Log
simple-auth.log.log-impl=com.example.simpleauthtest.domain.MyLog
```