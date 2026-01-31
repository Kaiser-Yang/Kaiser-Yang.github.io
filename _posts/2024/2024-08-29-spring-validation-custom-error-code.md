---
layout: post
title: Spring-Validation 自定义错误代码
date: 2024-08-29 10:38:09+0800
description:
tags:
  - 中文文章
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

在 [使用 `Spring-Validation` 进行参数校验](/blog/2024/spring-validation-intro) 中，我对如何使用
`Spring-Validation` 做了简单的介绍。但是只使用 `Spring-Validation` 还是有一些不足的，例如我们在登录的
时候需要检查用户名和密码是否正确，这时候 `Spring-Validation` 就很难实现了，我们往往需要自定义代码的
实现逻辑。

而对于自定义的校验没有通过的时候，我们也许要返回错误信息，如何将两者的错误信息进行统一是本文要讨论的
问题。

而如果只是单纯的进行统一，我们只需要返回错误信息即可进行统一。而仅仅返回错误信息是不够的，因为错误信息
往往是给用户查看的，对于开发者而言更多需要的是错误码，因为错误码不会因为语言的不同而改变，而且错误码
更加方便比较，这是因为对于错误信息而言，其中可能存在一些填入的参数，这样就可能导致错误信息不一致，
或者是在开发时书写错误信息的时候出现输入错误，而对于错误码我们可以很容易的避免这些问题。

# 返回信息
我们的基本要求是对于错误信息返回如下的 `JSON` 数据：

```json
{
  "code": 1,
  "message": "User id must be null when creating a new user"
}
```

除此之外，我们希望前端可以通过一个接口获取到所有的错误码以及简单描述，返回的数据应该具有以下的形式：

```json
{
  "1": "USERDTO_ID_NULL",
  "2": "USERDTO_ID_NOTNULL",
  "3": "USERDTO_USERNAME_SIZE",
  "4": "USERDTO_USERNAME_NOTBLANK",
  "5": "USERDTO_EMAIL_NOTBLANK",
  "6": "USERDTO_EMAIL_EMAIL",
  "7": "USERDTO_USERPASSWORD_SIZE",
  "8": "USERDTO_USERPASSWORD_NOTBLANK",
  "9": "USERSIGNINDTO_USERNAME_NOTBLANK",
  "10": "USERSIGNINDTO_USERPASSWORD_NOTBLANK",
  "11": "USERNAME_ALREADY_EXISTS",
  "12": "EMAIL_ALREADY_EXISTS",
  "13": "WRONG_SIGN_IN_INFORMATION",
  "14": "INVALID_TOKEN",
  "15": "ACCESS_DENIED",
  "16": "MESSAGE_CONVERSION_ERROR"
}
```

对于这样的数据，前端开发者可以很容易将其定义成一个 `enum` 类型，这样就可以很方便的进行错误码的比较。

# 自定义校验逻辑
对于自定义的校验逻辑的处理非常简单，我们只需要在校验没有通过的时候抛出一个异常即可，然后进行全局异常
的统一处理即可。
[使用 `Spring-Validation` 进行参数校验](/blog/2024/spring-validation-intro) 中介绍了如何设置全局异常处理器。

## 自定义错误码
当然，我们首先需要有错误码的信息，我们可以定义如下的 `enum` 类型：

```java
public enum ErrorCodeEnum {
    // This should be ignored, this is to make the ordinal of the enum start from 1
    ZERO_PLACEHOLDER,

    USERDTO_ID_NULL("UserDTO.id.Null"),
    USERDTO_ID_NOTNULL("UserDTO.id.NotNull"),
    USERDTO_USERNAME_SIZE("UserDTO.username.Size"),
    USERDTO_USERNAME_NOTBLANK("UserDTO.username.NotBlank"),
    USERDTO_EMAIL_NOTBLANK("UserDTO.email.NotBlank"),
    USERDTO_EMAIL_EMAIL("UserDTO.email.Email"),
    USERDTO_USERPASSWORD_SIZE("UserDTO.userPassword.Size"),
    USERDTO_USERPASSWORD_NOTBLANK("UserDTO.userPassword.NotBlank"),

    USERSIGNINDTO_USERNAME_NOTBLANK("UserSignInDTO.username.NotBlank"),
    USERSIGNINDTO_USERPASSWORD_NOTBLANK("UserSignInDTO.userPassword.NotBlank"),

    USERNAME_ALREADY_EXISTS("USERNAME_ALREADY_EXISTS"),
    EMAIL_ALREADY_EXISTS("EMAIL_ALREADY_EXISTS"),
    WRONG_SIGN_IN_INFORMATION("WRONG_SIGN_IN_INFORMATION"),

    INVALID_TOKEN("INVALID_TOKEN"),
    ACCESS_DENIED("ACCESS_DENIED"),

    MESSAGE_CONVERSION_ERROR("MESSAGE_CONVERSION_ERROR");

    // code means the error code in the message.properties
    private String code;

    ErrorCodeEnum(){}

    ErrorCodeEnum(String code) {
        this.code = code;
    }

    public String getCode() { return code; }
}
```

