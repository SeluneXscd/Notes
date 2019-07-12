# [Sping AOP-面向切面编程]([https://zh.wikipedia.org/wiki/%E9%9D%A2%E5%90%91%E5%88%87%E9%9D%A2%E7%9A%84%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1](https://zh.wikipedia.org/wiki/面向切面的程序设计))

关注点分离：旨在将**横切关注点**与业务主体进行进一步分离，以提高程序代码的模块化程度。即不同的问题交给不同的部分解决。

- 面向切面编程AOP（Aspect-oriented programming）
- 通用化功能代码的实现，对应的就是所谓的切面（Aspect）
- 业务功能代码和切面代码分开后，架构将变得高内聚低耦合
- 确保功能的完整性：切面最终要合并到业务中（Weave，织入）

## AOP的使用

### **AOP的三种织入方式**

- 编译时织入：需要特殊的Java编译器，如AspectJ
- 类加载时织入：需要特殊的Java编译器，如AspectJ和AspectWerkz
- 运行时织入：Spring采用的方式，通过动态代理的方式，实现简单

### **AOP的主要名词**

- Aspect：通用功能的代码实现
- Target：被织入Aspect的对象
- Join Point：可以被作为切入点的机会，所有的方法都可以作为切入点
- Pointcut：Aspect实际被应用在的Join Point，支持正则表达式
- Advice：通知，类里的方法以及这个方法如何织入到目标方法的方式
- Weaving：AOP的实现过程

**支持AOP应该先加入aop依赖**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

**RequestLogAspect.java** 设置Log日志切面

**@Pointcut注解修饰空方法的作用：**

定义一个方法, 用于声明切入点表达式. 一般地, 该方法中再不需要添入其他的代码。
使用 @Pointcut 来声明切入点表达式。
后面的其他通知直接使用方法名来引用当前的切入点表达式.。

```java
@Aspect // 标记切面
@Component // 标记组件
public class RequestLogAspect {

    // 定义一个Log
    private static final Logger logger = LoggerFactory.getLogger(RequestLogAspect.class);

    // 定义切面，正则表达式，com.selune.framework.web下的所有类的所有方法
    @Pointcut("execution(public * com.selune.framework.web..*.*(..))")
    public void webLog() {

    }

    // 方法运行前织入的通知
    @Before("webLog()")
    public void doBefore(JoinPoint joinPoint) {
        // 接收到请求，记录请求信息
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();

        // 记录下请求信息
        logger.info("URL= " + request.getRequestURL().toString());
        logger.info("IP= " + request.getRemoteAddr());
    }

    // 方法运行后织入的通知
    @AfterReturning(returning = "ret", pointcut = "webLog()")
    public void doAfterReturning(Object ret) {
        // 处理完请求，返回内容
        logger.info("Response= " + ret);
    }

}
```

**HelloController.java** Controller类

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    @ResponseBody
    public String hello() {
        String sentence = "Hello World";
        System.out.println(sentence);
        return sentence;
    }
}
```

**运行结果，console控制台**

```shell
2019-07-11 10:52:20.517  INFO 426 --- [nio-8080-exec-4] c.selune.framework.aop.RequestLogAspect  : URL= http://localhost:8080/hello
2019-07-11 10:52:20.518  INFO 426 --- [nio-8080-exec-4] c.selune.framework.aop.RequestLogAspect  : IP= 0:0:0:0:0:0:0:1
Hello World
2019-07-11 10:52:20.518  INFO 426 --- [nio-8080-exec-4] c.selune.framework.aop.RequestLogAspect  : Response= Hello World
```

### Advice的种类

- 前置通知 Before
- 后置通知 AfterRunning
- 异常通知 AfterThrowing
- 最终通知 After
- 环绕通知 Around

## AOP的原理

### AOP的实现

#### **JdkProxy(Jdk动态代理) 和 Cglib**

- 由AopProxyFactory根据AdvisedSupport对象的配置来决定
- 默认策略，如果目标类是接口，则用JdkProxy实现，否则用Cglib
- JdkProxy的核心：InvocationHandler接口和Proxy类
- Cglib：以继承的方式动态代理生成目标类的代理，修改字节码动态代理，被final标记的类，不能使用Cglib
- JdkProxy：通过Java的内部反射机制实现
- Cglib：借助ASM实现，ASM是一种能够操作字节码的框架
- 反射机制在生成类的过程中比较高效
- ASM在生成类之后的执行过程中比较高效，可以借助缓存，缓解ASM生成类的低效
- 代理模式：接口+真实实现类+代理类

#### **代理模式**

- 公用Payment接口

```java
public interface Payment {
    // 支付方法
    void pay();
}
```

- 用户支付类实现接口

```java
public class RealPayment implements Payment {
    @Override
    public void pay() {
        System.sout.println("用户支付，只关心支付功能，但不清楚内部");
    }
}
```

- AliPay代理接口

```java
public class AliPay implements Payment {
    // 代理接口需要调用Payment实例
    private Payment payment;
    
