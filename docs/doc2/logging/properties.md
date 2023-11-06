---
title: Common Configuration Parameters
keywords: keyword1, keyword2
desc: This is a lightweight framework for permission validation and access control based on SpringBoot. It is suitable for lightweight and progressive projects.
date: 2023-09-10
class: heading_no_counter
---

## Common Configuration Parameters
```properties
# Rate limiting log, default is NoLogging (no log).
# StdOut: Log to the console; jdk: Java's built-in logging system
simple-auth.log.limit-log-impl=StdOut

# Permission validation log, default is NoLogging
simple-auth.log.log-impl=StdOut

# Whether to display the user operation list in the permission validation log, default is false
simple-auth.log.show-opt-list=true

# Enable dynamic permission validation, default is false
simple-auth.func.dynamic-auth=true

# Enable dynamic access control, default is false
simple-auth.func.dynamic-limit=true

# Handler caching, default is true
simple-auth.func.handler-cache=true

# Default rate limiting algorithm, default is CompleteLimit, which accurately records all operations
simple-auth.func.limit-plan=tokenBucket
```