通常 `0` 代表执行成功，所以我们在上面的 `enum` 类型中加入了一个 `ZERO_PLACEHOLDER`，这样我们的错误码
就可以从 `1` 开始了。

为了能够让自己定义的错误也能使用 `MessageSource` 进行国际化处理，我们通过在 `enum` 中定义 `code` 将其
与 `message.properties` 中的错误码进行对应。这样我们只需要在 `message.properties` 添加对应的错误码即可：

```properties
# UserDTO validation messages
UserDTO.id.Null=User id must be null when creating a new user
UserDTO.id.NotNull=User id cannot be null
UserDTO.username.Size=Username must be between {min} and {max} characters
UserDTO.username.NotBlank=Username cannot be blank
UserDTO.email.NotBlank=Email cannot be blank
UserDTO.email.Email=Email must be a valid email address
UserDTO.userPassword.Size=Password must be between {min} and {max} characters
UserDTO.userPassword.NotBlank=Password cannot be blank

# UserSignInDTO validation messages
UserSignInDTO.username.NotBlank=Username cannot be blank
UserSignInDTO.userPassword.NotBlank=Password cannot be blank

USERNAME_ALREADY_EXISTS=Username already exists: {}
EMAIL_ALREADY_EXISTS=Email already exists: {}
WRONG_SIGN_IN_INFORMATION=Wrong sign in information

INVALID_TOKEN=Invalid token: {}
ACCESS_DENIED=Operation without privileges

MESSAGE_CONVERSION_ERROR=Error occurs while converting message
```

当然我们需要一个工具类来获取错误码的信息：

```java
// 标记 @Component 使其能够通过唯一的注解器对静态变量 MessageSource 进行注入
@Component
public class MessageSourceUtil {
    private static MessageSource messageSource;

    MessageSourceUtil(MessageSource messageSource) {
        MessageSourceUtil.messageSource = messageSource;
    }

    public static String getMessage(ErrorCodeEnum code, Object... args) {
        try {
            return messageSource.getMessage(code.getCode(), args, LocaleContextHolder.getLocale());
        } catch (Exception e) {
            // ignore
        }
        String message = messageSource.getMessage(code.getCode(), null, LocaleContextHolder.getLocale());
        Pattern pattern = Pattern.compile("\\{.*?\\}");
        Matcher matcher = pattern.matcher(message);
        int i = 0;
        while (matcher.find()) {
            message = message.replace(matcher.group(), args[i].toString());
            i++;
        }
        return message;
    }
}
```

之前我们在 [使用 `Spring-Validation` 进行参数校验](/blog/2024/spring-validation-intro) 中介绍了 `{min}`
、`{max}` 等不能通过 `MessageSource` 进行替换的参数，所以在这个工具类中，我们先尝试通过 `MessageSource`
获取信息，如果获取失败，我们就通过正则表达式进行替换。

最后我们需要自定义一个包含错误码的异常，用于在自定义校验出错的时候抛出给全局处理器统一处理：

```java
@Getter
@Setter
public class GenericException extends RuntimeException {
    private ErrorCodeEnum code;

    public GenericException(String message) {
        super(message);
    }

    public GenericException(ErrorCodeEnum code, Object... args) {
        super(MessageSourceUtil.getMessage(code, args));
        this.code = code;
    }
}
```

当然我们需要定义 `ErrorVO` 对象用于返回错误信息：

```java
public record ErrorVO(Integer code, String message) {
    public ErrorVO(ErrorCodeEnum errorCodeEnum, String message) {
        this(errorCodeEnum.ordinal(), message);
    }
}
```

经过上述操作后，我们便可以在全局的异常处理器中进行统一处理了：

```java
@ExceptionHandler(GenericException.class)
public ResponseEntity<ErrorVO> handleGenericException(
        GenericException e, HttpServletRequest request) {
    logger.error("Error caused by {}:\n {}", request.getRemoteAddr(), e.getMessage());
    switch (e.getCode()) {
        case INVALID_TOKEN:
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                    .body(new ErrorVO(e.getCode(), e.getMessage()));
        case ACCESS_DENIED:
            return ResponseEntity.status(HttpStatus.FORBIDDEN)
                    .body(new ErrorVO(e.getCode(), e.getMessage()));
        default:
            return ResponseEntity.status(HttpStatus.BAD_REQUEST)
                    .body(new ErrorVO(e.getCode(), e.getMessage()));
    }
}
```

# Spring Validation
在使用 `MessageSource` + `Spring Validation` 的时候，我们是通过
`@NotBlank(message = "{UserDTO.username.NotBlank}")` 来获取 `message.properties` 中的错误码的，
`{xxx.xxx.xxx}` 的语法是由 `Spring Validation` 提供的，而如果我们只是单纯这样进行注解的话，我们很难
获取到错误码的信息，而只能获取到错误信息 (当然可以通过比对字符串进行获取，但是由于存在参数占位符，
比较是困难的)，为了解决这个问题，我们可以迂回一下。

