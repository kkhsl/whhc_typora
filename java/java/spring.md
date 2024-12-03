## 事务

以下是一些常见的 **Spring 事务管理** 面试题，涵盖了基础概念、配置方式、事务传播行为、隔离级别等方面，帮助你更好地准备 Spring 事务相关的面试问题：

### 1. **Spring事务是什么？**

**回答：** Spring 事务管理是一个在 Spring 框架中提供的机制，它通过声明式或编程式方式管理数据库操作的事务。Spring 的事务管理支持在不同类型的事务管理器（如 JDBC、JPA、Hibernate 等）下工作，确保数据一致性和可靠性。事务可以保证多个操作的原子性，要么全部成功，要么全部回滚。

------

### 2. **Spring事务是如何工作的？**

**回答：** Spring 通过 AOP（面向切面编程）实现事务管理。具体工作原理如下：

- **声明式事务管理**（基于注解或 XML 配置）：通过 `@Transactional` 注解，Spring 会自动在方法执行前后插入事务管理代码，开启、提交或回滚事务。
- **编程式事务管理**：手动管理事务（`TransactionTemplate` 或 `PlatformTransactionManager`），通常用于更细粒度的控制。

Spring 会通过代理对象拦截事务方法，打开事务，在方法执行完成后提交事务或发生异常时回滚事务。

### 3. **`@Transactional` 注解的常见属性是什么？**

**回答：** `@Transactional` 注解用于声明方法或类需要事务管理，常见属性包括：

- **`rollbackFor`**: 指定哪些异常会导致事务回滚。默认是 `RuntimeException` 和 `Error` 类型。
- **`noRollbackFor`**: 指定哪些异常不会导致事务回滚。
- **`propagation`**: 事务的传播行为，定义事务如何在方法调用时传递。常见选项包括：
  - `REQUIRED`（默认）：如果当前存在事务，加入该事务；否则创建一个新事务。
  - `REQUIRES_NEW`：总是开启一个新事务，并挂起当前事务。
  - `NESTED`：支持事务嵌套，内部事务有自己的提交和回滚。
  - `SUPPORTS`：如果当前有事务，则加入；没有事务则不启动事务。
  - `MANDATORY`：当前必须存在事务，否则抛出异常。
  - `NOT_SUPPORTED`：不使用事务，挂起当前事务。
  - `NEVER`：不使用事务，如果有事务则抛出异常。
- **`isolation`**: 事务的隔离级别，定义事务的执行是否受其他事务影响。常见的隔离级别包括：
  - `READ_UNCOMMITTED`：最低的隔离级别，允许脏读。
  - `READ_COMMITTED`（默认）：防止脏读，但允许不可重复读和幻读。
  - `REPEATABLE_READ`：防止脏读和不可重复读，但仍然可能出现幻读。
  - `SERIALIZABLE`：最高的隔离级别，防止脏读、不可重复读和幻读，所有事务串行执行。
- **`timeout`**: 事务的超时时间，单位是秒。如果事务执行超过此时间，将抛出 `TransactionTimedOutException`。
- **`readOnly`**: 设置为 `true` 时，标记事务为只读，优化数据库的性能。适用于查询操作。

------

### 4. **事务的传播行为有哪些？**

**回答：** 事务传播行为定义了在调用方法时，当前事务是否会传递给被调用方法。Spring 中常见的传播行为有：

- **`REQUIRED`**（默认）：如果当前有事务，加入该事务；如果没有事务，创建一个新事务。
- **`REQUIRES_NEW`**：总是创建一个新事务，挂起当前事务。
- **`NESTED`**：创建一个嵌套事务，如果外部事务回滚，嵌套事务也会回滚。
- **`SUPPORTS`**：如果当前存在事务，则加入该事务；否则不启动事务。
- **`MANDATORY`**：当前方法必须在事务中执行，如果没有事务，则抛出异常。
- **`NOT_SUPPORTED`**：不支持事务，如果当前方法有事务，则挂起该事务。
- **`NEVER`**：当前方法不能在事务中执行，如果有事务则抛出异常。

------

### 5. **事务的隔离级别有哪些？**

**回答：** 事务的隔离级别定义了一个事务在执行时，是否可以看到其他事务的中间数据。常见的隔离级别包括：

- **`READ_UNCOMMITTED`**：允许脏读，事务可以读取到其他事务未提交的数据。最低的隔离级别。
- **`READ_COMMITTED`**：防止脏读，但允许不可重复读和幻读。默认的隔离级别。
- **`REPEATABLE_READ`**：防止脏读和不可重复读，但允许幻读。
- **`SERIALIZABLE`**：最高的隔离级别，防止脏读、不可重复读和幻读，所有事务串行执行。

