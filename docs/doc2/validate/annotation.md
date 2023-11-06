---
title: Quick Start - Permission Verification
keywords: keyword1, keyword2
desc: This is a lightweight framework for permission verification and access control based on SpringBoot. It is suitable for lightweight and progressive projects.
date: 2023-09-10
class: heading_no_counter
---

In this section, we will explain how to perform permission verification using annotations.

## Annotations

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
public @interface SimpleValidate {
    // The class to be used for validation. The validation functions within this class must return a Boolean.
    Class<?> value() default Object.class;
    // Names of the validation functions to be used.
    String[] methods() default {"validate"};
    // Strategy to handle validation rejection (default strategy throws ValidateException).
    Class<? extends ValidateRejectedStratagem> rejected() default DefaultValidateRejectedStratagem.class;
}
```

## Annotation Usage

The `SimpleValidate` annotation is used to specify the validation class and method for permission validation. The class provided in the `value` parameter contains validation methods that must return a `Boolean` and take the object to be validated as a parameter.

For example, a validation method in the validation class might look like this:

```java
public class MyValidateObj {
    // Validate that the user's age is between 1 and 99
    public Boolean fillUser(User user){
        final Integer age = user.getAge();
        if (age <= 1 || age >= 99){
            return false;
        }
        return true;
    }
}
```

To use the `SimpleValidate` annotation in a controller, add it to a method:

```java
@RestController
public class MyController {
    @GetMapping("/say")
    @SimpleValidate(value = MyValidateObj.class, methods = {"fillUser"})
    public String say(User user){
        return user.getName();
    }
}
```

If multiple methods in a controller need to use the same validation class, you can specify the validation class at the class level and reference it in the method-level annotations. When the class-level annotation specifies a validation class, you can skip the `value` parameter in the method-level annotations:

```java
@RestController
@SimpleValidate(value = MyValidateObj.class)
public class MyController {

    @GetMapping("/say")
    @SimpleValidate(methods = {"fillUser"})
    public String say(User user){
        System.out.println("say");
        return user.getName();
    }

    @GetMapping("/eat")
    @SimpleValidate(methods = {"partUser"})
    public String eat(User user){
        System.out.println("eat");
        return user.getName();
    }
}
```

### Rejection Strategy

The default rejection strategy is to throw a `ValidateException`. You can create a custom rejection strategy by implementing the `ValidateRejectedStratagem` interface:

```java
public class MyRejected implements ValidateRejectedStratagem {
    @Override
    public void doRejected(HttpServletRequest request, HttpServletResponse response, Object validationObj) {
    }
}
```

When a validation method returns `false`, the `doRejected` function of the rejection strategy will be invoked, and the `validationObj` parameter will contain the object that was just validated. For example, when the validation of a `User` object returns `false`, the `validationObj` parameter in `doRejected` will be the `User` object.