众所周知，`enum` 对于的 `valueOf` 方法可以通过名称来创建对应的 `enum` 对象，例如
`ErrorCodeEnum.valueOf("USERDTO_USERNAME_NOTBLANK")` 与 `ErrorCodeEnum.USERDTO_USERNAME_NOTBLANK` 是等价的，
而在 `Spring Validation` 中只会对 `{xxx.xxx.xxx}` 进行解析，而其他的部分将会保持不变，例如
`@NotBlank(message = "USERDTO_USERNAME_NOTBLANK {UserDTO.username.NotBlank}")` 解析后获取到的
错误信息是 `USERDTO_USERNAME_NOTBLANK Username cannot be blank`。这也就意味着我们只需要在使用注解的
地方多增加一个 `enum` 的名称即可：

```java
public record UserDTO(
    @Null(groups = CreateGroup.class, message = "USERDTO_ID_NULL {UserDTO.id.Null}")
    @NotNull(groups = UpdateGroup.class, message = "USERDTO_ID_NOTNULL {UserDTO.id.NotNull}")
    Long id,
    @Size(
            groups = {CreateGroup.class},
            min = ValidationConstant.MIN_USERNAME_LENGTH,
            max = ValidationConstant.MAX_USERNAME_LENGTH,
            message = "USERDTO_USERNAME_SIZE {UserDTO.username.Size}")
    @NotBlank(
            groups = {CreateGroup.class},
            message = "USERDTO_USERNAME_NOTBLANK {UserDTO.username.NotBlank}")
    String username,
    @Email(
            groups = {CreateGroup.class},
            message = "USERDTO_EMAIL_EMAIL {UserDTO.email.Email}")
    @NotBlank(
            groups = {CreateGroup.class},
            message = "USERDTO_EMAIL_NOTBLANK {UserDTO.email.NotBlank}")
    String email,
    @Size(
            groups = {CreateGroup.class},
            min = ValidationConstant.MIN_PASSWORD_LENGTH,
            max = ValidationConstant.MAX_PASSWORD_LENGTH,
            message = "USERDTO_USERPASSWORD_SIZE {UserDTO.userPassword.Size}")
    @NotBlank(
            groups = {CreateGroup.class},
            message = "USERDTO_USERPASSWORD_NOTBLANK {UserDTO.userPassword.NotBlank}")
    String userPassword) {}
```

当然着还没完，我们需要在全局异常处理器中对这种错误进行处理：

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<ErrorVO> handleMethodArgumentNotValidException(
        MethodArgumentNotValidException e, HttpServletRequest request) {
    // we only handle one validation message
    String codeAndMessage = e.getFieldError().getDefaultMessage();
    int firstSpaceIndex = codeAndMessage.indexOf(" ");
    // There must be a space and not at the end of the message
    assert firstSpaceIndex != -1;
    assert firstSpaceIndex != codeAndMessage.length() - 1;
    var exception = new GenericException(codeAndMessage.substring(firstSpaceIndex + 1));
    exception.setCode(ErrorCodeEnum.valueOf(codeAndMessage.substring(0, firstSpaceIndex)));
    return handleGenericException(exception, request);
}
```

再上面的处理中，我们提取到错误码后创建相应的 `GenericException` 对象，然后交给 `handleGenericException`
进行统一处理。

通过上面的操作后，我们已经成功对自定义校验逻辑以及 `Spring Validation` 进行了统一处理，而且我们还可以
通过 `MessageSource` 进行国陲化处理，这样我们就可以很方便的进行错误码的管理了。接下来简单介绍一下如何
返回错误码的信息。

# 返回错误码信息
我们只需要定义以下的 `controller` 即可：

```java
@RestController
// 标记只有在开发环境下才会被加载
@Profile(ApplicationConstant.DEV_PROFILE)
public class DevelopmentController {
    private Map<Integer, String> errorCodeConstant = new HashMap<>();

    // 这部分当然可以放入构造器中，放在这里是因为完整源代码中存在其他的逻辑
    @PostConstruct
    public void init() {
        for (ErrorCodeEnum code : ErrorCodeEnum.values()) {
            if (code == ErrorCodeEnum.ZERO_PLACEHOLDER) { continue; }
            errorCodeConstant.put(code.ordinal(), code.name());
        }
    }

    @GetMapping(ApiPathConstant.DEVELOPMENT_GET_ERROR_MESSAGE_API_PATH)
    public Map<Integer, String> getErrorMessage() {
        return errorCodeConstant;
    }
}
```

# 最后
上面的方法我个人认为并不是很优雅，但是对于我目前能想到的方法确实最好的，拓展性也不差，如果有更好的
方法，欢迎在评论区留言。
