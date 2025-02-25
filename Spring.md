## 1. Bean 的生命周期

> Bean 的生命周期是 Spring 容器管理 Bean 从创建到销毁的整个过程，主要包括实例化、属性赋值、初始化、销毁四个阶段

### 1. 创建 Bean 实例

 Spring 容器会根据配置文件或注解信息，使用 Java 反射机制实例化 Bean。

### 2. 属性赋值和依赖注入

Spring 会解析 Bean 的属性，例如 @Autowired、@Value、@Resource 等注解，并通过构造方法或 setter 方法将依赖注入到 Bean 中。

### 3. 初始化 Bean

在这个阶段 Spring 会进行一些额外的处理：

- 如果 Bean 实现了 BeanNameAware、BeanClassLoaderAware 或 BeanFactoryAware 接口，Spring 会调用相应的 set 方法，将 Bean 名称、类加载器和 BeanFactory 传递给 Bean。
- 如果 BeanPostProcessor 处理器，Spring 会在初始化前调用 postProcessBeforeInitialization() 方法。
- 如果 Bean 实现了 InitializingBean 接口，则会执行 afterPropertiesSet() 方法。
- 如果在配置中指定了 init-method，Spring 会调用该方法执行自定义初始化逻辑。
- 初始化后，Spring 还会调用 BeanPostProcessor 的 postProcessAfterInitialization() 方法。

### 4. 销毁 Bean

当 Spring 容器关闭时，他会销毁 Bean

- 如果 Bean 实现了 DisposableBean 接口，则会执行 destroy() 方法。
- 如果配置中定义了 destroy-method，则会调用指定的销毁方法。
- 如果使用 @PreDestroy 注解标记了销毁前的方法，Spring 也会执行该方法释放资源。



## 2. Spring IoC

> Spring IoC（控制反转）是 Spring 框架核心机制之一，负责管理对象的创建、依赖关系和生命周期，从而实现组件解耦，提升代码可维护性和扩展性。

### 1. IoC 的核心思想

将对象管理权从应用程序代码中转移到 Spring 容器。

### 2. IoC 实现

- Spring IoC 主要通过依赖注入 DI 来实现。
- Spring 通过 XML 配置，Java 注解，或 Java 代码定义 @Bean及其依赖关系，容器运行时会自动解析并注入相应的对象。

### 3. Spring 工作流程

1. IoC 容器初始化：
   1. Spring 解析 XML 配置或注解，获取所有 Bean 的定义信息，生成**BeanDefinition**
   2. **BeanDefinition** 存储了 Bean 的基本信息（类名、作用域、依赖），并注册到 IoC 容器的 **BeanDefinitionMap**中
   3. 这个阶段完成了 IoC 容器的初始化，但还未实例化 Bean
2. Bean 实例化以及依赖注入
   1. Spring 通过反射实例化那些未设置**lazy-init**且是单例模式的 Bean
   2. 依赖注入发生在这个阶段，Spring 根据 BeanDefinition 解析 Bean 之间的依赖关系，并通过构造方法、setter 方法或字段注入（@Autowired）完成对象的注入。
3. Bean 的使用
   1. 业务代码可以通过 @Autowired 或者 BeanFactory.getBean() 获取 Bean
   2. 对于设置了**lazy-init**的Bean或者非单例 Bean，他们的实例化不会在 IoC 容器初始化时完成，而是在第一次调用 getBean() 时及逆行创建和初始化，且 Spring 不会长期管理他们。

### 4. Spring IoC 主要解决三个问题

1. 减低解耦：组件之间通过接口和依赖注入解耦，增强了代码的灵活性。
2. 简化对象管理：开发者无需手动创建对象，Spring 统一管理 Bean生命周期。
3. 提升维护性：当需要修改依赖关系时，只需调整配置，而无需修改业务代码。



## 3. 什么是动态代理

> 动态代理是一种**运行时动态生成代理对象**，并在代理对象中**增强目标对象方法**的技术。被广泛应用于 AOP、权限控制、日志记录等场景，时程序更加灵活。**动态代理可以通过 JDK 原生的 Proxy 机制或 CGLIB 方式实现。**

### 1. JDK 动态代理

基于接口，适用于代理实现了接口的对象，当使用 JDK 动态代理时，主要分为四步。

1. 定义接口：由于动态代理是基于接口进行代理的，因此目标对象必须实现接口。
2. 创建并实现 `InvocationHandler` 接口，并在 invoke 方法中定义增强逻辑。
3. 生成代理对象：使用 `Proxy.newProxyInstance` 创建代理对象，代理对象内部会调用 invoke 方法。
4. 调用代理方法：调用代理对象的方法时，invoke 方法会被触发，执行增强逻辑，并最调用目标方法。

### 2. CGLIB

通过子类继承目标类，适用于没有实现接口的类。

1. 通过 Enhancer **创建代理对象**
2. 设置父类：CGLIB 代理基于子类继承，因此代理对象是目标类的子类。
3. 定义并实现 `MethodInterceptor` 接口，在 intercept 方法中增强目标方法。
4. 调用代理方法，并调用代理方法时的方法时，intercept 方法会被出发，执行增强逻辑，并最终调用目标方法。



