---
name: java-pattern-instructor
description: Use when asking how to implement GoF 23 design patterns in Java/Spring Boot projects, or needing guidance on pattern selection, implementation skeleton, or best practices for design patterns.
---

# Overview

## java-pattern-instructor

这个 skill 用于根据实际需求场景，指导如何正确选择和落地 GoF 23 种设计模式。

它的核心职责是：

- 帮助识别需求中的变化点与稳定点
- 指导选择最匹配的设计模式
- 提供符合 Spring Boot 项目的实现方案
- 指出常见落地误区和注意事项
- 给出最小必要实现的代码结构

这个 skill 的输出偏实现指导型：

- 先分析需求结构
- 再给出模式选择理由
- 然后提供实现骨架
- 最后说明扩展方向和避坑指南

---

# 定位区分

| skill | 职责 |
|-------|------|
| java-pattern-instructor（本 skill） | 从零开始实现设计模式，给出正确落地方式 |
| java-pattern-reviewer | 审查已有设计模式实现是否合理 |

**使用边界**：如果你已经有实现，在犹豫是否合理，用 java-pattern-reviewer；如果你还不知道怎么落地，用 java-pattern-instructor。

---

# 适用场景

## 1. 需求分析阶段

当你拿到一个新需求，需要判断是否需要上设计模式、上哪种模式时使用。

## 2. 技术方案设计

当你在设计技术方案，需要确认模式落点、层次划分、接口定义时使用。

## 3. 代码实现前

当你想确认实现思路是否正确，需要参考模式的标准实现骨架时使用。

## 4. 学习验证

当你想验证自己对某个设计模式的理解是否正确时使用。

---

# 输入方式

这个 skill 支持两种输入方式。

## 方式一：纯需求描述

提供自然语言需求，例如：

- 业务背景
- 需求目标
- 已知的变化点
- 已知的不变点

## 方式二：需求 + 已有方案

除了需求描述外，还提供你初步的技术方案或草稿，skill 会帮你验证方案是否合理并给出优化建议。

---

# 核心指导原则

## 1. 先识别再选择

**先确认需求中的三要素：**

- 稳定主干：什么是不变的
- 变化点：什么会扩展或变化
- 协作边界：谁会负责维护哪部分

**再匹配模式：**

- 变化点匹配不到具体模式时，优先轻量封装
- 变化点明确但模式不匹配时，不要强行模式化
- 只有 1-2 个变化分支时，优先 if-else 或简单策略注入

## 2. 模式选择的判断树

```
是否有多种可替换算法/策略？
  └─ YES → 策略模式 / 状态模式（看是否带状态迁移）
  └─ NO

主流程稳定但步骤有差异？
  └─ YES → 模板方法模式
  └─ NO

对象创建复杂或需要解耦创建者？
  └─ YES → 工厂模式（简单/抽象/工厂方法/建造者）
  └─ NO

需要把类接口转换成客户端期望的接口？
  └─ YES → 适配器模式
  └─ NO

需要让对象在内部状态变化时自动行为？
  └─ YES → 状态模式
  └─ NO

需要定义对象间一对多依赖关系？
  └─ YES → 观察者模式
  └─ NO

需要递归组合树形结构？
  └─ YES → 组合模式
  └─ NO

需要用对象封装替代类型码/分支？
  └─ YES → 桥接模式
  └─ NO

其他情况 → 优先不要上模式，先轻量实现
```

## 3. 层次落点原则

```
Controller 层
  └─ 不承载模式主逻辑
  └─ 只做参数校验、调用入口、返回结果

Business 层
  └─ 模式主入口
  └─ 策略分发入口
  └─ 模板方法骨架
  └─ 责任链装配
  └─ 状态流转入口

Service 层
  └─ 原子业务能力
  └─ 模式具体实现
  └─ 不承担流程编排

Mapper 层
  └─ 数据访问
  └─ 不承载任何模式逻辑
```

## 4. 抽象粒度原则

- 接口数量刚好：够用但不过度抽象
- 类职责单一：每个类只做一件事
- 避免空接口、空父类、过度工程化
- 优先组合，轻量依赖注入优于继承树

## 5. Spring 落地原则

- 优先通过 Spring 依赖注入实现解耦，而非工厂或 ServiceLocator
- 策略选择用 `Map<String, Strategy>` 或 `List<Handler>` 注入装配
- 注意 Spring 代理失效场景（自调用、事务边界）
- 分布式场景注意事务一致性边界

---

# GoF 23 种模式实现指导

## 创建型模式（5种）

### 1. 单例模式 Singleton

**适用场景：**
- 全局唯一配置对象
- 全局唯一资源访问器
- 无状态工具类

**实现指导：**

```java
// 推荐：枚举单例（防反射、防反序列化）
public enum RedisConfig {
    INSTANCE;

    private final JedisPool jedisPool;

    RedisConfig() {
        jedisPool = new JedisPool(...);
    }

    public JedisPool getJedisPool() {
        return jedisPool;
    }
}
```

```java
// 次选：饿汉式（如果确定需要提前初始化）
@Component
public class AppConfig {
    private static final AppConfig INSTANCE = new AppConfig();

    public static AppConfig getInstance() {
        return INSTANCE;
    }
}
```

```java
// 延迟初始化用双重检查锁
public class LazySingleton {
    private static volatile LazySingleton instance;

    public static LazySingleton getInstance() {
        if (instance == null) {
            synchronized (LazySingleton.class) {
                if (instance == null) {
                    instance = new LazySingleton();
                }
            }
        }
        return instance;
    }
}
```

**避坑：**
- 不要用 `getInstance()` 返回非线程安全对象
- Spring Bean 默认单例，不需要额外单例
- 枚举单例是 Java 最安全的单例实现

---

### 2. 工厂方法模式 Factory Method

**适用场景：**
- 创建逻辑可能扩展
- 将创建职责下沉到子类
- 解耦创建者与产品

**标准结构：**
```java
// 产品接口
public interface Product {
    void operation();
}

// 具体产品
public class ConcreteProduct implements Product {
    @Override
    public void operation() {
        // 具体实现
    }
}

// 创建者抽象
public abstract class Creator {
    // 工厂方法 - 子类实现
    protected abstract Product createProduct();

    // 业务方法
    public void businessMethod() {
        Product product = createProduct();
        product.operation();
    }
}

// 具体创建者
public class ConcreteCreator extends Creator {
    @Override
    protected Product createProduct() {
        return new ConcreteProduct();
    }
}
```