------

### 6. **什么是脏读、不可重复读和幻读？**

**回答：**

- **脏读（Dirty Read）**：一个事务可以读取另一个事务未提交的数据。这样可能会导致数据不一致。
- **不可重复读（Non-repeatable Read）**：一个事务在执行过程中读取的数据，另一事务在提交时修改了该数据，导致该事务读取到的值不一致。
- **幻读（Phantom Read）**：一个事务读取了某些数据后，另一事务插入了新数据，当第一个事务再次执行查询时，发现查询结果发生了变化。

------

### 7. **Spring 事务的回滚规则是什么？**

**回答：**

- 默认情况下，Spring 只会对 **`RuntimeException`** 和 **`Error`** 类型的异常进行回滚。
- 如果你希望某些 **`checked exceptions`** 也能够触发回滚，可以使用 `@Transactional(rollbackFor = Exception.class)` 来指定。
- 如果你希望某些异常不触发回滚，可以使用 `@Transactional(noRollbackFor = CustomException.class)` 来排除指定异常。

------

### 8. **如何解决事务丢失问题？**

**回答：** 事务丢失通常是因为数据库操作没有在同一个事务中执行，可能的原因包括：

- 事务管理器没有正确配置。
- 事务没有被正确传递到方法中。
- 手动捕获并处理了异常，没有重新抛出导致事务无法回滚。

解决方法：

- 确保 `@Transactional` 注解应用到正确的层次（服务层）。
- 使用 `TransactionAspectSupport.currentTransactionStatus().setRollbackOnly()` 手动回滚事务。
- 确保事务的传播行为设置正确。

------

### 9. **Spring 事务的事务管理器有哪些？**

**回答：** Spring 提供了多种事务管理器，可以根据使用的持久化框架选择：

- **`DataSourceTransactionManager`**：用于处理基于 JDBC 的事务。
- **`JpaTransactionManager`**：用于处理基于 JPA 的事务。
- **`HibernateTransactionManager`**：用于处理基于 Hibernate 的事务。
- **`ChainedTransactionManager`**：将多个事务管理器链接到一起，允许多种类型的事务管理器同时使用。

------

### 10. **Spring 事务管理器的配置方式有哪些？**

**回答：** Spring 事务管理器的配置方式有两种：

1. **基于注解的配置**：通过在服务类的方法上使用 `@Transactional` 注解来配置事务。
2. **基于 XML 的配置**：在 `applicationContext.xml` 或 `spring-config.xml` 中配置事务管理器及相关事务属性。

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>

<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="save*" propagation="REQUIRED" rollback-for="Exception"/>
    </tx:attributes>
</tx:advice>

<aop:config>
    <aop:pointcut id="transactionPointcut" expression="execution(* com.example.service.*.*(..))"/>
    <aop:advisor pointcut-ref="transactionPointcut" advice-ref="txAdvice"/>
</aop:config>
```

------

### 总结

准备 Spring 事务相关的面试题时，除了掌握事务的基本概念，还要深入理解 Spring 事务的传播行为、隔离级别、事务管理器配置、事务回滚等高级特性。实践中，通常会遇到需要处理事务的回滚规则、异常管理等问题，所以务必熟悉 `@Transactional` 注解的使用和常见的配置方法。



# Spring 动态代理

### Spring 动态代理

**Spring 动态代理**是 Spring 框架用来实现面向切面编程（AOP）的一种机制。它允许在程序运行时创建对象的代理，这些代理对象可以在不改变原有代码的情况下，为目标对象添加额外的功能。常见的功能包括事务管理、日志记录、权限控制、缓存等。

动态代理的核心思想是：通过代理对象来控制方法的执行，而不直接调用目标对象的方法。

### 动态代理的工作原理

Spring 的动态代理机制基于 **AOP**（面向切面编程），主要通过 **代理对象** 的方式来增强目标对象的功能。Spring 提供了两种类型的代理：

1. **JDK 动态代理**（基于接口的代理）
2. **CGLIB 代理**（基于类的代理）

### 1. **JDK 动态代理**

JDK 动态代理是基于 Java 原生的 `java.lang.reflect.Proxy` 类实现的，它要求目标对象至少实现一个接口。Spring 使用 JDK 动态代理生成一个接口类型的代理类，代理类会在运行时动态生成，并在目标对象的方法执行之前或之后执行增强逻辑。

#### JDK 动态代理的特点：

- 仅能为实现了接口的类生成代理。
- 代理对象与目标对象之间会共享相同的接口。

#### 示例：JDK 动态代理

```java
public interface UserService {
    void addUser(String name);
}

