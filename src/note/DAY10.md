# 31. 关于Spring框架

## 31.1. Spring框架的作用

Spring框架主要解决了创建对象、管理对象的相关问题。

创建对象，例如：

```java
User user = new User();
```

管理对象：Spring会在创建对象之后，完成必要的属性赋值等操作，并且，还会持有所创建的对象的引用，由于持久大量对象的引用，所以，Spring框架也通常被称之为“Spring容器”。

## 31.2. Spring框架创建对象的做法

Spring框架创建对象有2种做法，第1种是通过配置类中的`@Bean`方法，第2种是通过组件扫描。

关于`@Bean`方法：在任何配置类中，自定义返回对象的方法，并在方法上添加`@Bean`注解，则Spring会自动调用此方法，并且，获取此方法返回的对象，将对象保存在Spring容器中，例如：

```java
@Configuration
public class BeanFactory {
    
    @Bean
    public LocalDateTime localDateTime() {
        return LocalDateTime.now();
    }
    
    // 如果使用这种做法，则AlbumController不必使用组件扫描的做法
    @Bean
    public AlbumController albumController() {
        return new AlbumController();  
    }
    
    // 如果使用这种做法，则AlbumServiceImpl不必使用组件扫描的做法
    @Bean
    public AlbumServiceImpl albumServiceImpl() {
        return new AlbumServiceImpl();  
    }
    
}
```

关于组件扫描：需要通过`@ComponentScan`注解来指定扫描的根包，则Spring框架会在此根包下查找组件，并且创建这些组件的对象。

根包：某个包及其子孙包，例如，指定的根包是`cn.tedu.csmall.product`，则`cn.tedu.csmall.product`、`cn.tedu.csmall.product.controller`、`cn.tedu.csmall.product.service.impl`都属于根包的范围之内！

组件：在Spring框架中，添加了`@Component`及其衍生注解的，都是组件！常见的组件注解有：

- `@Component`：通用组件注解
- `@Controller`：应该添加在控制器类上
- `@Service`：应该添加在处理业务逻辑的类上
- `@Repository`：应该添加在处理数据访问（直接与数据源交互）的类上
- `@Configuration`：应该添加在配置类上

以上5个组件注解，除了`@Configuration`以外，另外4个在功能、用法、执行效果方面，在Spring框架的作用范围内是完全相同的，只是语义不同！Spring框架在处理`@Configuration`注解时，会使用CGLib的代理模式来创建对象，并且，被Spring实际使用的是代理对象。

提示：在Spring Boot项目中，启动类上的注解`@SpringBootApplication`，此注解的元注解包含`@ComponentScan`注解，所以，Spring Boot项目启动时就会执行组件扫描，扫描的根包就是启动类所在的包！

在开发实践中，如果需要创建非自定义类（例如Java官方的类，或其它框架中的类）的对象，**必须**使用`@Bean`方法，毕竟你不能在别人声明的类上添加组件注解，如果需要创建自定义类的对象，则**优先**使用组件扫描的做法，因为这种做法更加简单！

## 31.3. Spring管理的对象的作用域

Spring管理的对象默认是“单例”的，则在整个程序的运行过程中，随时可以获取或访问Spring容器中的“单例”对象！

**注意：Spring并没有实际使用单例模式！**

单例：单一实例（单一对象），即：在任意时间，某个类的对象最多只有1个！

如果需要Spring管理某个对象采取“非单例”的模式，可以通过`@Scope("prototype")`注解来实现！

> 提示：如果是通过`@Bean`方法创建对象，则`@Scope("prototype")`注解添加在`@Bean`方法上，如果是通过组件扫描创建对象，则`@Scope("prototype")`注解添加在组件类上。

如果没有Spring框架，自己手动实现单例效果，大致需要：

```java
public class Singleton {
    private static Singleton instance = new Singleton();
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        return instance;
    }
}
```