## 4. 动态代理与静态代理的区别

动态代理和静态代理都属于代理模式，它们都用于在**不修改目标对象代码的情况下增强其功能**。

### 1. 实现方式不同

- 静态代理需要手动编写代理类
- 动态代理在运行时生成代理类

### 2. 灵活性不同

- 静态代理不够灵活，代理类与目标类一一对应
- 动态代理更加灵活，适用于多种目标类

### 3. 维护成本不同

- 静态代理维护成本较高，因为每个目标类都需要一个代理类
- 动态代理维护成本较低，因为代理逻辑是通用的

### 4. 技术依赖不同

- 静态代理基于普通 Java 类实现
- 动态代理依赖于反射机制（JDK 动态代理）或字节码技术（CGLIB）

### 5. 适用场景不同

- 静态代理适用于简单的、目标类较少的场景
- 动态代理适合需要为多个目标类添加相同逻辑的场景。



## 5. Spring AOP 的执行流程

> Spring AOP（Aspect-Oriented Programming，面向切面编程）是一种通过代理机制实现方法增强的技术，它允许在不修改原始代码的情况下，对方法的执行过程进行扩展，如日志记录、事务管理、权限控制等。Spring AOP 主要依赖动态代理来实现，Spring AOP 的执行流程，主要分为六步。

### 1. 定义切面（Aspect）

可以使用 @Aspect 标注类，并在其中定义切点（Pointcut）和通知（Advice），如 @Before、@After、@Around 等。

### 2. 解析切点

Spring 会解析 @Poincut 表达式，确定需要增强的方法。

### 3. 创建代理对象

- 如果目标类实现了接口，Spring 使用 JDK 动态代理通过 Proxy.newProxyInstance 生成代理对象；
- 如果目标类没有实现接口，Spring 使用 CGLIB 动态代理，通过创建目标类的子类来生成代理对象。

### 4. 方法调用拦截

- JDK 动态代理，代理对象会拦截方法调用，并调用 InvovationHandler#invoke 方法，执行增强逻辑后，再调用目标方法；
- CGLIB 代理，代理对象则通过 MehtodInterceptor#intercept 代理方法调用，执行增强逻辑后，再调用目标方法。

### 5. 执行增强逻辑

根据通知类型，再方法执行前后或异常时，执行对应的 AOP 逻辑，如日志记录、事务提交等。

### 6. 执行目标方法

最终调用目标对象的方法，完成实际业务逻辑。



## 6. Spring 的事务什么情况下会失效

> Spring 的事务管理是 Spring 框架中非常重要的功能，它通过声明式事务（@Transaction 注解）编程式事务简化了事务的管理。然而在某些情况下，Spring 的事务可能会失效，导致事务无法正常混滚或提交。

### 1. 方法未被代理对象调用

- 当使用 @Transaction 注解时，Spring 会为目标类生成一个代理对象，并通过代理对象拦截方法调用以管理事务。
- 如果调用方法是通过类的**内部调用（ 即 this.method() ) 而不是通过代理对象调用时，则事务会失效。**这是因为代理对象无法拦截类内部的方法调用，导致事务逻辑未被执行。

### 2. 异常未被捕获或未触发回滚规则

- 当事务方法抛出异常时，Spring 默认只会在遇到 RuntimeException 或 Error 时回滚事务，而不会对**受检异常（Checked Exception）进行回滚。
- 如果开发者捕获了异常但未重新抛出，或未正确配置回滚规则（如通过 @Transaction(rollbackFor = Exception.class) )，事务也会失效。

### 3. 事务传播行为配置不当

- 当多个事务方法相互调用时，Spring 提供了多种事务传播行为（如 `REQUIRED`、`REQUIRES_NEW` 等）
- 如果传播行为配置不当，可能导致事务未按预期工作。例如：外部方法的事务传播行为未 `NOT_SUPPORTED` 或 `NEVER`，则内部方法的事务可能被挂起或完全不生效。

### 4. 数据库引擎不支持事务

- 当使用不支持事务的数据库引擎时，例如 MySQL 的 MyISAM 引擎不支持事务
- 即使代码中配置了事务管理，也无法生效，因此确保使用的数据库引擎（如 InnoDB）支持事务是非常重要的

### 5. 代理模式配置错误

- 当使用 @Transactional 注解时，Spring 默认使用 JDK 动态代理或 CGLIB 动态代理来管理事务。
- 如果目标类没有实现接口且未启用 CGLIB 代理（如未设置 proxyTargetClass = true），事务可能会失效。
- 此外，如果目标类被标记为 final 或方法被标记为 private，CGLIB 代理也无法完成，导致事务失效。

### 6. 事务管理器配置错误

- 当 Spring 容器中存在多个事务管理器时，如果未明确指定事务管理器（如通过 `@Transactional("transactionManagerName")`），可能导致事务管理器选择错误。这回导致事务无法正常工作，尤其是在多数据源场景下。