**避坑：**
- 简单场景优先用 Spring 依赖注入代替工厂
- 不要把业务逻辑塞进工厂
- 工厂方法适合扩展创建逻辑，不适合复杂对象构建（用建造者）

---

### 3. 抽象工厂模式 Abstract Factory

**适用场景：**
- 需要创建一族相关对象
- 产品族有多个维度变化
- 想提供库的同时隐藏具体实现类

**标准结构：**
```java
// 产品族 A 接口
public interface ProductA {
    void operationA();
}

// 产品族 B 接口
public interface ProductB {
    void operationB();
}

// 具体产品 A1
public class ConcreteProductA1 implements ProductA {}

// 具体产品 B1
public class ConcreteProductB1 implements ProductB {}

// 具体产品 A2
public class ConcreteProductA2 implements ProductA {}

// 具体产品 B2
public class ConcreteProductB2 implements ProductB {}

// 抽象工厂
public interface AbstractFactory {
    ProductA createProductA();
    ProductB createProductB();
}

// 具体工厂 1（产品族1）
public class ConcreteFactory1 implements AbstractFactory {
    @Override
    public ProductA createProductA() {
        return new ConcreteProductA1();
    }

    @Override
    public ProductB createProductB() {
        return new ConcreteProductB1();
    }
}

// 具体工厂 2（产品族2）
public class ConcreteFactory2 implements AbstractFactory {
    @Override
    public ProductA createProductA() {
        return new ConcreteProductA2();
    }

    @Override
    public ProductB createProductB() {
        return new ConcreteProductB2();
    }
}
```

**避坑：**
- 产品族稳定时才用抽象工厂
- 产品族扩展需要修改所有工厂实现
- 优先考虑工厂方法 + 策略组合

---

### 4. 建造者模式 Builder

**适用场景：**
- 对象创建需要多步骤
- 对象有大量可选参数
- 创建与表示分离

**标准结构：**
```java
// 目标对象
public class User {
    private final String name;       // 必填
    private final int age;           // 必填
    private final String email;      // 可选
    private final String phone;      // 可选
    private final String address;    // 可选

    // 私有构造
    private User(Builder builder) {
        this.name = builder.name;
        this.age = builder.age;
        this.email = builder.email;
        this.phone = builder.phone;
        this.address = builder.address;
    }

    // 静态内部 Builder
    public static class Builder {
        private final String name;   // 必填
        private final int age;       // 必填

        private String email;         // 可选
        private String phone;        // 可选
        private String address;      // 可选

        public Builder(String name, int age) {
            this.name = name;
            this.age = age;
        }

        public Builder email(String email) {
            this.email = email;
            return this;
        }

        public Builder phone(String phone) {
            this.phone = phone;
            return this;
        }

        public Builder address(String address) {
            this.address = address;
            return this;
        }

        public User build() {
            return new User(this);
        }
    }
}

// 使用
User user = new User.Builder("张三", 25)
    .email("zhangsan@example.com")
    .phone("13800138000")
    .build();
```

**Spring Boot 中的变体：**
```java
// 对于需要组装的 Bean，用 @Configuration + @Bean
@Configuration
public class MyBatisConfig {

    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSource);
        // 配置其他参数
        return factoryBean.getObject();
    }
}
```

**避坑：**
- 静态内部 Builder 适合类本身定义
- 独立 Builder 适合跨模块或复杂构建逻辑
- Lombok `@Builder` 是常见简化方式
- 不可变对象优先用 Builder

---

### 5. 原型模式 Prototype

**适用场景：**
- 对象创建成本高（数据库查询、远程调用）
- 对象结构复杂但需要复制
- 避免子类化创建过程

**标准结构：**
```java
// 实现 Cloneable 接口
public class PrototypeBean implements Cloneable {
    private String name;
    private DeepObject deepObject;  // 包含引用类型

    @Override
    public PrototypeBean clone() {
        try {
            PrototypeBean clone = (PrototypeBean) super.clone();
            // 深拷贝引用类型
            clone.deepObject = this.deepObject.clone();
            return clone;
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }
}

// 使用原型管理器
public class PrototypeManager {
    private Map<String, PrototypeBean> prototypes = new HashMap<>();

    public void register(String key, PrototypeBean prototype) {
        prototypes.put(key, prototype);
    }

    public PrototypeBean get(String key) {
        return prototypes.get(key).clone();
    }
}
```

**避坑：**
- 深拷贝/浅拷贝需要明确处理
- `Cloneable` 的 clone() 是浅拷贝
- 优先考虑序列化实现深拷贝
- Spring 中的 scope="prototype" 不是原型模式

---

## 结构型模式（7种）

### 6. 适配器模式 Adapter

**适用场景：**
- 集成第三方库/外部系统
- 接口不兼容但需要协同工作
- 统一多个不同接口

**标准结构：**

```java
// 目标接口（客户端期望的接口）
public interface Target {
    void request();
}

// 待适配类（第三方或遗留）
public class Adaptee {
    public void specificRequest() {
        System.out.println("Adaptee's specific request");
    }
}

// 类适配器（通过继承）
public class ClassAdapter extends Adaptee implements Target {
    @Override
    public void request() {
        specificRequest();  // 调用父类方法
    }
}

// 对象适配器（通过组合）
public class ObjectAdapter implements Target {
    private final Adaptee adaptee;

    public ObjectAdapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }

    @Override
    public void request() {
        adaptee.specificRequest();
    }
}
```

**Spring Boot 落地：**
```java
// 适配外部支付平台
@Component
public class PaymentAdapter implements PaymentGateway {

    private final ThirdPartyPaySDK thirdPartySDK;

    public PaymentAdapter(ThirdPartyPaySDK thirdPartySDK) {
        this.thirdPartySDK = thirdPartySDK;
    }

    @Override
    public PayResult pay(PayRequest request) {
        // 转换请求格式
        ThirdPartyPayRequest converted = convert(request);
        // 调用第三方
        ThirdPartyPayResponse response = thirdPartySDK.pay(converted);
        // 转换返回格式
        return convertResponse(response);
    }
}
```

