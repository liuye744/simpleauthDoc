---
title: XML配置
keywords: keyword1, keyword2
desc: 这是一款基于SpringBoot的轻量化的权限校验和访问控制的框架。适用于轻量级以及渐进式的项目。
date: 2023-09-10
---

## 配置相关配置文件
默认的配置文件为classpath目录下的`simpleauth.xml`,可以通过如下配置添加其他配置文件
```xml
<simpleauth>
	<configs>
		<config>handler.xml</config>
		<config>limit.xml</config>
		<config>paths.xml</config>
		<config>handlerChain.xml</config>
	</configs>
</simpleauth>
```
## 配置Handler
paths可以只指定id，具体的路径可以单独声明。`scope`默认为singleton，即所有用到这个Handler的函数均使用同一个实例。
```xml
<simpleauth>
    <handler  id="test">
        <class>com.example.simpleauthtest.handler.MyHandler</class>
        <scope>singleton</scope>
        <paths id="myHandlerPath"/>
    </handler>

    <handler id="myHandler2">
        <class>com.codingcube.simpleauth.security.handler.session.SessionMigratorHandler</class>
        <paths id="myHandlerPath3">
            <path>/say/*</path>
            <path>/eat</path>
        </paths>
    </handler>
</simpleauth>
```
## 配置Limit
60s内只能访问两次，超过之后被禁止60s
```xml
<simpleauth>
    <!--注册Limit-->
    <limit id="MyLimit">
        <times>2</times>
		<seconds>60</seconds>
		<ban>60</ban>
        <paths id="myLimitPath">
            <path>/say</path>
            <path>/eat</path>
        </paths>
    </limit>
</simpleauth>
```
或者可以更详细的配置
```xml
<simpleauth>
    <!--注册Limit-->
    <limit name="MyLimit" id="MyLimit">
        <times>1</times>
		<seconds>60</seconds>
		<ban>60</ban>
        <signStrategic>com.codingcube.simpleauth.auth.strategic.DefaultItemStrategic</signStrategic>
        <itemStrategic>com.codingcube.simpleauth.auth.strategic.DefaultSignStrategic</itemStrategic>
        <effectiveStrategic>com.codingcube.simpleauth.limit.strategic.DefaultEffectiveStrategic</effectiveStrategic>
        <tokenLimit>com.codingcube.simpleauth.limit.util.TokenBucket</tokenLimit>
        <paths id="myLimitPath">
            <path>/say</path>
            <path>/eat</path>
        </paths>
    </limit>
</simpleauth>
```


## 配置HandlerChain

```xml
<simpleauth>
    <handlerChain id="MyLimit">
		<list>
			<handler id="test"/>
			<handler>
				<class>com.example.simpleauthtest.handler.MyHandler2</class>
			</handler>
		</list>
		<paths id="myPath"/>
    </handlerChain>
</simpleauth>
```

## 配置Paths
```xml
<simpleauth>
    <paths id="myPath">
        <permission>myPath</permission>
        <path>/say/*</path>
        <path>/eat</path>
    </paths>
    <paths id="myHandlerPath2">
        <path>/sleep</path>
        <path>/eat</path>
    </paths>
</simpleauth>
```

## 总结
当然你也可以把所有内容均配置到`simpleauth.xml`内
```xml
<simpleauth>
	<configs>
        <!--若后缀为xml,则配置文件中可以不添加后缀-->
		<config>handler</config>
		<config>limit</config>
		<config>paths</config>
		<config>handlerChain</config>
	</configs>
    <handler  id="test">
        <class>com.example.simpleauthtest.handler.MyHandler</class>
        <scope>singleton</scope>
        <paths id="myHandlerPath"/>
    </handler>
    <limit id="MyLimit">
        <times>1</times>
		<seconds>60</seconds>
		<ban>60</ban>
        <paths id="myLimitPath">
            <path>/say</path>
            <path>/eat</path>
        </paths>
    </limit>
    <handlerChain id="MyLimit">
		<list>
			<handler id="test"/>
			<handler>
				<class>com.example.simpleauthtest.handler.MyHandler2</class>
			</handler>
		</list>
		<paths id="myPath"/>
    </handlerChain>
    <paths id="myPath">
        <permission>myPath</permission>
        <path>/say/*</path>
        <path>/eat</path>
    </paths>
    <paths id="myHandlerPath2">
        <path>/say/*</path>
        <path>/eat</path>
    </paths>
</simpleauth>
```