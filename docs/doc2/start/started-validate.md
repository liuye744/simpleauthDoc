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
    <version>1.4.7.RELEASE</version>
</dependency>
```

## Step 2: Create Validation Class and Method
The method return value must be Boolean, and there must be exactly one object to be validated as a parameter.

```java
// Example object used
public class User {
    String name;
    Integer age;
    String phone;
    // Omitting getter, setter, etc.
}

// Requirement: Validate that the age (age) is between 1-99 and the length of the nickname (name) is 5-16
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

## Step 3: Add Annotations to Controller
Add to an individual function in the Controller. If the validation fails, a `ValidateException` will be thrown.

```java
// Controller object, where methods is the function name in MyValidateObj
@RestController
public class MyController {
    @GetMapping("/say")
    @SimpleValidate(value = MyValidateObj.class, methods = {"fillUser"})
    public String say(User user){
        System.out.println("Controller "+user);
        return user.getName();
    }
}
```

## Other Examples

### Use Case 1: Multiple Functions in the Same Controller Performing the Same Validation
`/say` needs to validate the name and age fields.
`/eat` only needs to validate whether the phone field is a valid mobile number.

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
        // Check if it is empty, if empty return false for validation failure
        if (phone == null){
            return false;
        }
        // Regular expression to validate if the phone number is legal
        Pattern pattern = Pattern.compile("^(13[0-9]|14[01456879]|15[0-35-9]|16[2567]|17[0-8]|18[0-9]|19[0-35-9])\\d{8}$");
        return pattern.matcher(phone).matches();
    }
}
```
### Use Case 2: Simplifying the Validation Process with Utility Classes
This part achieves the same functionality as Use Case 1.

```java
public class MyValidateObj {
    public Boolean fillUser(User user){
        // The notFalse function takes multiple boolean values, and returns false if any of them are false.
        // The lengthRange function validates whether the string length is between 5-16 (inclusive).
        // The range function validates whether the number is between 1-99 (inclusive).
        return SVU.notFalse(
            SVU.lengthRange( user.getName(), 5, 16),
            SVU.range( user.getAge(), 1, 99)
        );
    }

    public Boolean partUser(User user){
        // Regular expression validation for phone matching the regular expression.
        // The Regex class provides a large number of commonly used regular expression string constants.
        return SVU.pattern(user.getPhone(), Regex.CHINESE_MOBIL_PHONE_NUMBER);
    }
}
```

### Use Case 3: Throwing Different Exceptions on Validation Failure
This part achieves the same functionality as the previous example.

```java
// After validation fails, a ValidateException will be thrown, and the input prompt message will be carried in the exception object's message.
public class MyValidateObj {
    public Boolean fillUser(User user){
        return SVU.notFalse(
            SVU.lengthRange("The nickname length needs to be between 5-16", user.getName(), 5, 16),
            SVU.range("The age needs to be between 1-99", user.getAge(), 1, 99)
        );
    }

    public Boolean partUser(User user){
        return SVU.pattern("Invalid mobile phone number format", user.getPhone(), Regex.CHINESE_MOBIL_PHONE_NUMBER);
    }
}
// It is recommended to catch ValidateException in Advice and handle it.
@ControllerAdvice
public class MyExceptionHandler {
    @ExceptionHandler(value = ValidateException.class)
    @ResponseBody
    public String exceptionHandler(ValidateException e){
        return e.getMessage();
    }
}
```
With the above code, when accessing `http://localhost:8080/say?name=CodingCube`, it will return `The age needs to be between 1-99`.