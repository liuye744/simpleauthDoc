---
title: 快速上手-权限校验
keywords: keyword1, keyword2
desc: 这是一款基于SpringBoot的轻量化的权限校验和访问控制的框架。适用于轻量级以及渐进式的项目。
date: 2023-09-10
---
本节中将会介绍如何使用工具类简化参数的校验

## 常用校验函数
工具类的类名为SVU(SimpleValidateUtils)
```java
//传入多个参数，若存在null值则返回false
public static boolean notNull(Object ...args);
//判断base是否在start与end之间(包含等于start或end)，成立则返回true
public static boolean range(Integer base, Integer start, Integer end);
//判断字符串content的长度是否在start与end之间(包含等于start或end)，成立则返回true
public static boolean lengthRange(String content, Integer start, Integer end);
//判断content 是否符合regex的正则校验，符合返回true
public static boolean pattern(String content, String regex);
//当多个参数中存在false则返回false
public static boolean notFalse(boolean ...args);
```
## 多个参数校验
可以使用notFalse函数将多个判断条件连接起来，当其中一个
```java
public Boolean fillUser(User user){
    return SVU.notFalse(
        SVU.lengthRange(user.getName(), 5, 16),
        SVU.range(user.getAge(), 1, 99)
    );
}
```
## 快捷抛出异常
SVU类的大多数函数都有一个功能相同的带有message的函数，带有message的函数在校验失败后不返回false而是抛出`ValidateException`，并将传入的message封装到异常中。
```java
public Boolean fillUser(User user){
    return SVU.notFalse(
        SVU.lengthRange("昵称长度不符合5-16",user.getName(), 5, 16),
        SVU.range("年龄只能为1-99", user.getAge(), 1, 99)
    );
}
```
可以捕获异常处理验证信息
```java
//全局捕获异常并返回异常信息
@ControllerAdvice
public class MyExceptionHandler {
    @ExceptionHandler(value = ValidateException.class)
    @ResponseBody
    public String exceptionHandler(ValidateException e){
        return e.getMessage();
    }
}
```
## 使用延迟函数提高效率
### 因由
上述使用`notFalse`函数时会先计算每一个判断条件的真伪，后传入notFalse。
当校验失败且没有抛出异常时若前一个已经判断为False将不会终止，如此会造成效率的降低，建议使用SVU中的Delay函数延迟条件的判断。
### 使用
SVU中有一个重载的`notFalse`函数
`public static boolean notFalse(BooleanCompute ...booleanComputes);`
SVU中的Delay函数则会返回一个`BooleanCompute`对象将操作封装起来延迟判断。
例如：
```java
public Boolean fillUser(User user){
    return SVU.notFalse(
        SVU.lengthRangeDelay(user.getName(), 5, 16),
        SVU.rangeDelay(user.getAge(), 1, 99)
    );
}
```
当第一个`lengthRangeDelay`函数校验返回false后第二个`rangeDelay`则不会运行直接返回false;
## 可为空的参数校验

若部分需要校验的参数可为空，若不为空则需要校验是否符合要求，则可以使用`SVOU`类(SimpleValidateOptionalUtil)。
SVOU类中的函数名和SVU中的相同，只是当传入的参数为null时返回true

示例：电话为可选项，传入后不为null需要校验是否合法
```java
public Boolean fillUser(User user){
    //Regex类提供了常用正则表达式
    SVOU.pattern(user.getPhone(), Regex.CHINESE_MOBIL_PHONE_NUMBER);
    return SVU.notFalse(
        SVU.lengthRangeDelay(user.getName(), 5, 16),
        SVU.rangeDelay(user.getAge(), 1, 99)
    );
}
```