public class UserServiceImpl implements UserService {
    @Override
    public void addUser(String name) {
        System.out.println("Adding user: " + name);
    }
}

public class LoggingInvocationHandler implements InvocationHandler {
    private Object target;

    public LoggingInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Method " + method.getName() + " is being called");
        return method.invoke(target, args);
    }
}

public class Main {
    public static void main(String[] args) {
        UserService userService = new UserServiceImpl();
        UserService proxy = (UserService) Proxy.newProxyInstance(
                userService.getClass().getClassLoader(),
                userService.getClass().getInterfaces(),
                new LoggingInvocationHandler(userService)
        );

        proxy.addUser("John");
    }
}
```

输出：

```java
Method addUser is being called
Adding user: John
```

在上面的代码中，`LoggingInvocationHandler` 是一个自定义的 `InvocationHandler`，用于对方法执行做增强（如日志记录）。通过 `Proxy.newProxyInstance` 创建了一个代理对象 `proxy`，该代理对象会在调用 `addUser` 方法时，先执行日志打印逻辑，然后再执行目标方法。

#### JDK 动态代理的局限性：

- 只能代理实现了接口的类。无法直接为没有实现接口的类生成代理。

### 2. **CGLIB 代理**

CGLIB（Code Generation Library）是一个功能强大的字节码生成库，Spring 使用 CGLIB 来为没有实现接口的类生成动态代理。CGLIB 代理通过继承目标类来生成代理对象，这与 JDK 动态代理有所不同，JDK 动态代理通过接口来实现代理。

#### CGLIB 代理的特点：

- 可以为没有接口的类生成代理。
- 通过子类化（继承）目标类来生成代理，代理类会重写目标类的非 `final` 方法。
- 因为代理类继承目标类，因此目标类的 `final` 方法不能被代理。

#### 示例：CGLIB 代理

```java
public class UserService {
    public void addUser(String name) {
        System.out.println("Adding user: " + name);
    }
}

public class UserServiceInterceptor implements MethodInterceptor {
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("Method " + method.getName() + " is being called");
        return proxy.invokeSuper(obj, args); // 调用目标类的方法
    }
}

public class Main {
    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(UserService.class);
        enhancer.setCallback(new UserServiceInterceptor());

        UserService proxy = (UserService) enhancer.create();
        proxy.addUser("John");
    }
}
```

输出：

```
Method addUser is being called
Adding user: John
```

在上面的代码中，`Enhancer` 是 CGLIB 提供的类，用于生成代理类。`UserServiceInterceptor` 实现了 `MethodInterceptor` 接口，用来增强目标类的方法执行逻辑。

#### CGLIB 代理的局限性：

- CGLIB 生成代理类时，需要继承目标类，因此目标类不能是 `final` 类，且目标类的方法不能是 `final`。
- 相较于 JDK 动态代理，CGLIB 生成的代理类性能稍差。

### 3. **Spring AOP 中的代理类型选择**

在 Spring 中，AOP 会根据目标对象是否实现接口来选择使用哪种类型的代理：

- 如果目标对象实现了一个或多个接口，Spring 会使用 **JDK 动态代理**。
- 如果目标对象没有实现接口，Spring 会使用 **CGLIB 代理**（需要配置 CGLIB 依赖）。

#### 自定义代理类型：

可以通过设置 Spring 配置文件或注解来强制选择代理类型：

- **基于接口的代理**：使用 `@EnableAspectJAutoProxy(proxyTargetClass = false)`
- **基于类的代理**：使用 `@EnableAspectJAutoProxy(proxyTargetClass = true)`

#### 示例：选择 JDK 动态代理或 CGLIB

```java
@Configuration
@EnableAspectJAutoProxy(proxyTargetClass = false)  // JDK 动态代理
public class AppConfig {
}
@Configuration
@EnableAspectJAutoProxy(proxyTargetClass = true)  // CGLIB 代理
public class AppConfig {
}
```

### 4. **Spring AOP 中的动态代理**

在 Spring AOP 中，动态代理通过接口或者类生成代理对象。Spring 默认使用 JDK 动态代理来处理 **接口类型的代理**，如果目标类没有实现接口，则 Spring 会选择使用 **CGLIB 代理**。

#### 示例：Spring AOP 使用动态代理

```java
@Aspect
@Component
public class LoggingAspect {
    @Before("execution(* com.example.service.*.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println("Method " + joinPoint.getSignature().getName() + " is being called");
    }
}

