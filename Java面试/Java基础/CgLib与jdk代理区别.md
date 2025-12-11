# 🚀 一句话总结（最佳面试回答）

**JDK 动态代理基于接口（Interface），只能代理接口；  
CGLIB 基于继承（子类化），通过生成子类来代理类本身。  
JDK 通过反射调用，性能略低；CGLIB 通过 ASM 字节码生成，性能更高。**

---

# 1️⃣ JDK 动态代理的原理

### 核心：**基于接口**，只能代理接口。

底层依赖：

- `java.lang.reflect.Proxy`
    
- `InvocationHandler`
    

工作机制：

1. JDK 在运行时生成一个 **代理对象（Proxy$xx 类）**
    
2. 这个代理类 **实现了目标对象的接口**
    
3. 方法调用最终会落到 `InvocationHandler.invoke()`
    

示例：

`MyInterface proxy = (MyInterface) Proxy.newProxyInstance(     clazzLoader,     new Class[]{MyInterface.class},     new MyInvocationHandler(target) );`

👉 若目标类没有接口，**JDK 无法代理**
# 2️⃣ CGLIB 动态代理的原理

### 核心：**基于继承（Subclassing）**，给目标类创建子类。

底层依赖：

- ASM 字节码操作库
    
- `Enhancer` 类生成子类
    

工作机制：

1. CGLIB 生成一个类：`TargetClass$$EnhancerByCGLIB`
    
2. 该类 **继承目标类**
    
3. 覆盖方法，在方法前后插入增强逻辑（MethodInterceptor）
    

示例：

```
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(Target.class);
enhancer.setCallback(new MethodInterceptorImpl());
Target proxy = (Target) enhancer.create();

```

👉 只要目标类不是 `final`，CGLIB 都能代理


# 3️⃣ 两者的核心区别（面试必答点）

|特性|JDK 动态代理|CGLIB 动态代理|
|---|---|---|
|**是否依赖接口**|✔ 必须有接口|❌ 不需要接口|
|**实现方式**|通过 Java 反射|通过 ASM 生成字节码（继承）|
|**性能**|调用慢（反射）|调用快（直接方法调用）|
|**能否代理 final 类**|可以（类不用继承）|❌ 不能（final 无法继承）|
|**能否代理 final 方法**|❌ 不行（反射无法增强 final 方法）|❌ 同样不行（子类无法覆盖 final 方法）|
|**内存消耗**|较少|较多（需要生成子类）|
|**Spring 默认选择**|默认优先 JDK|没接口才走 CGLIB|
# 4️⃣ Spring 为什么默认用 JDK 动态代理？

Spring 的 AOP 默认策略：

- **如果有接口 → 使用 JDK 动态代理**
    
- **如果没有接口 → 使用 CGLIB**
    

原因：

1. JDK 动态代理更标准、兼容性好
    
2. CGLIB 需要生成子类，成本高一点
    
3. 避免继承导致的问题（final、构造器调用等）
    

👉 如果你想强制 Spring 使用 CGLIB，需要：

`spring:   aop:     proxy-target-class: true`

---

# 5️⃣ 哪些情况下必须用 CGLIB？

✔ 目标类 **没有接口**  
✔ 你想代理 **类本身** 而不是接口  
✔ 你想提高性能（高频调用场景）

典型例子：

- Spring AOP 代理一个没有实现接口的 Service 类
    
- MyBatis Mapper 在旧版本中也曾使用 CGLIB（现在是 JDK）
    

---

# 6️⃣ 哪些情况下必须用 JDK 动态代理？

✔ 代码必须通过接口访问，而不是类  
✔ 你不希望生成子类以避免继承问题  
✔ 想避免 CGLIB 与 final 冲突

---

# 7️⃣ 面试最佳回答（强烈推荐背诵）

> **JDK 动态代理基于接口，必须有接口，通过反射实现，性能略低。  
> CGLIB 基于继承，通过生成子类并重写方法来增强，依赖 ASM，性能更高。  
> JDK 不能代理类；CGLIB 不能代理 final 类和 final 方法。  
> Spring 默认用 JDK，有接口用 JDK，没有接口才使用 CGLIB。**