**避坑：**
- 适配器只做接口转换，不承载业务逻辑
- 不要把业务判断塞进适配器
- 适配器是单向的，不要双向适配
- 优先对象组合，少用类继承

---

### 7. 装饰器模式 Decorator

**适用场景：**
- 动态添加额外职责
- 需要子类化扩展功能（但继承会爆炸）
- 可以在运行时装饰对象

**标准结构：**
```java
// 基础组件接口
public interface Component {
    String operation();
}

// 具体组件
public class ConcreteComponent implements Component {
    @Override
    public String operation() {
        return "ConcreteComponent";
    }
}

// 基础装饰器
public abstract class Decorator implements Component {
    protected final Component wrapped;

    public Decorator(Component component) {
        this.wrapped = component;
    }

    @Override
    public String operation() {
        return wrapped.operation();
    }
}

// 具体装饰器 A
public class ConcreteDecoratorA extends Decorator {

    public ConcreteDecoratorA(Component component) {
        super(component);
    }

    @Override
    public String operation() {
        return "DecoratorA(" + super.operation() + ")";
    }
}

// 具体装饰器 B
public class ConcreteDecoratorB extends Decorator {

    public ConcreteDecoratorB(Component component) {
        super(component);
    }

    @Override
    public String operation() {
        return "DecoratorB(" + super.operation() + ")";
    }
}

// 使用
Component component = new ConcreteDecoratorB(
    new ConcreteDecoratorA(new ConcreteComponent())
);
```

**Java IO 经典示例：**
```java
// BufferedReader 装饰 FileReader
BufferedReader reader = new BufferedReader(new FileReader("file.txt"));
// LineNumberReader 再装饰 BufferedReader
LineNumberReader lnr = new LineNumberReader(new BufferedReader(new FileReader("file.txt")));
```

**Spring Boot 落地：**
```java
// 装饰 Spring Bean 实现日志增强
@Component
@Primary
public class LoggingService implements OrderService {

    private final OrderService target;

    public LoggingService(OrderService target) {
        this.target = target;
    }

    @Override
    public Order createOrder(OrderDTO dto) {
        log.info("Creating order: {}", dto);
        Order result = target.createOrder(dto);
        log.info("Order created: {}", result.getId());
        return result;
    }
}
```

**避坑：**
- 装饰器顺序很重要
- 装饰器与被装饰对象实现同一接口
- 装饰器要委托给被装饰对象，不要自己做全部工作
- 不要过度装饰

---

### 8. 代理模式 Proxy

**适用场景：**
- 远程代理（RPC、Feign）
- 虚代理（延迟加载）
- 保护代理（权限控制）
- 智能引用（访问监控）

**标准结构：**
```java
// 接口
public interface Subject {
    void request();
}

// 真实主题
public class RealSubject implements Subject {
    @Override
    public void request() {
        System.out.println("RealSubject request");
    }
}

// 代理
public class Proxy implements Subject {
    private RealSubject realSubject;

    @Override
    public void request() {
        if (realSubject == null) {
            realSubject = new RealSubject();
        }
        // 前置处理
        before();
        realSubject.request();
        // 后置处理
        after();
    }

    private void before() {
        System.out.println("Before request");
    }

    private void after() {
        System.out.println("After request");
    }
}
```

**Spring AOP 代理：**
```java
// Spring 通过 AOP 自动创建代理
@Component
@Aspect
public class LoggingAspect {

    @Around("execution(* com.example.service.*.*(..))")
    public Object logAround(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("Before: {}", joinPoint.getSignature());
        Object result = joinPoint.proceed();
        log.info("After: {}", joinPoint.getSignature());
        return result;
    }
}
```

**Feign 远程代理：**
```java
// Feign 自动为接口创建代理实现
@FeignClient(name = "user-service", url = "http://localhost:8080")
public interface UserClient {
    @GetMapping("/user/{id}")
    User getUser(@PathVariable Long id);
}
```

**避坑：**
- Spring AOP 基于代理，有代理失效场景（自调用）
- 静态代理需要维护与目标对象相同的接口
- 动态代理（JDK Proxy）要求目标实现接口，cglib 不要求
- 代理对象与真实对象要区分清楚

---

### 9. 桥接模式 Bridge

**适用场景：**
- 多维度变化（抽象与实现均可独立扩展）
- 不希望继承导致类爆炸
- 实现与抽象可以绑定也可以解绑

**标准结构：**
```java
// 实现维度接口
public interface Color {
    String fill();
}

// 具体实现
public class Red implements Color {
    @Override
    public String fill() {
        return "Red";
    }
}

public class Blue implements Color {
    @Override
    public String fill() {
        return "Blue";
    }
}

// 抽象维度（使用组合而非继承实现）
public abstract class Shape {
    protected Color color;  // 桥接

    protected Shape(Color color) {
        this.color = color;
    }

    abstract void draw();
}

// 具体形状
public class Circle extends Shape {
    public Circle(Color color) {
        super(color);
    }

    @Override
    public void draw() {
        System.out.println("Drawing " + color.fill() + " circle");
    }
}

public class Square extends Shape {
    public Square(Color color) {
        super(color);
    }

    @Override
    public void draw() {
        System.out.println("Drawing " + color.fill() + " square");
    }
}

// 使用
Shape redCircle = new Circle(new Red());
Shape blueSquare = new Square(new Blue());
```

**Spring Boot 落地：**
```java
// 消息发送维度
public interface MessageSender {
    void send(String message);
}

// 短信发送
public class SmsSender implements MessageSender {
    @Override
    public void send(String message) {
        // 发送短信
    }
}

// 邮件发送
public class EmailSender implements MessageSender {
    @Override
    public void send(String message) {
        // 发送邮件
    }
}

// 消息类型抽象
public abstract class Notification {
    protected MessageSender sender;

    protected Notification(MessageSender sender) {
        this.sender = sender;
    }

    abstract void notifyUser(String message);
}

// 重要通知
public class UrgentNotification extends Notification {
    public UrgentNotification(MessageSender sender) {
        super(sender);
    }

    @Override
    void notifyUser(String message) {
        sender.send("[URGENT] " + message);
    }
}
```

**避坑：**
- 识别正确的两个独立变化维度
- 用组合代替继承
- 抽象与实现可以单独变化，不要过度设计

