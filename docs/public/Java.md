# Java

## 值传递和引用传递

```
值传递（pass by value）是指在调用函数时将实际参数复制一份传递到函数中，这样在函数中如果对参数进行修改，将不会影响到实际参数。
引用传递（pass by reference）是指在调用函数时将实际参数的地址直接传递到函数中，那么在函数中对参数所进行的修改，将影响到实际参数。

```

## 方法调用

```
 某个类有两个重载方法：void f(String s) 和 void f(Integer i)，那么f(null)的会调用哪个方法？
解释：1）精确匹配->2）基本数据类型（自动转换成更大范围）->3）封装类（自动拆箱与装箱）->4）子类向上转型依次匹配->5）可变参数匹配。  
子类向上转型，两者的父类都是object类（null默认类型是object），因而会同时匹配上两者，编译器会报Ambiguous method call. Both 错误
```

## Spring事务失效场景
![图片](../../document/images/public/事务失效.img)

### 1、访问权限问题

含有事务方法，定义了错误的访问权限，就会导致事务功能出问题，例如：

```java
@Service
public class UserService {
    
    @Transactional
    private void add(UserModel userModel) {
         saveData(userModel);
         updateData(userModel);
    }
}
```
**add方法的访问权限被定义成了`private`，这样会导致事务失效，spring要求被代理方法必须是`public`的。**

**原理：**  
在`AbstractFallbackTransactionAttributeSource`类的`computeTransactionAttribute`方法中有个判断，如果目标方法不是public，则`TransactionAttribute`返回null，即不支持事务。

```java
protected TransactionAttribute computeTransactionAttribute(Method method, @Nullable Class<?> targetClass) {
    // Don't allow no-public methods as required.
    if (allowPublicMethodsOnly() && !Modifier.isPublic(method.getModifiers())) {
      return null;
    }

    // The method may be on an interface, but we need attributes from the target class.
    // If the target class is null, the method will be unchanged.
    Method specificMethod = AopUtils.getMostSpecificMethod(method, targetClass);

    // First try is the method in the target class.
    TransactionAttribute txAttr = findTransactionAttribute(specificMethod);
    if (txAttr != null) {
      return txAttr;
    }

    // Second try is the transaction attribute on the target class.
    txAttr = findTransactionAttribute(specificMethod.getDeclaringClass());
    if (txAttr != null && ClassUtils.isUserLevelMethod(method)) {
      return txAttr;
    }

    if (specificMethod != method) {
      // Fallback is to look at the original method.
      txAttr = findTransactionAttribute(method);
      if (txAttr != null) {
        return txAttr;
      }
      // Last fallback is the class of the original method.
      txAttr = findTransactionAttribute(method.getDeclaringClass());
      if (txAttr != null && ClassUtils.isUserLevelMethod(method)) {
        return txAttr;
      }
    }
    return null;
  }
```

### 2、方法修饰符问题
**方法用final、static修饰,导致事务失效**

**原理：**
**spring事务底层使用了aop，也就是通过jdk动态代理或者cglib，帮我们生成了代理类，在代理类中实现的事务功能。**
**如果某个方法用final修饰了，那么在它的代理类中，就无法重写该方法，而添加事务功能。**

*错误示例：*
```java
@Service
public class UserService {

    @Transactional
    public final void add(UserModel userModel){
        saveData(userModel);
        updateData(userModel);
    }
}
```

### 3、方法内部调用
在同一个类中的方法直接内部调用，会导致事务失效
```java
@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;

  
    public void add(UserModel userModel) {
        userMapper.insertUser(userModel);
        updateStatus(userModel);
    }

    @Transactional
    public void updateStatus(UserModel userModel) {
        doSameThing();
    }
}
```
事务方法add中，直接调用事务方法updateStatus。由于updateStatus方法拥有事务的能力是因为spring aop生成代理了对象，但是这种方法直接调用了this对象的方法，所以updateStatus方法不会生成事务。

**解决方式：**
①通过AopContent类：
在该Service类中使用AopContext.currentProxy()获取代理对象
```java
@Servcie
public class ServiceA {

   public void save(User user) {
         queryData1();
         queryData2();
         ((ServiceA)AopContext.currentProxy()).doSave(user);
   }

   @Transactional(rollbackFor=Exception.class)
   public void doSave(User user) {
       addData1();
       updateData2();
    }
 }
```

