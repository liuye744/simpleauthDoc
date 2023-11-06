---
title: Quick Start - Parameter Validation
keywords: keyword1, keyword2
desc: This is a lightweight framework for permission validation and access control based on SpringBoot. It is suitable for lightweight and progressive projects.
date: 2023-09-10
class: heading_no_counter
---

In this section, we will explain how to validate parameters using annotations.

## Step 1: Add Maven Dependency
```xml
<dependency>
    <groupId>io.github.liuye744</groupId>
    <artifactId>simpleAuth-spring-boot-starter</artifactId>
    <version>1.4.0.RELEASE</version>
</dependency>
```

## Step 2: Add Annotations

You can add annotations to individual functions within a Controller. If the validation limit is exceeded, a `ValidateException` will be thrown.

### Use Case 1: Simple Validation of User Name and Age
Validate that the age (age) is between 1 and 99 and that the length of the nickname (name) is between 5 and 16.

```java
// User class
public class User {
    String name;
    Integer age;
    String phone;
    // Getters and setters omitted
}

// Controller where methods refer to functions in MyValidateObj
@RestController
public class MyController {
    @GetMapping("/say")
    @SimpleValidate(value = MyValidateObj.class, methods = {"fillUser"})
    public String say(User user){
        System.out.println("Controller " + user);
        return user.getName();
    }
}
// Validation class (returns a Boolean and has exactly one object to validate as a parameter)
public class MyValidateObj {
    public Boolean fillUser(User user){
        final Integer age = user.getAge();
        if (age <= 1 || age >= 99){
            return false;
        }
        final int nameLength = user.getName().length();
        if (nameLength <= 5 || nameLength >= 16){
            return false;
        }
        return true;
    }
}
```

### Use Case 2: Reusing the Same Validation for Multiple Functions in the Same Controller
The `/say` endpoint requires validation of the `name` and `age` fields, while the `/eat` endpoint only requires validation of the `phone` field for a valid mobile phone number.

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
// Validation class
public class MyValidateObj {
    public Boolean fillUser(User user){
        final Integer age = user.getAge();
        if (age <= 1 || age >=99){
            return false;
        }
        final int nameLength = user.getName().length();
        if (nameLength <= 5 || nameLength >= 16){
            return false;
        }
        return true;
    }

    public Boolean partUser(User user){
        final String phone = user.getPhone();
        if (phone == null){
            return false;
        }
        Pattern pattern = Pattern.compile("^(13[0-9]|14[01456879]|15[0-35-9]|16[2567]|17[0-8]|18[0-9]|19[0-35-9])\\d{8}$");
        return pattern.matcher(phone).matches();
    }
}
```

### Use Case 3: Simplifying Validation Using a Utility Class
This section demonstrates the same functionality as Use Case 2.

```java
public class MyValidateObj {
    public Boolean fillUser(User user){
        return SVU.notFalse(
            SVU.lengthRange( user.getName(), 5, 16),
            SVU.range( user.getAge(), 1, 99)
        );
    }

    public Boolean partUser(User user){
        return SVU.pattern(user.getPhone(), Regex.CHINESE_MOBIL_PHONE_NUMBER);
    }
}
```

### Use Case 4: Throwing Different Exceptions on Validation Failure
This section demonstrates the same functionality as the previous example. In this case, a `ValidateException` will be thrown when validation fails, and the input message will be carried in the exception's message.

```java
// The ValidateException will be thrown on validation failure, and the provided message will be carried in the exception's message.
public class MyValidateObj {
    public Boolean fillUser(User user){
        return SVU.notFalse(
            SVU.lengthRange("昵称长度需要在5-16之间", user.getName(), 5, 16),
            SVU.range("年龄需要在1-99之间", user.getAge(), 1, 99)
        );
    }

    public Boolean partUser(User user){
        return SVU.pattern("手机号格式不合法", user.getPhone(), Regex.CHINESE_MOBIL_PHONE_NUMBER);
    }
}
// It's recommended to catch ValidateException in an advice class and handle it there.
@ControllerAdvice
public class MyExceptionHandler {
    @ExceptionHandler(value = ValidateException.class)
    @ResponseBody
    public String exceptionHandler(ValidateException e){
        return e.getMessage();
    }
}
```

With the provided code, accessing `http://localhost:8080/say?name=CodingCube` will return the message `"年龄需要在1-99之间"` if validation fails.