```java
public class Singleton {
    private static final Object lock = new Object();
    private static Singleton instance;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (lock) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

Spring管理的单例对象，默认情况下预加载的！可以通过`@Lazy`注解配置为懒加载的！

> 提示：如果是通过`@Bean`方法创建对象，则`@Lazy`注解添加在`@Bean`方法上，如果是通过组件扫描创建对象，则`@Lazy`注解添加在组件类上。

## 31.4. 自动装配机制

Spring的自动装配机制表现为：当Spring管理的类的属性需要被自动赋值，或Spring调用的方法的参数需要值时，Spring会自动从容器中找到合适的值，为属性 / 参数自动赋值！

当类的属性需要值时，可以在属性上添加`@Autowired`注解。

关于被Spring调用的方法，主要表现为：构造方法、配置类中的`@Bean`方法等。

关于调用构造方法：

- 如果类中存在无参数构造方法（无论是否存在其它构造方法），Spring会自动调用无参数构造方法
- 如果类中仅有1个构造方法，Spring会自动尝试调用，且，如果此构造方法有参数，Spring会自动尝试从容器中查找合适的值用于调用此构造方法
- 如果类中有多个构造方法，且都是有参数的，Spring不会自动调用任何构造方法，且会报错
- 如果希望Spring调用特定的构造方法，应该在那一个构造方法上添加`@Autowired`注解

关于在属性上使用`@Autowired`时的提示：`Field injection is not recommended`，其意思是“字段注入是不推荐的”，因为，开发工具认为，你有可能在某些情况下自行创建当前类的对象，例如自行编写代码：`AlbumController albumController = new AlbumController();`，由于是自行创建的对象，Spring框架在此过程中是不干预的，则类的属性`IAlbumService albumService`将不会由Spring注入值，如果此时你也没有为这个属性赋值，则这个的属性就是`null`，如果还执行类中的方法，就可能导致NPE（`NullPointerException`），这种情况可能发生在单元测试中。开发工具建议使用构造方法注入，即使用带参数的构造方法，且通过构造方法为属性赋值，并且类中只有这1个构造方法，在这种情况下，即使自行创建对象，由于唯一的构造方法是带参数的，所以，创建对象时也会为此参数赋值，不会出现属性没有值的情况，所以，通过构造方法为属性注入值的做法被认为是安全的，是建议使用的做法！但是，在开发实践，通常并不会使用构造方法注入属性的值，因为，属性的增、减都需要调整构造方法，并且，如果类中需要注入值的属性较多，也会导致构造方法的参数较多，不是推荐的！

关于合适的值：Spring框架会查找容器中匹配类型的对象的数量：

- 0个：无法装配，需要判断`@Autowired`注解的`required`属性：
  - `true`：在加载Spring时直接报错`NoSuchBeanDefinitionException`
  - `false`：放弃自动装配，且尝试自动装配的属性将是默认值
- 1个：直接装配，且装配成功
- 超过1个：尝试按照名称来匹配，如果均不匹配，则在加载Spring时直接报错`NoUniqueBeanDefinitionException`，按照名称匹配时，要求被装配的变量名与Bean Name保持一致

关于Bean Name：每个Spring Bean都有一个Bean Name，如果是通过`@Bean`方法创建的对象，则Bean Name就是方法名，或通过`@Bean`注解参数来指定名称，如果是通过组件扫描的做法来创建的对象，则Bean Name默认是将类名首字母改为小写的名称（例如，类名为`AlbumServiceImpl`，则Bean Name为`albumServiceImpl`）（此规则只适用于类名中第1个字母大写、第2个字母小写的情况，如果不符合此情况，则Bean Name就是类名），Bean Name也可以通过`@Component`等注解的参数进行配置，或者，你还可以在需要装配的属性上使用`@Qualifier`注解来指定装配哪个Bean Name对应的Spring Bean。

另外，在处理属性的自动装配上，还可以使用`@Resource`注解取代`@Autowired`注解，`@Resource`是先根据名称尝试装配，再根据类型装配的机制！

## 31.5. 关于IoC和DI

IoC = Inversion of Control，控制反转，表现为“将对象的管理权交给了框架”

DI = Dependency Injection，依赖注入，表现为“为类中的属性自动赋值”

Spring框架通过DI实现了IoC。

# 





