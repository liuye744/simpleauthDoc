---
title: XML Configuration
keywords: keyword1, keyword2
desc: This is a lightweight framework for permission validation and access control based on SpringBoot. It is suitable for lightweight and progressive projects.
date: 2023-09-10
---

## Configure Related Configuration Files
The default configuration file is `simpleauth.xml` in the classpath. You can add other configuration files as follows:

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

## Configure Handlers
You can specify `id` for paths and declare specific paths separately. The `scope` is set to `singleton` by default, meaning that all functions that use this Handler will share the same instance.

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

## Configure Limits
Allow only two requests within 60 seconds, and block access for 60 seconds if the limit is exceeded.

```xml
<simpleauth>
    <!-- Register Limit -->
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

You can provide more detailed configuration as shown below:

```xml
<simpleauth>
    <!-- Register Limit -->
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

## Configure Handler Chains

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

## Configure Paths

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

## Summary
Of course, you can also configure everything in `simpleauth.xml`:

```xml
<simpleauth>
	<configs>
        <!-- If the file extension is XML, you can omit the extension in the configuration file -->
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
			</handler

>
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