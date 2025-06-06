---
layout: post
title: Spring Validation Introduction
date: 2024-08-23 10:24:05+0800
last_updated: 2025-06-05 21:21:30+0800
description:
tags:
  - Spring
  - Spring MVC
  - Spring Boot
  - Spring Validation
categories: Java
featured:
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

When developing `web` projects, we often need to validate the data received from the frontend.
As a backend developer, we should never trust the data sent by the frontend.

And we should try to avoid using a lot of `if else` statements for validation,
as it can make the code bloated and hard to maintain.

This post introduces how to use Spring Validation to validate data.

## JSR 303

Spring Validation supports the annotations defined in `JSR 303`.
We can use the annotations in JSR 303 to validate data.

JSR 303 annotations are easy to use.
We can add annotations to the properties of a `DTO` (Data Transfer Object) class,
and then use `@Valid` or `@Validated` annotations in the `Controller` to validate the data.

For example, when registering a user, we receive the username, password,
and email from the frontend.
We can create a `DTO` class like this:

```java
@Data
public class UserDTO {
    @Size(min = 1, max = 20, message = "Username length must be between 1 and 20 characters")
    private String username;

    @Email(message = "Email format is incorrect")
    @NotBlank(message = "Email cannot be blank")
    private String email;

    @Size(min = 6, max = 20, message = "Password length must be between 6 and 20 characters")
    private String userPassword;
}
```

The annotations used in the above example are self-explanatory.
We now just need to add `@Validated`
or `@Valid` annotation to the method parameter in the `Controller`:

```java
@Controller
@RequestMapping("/user")
public class UserController {
    @Autowired private UserService userService;

    @PostMapping
    public ResponseEntity<Void> createUser(@Validated @RequestBody UserDTO user) {}
}
```

Once it is finished,
the `Spring` framework will automatically validate the data received from the frontend.
And if the data does not meet the validation requirements,
it will throw a `MethodArgumentNotValidException` exception,
which will result in a `400` status code and a `JSON` formatted error message.
And we can customize the exception handler to return a more user-friendly error message.

## Custom Exception Handler

There are two types of exception handlers in Spring:
global exception handlers and local exception handlers.
Local exception handlers only handle exceptions in the current `Controller`,
while global exception handlers handle all exceptions.
Local exception handlers have a higher priority.

### Global Exception Handler