---

### 10. 组合模式 Composite

**适用场景：**
- 树形结构（菜单、机构、文件目录）
- 部分-整体层次结构
- 客户端忽略个体与整体的差异

**标准结构：**
```java
// 组件接口
public interface Component {
    void operation();
    default void add(Component component) {
        throw UnsupportedOperationException();
    }
    default void remove(Component component) {
        throw UnsupportedOperationException();
    }
    default Component getChild(int index) {
        throw UnsupportedOperationException();
    }
}

// 叶子节点
public class Leaf implements Component {
    private String name;

    public Leaf(String name) {
        this.name = name;
    }

    @Override
    public void operation() {
        System.out.println("Leaf: " + name);
    }
}

// 容器节点
public class Composite implements Component {
    private List<Component> children = new ArrayList<>();
    private String name;

    public Composite(String name) {
        this.name = name;
    }

    @Override
    public void operation() {
        System.out.println("Composite: " + name);
        for (Component child : children) {
            child.operation();
        }
    }

    @Override
    public void add(Component component) {
        children.add(component);
    }

    @Override
    public void remove(Component component) {
        children.remove(component);
    }
}
```

**Spring Boot 落地（菜单树）：**
```java
public interface MenuComponent {
    String getName();
    void display();
    BigDecimal getPrice();
}

@Service
public class MenuItem implements MenuComponent {
    private String name;
    private BigDecimal price;

    // getters, setters, display(), getPrice()
}

@Service
public class MenuCategory implements MenuComponent {
    private String name;
    private List<MenuComponent> children = new ArrayList<>();

    public void add(MenuComponent component) {
        children.add(component);
    }

    @Override
    public BigDecimal getPrice() {
        return children.stream()
            .map(MenuComponent::getPrice)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }

    // other methods...
}
```

**避坑：**
- 组合对象与叶子对象要有一致接口
- 子节点操作要递归到底
- 组合模式适合稳定树结构，不适合频繁增删结构

---

### 11. 外观模式 Facade

**适用场景：**
- 为复杂子系统提供统一入口
- 解耦客户端与子系统
- 分层架构中的服务编排层

**标准结构：**
```java
// 子系统 A
public class SubSystemA {
    public void methodA() {
        System.out.println("SubSystemA method");
    }
}

// 子系统 B
public class SubSystemB {
    public void methodB() {
        System.out.println("SubSystemB method");
    }
}

// 子系统 C
public class SubSystemC {
    public void methodC() {
        System.out.println("SubSystemC method");
    }
}

// 外观类
public class Facade {
    private SubSystemA a = new SubSystemA();
    private SubSystemB b = new SubSystemB();
    private SubSystemC c = new SubSystemC();

    public void operation() {
        a.methodA();
        b.methodB();
        c.methodC();
    }
}

// 使用
Facade facade = new Facade();
facade.operation();
```

**Spring Boot 落地：**
```java
// 对外统一服务入口
@Service
public class OrderFacade {

    private final OrderService orderService;
    private final InventoryService inventoryService;
    private final PaymentService paymentService;

    public OrderFacade(OrderService orderService,
                       InventoryService inventoryService,
                       PaymentService paymentService) {
        this.orderService = orderService;
        this.inventoryService = inventoryService;
        this.paymentService = paymentService;
    }

    @Transactional
    public OrderResult createOrder(OrderDTO dto) {
        // 库存检查
        inventoryService.checkAndDeduct(dto.getItems());

        // 支付扣款
        paymentService.deduct(dto.getUserId(), dto.getAmount());

        // 创建订单
        return orderService.create(dto);
    }
}
```

**避坑：**
- 外观不是把所有业务逻辑堆在一起
- 外观不替代 Service，仍需要分层
- 外观可以包含业务流程编排，但不要承载复杂业务判断

---

### 12. 享元模式 Flyweight

**适用场景：**
- 对象数量大且占用内存高
- 对象大部分状态可外部化
- 相同对象可共享

**标准结构：**
```java
// 享元接口
public interface Flyweight {
    void operation(String extrinsicState);
}

// 具体享元（内部状态）
public class ConcreteFlyweight implements Flyweight {
    private String intrinsicState;

    public ConcreteFlyweight(String intrinsicState) {
        this.intrinsicState = intrinsicState;
    }

    @Override
    public void operation(String extrinsicState) {
        System.out.println("Intrinsic: " + intrinsicState + ", Extrinsic: " + extrinsicState);
    }
}

// 享元工厂
public class FlyweightFactory {
    private Map<String, Flyweight> pool = new HashMap<>();

    public Flyweight getFlyweight(String key) {
        if (!pool.containsKey(key)) {
            pool.put(key, new ConcreteFlyweight(key));
        }
        return pool.get(key);
    }
}

// 使用
FlyweightFactory factory = new FlyweightFactory();
Flyweight fw1 = factory.getFlyweight("A");
Flyweight fw2 = factory.getFlyweight("A");  // 复用

fw1.operation("operation1");  // Intrinsic: A, Extrinsic: operation1
fw2.operation("operation2");  // Intrinsic: A, Extrinsic: operation2
```

**Spring Boot 落地（字符串常量池、连接池）：**
```java
// 数据库连接池（HikariCP 等已经是享元实现）
@Configuration
public class DataSourceConfig {

    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        config.setUsername("root");
        config.setMaximumPoolSize(10);  // 有限数量的连接复用
        return new HikariDataSource(config);
    }
}

// 线程池
@Bean
public ExecutorService executorService() {
    return Executors.newFixedThreadPool(10);  // 有限线程复用
}
```

**避坑：**
- 享元模式不适合小对象
- 内部状态与外部状态要区分清楚
- 线程安全问题要注意
- Spring 默认单例 Bean 已经是享元的思路

---

## 行为型模式（11种）

### 13. 策略模式 Strategy

**适用场景：**
- 多种算法/策略可替换
- 需要在运行时选择算法
- 消除大量条件分支

