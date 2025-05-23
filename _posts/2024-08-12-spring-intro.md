---
layout: post
title: The Brief Introduction of Spring
date: 2024-08-12 11:59:00+0800
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
We usually use `@Configuration` or `Component` to register those classes as beans.

### `@Bean`

`@Bean` is an annotation used to specify that a method is a bean producer,
and the return value is a bean.

`@Bean` can specify the name of the bean.
If the name of the bean is not specified,
the name of the bean defaults to the method name.

### `@Named` and `@ManagedBean`

These two annotations are used to specify that the current class is a bean,
which are similar to `@Bean` but from `JSR330`.

### `@Order`, `@Priority`

Annotations for specifying the priorities of beans.

For `@Order` and `@Priority`, lower value means higher priority.

When injecting beans to collections, the beans with higher priorities will appear
at the beginning part.

### `@Primary` and `@Fallback`

`@Primary` is used to specify that the current bean is the primary bean,
which means that when there are multiple beans of the same type,
the bean marked with `@Primary` will be injected.

`@Fallback` is used to specify that the current bean is a fallback bean,
which means that when there are multiple beans of the same type,
the bean marked with `@Fallback` will not be injected.

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

| Scope         | Description |
| ---           | --- |
| `singleton`   | The default, only one instance |
| `prototype`   | Every injection will create a new instance |
| `request`     | Each `HTTP` request will create a new instance |
| `session`     | Each `HTTP` session will create a new instance |
| `application` | Each `ServletContext` will create a new instance |
| `websockt`    | Each `WebSocket` will create a new instance |

You can use those annotations below to specify the scope of a bean:

* `@Scope("singleton")` or `@Singleton`
* `@Scope("prototype")` or `@Prototype`
* `@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)`
or `@RequestScope`
* `@Scope("session", proxyMode = ScopedProxyMode.TARGET_CLASS)`
or `@SessionScope`
* `@Scope(value = "application", proxyMode = ScopedProxyMode.TARGET_CLASS)`
or `@ApplicationScope`
* `@Scope(value = "websocket", proxyMode = ScopedProxyMode.TARGET_CLASS)`

For `request`, `session`, `application`, and `websocket`,
we must specify `proxyMode = ScopedProxyMode.TARGET_CLASS` when using `@Scope`.
This is because the life cycle of these scopes is shorter.
Singleton objects may depend on these objects,
and if `proxyMode` is not used,
the dependencies in the singleton may be expired.

## Autowiring

The autowiring is the process of automatically injecting dependencies into a bean.
Those below are the common autowiring modes in Spring:

| Mode          | Description |
| ---           | --- |
| `byName`      | Autowired by the name of the bean |
| `byType`      | Autowired by the type of the bean |

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

* `@Value("#{}")`: [Spring Expression Language (SpEL)](https://docs.spring.io/spring-framework/reference/core/expressions.html)
* `@Value("${}")`: Properties

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