@Service
public class UserService {
    public void addUser(String name) {
        System.out.println("Adding user: " + name);
    }
}
```

在这个例子中，`LoggingAspect` 是一个 AOP 切面，用于增强 `UserService` 中的方法。当 `addUser` 方法被调用时，`LoggingAspect` 会在方法执行前输出日志。

Spring AOP 会根据目标对象是否实现接口来决定是否使用 JDK 动态代理或 CGLIB 代理。

### 5. **总结**

- **动态代理** 是一种在运行时动态创建代理对象并增强目标对象的方法。
- **JDK 动态代理** 通过接口生成代理对象，适用于目标对象实现了接口的情况。
- **CGLIB 代理** 通过子类化目标类生成代理对象，适用于目标对象没有实现接口的情况。
- 在 Spring 中，动态代理主要用于 AOP（面向切面编程），为目标方法添加额外的功能（如事务、日志、缓存等）。



# 动态代理(面试题)

动态代理是 Java 中的一个重要概念，特别是在 AOP（面向切面编程）和 Spring 框架中应用广泛。在面试中，了解动态代理的原理、使用场景以及相关的优缺点是非常重要的。以下是一些常见的动态代理面试题，涵盖了理论基础和实际应用。

### 1. **什么是动态代理？**

**答案：**
动态代理是一种在运行时创建代理对象的技术。代理对象可以在不修改原始对象代码的情况下，增强目标对象的方法或行为。Java 中有两种主要的动态代理方式：

- **JDK 动态代理**：基于接口的代理，只能代理接口的方法。
- **CGLIB 动态代理**：基于继承的代理，可以代理类的方法，包括非接口方法。

### 2. **JDK 动态代理和 CGLIB 动态代理的区别是什么？**

**答案：**

- **JDK 动态代理**：是基于接口的代理，只能代理接口的方法。它使用 `java.lang.reflect.Proxy` 类和 `InvocationHandler` 接口实现。
- **CGLIB 动态代理**：是基于类的代理，通过生成目标类的子类来实现代理。它使用字节码增强技术，不能代理 `final` 类和 `final` 方法。

**优缺点对比：**

- JDK 动态代理：需要目标对象实现接口；性能较好，内存占用较小。
- CGLIB 动态代理：不要求目标对象实现接口，支持对类的增强，但内存占用较大，性能相对较差。

### 3. **JDK 动态代理如何工作？**

**答案：**
JDK 动态代理通过 `Proxy.newProxyInstance()` 方法创建代理对象，并使用 `InvocationHandler` 接口来定义代理逻辑。当代理对象的方法被调用时，实际调用的是 `InvocationHandler` 的 `invoke()` 方法。在 `invoke()` 方法中，可以实现增强逻辑，例如日志、事务管理等。

示例代码：

```java
interface Hello {
    void sayHello();
}

class HelloImpl implements Hello {
    public void sayHello() {
        System.out.println("Hello, world!");
    }
}

class MyInvocationHandler implements InvocationHandler {
    private Object target;

    public MyInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Before method call");
        Object result = method.invoke(target, args);
        System.out.println("After method call");
        return result;
    }
}