**标准结构：**
```java
// 策略接口
public interface Strategy {
    BigDecimal calculate(Order order);
}

// 具体策略 A
@Component
public class NormalDiscountStrategy implements Strategy {
    @Override
    public BigDecimal calculate(Order order) {
        return order.getAmount().multiply(new BigDecimal("0.9"));
    }
}

// 具体策略 B
@Component
public class VipDiscountStrategy implements Strategy {
    @Override
    public BigDecimal calculate(Order order) {
        return order.getAmount().multiply(new BigDecimal("0.7"));
    }
}

// 上下文
@Service
public class DiscountService {

    private final Map<String, Strategy> strategyMap;

    public DiscountService(List<Strategy> strategies) {
        this.strategyMap = strategies.stream()
            .collect(Collectors.toMap(
                s -> s.getClass().getSimpleName(),
                s -> s
            ));
    }

    public BigDecimal discount(Order order, String strategyType) {
        Strategy strategy = strategyMap.get(strategyType);
        if (strategy == null) {
            throw new IllegalArgumentException("Unknown strategy: " + strategyType);
        }
        return strategy.calculate(order);
    }
}
```

**Spring Boot 落地：**
```java
@Configuration
public class StrategyConfig {

    @Bean
    public Map<String, PaymentStrategy> paymentStrategyMap(
            List<PaymentStrategy> strategies) {
        return strategies.stream()
            .collect(Collectors.toMap(
                s -> s.getType(),  // 每个策略有自己的 type
                s -> s
            ));
    }
}

@Service
public class PaymentService {

    private final Map<String, PaymentStrategy> strategyMap;

    public PaymentService(Map<String, PaymentStrategy> strategyMap) {
        this.strategyMap = strategyMap;
    }

    public void pay(Order order, String paymentType) {
        PaymentStrategy strategy = strategyMap.get(paymentType);
        strategy.pay(order);
    }
}
```

**避坑：**
- 策略接口要稳定，不要经常变
- 策略之间不要互相调用
- 策略选择逻辑不要放在 Client
- 简单场景不要硬上策略，Spring 注入可能就够了

---

### 14. 模板方法模式 Template Method

**适用场景：**
- 主流程稳定
- 部分步骤需要子类实现或扩展
- 算法骨架不变但具体步骤可变

**标准结构：**
```java
// 抽象模板类
public abstract class AbstractTemplate {

    // 模板方法
    public final void process() {
        step1();
        step2();
        step3();
        hook();  // 钩子方法，可选
    }

    // 通用步骤
    protected void step1() {
        System.out.println("Step 1");
    }

    // 需要子类实现的抽象方法
    protected abstract void step2();

    // 具体步骤
    protected void step3() {
        System.out.println("Step 3");
    }

    // 钩子方法（默认空实现）
    protected void hook() {
        // 可选扩展点
    }
}

// 具体实现
public class ConcreteTemplate extends AbstractTemplate {

    @Override
    protected void step2() {
        System.out.println("Concrete step 2");
    }

    @Override
    protected void hook() {
        System.out.println("Hook implementation");
    }
}
```

**Spring Boot 落地（JDBC 模板）：**
```java
// Spring JdbcTemplate 本身就是模板方法模式
@Service
public class UserDao {

    private final JdbcTemplate jdbcTemplate;

    public UserDao(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public User findById(Long id) {
        return jdbcTemplate.queryForObject(
            "SELECT * FROM users WHERE id = ?",
            new Object[]{id},
            (rs, rowNum) -> {
                User user = new User();
                user.setId(rs.getLong("id"));
                user.setName(rs.getString("name"));
                return user;
            }
        );
    }
}
```

**MyBatis 抽象：**
```java
public abstract class BaseService<T> {

    protected abstract T mapper(T record);

    @Transactional
    public boolean save(T record) {
        return mapper(record) > 0;
    }

    public T getById(Long id) {
        return getBaseMapper().selectById(id);
    }
}
```

**避坑：**
- 不要用继承实现所有方法
- 主流程不稳定时不要硬套模板方法
- 优先用组合而非继承
- 钩子方法不要暴露太多

---

### 15. 责任链模式 Chain of Responsibility

**适用场景：**
- 多个对象处理请求，形成链
- 请求发送者与接收者解耦
- 链可以动态配置

**标准结构：**
```java
// 处理器接口
public interface Handler {
    void setNext(Handler handler);
    void handle(Request request);
}

// 抽象基类
public abstract class AbstractHandler implements Handler {
    private Handler next;

    @Override
    public void setNext(Handler handler) {
        this.next = handler;
    }

    @Override
    public void handle(Request request) {
        if (doHandle(request)) {
            return;
        }
        if (next != null) {
            next.handle(request);
        }
    }

    protected abstract boolean doHandle(Request request);
}

// 具体处理器 A
@Component
public class ValidationHandler extends AbstractHandler {

    @Override
    protected boolean doHandle(Request request) {
        if (request.getData() == null) {
            throw new IllegalArgumentException("Data is null");
        }
        return false;  // 继续传递
    }
}

// 具体处理器 B
@Component
public class AuthHandler extends AbstractHandler {

    @Override
    protected boolean doHandle(Request request) {
        if (request.getToken() == null) {
            throw new IllegalArgumentException("Token is required");
        }
        return false;
    }
}

// 装配
@Configuration
public class HandlerConfig {

    @Bean
    public Handler handlerChain(ValidationHandler v, AuthHandler a, BusinessHandler b) {
        v.setNext(a);
        a.setNext(b);
        return v;
    }
}
```

**Spring Boot 落地（过滤器链）：**
```java
// Servlet Filter 链
@Component
public class MyFilter {

    @Autowired
    private Handler handlerChain;

    @Override
    protected void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
        Request req = new Request(request);
        handlerChain.handle(req);
        chain.doFilter(request, response);
    }
}

// WebMVC 拦截器
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new AuthInterceptor())
            .addPathPatterns("/api/**")
            .excludePathPatterns("/api/public/**");
    }
}
```

**避坑：**
- Handler 必须真正独立，不能共享状态
- 链路过长要警惕，可能设计有问题
- Handler 之间不要循环依赖
- 链是顺序执行，并发场景要注意

---

### 16. 状态模式 State

**适用场景：**
- 对象行为取决于内部状态
- 状态转换规则复杂
- 消除大量条件分支

