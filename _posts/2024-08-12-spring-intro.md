---
layout: post
title: The Brief Introduction of Spring
date: 2024-08-12 11:59:00+0800
last_updated: 2025-06-05 17:11:58+0800
description:
tags:
  - Spring
categories: Java
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

For the Spring Framework, there are two most important concepts: IoC and AOP.

This post will introduce these two concepts, some other important concepts in Spring
and some commonly used annotations in Spring.

## What is a Bean?

A bean is an object that is instantiated, assembled, and managed by a Spring IoC container.

## Bean and IoC

Without using Spring for development,
if you need to create an object, you usually create it with the `new` keyword.
In this case,
the creation and destruction of the object are controlled by the programmer (or your source code).
In contrast,
IoC refers to Inversion of Control,
which means that
the creation and destruction of objects are no longer controlled by the programmer
(or your source code).

In Spring,
the creation and destruction of objects are controlled by the Spring's IoC container.
Programmers only need to specify the configuration of the object,
and the Spring container will create or destroy the object automatically.

With IoC, we can finish some work easily.
For example, if we implement an interface with different implementations,
and then inject one implementation into the class that uses it.
We can easily switch between different implementations
without modifying the code of the class that uses it.

## Dependency Injection

Dependency Injection is a design pattern used to implement IoC.
It allows a class to receive its dependencies
from an external source rather than creating them itself.

## Use Annotations to Add a Bean

### `@Configuration`

`@Configuration` is an annotation used to specify that the current class is a configuration class,
which is equivalent to a `Spring` configuration `xml` file.
In the configuration class,
you can use the `@Bean` annotation to register beans into the `Spring` container.

### `@Import`

`@Import` annotation in Spring is used to include additional configuration classes or components
into the application context.
It allows you to modularize your configuration by combining multiple configuration sources
into a single context.

We rarely use this annotation in Spring Boot,
because Spring Boot automatically scans the package of the class
where `@SpringBootApplication` is located and its sub-packages.
We usually use `@Configuration` or `@Component` to register those classes as beans.

### `@Bean`

`@Bean` is an annotation used to specify that a method is a bean producer,
and the return value is a bean.

`@Bean` can specify the name of the bean.
If the name of the bean is not specified,
the name of the bean defaults to the method name.

### `@Named` and `@ManagedBean`

These two annotations are used to specify that the current **class** is a bean,
which are similar to `@Bean` but from `JSR330`.

### `@Order`

This is used to specify the order of beans in the IoC container.

**NOTE**: `@Order` can not be used to specify the injection priority of beans.

### `@Priority`

This is used to specify the priority of a bean.
Smaller values indicate higher priority.

When injecting beans,
the bean with the highest priority will be injected first.

### `@Primary` and `@Fallback`

`@Primary` is used to specify that the current bean is the primary bean,
which means that when there are multiple beans of the same type,
the bean marked with `@Primary` will be injected.

`@Fallback` is used to specify that the current bean is a fallback bean,
which means that when there are multiple beans of the same type,
the bean marked with `@Fallback` will not be injected.

**NOTE**: `@Primary` is higher priority than `@Priority`.

### `@DependsOn`

This annotation is used to mark this bean will depend on another bean,
which means the depended bean will be loaded before current bean.

### `@Lazy`

When marked with `@Lazy`, the bean will be loaded lazily, which means
it will be loaded when it is first needed.

### `@Component`

`@Component` annotation is used to specify that the current class is a bean.
The default name of the bean is the class name with the first letter in lowercase.

### `@Controller`

`@Controller` is similar to `@Component`, but it indicates a controller.

### `@RestController`

`@RestController` is a specialized version of `@Controller`.
It is used to indicate that the class is a controller
and that the methods in the class will return JSON or XML data.

This annotation is a combination of `@Controller` and `@ResponseBody`.

### `@Service`

`@Service` is similar to `@Component`, but it indicates a service.

### `@Repository`

`@Repository` is similar to `@Component`, but it indicates a repository (DAO).

### `@ComponentScan`

`@ComponentScan` is an annotation used to specify which classes will be scanned by Spring.
For example, `@ComponentScan("com.example")` indicates scanning `com.example`.

In most cases, we do not need to use this annotation in Sprint Boot.
This is because Spring Boot will automatically scan the package
where the class with `@SpringBootApplication` is located and its sub-packages.
This is because `@SpringBootApplication` contains the `@ComponentScan` annotation.

### `@Description`

`@Description` is an annotation used to add a description to a bean.
We can use `Spring`'s `BeanFactory` to get the description of the bean.

### `@Profile`

`@Profile` is an annotation used to specify the environment of a bean.
For example, `@Profile("dev")` indicates
that this bean will only be effective in the `dev` environment.

### `@Nullable`

`@Nullable` is an annotation used to mark a field that can be `null`.
This means that when `Spring` injects the bean,
if it does not find the corresponding bean,
it will not throw an exception.

### `@NonNull`

`@NonNull` is an annotation used to mark a field that cannot be `null`.
When `Spring` injects the bean and does not find the corresponding bean,
it will throw an exception.

## Bean Scopes

