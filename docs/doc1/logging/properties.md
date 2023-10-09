---
title: 常用配置参数
keywords: keyword1, keyword2
desc: 这是一款基于SpringBoot的轻量化的权限校验和访问控制的框架。适用于轻量级以及渐进式的项目。
date: 2023-09-10
---

## 常用配置参数
```properties
# 限流日志，默认为NoLogging，即不打印日志。
# StdOut：输出到控制台；jdk：java自带的日志系统
simple-auth.log.limit-log-impl=StdOut

# 权限校验日志，默认为NoLogging
simple-auth.log.log-impl=StdOut

# 权限校验日志中是否输出用户操作列表，默认为false
simple-auth.log.show-opt-list=true

# 是否开启动态权限校验，默认为false
simple-auth.func.dynamic-auth=true

# 是否开启动态限流控制，默认为false
simple-auth.func.dynamic-limit=true

# handler 是否缓存。默认为true
simple-auth.func.handler-cache=true

# 默认限流算法。默认为CompleteLimit,即所有操作全部准确记录
simple-auth.func.limit-plan=tokenBucket
```