**标准结构：**
```java
// 状态接口
public interface State {
    void handle(Context context);
    State getState();  // 获取当前状态标识
}

// 具体状态 A
public class StateA implements State {

    @Override
    public void handle(Context context) {
        System.out.println("StateA handling");
        context.setState(new StateB());  // 状态转换
    }

    @Override
    public State getState() {
        return this;
    }
}

// 具体状态 B
public class StateB implements State {

    @Override
    public void handle(Context context) {
        System.out.println("StateB handling");
        context.setState(new StateA());
    }

    @Override
    public State getState() {
        return this;
    }
}

// 上下文
public class Context {
    private State state;

    public Context(State initialState) {
        this.state = initialState;
    }

    public void setState(State state) {
        this.state = state;
    }

    public void request() {
        state.handle(this);
    }
}
```

**Spring Boot 落地（订单状态机）：**
```java
// 订单状态
public enum OrderStatus {
    PENDING, PAID, SHIPPED, DELIVERED, CANCELLED
}

// 订单状态接口
public interface OrderStateHandler {
    OrderStatus getStatus();
    void handle(OrderContext context);
}

// 状态处理器
@Service
public class PaidOrderHandler implements OrderStateHandler {

    @Override
    public OrderStatus getStatus() {
        return OrderStatus.PAID;
    }

    @Override
    public void handle(OrderContext context) {
        Order order = context.getOrder();
        // 执行已支付状态的处理逻辑
        order.setPaidTime(LocalDateTime.now());
        context.save(order);
    }
}

// 状态机服务
@Service
public class OrderStateMachine {

    private final Map<OrderStatus, OrderStateHandler> handlers;

    public OrderStateMachine(List<OrderStateHandler> handlers) {
        this.handlers = handlers.stream()
            .collect(Collectors.toMap(OrderStateHandler::getStatus, h -> h));
    }

    public void transition(Order order, OrderStatus targetStatus) {
        OrderStateHandler handler = handlers.get(targetStatus);
        if (handler == null) {
            throw new IllegalStateException("No handler for status: " + targetStatus);
        }
        OrderContext context = new OrderContext(order);
        handler.handle(context);
    }
}
```

**避坑：**
- 状态转换规则要集中管理
- 状态类之间不要直接调用
- 主入口不要再有大量 if-else 判断状态
- 简单状态变化用枚举 + 策略就够了

---

### 17. 观察者模式 Observer

**适用场景：**
- 一对多依赖关系
- 当对象变化时需要通知其他对象
- 解耦发布与订阅

**标准结构：**
```java
// 主题接口
public interface Subject {
    void attach(Observer observer);
    void detach(Observer observer);
    void notifyAllObservers();
}

// 观察者接口
public interface Observer {
    void update(Subject subject);
}

// 具体主题
public class ConcreteSubject implements Subject {
    private List<Observer> observers = new ArrayList<>();
    private int state;

    public int getState() {
        return state;
    }

    public void setState(int state) {
        this.state = state;
        notifyAllObservers();
    }

    @Override
    public void attach(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void detach(Observer observer) {
        observers.remove(observer);
    }

    @Override
    public void notifyAllObservers() {
        for (Observer observer : observers) {
            observer.update(this);
        }
    }
}

// 具体观察者
public class ConcreteObserver implements Observer {
    private String name;

    public ConcreteObserver(String name) {
        this.name = name;
    }

    @Override
    public void update(Subject subject) {
        ConcreteSubject s = (ConcreteSubject) subject;
        System.out.println(name + " received update, state: " + s.getState());
    }
}
```

**Spring Boot 落地（ApplicationEvent）：**
```java
// 事件
public class OrderCreatedEvent extends ApplicationEvent {
    private final Order order;

    public OrderCreatedEvent(Object source, Order order) {
        super(source);
        this.order = order;
    }

    public Order getOrder() {
        return order;
    }
}

// 事件发布者
@Service
@RequiredArgsConstructor
public class OrderService {

    private final ApplicationEventPublisher eventPublisher;

    public void createOrder(OrderDTO dto) {
        Order order = saveOrder(dto);
        // 发布事件
        eventPublisher.publishEvent(new OrderCreatedEvent(this, order));
    }
}

// 事件监听器
@Component
public class OrderNotificationListener {

    @EventListener
    @Async  // 异步处理
    public void handleOrderCreated(OrderCreatedEvent event) {
        Order order = event.getOrder();
        // 发送通知
        sendNotification(order);
    }
}
```

**避坑：**
- 监听器不要反向修改发布者对象
- 注意事务边界，事件发布在事务提交后
- 异步监听要做好异常处理
- 不要把同步主流程拆成异步监听

---

### 18. 命令模式 Command

**适用场景：**
- 请求封装成对象
- 支持撤销/重做操作
- 日志记录、事务管理
- 队列请求

**标准结构：**
```java
// 命令接口
public interface Command {
    void execute();
    void undo();  // 可选撤销
}

// 具体命令
public class ConcreteCommand implements Command {
    private Receiver receiver;

    public ConcreteCommand(Receiver receiver) {
        this.receiver = receiver;
    }

    @Override
    public void execute() {
        receiver.action();
    }

    @Override
    public void undo() {
        receiver.undoAction();
    }
}

// 接收者
public class Receiver {
    public void action() {
        System.out.println("Receiver action");
    }

    public void undoAction() {
        System.out.println("Receiver undo");
    }
}

// 调用者
public class Invoker {
    private Command command;

    public void setCommand(Command command) {
        this.command = command;
    }

    public void executeCommand() {
        command.execute();
    }

    public void undoCommand() {
        command.undo();
    }
}
```

**Spring Boot 落地（任务队列）：**
```java
// 命令接口
public interface Command {
    void execute();
    String getCommandType();
}

// 具体命令
@Component
public class SendSmsCommand implements Command {

    private final SmsService smsService;
    private SmsRequest request;

    public SendSmsCommand(SmsService smsService) {
        this.smsService = smsService;
    }

    public void setRequest(SmsRequest request) {
        this.request = request;
    }

    @Override
    public void execute() {
        smsService.send(request);
    }

    @Override
    public String getCommandType() {
        return "SMS";
    }
}

// 命令处理器
@Service
public class CommandHandler {

    private final Map<String, Command> commandMap;

    public CommandHandler(List<Command> commands) {
        this.commandMap = commands.stream()
            .collect(Collectors.toMap(Command::getCommandType, c -> c));
    }

    public void handle(String type, Object request) {
        Command command = commandMap.get(type);
        if (command instanceof SendSmsCommand) {
            ((SendSmsCommand) command).setRequest((SmsRequest) request);
        }
        command.execute();
    }
}
```