The scopes of Spring beans are as follows:

| Scope         | Description                                      |
| ------------- | ------------------------------------------------ |
| `singleton`   | The default, only one instance                   |
| `prototype`   | Every injection will create a new instance       |
| `request`     | Each `HTTP` request will create a new instance   |
| `session`     | Each `HTTP` session will create a new instance   |
| `application` | Each `ServletContext` will create a new instance |
| `websockt`    | Each `WebSocket` will create a new instance      |

You can use those annotations below to specify the scope of a bean:

- `@Scope("singleton")` or `@Singleton`
- `@Scope("prototype")` or `@Prototype`
- `@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)`
  or `@RequestScope`
- `@Scope(value = "session", proxyMode = ScopedProxyMode.TARGET_CLASS)`
  or `@SessionScope`
- `@Scope(value = "application", proxyMode = ScopedProxyMode.TARGET_CLASS)`
  or `@ApplicationScope`
- `@Scope(value = "websocket", proxyMode = ScopedProxyMode.TARGET_CLASS)`

For `request`, `session`, `application`, and `websocket`,
we must specify `proxyMode = ScopedProxyMode.TARGET_CLASS` when using `@Scope`.
This is because the life cycle of these scopes is shorter.
Singleton objects may depend on these objects,
and if `proxyMode` is not used,
the dependencies in the singleton may be expired.

## Autowiring

The autowiring is the process of automatically injecting dependencies into a bean.
Those below are the common autowiring modes in Spring:

| Mode     | Description                       |
| -------- | --------------------------------- |
| `byName` | Autowired by the name of the bean |
| `byType` | Autowired by the type of the bean |

### `@Autowired`

`@Autowired` is an annotation used to specify that a field, constructor, or method
is to be autowired.

By default, `@Autowired` will use `byType` to autowire the bean.
If there are multiple beans of the same type or none at all,
Spring will throw an exception.

When using `@Autowired` to an array or collection type field,
Spring will inject all matching beans, and the order of the beans
is determined by the priority of the beans.

If the `@Autowired` annotation is placed on a `Map` type field,
Spring will inject all matching beans,
with the bean name as the key and the bean as the value.

You can use `@Autowired(required = false)` to indicate that the dependency is optional.
This means that if the bean is not found,
Spring will not throw an exception.

If there are multiple beans of the same type,
the bean annotated with `@Primary` will be injected.

**NOTE**: As for Spring Framework 4.3,
you can omit the `@Autowired` annotation on a constructor,
if the class contains only one constructor.

**NOTE**: A non-required method will not be called at all if its dependency
(or one of its dependencies, in case of multiple arguments)
is not available.
A non-required field will not get populated at all in such cases,
leaving its default value in place.

### `@Qualifier`

You can use `@Qualifier` with `@Autowired` to specify the name of the bean to be injected.

### `@Inject`

`@Inject` is an annotation from `JSR330`, which is similar to `@Autowired` to some extent.

### `@Resource`

This annotation is from `JSR250`,
you can use this annotation to inject a bean by its name.
However, `@Resource` is only allowed to be used on fields and setter methods.
When the name is not explicitly specified,
if the annotation is placed on a field,
the name of the field will be used as the bean name.
If the annotation is placed on a setter method,
the name of the setter method (with `set` removed and the first letter in lowercase)
will be used as the bean name.

Besides, `@Resource` can also be used to inject a bean by its `type` property.

When only the `name` is specified,
`Spring` will inject the bean by its name.
When only the `type` is specified,
`Spring` will inject the bean by its type.
When both `name` and `type` are specified,
`Spring` will first try to inject the bean by its name;
if the bean is not found,
`Spring` will then try to inject the bean by its type.

### `@Value`

`@Value` is an annotation used to inject values into fields, methods, or constructor parameters.
Those below are the common usages of `@Value`:

- `@Value("#{}")`: [Spring Expression Language (SpEL)](https://docs.spring.io/spring-framework/reference/core/expressions.html)
- `@Value("${}")`: Properties

Before using `@Value`, you may need to write a configuration file.
The following is a brief introduction to how to read configuration files.
For a `Spring Boot` project,
after initialization,
there will be an `application.properties` file in the `src/main/resources` directory.
This file is the configuration file for `Spring Boot`,
and you can write some configuration information in this file.
Then, you can read it using the `@Value` annotation.
If this file does not exist, you can create it,
and `Spring` will automatically load the `application.properties` or `application.yml`
files under the following paths (the priority of these paths is from high to low):

1. `file:./config`
2. `file:./`
3. `classpath:/config/`
4. `classpath:/`

When there are both `application.properties` and `application.yml` files,
`application.properties` will override `application.yml`.

We can place the configuration file in the `src/main/resources` directory.
For example, we can place `XXXConfig.properties` in the `src/main/resources` directory,
and then we can read the configuration file using the `@PropertySource`:

```java
@Configuration
@PropertySource("classpath:XXXConfig.properties")
public class XXXConfig {}
```

After that, we can use `@Value` to read the configuration file.
As for Spring 6, we can assign `yml` files to `@PropertySource`
without specifying the `factory` attribute.

## Other Annotations

### `@NonNullApi`

`@NonNullApi` is an annotation used to mark that all parameters and return values
of the methods in the current package are not allowed to be `null`.
This annotation is usually placed above the `package` statement.

### `@NonNullFields`

`@NonNullFields` is an annotation used to mark that all fields in the current package
are not allowed to be `null`.
This annotation is usually placed above the `package` statement.

## AOP

AOP (Aspect-Oriented Programming) is a programming paradigm
that allows you to separate cross-cutting concerns from the main business logic.

In Spring, AOP is used to provide declarative transaction management,
logging, security, and other cross-cutting concerns.

AOP allows you to define aspects, which are reusable modules
that encapsulate cross-cutting concerns.

### `@Aspect`

This annotation is used to define a class as an aspect.

### `@PointCut`

This annotation is used to define a method as a pointcut.

This annotation can reduce the code duplication. For example,
if you want to add multiple aspect to a same pointcut,
you can use `@PointCut` to define the pointcut once,
then add the aspect to the name of the pointcut (the method name).

```java
@Aspect
@Component
public Class MyAspect {
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceLayer() {}

    @Before("serviceLayer()")
    public void beforeServiceLayer(JoinPoint joinPoint) {
        System.out.println("Before method: " + joinPoint.getSignature());
    }

    @After("serviceLayer()")
    public void afterServiceLayer(JoinPoint joinPoint) {
        System.out.println("After method: " + joinPoint.getSignature());
    }
}
```

### `@Before`

This annotation is used to define a method as a before advice.

### `@After`

This annotation is used to define a method as an after advice.

### `@Around`

This annotation is used to define a method as an around advice.

### `@AfterReturning`

This advice will be executed after the target method returns successfully,
which means it will not be executed if the target method throws an exception.

### `@AfterThrowing`

This advice will be executed after the target method throws an exception.

**NOTE**: When there is multiple advice for the same join point,
The order of execution is shown below:

- `@Around` (before `proceed()`)
- `@Before`
- Target method execution
- `@After`
- `@AfterReturning` or `@AfterThrowing`
- `@Around` (after `proceed()`)

## Transaction

### `@Transactional`

`@Transactional` is an annotation used to specify that a method or class is transactional.

You can specify the isolation level of the transaction using the `isolation` attribute.

There is a `propagation` attribute that specifies the transaction propagation behavior.
This attribute is used to specify how the transaction should behave
when it is called from another transaction. The possible values are:

- `Required`: The default behavior, if a transaction already exists,
  the method will run within that transaction; otherwise, a new transaction will be created.
  In this options, no matter the caller or callee throws an exception
  (even you have catch the exception thrown by the callee in the caller),
  the whole transaction will be rolled back.
- `Requires New`: A new transaction will always be created,
  and it is independent of the caller's transaction.
  In this options, if the caller throws an exception,
  the callee's transaction will not be affected;
  if the callee throws an exception and the caller catches it,
  the callee's transaction will be rolled back,
  but the caller's transaction will not be affected;
  if the callee throws an exception and the caller does not catch it,
  both the callee's and caller's transactions will be rolled back.
- `Nested`: A new transaction will be created,
  and it will be a nested transaction of the caller's transaction.
  In this options, if the caller throws an exception,
  both the callee's and caller's transactions will be rolled back;
  if the callee throws an exception and the caller catches it,
  the callee's transaction will be rolled back,
  but the caller's transaction will not be affected;
  if the callee throws an exception and the caller does not catch it,
  both the callee's and caller's transactions will be rolled back.
- `Supports`: If a transaction already exists, the method will run within that transaction;
  otherwise, it will run without a transaction.
- `Mandatory`: If a transaction already exists, the method will run within that transaction;
  Otherwise, an exception will be thrown.
- `Not Supported`: If a transaction already exists, it will be suspended,
  and the method will run without a transaction.
- `Never`: If a transaction already exists, an exception will be thrown;

### Lock in Transaction

When you want to use Java locks in Spring transaction,
you should be careful.

From Spring, we know that the Spring transaction is implemented by AOP,
and AOP is implemented by dynamic proxy. Be more specific, if a class has implemented
at least one interface, Spring will use JDK dynamic proxy to create a proxy object,
otherwise, Spring will use CGLIB to create a proxy object.

When you use `@Transactional` on a method, the method will be wrapped in a proxy object.
You should be aware that the sequence of the method calls is important. The lock may be released
before the transaction is committed, which may cause unexpected behavior.

1. Start a transaction.
1. Call the actual method.
1. Acquire a lock in the actual method.
1. Perform some operations on the database.
1. Release the lock in the actual method. (Other methods may acquire the lock to update)
1. Commit or rollback the transaction.

However, in some situations, you can use lock in transaction. For example,
if you want to update the shared data rather than the data in the database,
you can use lock in transaction.

1. Start a transaction.
1. Call the actual method.
1. Acquire a lock in the actual method.
1. Perform some operations on the shared data (such as a shared object).
1. Release the lock in the actual method. (It's OK)
1. Perform some operations on the database.
1. Commit or rollback the transaction.
