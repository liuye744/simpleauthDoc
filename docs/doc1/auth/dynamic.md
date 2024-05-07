---
title: 动态权限校验
keywords: keyword1, keyword2
desc: 这是一款基于SpringBoot的轻量化的权限校验和访问控制的框架。适用于轻量级以及渐进式的项目。
date: 2023-09-10
class: heading_no_counter
---


## 开启动态权限校验
在Application启动类上加入`@EnableDynamicAuthRequest`注解

```java
@SpringBootApplication
@EnableDynamicAuthRequest
public class SimpleAuthTestApplication {
    public static void main(String[] args) {
        SpringApplication.run(SimpleAuthTestApplication.class, args);
    }
}
```
或者在配置文件中配置
```properties
simple-auth.func.dynamic-auth=true
```
## 创建AuthItemProvider
```java
@Component
public class MyAuthItemProvider implements RequestAuthItemProvider {
    @Override
    public List<RequestAuthItem> getRequestAuthItem() {
        List<RequestAuthItem> list = new ArrayList<>();
        //访问`/say`路径时使用MyHandler处理，并携带了请求权限visitor。
        //注：具体请求权限的处理交给MyHandler,这里只是携带
        //注：RequestAuthItem的路径也可传入List<String>，表示多个路径处理方式相同
        //注：可通过数据库初始化RequestAuthItem，以动态的调整权限校验
        list.add(new RequestAuthItem("/say", "visitor", MyHandler.class));
        list.add(new RequestAuthItem(MyHandlerChain.class, "/user/*", "vip"));
        return list;
    }
}
//处理'/say'路径的Handler
@Component
public class MyHandler extends AutoAuthHandler {
    @Override
    public boolean isAuthor(HttpServletRequest request, String permission) {
        //参数name是否与permission相同。相同则放行
        final String name = request.getParameter("name");
        return name.equals(permission);
    }
}
//处理'/user/*'路径的HandlerChain
@Component
public class MyHandlerChain extends AutoAuthHandlerChain {
    @Override
    public void addChain() {
        addLast(MyHandler1.class).addLast(MyHandler2.class);
    }
}
//MyHandlerChain中的Handler
@Component
public class MyHandler1 extends AutoAuthHandler {
    @Override
    public boolean isAuthor(HttpServletRequest request, String permission) {
        //查询数据库添加用户permissions，为方便演示直接手动添加一个"visitor"
        ArrayList<String> permissions = new ArrayList<>();
        permissions.add("visitor");
        setPermissions(permissions);
        //返回true直接放行
        return true;
    }
}
@Component
public class MyHandler2 extends AutoAuthHandler {
    @Override
    public boolean isAuthor(HttpServletRequest request, String permission) {
        final ArrayList<String> permissions = getPermissions();
        return !permissions.contains(permission);
    }
}

```