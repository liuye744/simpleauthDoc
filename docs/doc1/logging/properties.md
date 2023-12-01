---
title: 常用配置参数
keywords: keyword1, keyword2
desc: 这是一款基于SpringBoot的轻量化的权限校验和访问控制的框架。适用于轻量级以及渐进式的项目。
date: 2023-09-10
class: heading_no_counter
---

## Auth常用配置
自己实现的Log参考:[自定义日志](/doc1/logging/index.html)
```properties
# 权限校验日志，默认为NoLogging，即不打印日志。
# StdOut：输出到控制台；jdk：java自带的日志系统
simple-auth.log.log-impl=StdOut

# 是否开启动态权限校验，默认为false
simple-auth.func.dynamic-auth=true

# Auth全局默认拒绝策略
simple-auth.auth.default-rejected=com.example.simpleauthvalidatetest.controller.MyRejected

# handler 是否缓存，默认为true，不建议更改
simple-auth.func.handler-cache=true
```

## Limit常用配置
```properties
# 限流日志，默认为NoLogging，即不打印日志。
# StdOut：输出到控制台；jdk：java自带的日志系统
simple-auth.log.limit-log-impl=StdOut

# 权限校验日志中是否输出用户操作列表，默认为false
simple-auth.log.show-opt-list=true

# 是否开启动态限流控制，默认为false
simple-auth.func.dynamic-limit=true

# 默认限流算法。默认为CompleteLimit,即所有操作全部准确记录.
# tokenbucket 为令牌桶模式
simple-auth.func.limit-plan=tokenBucket

# Limit全局默认拒绝策略
simple-auth.limit.default-rejected=com.example.simpleauthvalidatetest.controller.MyRejected

# 过期操作数据定时清理，开启后默认会在每周1凌晨3点半清理过期数据
simple-auth.clear.clear=true
simple-auth.clear.cron=0 30 3 * * 1

```

## Validate常用配置
```properties
# 全局默认参数检验类
simple-auth.validate.default-validate-object=com.example.simpleauthvalidatetest.validate.MyValidateObj

# validate全局默认拒绝策略
simple-auth.validate.default-rejected=com.example.simpleauthvalidatetest.controller.MyRejected
```
