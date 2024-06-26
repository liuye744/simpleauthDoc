---
title: 注解
keywords: keyword1, keyword2
desc: 这是一款基于SpringBoot的轻量化的权限校验和访问控制的框架。适用于轻量级以及渐进式的项目。
date: 2023-09-10
class: heading_no_counter
---


## 注解参数
```java
public @interface SimpleAuth {
    //接口权限名，支持SpEL表达式
    String value() default "";
    //鉴权Handler的Class，默认扫描是否permissions中是否存在接口对应的value
    Class<? extends AutoAuthHandler>[] handler() default DefaultAuthHandler.class;
    //鉴权Handler链的Class
    Class<? extends AutoAuthHandlerChain>[] handlerChain() default DefaultAuthHandlerChain.class;
    //拒绝策略，默认为DefaultAuthRejectedStratagem，抛出PermissionsException("lack of permissions")异常
    Class<? extends AuthRejectedStratagem> rejected() default NullTarget.class;
}
```
## Handler（鉴权处理器）
需要继承AutoAuthHandler，如果用到Spring中的其他Bean可以使用@Component注册为Bean(推荐添加@Component注解)。
验证失败则会抛出`PermissionsException`异常
```java
public class MyAutoAuthHandler extends MyAutoAuthHandler {

    /**
     * 判断是否符合条件
     * @param permission 需要的权限名 
     * @return 返回true则符合(放行)
     */
    @override
    public boolean isAuthor(HttpServletRequest request, String permission){
        //验证通过返回true放行，失败则返回false
        return true;
    }

}
若没有使用@Component，首次调用Handler则通过无参构造函数创建新实例后缓存，之后的调用则会查询缓存中对应的Handler对象，可以使用`simple-auth.func.handler-cache=true`配置关闭缓存。
```
## HandlerChain（权限处理器链）
需要继承AutoAuthHandlerChain，在`addChain`函数中添加Handler将多个Handler组合在一起方便模块化调用。同样的也可以选择注册为Bean
```java
public abstract class AutoAuthHandlerChain {
    public abstract void addChain();
}
```
常用的函数
`addLast(Class<? extends AutoAuthHandler> autoAuthHandler)`:添加Handler到尾部，参数也可以为Handler的Bean名称。
`addFirst(Class<? extends AutoAuthHandler> auto)`:添加Handler到头部

## AuthRejectedStratagem（鉴权拒绝策略）
默认会抛出PermissionsException("lack of permissions")异常，一般不建议修改
默认拒绝策略如下
```java
public class DefaultAuthRejectedStratagem implements AuthRejectedStratagem {
    @Override
    public void doRejected(HttpServletRequest request, HttpServletResponse response, LogAuthFormat authFormat) {
        throw new PermissionsException("lack of permissions");
    }
}
```
可以自己创建类继承AuthRejectedStratagem接口并重写doRejected方法。
需要使用时可通过注解配置
```java
//新建的拒绝策略
public class MyAuthRejectedStratagem implements AuthRejectedStratagem {
    @Override
    public void doRejected(HttpServletRequest request, HttpServletResponse response, LogAuthFormat authFormat) {
        throw new PermissionsException("this is my AuthRejectedStratagem");
    }
}
//Controller
@GetMapping("/checkNameWhetherCodingCube2")
@SimpleAuth(handler = MyHandler.class, rejected = MyAuthRejectedStratagem.class)
public String checkNameWhetherCodingCube(String name){
    return "Hello World";
}
```
