---
title: JSON配置
keywords: keyword1, keyword2
desc: 这是一款基于SpringBoot的轻量化的权限校验和访问控制的框架。适用于轻量级以及渐进式的项目。
date: 2023-09-10
---
推荐使用XML配置
## 配置相关配置文件
```json
{  
    "configs": [
        "handler.json",
        "limit.json",
        "handlerChain.json",
        "paths.json"
    ]
}
```
## 配置Handler
```json
{
    "handler": [
        {
        "id": "test",
        "class": "com.example.simpleauthtest.handler.MyHandler",
        "scope": "singleton",
        "pathsId":"test"
        },
        {
        "id": "myHandlerId2",
        "class": "com.example.simpleauthtest.handler.MyHandler",
        "scope": "singleton",
        "paths": {
            "id": "myHandlerPath2",
            "path": [
            "/say",
            "/eat"
            ]
        }
        }
  ]
}
```
## 配置Limit
```json
{
    "limit": [
        {
        "id": "limitId",
        "times": 1,
        "seconds": 2,
        "ban": 3,
        "pathsId": "myLimitPath"
        }
    ]
}
```
## 配置HandlerChain
```json
{
    "handlerChain": [
        {
        "id": "myHandlerChain",
        "list": [
            {
            "id": "test"
            },
            {
            "class": "com.example.simpleauthtest.handler.MyHandler"
            }
        ],
        "paths": {
            "id": "myPath"
        }
        },
        {
        "id": "myHandlerChain2",
        "list": [
            {
            "id": "test"
            },
            {
            "class": "com.example.simpleauthtest.handler.MyHandler"
            }
        ],
        "paths": {
            "id": "myPath"
        }
        }
    ]
}
```

## 配置Paths
```json
[
    "paths": [
        {
        "id": "myLimitPath",
        "path": [
            "/say",
            "/eat",
            "/sleep"
        ]
        },
        {
        "id": "myPath",
        "path": [
            "/say",
            "/eat",
            "/sleep"
        ]
        }
    ]
]
```