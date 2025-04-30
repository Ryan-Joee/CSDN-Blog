# AOP编程的细节

## 一、AOP编程概述

**AOP（面向切面编程，Aspect-Oriented Programming）**是一种编程范式，旨在将程序中的关注点（如日志、事务、权限检查等）与业务逻辑分离，从而提高代码的模块化和可维护性。

AOP 主要通过“切面”（Aspect）来实现这一目标。切面是一些关注点的集合，它可以跨越多个类或方法。例如，**记录日志、性能监控、事务管理**等都可以作为切面进行处理，而不需要在每个方法中重复编写相同的代码。

**AOP 的核心概念**：

- **切面（Aspect）**：定义了横切关注点的代码（如日志记录、事务管理等）。

- **连接点（Joinpoint）**：程序中能够插入切面代码的地方。通常是方法的执行，例如方法调用、字段的访问等。

- **通知（Advice）**：切面中的具体操作，指明在什么时机执行特定的代码。根据执行时机的不同，通知有几种类型：

- **前置通知（Before）**：方法执行前执行的代码。

- **后置通知（After）**：方法执行后执行的代码。

- **环绕通知（Around）**：在方法执行前后都执行的代码，甚至可以决定是否执行该方法。

- **异常通知（AfterThrowing）**：方法抛出异常后执行的代码。

- **切入点（Pointcut）**：用于定义哪些连接点需要应用切面。切入点表达式通常用来指定在哪些方法或类上应用通知。

- **织入（Weaving）**：将切面应用到目标对象的过程。织入可以在编译时、类加载时或运行时进行。

**AOP 的优势**：

- **解耦：**将关注点从业务逻辑中分离，增强了模块化，避免了重复代码。

- **代码复用**：可以将通用功能（如日志、事务管理等）提取出来，供多个类复用。

- **增强可维护性：**在不修改原有代码的情况下，增强程序的功能。



## 二、三大通知演示和细节

### 1. **前置通知（Before）** 示例：

前置通知会在目标方法执行前被调用。

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;

@Aspect // 标注当前方法为切面
@Component // 纳入Spring IoC容器管理
public class BeforeAdvice {

    @Before("execution(* com.example.service.*.*(..))")  // 切入点，匹配com.example.service包下的所有方法
    public void beforeMethod() {
        System.out.println("前置通知：方法执行前");
    }
}
```

### 2. **后置通知（After）** 示例：

后置通知会在目标方法执行后被调用，不管目标方法是否抛出异常。

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.After;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class AfterAdvice {

    @After("execution(* com.example.service.*.*(..))")  // 切入点，匹配com.example.service包下的所有方法
    public void afterMethod() {
        System.out.println("后置通知：方法执行后");
    }
}
```

### 3. **环绕通知（Around）** 示例：

环绕通知在目标方法执行前后都可以进行操作，并且可以控制目标方法是否执行。

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.ProceedingJoinPoint;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class AroundAdvice {

    @Pointcut("execution(* com.example.service.*.*(..))")  // 定义切入点
    public void pointcut() {}

    @Around("pointcut()")  // 使用环绕通知
    public Object aroundMethod(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("环绕通知：方法执行前");

        Object result = joinPoint.proceed();  // 执行目标方法

        System.out.println("环绕通知：方法执行后");
        return result;
    }
}
```

### 4. 使用说明：

- **前置通知（Before）**：`@Before` 注解表示方法执行前触发。
- **后置通知（After）**：`@After` 注解表示方法执行后触发（无论是否抛出异常）。
- **环绕通知（Around）**：`@Around` 注解表示方法执行前后都触发，并且通过 `ProceedingJoinPoint` 控制目标方法的执行。`joinPoint.proceed()` 会继续执行目标方法。

### 5. 可以这么记忆：

- `@Before`：只能“提前说句话”，不能阻止你继续往前走。
- `@Around`：你是“守门员”，你可以决定**放不放人进去**，还可以在**人进来之后再检查一遍行李**。
- `@After` / `@AfterReturning`：只能“人在屋子里出来后”，做点“事后处理”，但无法影响中途执行。

### 6. 特别注意：环绕通知

- 当在切面中使用了**环绕通知@Around**时，必须调用**joinPoint.proceed(); **才能保证标注了切点处的目标方法会执行。

- 其次，如果目标方法有返回结果集，那么在环绕通知里必须返回**joinPoint.proceed(); **，这样目标方法返回的响应体结果集才会返回给前端。