public class Main {
    public static void main(String[] args) {
        Hello hello = new HelloImpl();
        Hello proxy = (Hello) Proxy.newProxyInstance(
                hello.getClass().getClassLoader(),
                hello.getClass().getInterfaces(),
                new MyInvocationHandler(hello));

        proxy.sayHello();
    }
}
```

运行结果：

```
Before method call
Hello, world!
After method call
```

### 4. **动态代理的 `InvocationHandler` 的 `invoke()` 方法是如何工作的？**

**答案：**
`InvocationHandler` 接口的 `invoke()` 方法在代理对象的方法被调用时执行。它接收三个参数：

- `proxy`：代理对象本身。
- `method`：被调用的方法。
- `args`：方法的参数。

`invoke()` 方法的常见用途包括：

- 在调用目标方法之前做一些处理（如日志记录、权限检查、事务管理等）。
- 调用目标方法（通过反射）。
- 在调用目标方法之后做一些处理（如性能统计、结果缓存等）。

### 5. **动态代理能否用于非接口类的代理？**

**答案：**
**不能**，JDK 动态代理仅适用于接口代理。如果目标类没有实现接口，JDK 动态代理无法创建代理对象。这时可以使用 CGLIB 动态代理，CGLIB 通过继承目标类生成子类来进行代理。

### 6. **Spring 中的 AOP 是如何实现的？**

**答案：** Spring AOP 主要通过代理模式实现，默认使用 **JDK 动态代理**（如果目标类实现了接口）。如果目标类没有实现接口，则使用 **CGLIB 动态代理**。Spring AOP 使用 AOP 代理对象来处理切面（例如事务管理、日志等）。

- **JDK 动态代理**：代理的是接口。
- **CGLIB 动态代理**：代理的是类，通过子类继承实现。

### 7. **如何判断 Spring 使用的是 JDK 动态代理还是 CGLIB 动态代理？**

**答案：** 可以通过两种方式来判断 Spring 使用的是哪种代理：

- 如果目标类实现了接口，则 Spring 会使用 JDK 动态代理。
- 如果目标类没有实现接口，则 Spring 会使用 CGLIB 动态代理。

**查看代理类的类型：**

```java
System.out.println(proxy.getClass().getName());
```

- 如果输出类似 `com.sun.proxy.$Proxy1`，则说明使用的是 JDK 动态代理。
- 如果输出类似 `com.example.TargetClass$$EnhancerByCGLIB$$12345678`，则说明使用的是 CGLIB 动态代理。

### 8. **动态代理在 Spring 中的应用场景是什么？**

**答案：** 动态代理在 Spring 中主要用于 AOP（面向切面编程），应用场景包括：

- **事务管理**：通过动态代理将事务控制逻辑织入到目标方法中。
- **日志记录**：动态代理可以拦截目标方法的调用，记录日志。
- **权限检查**：在目标方法执行之前，检查是否有权限进行操作。
- **缓存**：动态代理可以在目标方法调用前后做缓存处理，避免重复计算。

### 9. **Spring AOP 中的 `@Around` 注解是如何使用的？**

**答案：** `@Around` 注解用于环绕通知，它可以在目标方法执行前后做增强。`@Around` 通常用于实现自定义的行为，例如事务管理、日志记录等。

示例：

```java
@Aspect
@Component
public class LogAspect {

    @Around("execution(* com.example.service.*.*(..))")
    public Object logMethodExecution(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("Method execution start");
        Object result = joinPoint.proceed();
        System.out.println("Method execution end");
        return result;
    }
}
```

### 10. **CGLIB 动态代理如何处理 `final` 方法？**

**答案：** CGLIB 动态代理通过生成目标类的子类来进行代理，但它无法代理 `final` 方法或 `final` 类。如果目标方法是 `final` 的，CGLIB 会抛出 `Cannot subclass final method` 异常。因此，CGLIB 代理不适用于含有 `final` 方法的类。

### 11. **如何避免代理类的多次调用？**

**答案：** 在 AOP 中，有时可能会遇到由于代理嵌套调用导致的递归问题。为了解决这个问题，可以使用 `@Transactional` 注解结合 `@Aspect` 或 `@Around` 注解来避免多次调用和重复处理。

### 12. **如何在 Spring 中使用 JDK 动态代理？**

**答案：** Spring 默认情况下会使用 JDK 动态代理进行接口代理，如果目标对象实现了接口，Spring 会为该接口创建代理。如果目标对象没有实现接口，Spring 会回退到 CGLIB 代理。

例子：

```java
@Service
public class MyService implements MyServiceInterface {
    public void myMethod() {
        System.out.println("My method executed");
    }
}

@Aspect
@Component
public class MyAspect {
    @Before("execution(* com.example.service.MyServiceInterface.*(..))")
    public void beforeMethod() {
        System.out.println("Before method");
    }
}
```

------

通过对这些问题的准备，可以帮助你在面试中展示你对 Java 动态代理及其在 Spring 中应用的理解，进而提升面试的表现。



# IOC（）

> - 控制反转
>   - ioc容器
>     - BeanDefinitionReader
>     - BeanFactory  + 反射 = 实例化
>   - 控制反转
>   - map集合存储bean对象
> - 依赖注入
>   - set注入
>   - 构造注入
> - ioc创建方式
>   - BeanFactory
>     - ApplicationContext
>       - ClassPathXmlApplicationContext：读取类路径下的xml
>       - FileSystemXmlApplicationContext：通过文件系统读取xml
>       - ConfigurableApplicatoinContext：关闭刷新上下文
>       - WebApplicationContext：基于web环境ioc容器对象
> - 生命周期
> - 注解
>   - @Component
>   - Respository
>   - Service
>   - Controller
