---
title: JSON Configuration
keywords: keyword1, keyword2
desc: This is a lightweight framework for permission validation and access control based on SpringBoot. It is suitable for lightweight and progressive projects.
date: 2023-09-10
---
Recommended: Use XML Configuration

## Configure Related Configuration Files
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

## Configure Handlers
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

## Configure Limits
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

## Configure HandlerChain
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

## Configure Paths
```json
{
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
}
```