**避坑：**
- 命令要支持序列化（用于日志、队列）
- 撤销/重做要正确维护状态
- 命令粒度要合适，不要过细拆分
- 简单操作不需要命令模式

---

### 19. 迭代器模式 Iterator

**适用场景：**
- 遍历聚合对象不暴露内部结构
- 支持多种遍历方式
- 为聚合对象提供统一访问接口

**标准结构：**
```java
// 迭代器接口
public interface Iterator<T> {
    boolean hasNext();
    T next();
}

// 聚合接口
public interface Aggregate<T> {
    Iterator<T> createIterator();
}

// 具体迭代器
public class ConcreteIterator<T> implements Iterator<T> {
    private final List<T> list;
    private int index = 0;

    public ConcreteIterator(List<T> list) {
        this.list = list;
    }

    @Override
    public boolean hasNext() {
        return index < list.size();
    }

    @Override
    public T next() {
        if (hasNext()) {
            return list.get(index++);
        }
        return null;
    }
}

// 具体聚合
public class ConcreteAggregate<T> implements Aggregate<T> {
    private final List<T> list = new ArrayList<>();

    public void add(T item) {
        list.add(item);
    }

    @Override
    public Iterator<T> createIterator() {
        return new ConcreteIterator<>(list);
    }
}
```

**Java 内置迭代器：**
```java
// Java 集合框架已经实现了迭代器模式
List<String> list = Arrays.asList("a", "b", "c");
Iterator<String> iterator = list.iterator();

while (iterator.hasNext()) {
    String item = iterator.next();
    System.out.println(item);
}

// for-each 语法
for (String item : list) {
    System.out.println(item);
}
```

**Spring Boot 落地：**
```java
// 自定义集合遍历
@Service
public class MenuIteratorService {

    public void printMenu(MenuComponent root) {
        Iterator<MenuComponent> iterator = root.createIterator();
        while (iterator.hasNext()) {
            MenuComponent component = iterator.next();
            component.display();
        }
    }
}
```

**避坑：**
- Java 内置集合已实现迭代器，不需要自己写
- 自定义迭代器要支持 fail-fast
- 迭代器不要在遍历时修改集合

---

### 20. 中介者模式 Mediator

**适用场景：**
- 对象间通信复杂，形成网状结构
- 想减少对象间直接依赖
- 集中管理对象间交互逻辑

**标准结构：**
```java
// 中介者接口
public interface Mediator {
    void notify(Colleague colleague, String event);
}

// 具体同事
public abstract class Colleague {
    protected Mediator mediator;

    public Colleague(Mediator mediator) {
        this.mediator = mediator;
    }
}

// 具体同事 A
public class ConcreteColleagueA extends Colleague {

    public ConcreteColleagueA(Mediator mediator) {
        super(mediator);
    }

    public void doSomething() {
        System.out.println("ColleagueA does something");
        mediator.notify(this, "A_done");
    }
}

// 具体同事 B
public class ConcreteColleagueB extends Colleague {

    public ConcreteColleagueB(Mediator mediator) {
        super(mediator);
    }

    public void doSomething() {
        System.out.println("ColleagueB does something");
        mediator.notify(this, "B_done");
    }

    public void reactOnA() {
        System.out.println("ColleagueB reacts to A");
    }
}

// 具体中介者
public class ConcreteMediator implements Mediator {
    private ConcreteColleagueA a;
    private ConcreteColleagueB b;

    public void setColleagues(ConcreteColleagueA a, ConcreteColleagueB b) {
        this.a = a;
        this.b = b;
    }

    @Override
    public void notify(Colleague colleague, String event) {
        if (colleague == a && "A_done".equals(event)) {
            b.reactOnA();
        }
    }
}
```

**Spring Boot 落地：**
```java
// Spring MessageBroker 就是中介者
@Service
public class ChatRoomService {

    private final SimpMessagingTemplate template;

    public void sendMessage(String roomId, String user, String message) {
        // 消息由 broker 分发给所有订阅者
        template.convertAndSend("/topic/room/" + roomId, new ChatMessage(user, message));
    }
}

// 或者使用 @MessageMapping
@Controller
public class ChatController {

    @MessageMapping("/chat/{roomId}")
    @SendTo("/topic/room/{roomId}")
    public ChatMessage chat(@DestinationVariable String roomId, ChatMessage message) {
        return message;
    }
}
```

**避坑：**
- 中介者不要变成上帝对象
- 同事对象之间不应该完全不知道彼此
- 简单场景优先直接调用

---

### 21. 备忘录模式 Memento

**适用场景：**
- 保存对象状态快照
- 支持撤销/回滚功能
- 不破坏封装性

**标准结构：**
```java
// 备忘录（快照）
public class Memento {
    private final String state;

    public Memento(String state) {
        this.state = state;
    }

    public String getState() {
        return state;
    }
}

// 原发器
public class Originator {
    private String state;

    public Memento save() {
        return new Memento(state);
    }

    public void restore(Memento memento) {
        this.state = memento.getState();
    }

    public String getState() {
        return state;
    }

    public void setState(String state) {
        this.state = state;
    }
}

// 负责人
public class Caretaker {
    private Memento memento;

    public void setMemento(Memento memento) {
        this.memento = memento;
    }

    public Memento getMemento() {
        return memento;
    }
}
```

**Spring Boot 落地：**
```java
// 草稿状态保存
@Service
public class DraftService {

    private final Map<Long, DraftMemento> drafts = new ConcurrentHashMap<>();

    public void saveDraft(Long userId, Order order) {
        DraftMemento memento = new DraftMemento(order);
        drafts.put(userId, memento);
    }

    public Order getDraft(Long userId) {
        DraftMemento memento = drafts.get(userId);
        return memento != null ? memento.getOrder() : null;
    }

    // 内部备忘录
    private static class DraftMemento {
        private final Order order;
        private final LocalDateTime savedAt;

        DraftMemento(Order order) {
            this.order = order;
            this.savedAt = LocalDateTime.now();
        }

        Order getOrder() {
            return order;
        }
    }
}
```