    /**
     * 构造方法
     */
    AliPay(Payment payment) {
        this.payment = payment;
    }
    
    /**
     * 代理类，需要调用Payment的方法
     */
    @Override
    public void pay() {
        beforePay();
        payment.pay();
        afterPay();
    }
    
    /**
     * 支付前
     */
    private void beforePay() {
        System.out.println("从银行取款");
    }
    
    /**
     * 支付后
     */
    private void afterPay() {
        System.out.println("打款给用户");
    }
}
```

- 主函数

```java
public class ProxyDemo {
    
    public static void main(String[] args) {
        Payment proxy = new AliPay(new RealPayment());
        proxy.pay();
    }
}
```

- 运行结果

```shell
从银行取款
作为用户，只关心支付功能
打款给用户
```

#### Spring里的代理模式的实现

- 真实实现类的逻辑包含在了getBean方法里
- getBean方法返回的实际上是Proxy的实例
- Proxy实例是Spring采用JdkProxy或Cglib动态生成的，SpringAOP只能作用于Spring容器的原因

#### Spring事务

- [ACID](https://zh.wikipedia.org/wiki/ACID)

  是指数据库管理系统(DBMS)在写入或更新资料的过程中，为保证事务(transaction)是正确可靠的，所必须具备的四个特性：

  - Atomicity（原子性）：一个事务（transaction）中的所有操作，或者全部完成，或者全部不完成，不会结束在中间某个环节。

    事务在执行过程中发生错误，会被回滚(数据管理))（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。即，事务不可分割、不可约简。

  - Consistency（一致性）：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设[约束](https://zh.wikipedia.org/wiki/数据完整性)、[触发器](https://zh.wikipedia.org/wiki/触发器_(数据库))、[级联回滚](https://zh.wikipedia.org/wiki/级联回滚)等。

  - Isolation（隔离性）：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括未提交读（Read uncommitted）、提交读（read committed）、可重复读（repeatable read）和串行化（Serializable）。

  - Durability（持久性）：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

  在数据库系统中，一个事务是指：由一系列数据库操作组成的一个完整的逻辑过程。

  例如银行转帐，从原账户扣除金额，以及向目标账户添加金额，这两个数据库操作的总和，构成一个完整的逻辑过程，不可拆分。

  这个过程被称为一个事务，具有ACID特性。

- [隔离级别](https://zh.wikipedia.org/wiki/事務隔離#隔离级别)

  为了获取更高的隔离等级，数据库系统的锁机制或者多版本并发控制机制都会影响并发。

  在很多数据库系统中，多数的数据库事务都避免高等级的隔离等级（如可串行化）从而减少对系统的锁定开销。
  
  - **可串行化**
  
    最高的隔离级别。
  
    在基于锁机制[并发控制](https://zh.wikipedia.org/wiki/并发控制)的DBMS实现[可串行化](https://zh.wikipedia.org/w/index.php?title=可串行化&action=edit&redlink=1)，要求在选定对象上的读锁和写锁保持直到事务结束后才能释放。在[SELECT](https://zh.wikipedia.org/wiki/Select_(SQL)) 的查询中使用一个“WHERE”子句来描述一个范围时应该获得一个“范围锁”（range-locks）。这种机制可以避免“幻读”（phantom reads）现象（详见下文）。
  
    当采用不基于锁的[并发控制](https://zh.wikipedia.org/wiki/并发控制)时不用获取锁。但当系统探测到几个并发事务有“写冲突”的时候，只有其中一个是允许提交的。这种机制的详细描述见“[快照隔离](https://zh.wikipedia.org/wiki/快照隔离)”

  - **可重复读**

  在可重复读（REPEATABLE READS）隔离级别中，基于锁机制并发控制的DBMS需要对选定对象的读锁（read locks）和写锁（write locks）一直保持到事务结束，但不要求“范围锁”，因此可能会发生“幻影读”。

  - **提交读**

  在提交读（READ COMMITTED）级别中，基于锁机制并发控制的DBMS需要对选定对象的写锁一直保持到事务结束，但是读锁在SELECT操作完成后马上释放（因此“不可重复读”现象可能会发生，见下面描述）。和前一种隔离级别一样，也不要求“范围锁”。

  - **未提交读**

  未提交读（READ UNCOMMITTED）是最低的隔离级别。允许“脏读”（dirty reads），事务可以看到其他事务“尚未提交”的修改。

- 事务传播

  - **PROPAGATION_REQUIRED** ，默认的spring事务传播级别，使用该级别的特点是，如果上下文中已经存在事务，那么就加入到事务中执行，如果当前上下文中不存在事务，则新建事务执行。所以这个级别通常能满足处理大多数的业务场景。

  - **PROPAGATION_SUPPORTS** ，从字面意思就知道，supports，支持，该传播级别的特点是，如果上下文存在事务，则支持事务加入事务，如果没有事务，则使用非事务的方式执行。所以说，并非所有的包在transactionTemplate.execute中的代码都会有事务支持。这个通常是用来处理那些并非原子性的非核心业务逻辑操作。应用场景较少。

  - **PROPAGATION_MANDATORY** ， 该级别的事务要求上下文中必须要存在事务，否则就会抛出异常！配置该方式的传播级别是有效的控制上下文调用代码遗漏添加事务控制的保证手段。比如一段代码不能单独被调用执行，但是一旦被调用，就必须有事务包含的情况，就可以使用这个传播级别。

  - **PROPAGATION_REQUIRES_NEW** ，从字面即可知道，new，每次都要一个新事务，该传播级别的特点是，每次都会新建一个事务，并且同时将上下文中的事务挂起，执行当前新建事务完成以后，上下文事务恢复再执行。
  
    这是一个很有用的传播级别，举一个应用场景：现在有一个发送100个红包的操作，在发送之前，要做一些系统的初始化、验证、数据记录操作，然后发送100封红包，然后再记录发送日志，发送日志要求100%的准确，如果日志不准确，那么整个父事务逻辑需要回滚。
	    怎么处理整个业务需求呢？就是通过这个PROPAGATION_REQUIRES_NEW 级别的事务传播控制就可以完成。发送红包的子事务不会直接影响到父事务的提交和回滚。
  
  - **PROPAGATION_NOT_SUPPORTED** ，这个也可以从字面得知，not supported ，不支持，当前级别的特点就是上下文中存在事务，则挂起事务，执行当前逻辑，结束后恢复上下文的事务。
  
    这个级别有什么好处？可以帮助你将事务极可能的缩小。我们知道一个事务越大，它存在的风险也就越多。所以在处理事务的过程中，要保证尽可能的缩小范围。比如一段代码，是每次逻辑操作都必须调用的，比如循环1000次的某个非核心业务逻辑操作。这样的代码如果包在事务中，势必造成事务太大，导致出现一些难以考虑周全的异常情况。所以这个事务这个级别的传播级别就派上用场了。用当前级别的事务模板抱起来就可以了。
  
  - **PROPAGATION_NEVER** ，该事务更严格，上面一个事务传播级别只是不支持而已，有事务就挂起，而PROPAGATION_NEVER传播级别要求上下文中不能存在事务，一旦有事务，就抛出runtime异常，强制停止执行！这个级别上辈子跟事务有仇。
  
  - **PROPAGATION_NESTED** ，字面也可知道，nested，嵌套级别事务。该传播级别特征是，如果上下文中存在事务，则嵌套事务执行，如果不存在事务，则新建事务。