To define a global exception handler,
we can create a class and add the `@RestControllerAdvice` or `@ControllerAdvice` annotation
(Spring implement these by `AOP`):

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ResponseStatus(HttpStatus.BAD_REQUEST) // Specify the response status code
    @ExceptionHandler(MethodArgumentNotValidException.class) // Specify the exception type to handle
    public Map<String, String> handleMethodArgumentNotValidException(
        MethodArgumentNotValidException e
    ) {
        Map<String, String> errors = new HashMap<>();
        e.getBindingResult().getFieldErrors().forEach((error) -> {
            String fieldName = error.getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        return errors;
    }
}
```

In the above code,
we return a `Map` object containing the field names and error messages,
which make it possible to clearly see which field has an error in the response body.
For example, when the username is empty, we can see the following response body:

```json
{
  "username": "Username length must be between 1 and 20 characters"
}
```

### Local Exception Handler

If we want to handle exceptions only in a specific `Controller`,
we just need to define the exception handling method in that `Controller` class.

## Group Validation

In some cases, we may need to validate the same `DTO` object in different ways.
For example, when creating a user, we may not need to validate the `id` field,
but when updating a user, we need to validate the `id` field.
We can use group validation to solve this problem.

First, we need to define groups in the `DTO` object (here we use an inner class for demonstration):

```java
@Data
public class UserDTO {
    @NotNull(groups = Update.class)
    @Null(groups = Create.class)
    private Long id;

    @Size(
        groups = {Update.class, Create.class},
        min = 1, max = 20,
        message = "Username length must be between 1 and 20 characters")
    private String username;

    @Email(groups = {Update.class, Create.class}, message = "Email format is incorrect")
    @NotBlank(groups = {Update.class, Create.class}, message = "Email cannot be blank")
    private String email;

    @Size(
        groups = {Update.class, Create.class},
        min = 6, max = 20,
        message = "Password length must be between 6 and 20 characters")
    private String userPassword;

    public interface Update {}

    public interface Create {}
}
```

Then we can use the `@Validated` (we can not use `@Valid` here)
annotation to specify the validation group in the `Controller`:

```java
@Controller
@RequestMapping("/user")
public class UserController {
    @Autowired private UserService userService;

    @PostMapping
    public ResponseEntity<Void> createUser(
        @Validated(UserDTO.Create.class) @RequestBody UserDTO user
    ) {
        return null;
    }

    @PatchMapping
    public ResponseEntity<Void> updateUser(
        @Validated(UserDTO.Update.class) @RequestBody UserDTO user
    ) {
        return null;
    }
}
```

## `@Validated` VS `@Valid`

The difference between `@Validated` and `@Valid` is as follows:

* `@Valid` is defined in `JSR 303` while `@Validated` is provided by `Spring`.
* `@Validated` supports group validation while `@Valid` does not.
* `@Validated` can be used on classes while `@Valid` cannot.

**NOTE**: When using `@Validated` on a class, you can use both `JSR 303` and `Hibernate Validator`
annotations to validate method parameters.

## Annotations

Those annotations defined in `JSR 303` are very useful for validating data:

| Annotation | Description |
| --- | --- |
| `@Null` | Must be `null` |
| `@NotNull` | Must not be `null` |
| `@NotBlank` | Not empty, not `null`, and at least one non-whitespace character |
| `@AssertFalse` | Must be `false` |
| `@AssertTrue` | Must be `true` |
| `@Min` | Must be greater than or equal to the specified value |
| `@Max` | Must be less than or equal to the specified value |
| `@DecimalMin` | Similar to `@Min`, but for `Number` and `String` objects |
| `@DecimalMax` | Similar to `@Max`, but for `Number` and `String` objects |
| `@Digits` | Must be a valid number with specified integer and fraction digits |
| `@Past` | `Date` object must be before the time |
| `@Future` | `Date` object must be after the time |
| `@Size` | Size must be in the range |
| `@Pattern` | `String` must match the specified regular expression |

Those annotations below are defined in `Hibernate Validator`:

| Annotation | Description |
| --- | --- |
| `@Email` | `String` must be a valid email address |
| `@URL` | `String` must be a valid URL |

## `MessageSource`

In the examples before, we directly wrote the error messages in the annotations.
This approach is not conducive to development.
For example, when writing tests, we need to check if the error messages are consistent,
we have to write the error messages again.
And when modifying the error messages, we need to modify them in multiple places.

In order to solve this problem, we can use `MessageSource` to manage error messages.

First, we need to specify the location of the error message file in the `application.yml`
(or `application.properties`):

```yaml
spring:
  messages:
    basename: message/message
    encoding: UTF-8
```

This configuration indicates that our error message file is located
in the `classpath:message/` directory with `message` as base name.
And the file name is `message.properties` (we can only use `properties` files).

Now we need to create the error message file.
And we can put the error messages in this file:

```properties
# userDTO validation messages
Size.userDTO.username=Username must be between {2} and {1} characters
Email.userDTO.email=Email must be a valid email address
NotBlank.userDTO.email=Email cannot be blank
Size.userDTO.userPassword=Password must be between {2} and {1} characters
```

After this, we can use the `MessageSource` to get the error messages:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @Autowired private MessageSource messageSource;

    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Map<String, String> handleMethodArgumentNotValidException(
        MethodArgumentNotValidException e
    ) {
        Map<String, String> errors = new HashMap<>();
        e.getBindingResult().getFieldErrors().forEach((error) -> {
            String fieldName = error.getField();
            String errorMessage = messageSource.getMessage(error.getCodes()[0],
                error.getArguments(), LocaleContextHolder.getLocale());
            errors.put(fieldName, errorMessage);
        });
        return errors;
    }
}
```

In test cases, we can use the `MessageSource` to get the error messages as well:

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
public class UserControllerTest {
    @Autowired private MockMvc mvc;
    @Autowired private MessageSource messageSource;

    @Test
    public void testCreateUserInvalid() throws Exception {
        String user =
                """
                {
                    "name": "test",
                    "email": "invalid email address",
                    "userPassword": ""
                }
                """;
        mvc.perform(
            MockMvcRequestBuilders.post("/user")
                .contentType(MediaType.APPLICATION_JSON)
                .content(user))
            .andExpectAll(
                status().isBadRequest(),
                jsonPath(
                    "$.email",
                    equalTo(messageSource.getMessage("Email.userDTO.email",
                            null,
                            LocaleContextHolder.getLocale()))),
                jsonPath(
                    "$.userPassword",
                    equalTo(messageSource.getMessage(
                            "Size.userDTO.userPassword",
                            new Object[] {
                                null,
                                ConstantProperty.MAX_PASSWORD_LENGTH,
                                ConstantProperty.MIN_PASSWORD_LENGTH},
                            null))),
                content().contentType(MediaType.APPLICATION_JSON));
    }
```

## Path Variable and Request Parameter Validation

When we want to validate parameters obtained through `@PathVariable` and `@RequestParam`,
we must add the `@Validated` annotation to the controller class. This annotation is provided by
`Spring` to implement more flexible functionality than `JSR-303`.

In the documentation of `@Validated`, it states that:

> Applying this annotation at the method level allows for overriding the validation groups for
a specific method but does not serve as a pointcut; a class-level annotation is nevertheless
necessary to trigger method validation for a specific bean to begin with.

After adding the `@Validated` annotation to the controller class, we can then use
annotations such as `@Email`, `@NotBlank`, etc., to validate parameters obtained from
`@PathVariable` and `@RequestParam`:

```java
@Validated
@RestController
public class UserController {
    @Autowired private UserService userService;

    @GetMapping(ApiPathConstant.USER_CHECK_EMAIL_VALIDITY_API_PATH)
    public void checkEmailValidity(
            @Email(message = "USERDTO_EMAIL_EMAIL {UserDTO.email.Email}")
            @NotBlank(message = "USERDTO_EMAIL_NOTBLANK {UserDTO.email.NotBlank}")
            @RequestParam("email")
            String email) {
        QueryWrapper<UserPO> wrapper = new QueryWrapper<UserPO>();
        wrapper.eq("email", email);
        if (userService.exists(wrapper)) {
            throw new GenericException(ErrorCodeEnum.EMAIL_ALREADY_EXISTS, email);
        }
    }
}
```

In the above code, when requests reach the `checkEmailValidity` method,
Spring will automatically validate the `email` parameter.
It is worth noting that if the validation fails,
it will throw a `ConstraintViolationException` instead of a `MethodArgumentNotValidException`.
To handle this exception globally, we can create a global exception handler as follows:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<ErrorVO> handleConstraintViolationException(
            ConstraintViolationException e, HttpServletRequest request) {
        // do something
    }
}
```
### Internationalization

We can use `MessageSource` to support internationalization in our application.

Actually, we just need to create multiple `properties` files for different languages.
For example, we can create `message_en.properties` for English,
`message_zh_CN.properties` for Chinese, and so on.

When browsers send requests, they will include the `Accept-Language` header
on which Spring will automatically select the appropriate `properties` file
based.

## References

* [Validation with Spring Boot - the Complete Guide](https://reflectoring.io/bean-validation-with-spring-boot/#validating-path-variables-and-request-parameters)