**避坑：**
- 备忘录要支持序列化
- 状态快照不要过于频繁
- 备忘录要隔离原发器与负责人

---

### 22. 解释器模式 Interpreter

**适用场景：**
- 特定领域语言（DSL）
- 规则引擎
- 简单语法解析

**标准结构：**
```java
// 表达式接口
public interface Expression {
    boolean interpret(String context);
}

// 终结符表达式
public class TerminalExpression implements Expression {
    private String data;

    public TerminalExpression(String data) {
        this.data = data;
    }

    @Override
    public boolean interpret(String context) {
        return context.contains(data);
    }
}

// 非终结符表达式 - And
public class AndExpression implements Expression {
    private Expression expr1, expr2;

    public AndExpression(Expression expr1, Expression expr2) {
        this.expr1 = expr1;
        this.expr2 = expr2;
    }

    @Override
    public boolean interpret(String context) {
        return expr1.interpret(context) && expr2.interpret(context);
    }
}

// 非终结符表达式 - Or
public class OrExpression implements Expression {
    private Expression expr1, expr2;

    public OrExpression(Expression expr1, Expression expr2) {
        this.expr1 = expr1;
        this.expr2 = expr2;
    }

    @Override
    public boolean interpret(String context) {
        return expr1.interpret(context) || expr2.interpret(context);
    }
}
```

**Spring Boot 落地（SpEL 或规则引擎）：**
```java
// Spring Expression Language
@Service
public class RuleEvaluator {

    public boolean evaluate(String expression, Map<String, Object> context) {
        ExpressionParser parser = new SpelExpressionParser();
        Expression exp = parser.parseExpression(expression);
        return exp.getValue(context, Boolean.class);
    }
}

// 使用
@Service
@RequiredArgsConstructor
public class DiscountService {

    private final RuleEvaluator evaluator;

    public BigDecimal calculate(Order order) {
        Map<String, Object> context = new HashMap<>();
        context.put("orderAmount", order.getAmount());
        context.put("userLevel", order.getUser().getLevel());

        String rule = "orderAmount > 100 && userLevel == 'VIP'";

        if (evaluator.evaluate(rule, context)) {
            return order.getAmount().multiply(new BigDecimal("0.8"));
        }
        return order.getAmount();
    }
}
```

**避坑：**
- 解释器适合简单语法，复杂语法用 ANTLR
- 优先考虑现有规则引擎（Drools）
- 表达式注入要注意安全风险

---

### 23. 访问者模式 Visitor

**适用场景：**
- 对象结构稳定但操作经常变
- 操作与对象结构分离
- 对集合中每种类型做不同操作

**标准结构：**
```java
// 元素接口
public interface Element {
    void accept(Visitor visitor);
}

// 具体元素 A
public class ElementA implements Element {

    @Override
    public void accept(Visitor visitor) {
        visitor.visitA(this);
    }

    public String operationA() {
        return "ElementA";
    }
}

// 具体元素 B
public class ElementB implements Element {

    @Override
    public void accept(Visitor visitor) {
        visitor.visitB(this);
    }

    public String operationB() {
        return "ElementB";
    }
}

// 访问者接口
public interface Visitor {
    void visitA(ElementA element);
    void visitB(ElementB element);
}

// 具体访问者
public class ConcreteVisitor implements Visitor {

    @Override
    public void visitA(ElementA element) {
        System.out.println("Visiting A: " + element.operationA());
    }

    @Override
    public void visitB(ElementB element) {
        System.out.println("Visiting B: " + element.operationB());
    }
}

// 对象结构
public class ObjectStructure {
    private List<Element> elements = new ArrayList<>();

    public void add(Element element) {
        elements.add(element);
    }

    public void accept(Visitor visitor) {
        for (Element element : elements) {
            element.accept(visitor);
        }
    }
}
```

**Spring Boot 落地：**
```java
// 报表导出场景
public interface ReportVisitor {
    void visit(SalesReport report);
    void visit(UserReport report);
    void visit(OrderReport report);
}

public class PdfReportVisitor implements ReportVisitor {

    @Override
    public void visit(SalesReport report) {
        // PDF 导出逻辑
    }

    @Override
    public void visit(UserReport report) {
        // PDF 导出逻辑
    }

    @Override
    public void visit(OrderReport report) {
        // PDF 导出逻辑
    }
}

public class ExcelReportVisitor implements ReportVisitor {

    @Override
    public void visit(SalesReport report) {
        // Excel 导出逻辑
    }

    // ...
}
```

**避坑：**
- 对象结构稳定才用访问者模式
- 增加新元素要修改所有访问者
- 优先考虑策略模式或反射
- 访问者模式会增加代码复杂度

---

# 输出格式要求

每次模式选择与实现指导时，按以下结构输出：

## 一、需求分析

- 稳定主干
- 变化点
- 协作边界

## 二、模式选择

- 推荐模式
- 选择理由
- 不选其他模式的原因

## 三、模式结构

用 ASCII 图或结构说明：
- 角色划分
- 类职责
- 协作关系

## 四、代码骨架

给出核心实现代码：
- 接口/抽象类
- 具体实现
- 装配方式（Spring 如何注入）

## 五、扩展方向

- 如何新增实现
- 注意事项

## 六、避坑指南

- 常见错误
- 如何避免

---

# 使用方式

按以下格式提供输入：

```text
请帮我设计一个适合下面需求的 Java 设计模式实现方案。

【需求背景】
{{业务背景、目标}}

【已知变化点】
{{哪些地方可能会扩展、变化}}

【已知不变点】
{{哪些是稳定的}}

【约束条件】
{{技术栈、Spring Boot 版本、数据库、事务要求等}}

【可选】初步方案
{{如果你已经有初步想法}}
```

---

# 硬约束

## 1. 先识别再推荐

必须先分析需求结构，再给出模式建议。不能不问需求就直接推荐模式。

## 2. 简单优先

能用 if-else 解决的不用策略，能用 Spring 注入解决的不用工厂。

## 3. 渐进式设计

不要一开始就设计完整模式，先轻量落地，根据需要再演进。

## 4. 上下文敏感

Spring Boot 项目要考虑 Spring 生命周期、事务边界、代理失效场景。

## 5. 不强推模式

如果当前需求不需要模式，要明确告知不要过度设计。
