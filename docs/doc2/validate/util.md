---
title: Quick Start - Parameter Validation
keywords: keyword1, keyword2
desc: This is a lightweight framework for permission verification and access control based on SpringBoot. It is suitable for lightweight and progressive projects.
date: 2023-09-10
class: heading_no_counter
---

In this section, we will explain how to simplify parameter validation using utility functions.

## Common Validation Functions

The utility class for validation is named `SVU` (SimpleValidateUtils). Here are some of the commonly used validation functions provided by `SVU`:

```java
// Checks if any of the provided arguments are null. Returns false if any argument is null.
public static boolean notNull(Object ...args);

// Checks if 'base' is within the range defined by 'start' and 'end' (inclusive). Returns true if it's within the range.
public static boolean range(Integer base, Integer start, Integer end);

// Checks if the length of 'content' is within the range defined by 'start' and 'end' (inclusive). Returns true if it's within the range.
public static boolean lengthRange(String content, Integer start, Integer end);

// Checks if 'content' matches the regular expression 'regex'. Returns true if it matches the regex.
public static boolean pattern(String content, String regex);

// Returns false if any of the provided boolean values is false.
public static boolean notFalse(boolean ...args);
```

## Multi-Parameter Validation

You can use the `notFalse` function to chain multiple validation conditions. If any of the conditions return `false`, the entire validation will fail.

For example:

```java
public Boolean fillUser(User user){
    return SVU.notFalse(
        SVU.lengthRange(user.getName(), 5, 16),
        SVU.range(user.getAge(), 1, 99)
    );
}
```

## Shortcut for Throwing Exceptions

Most functions in the `SVU` class have an overloaded version that includes a message parameter. Instead of returning `false` when validation fails, these functions throw a `ValidateException` with the provided message. This allows you to handle validation failures with more informative error messages.

For example:

```java
public Boolean fillUser(User user){
    return SVU.notFalse(
        SVU.lengthRange("Name length should be between 5 and 16", user.getName(), 5, 16),
        SVU.range("Age must be between 1 and 99", user.getAge(), 1, 99)
    );
}
```

You can catch the `ValidateException` and handle the validation message in a global exception handler:

```java
@ControllerAdvice
public class MyExceptionHandler {
    @ExceptionHandler(value = ValidateException.class)
    @ResponseBody
    public String exceptionHandler(ValidateException e){
        return e.getMessage();
    }
}
```

## Use Delay Functions for Improved Efficiency

Using the `notFalse` function as shown earlier computes all the validation conditions even if one of them returns `false`. To improve efficiency, you can use the `Delay` functions provided by `SVU`. 

The `notFalse` function has an overloaded version that accepts `BooleanCompute` objects. The `Delay` functions return a `BooleanCompute` object, encapsulating the validation condition, and delays its execution until necessary.

Here's an example:

```java
public Boolean fillUser(User user){
    return SVU.notFalse(
        SVU.lengthRangeDelay(user.getName(), 5, 16),
        SVU.rangeDelay(user.getAge(), 1, 99)
    );
}
```

In this example, if the first `lengthRangeDelay` function returns `false`, the second `rangeDelay` function won't be executed, resulting in improved efficiency.

## Optional Parameter Validation

If some parameters need to be validated only when they are not null, you can use the `SVOU` class (SimpleValidateOptionalUtil). The functions in `SVOU` have the same names as those in `SVU`, but they return true when the parameter is null.

Here's an example where the phone number is an optional parameter, and validation is required only when it's not null:

```java
public Boolean fillUser(User user){
    // The Regex class provides commonly used regular expressions.
    SVOU.pattern(user.getPhone(), Regex.CHINESE_MOBIL_PHONE_NUMBER);
    return SVU.notFalse(
        SVU.lengthRangeDelay(user.getName(), 5, 16),
        SVU.rangeDelay(user.getAge(), 1, 99)
    );
}
```

This code will only validate the phone number if it's not null.