### 4、未被spring管理
通常情况下，通过@Controller、@Service、@Component、@Repository等注解，可以自动实现bean实例化和依赖注入的功能

### 5、多线程调用
两个方法不在同一个线程中，获取到的数据库连接不一样，从而是两个不同的事务
```java
@Slf4j
@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;
    @Autowired
    private RoleService roleService;

    @Transactional
    public void add(UserModel userModel) throws Exception {
        userMapper.insertUser(userModel);
        new Thread(() -> {
            roleService.doOtherThing();
        }).start();
    }
}

@Service
public class RoleService {

    @Transactional
    public void doOtherThing() {
        System.out.println("保存role表数据");
    }
}
```

### 6、表不支持事务
在mysql5之前，默认的数据库引擎是myisam。
好处：索引文件和数据文件是分开存储的，对于查多写少的单表操作，性能比innodb更好。
在创建表的时候，只需要把ENGINE参数设置成MyISAM即可：
```sql
CREATE TABLE `category` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `one_category` varchar(20) COLLATE utf8mb4_bin DEFAULT NULL,
  `two_category` varchar(20) COLLATE utf8mb4_bin DEFAULT NULL,
  `three_category` varchar(20) COLLATE utf8mb4_bin DEFAULT NULL,
  `four_category` varchar(20) COLLATE utf8mb4_bin DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin
```
**myisam缺陷：不支持事务**。

如果只是单表操作还好，不会出现太大的问题。但如果需要跨多张表操作，由于其不支持事务，数据极有可能会出现不完整的情况。

此外，myisam还不支持行锁和外键。
实际业务场景中，myisam使用的并不多。在mysql5以后，myisam已经逐渐退出了历史的舞台，取而代之的是innodb。

### 7、未开启事务
①springboot项目：通过DataSourceTransactionManagerAutoConfiguration类，已经默认开启了事务，
只需要配置spring.datasource相关参数即可
②传统的spring项目，则需要在applicationContext.xml文件中，手动配置事务相关参数。如果忘了配置，事务肯定是不会生效的
```xml
<!-- 配置事务管理器 --> 
<bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" 		  id="transactionManager"> 
    <property name="dataSource" ref="dataSource"></property> 
</bean> 
<tx:advice id="advice" transaction-manager="transactionManager"> 
    <tx:attributes> 
        <tx:method name="*" propagation="REQUIRED"/>
    </tx:attributes> 
</tx:advice> 
<!-- 用切点把事务切进去 --> 
<aop:config> 
    <aop:pointcut expression="execution(* com.susan.*.*(..))" id="pointcut"/> 
    <aop:advisor advice-ref="advice" pointcut-ref="pointcut"/> 
</aop:config> 
```

### 8、自己吞了异常
**在代码中手动try...catch了异常 导致 事务不会回滚。**
如果想要spring事务能够正常回滚，必须抛出它能够处理的异常。如果没有抛异常，则spring认为程序是正常的。
```java
@Slf4j
@Service
public class UserService {
    
    @Transactional
    public void add(UserModel userModel) {
        try {
            saveData(userModel);
            updateData(userModel);
        } catch (Exception e) {
            log.error(e.getMessage(), e);
        }
    }
}
```

### 9.手动抛了别的异常
**spring事务，默认情况下只会回滚`RuntimeException`（运行时异常）和`Error`（错误），对于普通的Exception（非运行时异常），它不会回滚.**
```java
@Slf4j
@Service
public class UserService {
    
    @Transactional
    public void add(UserModel userModel) throws Exception {
        try {
             saveData(userModel);
             updateData(userModel);
        } catch (Exception e) {
            log.error(e.getMessage(), e);
            throw new Exception(e);
        }
    }
}
```

### 10.自定义了回滚异常
**自定义回滚异常与程序抛出的异常不一致时，事务失效。**
建议一般情况下，将该参数设置成：Exception或Throwable。
```java
@Slf4j
@Service
public class UserService {
    
    @Transactional(rollbackFor = BusinessException.class)
    public void add(UserModel userModel) throws Exception {
       saveData(userModel);
       updateData(userModel);
    }
}
```

