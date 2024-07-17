
# 总流程图

![](https://xuemingde.com/pages/image/2022/03/09/ufeQLe.jpg)



# 入口类

```java
@SpringBootApplication
public class DevServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(DevServiceApplication.class,args);
    }
}
```



## SpringBootApplication注解

> Spring Boot应用标注在某个类上说明这个类是SpringBoot的主配置类，SpringBoot就应该运行这个类的main方法来启动SpringBoot应用

SpringBootApplication注解是Spring Boot的核心注解，它其实是一个组合注解：

```java
@Target(ElementType.TYPE) //// 注解的适用范围，其中TYPE用于描述类、接口（包括包注解类型）或enum声明
@Retention(RetentionPolicy.RUNTIME) //// 注解的生命周期，保留到class文件中（三个生命周期）
@Documented //// 表明这个注解应该被javadoc记录
@Inherited //// 子类可以继承该注解
@SpringBootConfiguration //// 继承了Configuration，表示当前是注解类
@EnableAutoConfiguration //// 开启springboot的注解功能，springboot的四大神器之一，其借助@import的帮助
@ComponentScan(excludeFilters = {
        @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) }) //// 扫描路径设置
public @interface SpringBootApplication {
...
}
```



从源码可以看出，这个注解是**@SpringBootConfiguration**，**@EnableAutoConfiguration**以及**@ComponentScan**这三个注解的组合



### @SpringBootConfiguration

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
	@AliasFor(annotation = Configuration.class)
	boolean proxyBeanMethods() default true;

}
```

*Spring Boot的配置类；标注在某个类上，表示一个类提供了Spring Boot应用程序*

看SpringBootConfiguration的源码可以看出来，他其实继承与**@Configuration**，二者功能也一样，标注当前类是配置类，并且会将当前类声明的一个或多个@Bean注解标记的方法的实例纳入到spring容器中，并且实例名就是方法名。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {
    @AliasFor(
        annotation = Component.class
    )
    String value() default "";

    boolean proxyBeanMethods() default true;
}
```

配置类相当于配置文件；配置类也是容器中的一个组件，它使用了@Component这个注解。

@Configuration注解使用了@Component注解，但两个注解还是有区别的。



### @EnableAutoConfiguration

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}

```

*告诉SpringBoot开启自动配置功能，这样自动配置才能生效，最为重要*

<u>使用@EnableAutoConfiguration这个注解开启自动扫描，然后使用select选择挑选满足条件的文件，并且使用SpringFactoriesLoader进行实例化。最后加载到IOC容器里面，即ApplicationContext中。</u>



@Import注解，它会加载AutoConfigurationImportSelector类，然后就会触发这个类的selectImports()方法。根据返回的String数组(配置类的Class的名称)加载配置类。

```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware,
ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {
    //返回的String[]数组，是配置类Class的类名
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        if (!isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        }
        AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
        //返回配置类的类名
        return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
    }
}

```

我们一直点下去，就可以找到最后的幕后英雄，就是SpringFactoriesLoader类，通过loadSpringFactories()方法加载META-INF/spring.factories中的配置类。

![](https://xuemingde.com/pages/image/2022/03/09/EaiL64.jpg)

![](https://xuemingde.com/pages/image/2022/03/09/HbEGa5.jpg)

这里使用了spring.factories文件的方式加载配置类，提供了很好的扩展性。

所以@EnableAutoConfiguration注解的作用其实就是开启自动配置，自动配置主要则依靠这种加载方式来实现。



![](https://xuemingde.com/pages/image/2022/03/09/9AKjzk.jpg)







```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import({Registrar.class})
public @interface AutoConfigurationPackage {
}
```

![](https://xuemingde.com/pages/image/2022/03/09/K5nsWR.png)





### @ComponentScan

@ComponentScan就是自动扫描并加载符合条件的组件（比如@Component和@Repository等）或者bean定义，最终将这些bean定义加载到IOC容器中去 

所以**启动类最好定义在Root package下**，因为一般我们在使用@SpringBootApplication时，都不指定basePackages的。



## SpringApplication类

SpringApplication类实例化

```java

	/**
	 * Create a new {@link SpringApplication} instance. The application context will load
	 * beans from the specified primary sources (see {@link SpringApplication class-level}
	 * documentation for details. The instance can be customized before calling
	 * {@link #run(String...)}.
	 * @param resourceLoader the resource loader to use
	 * @param primarySources the primary bean sources
	 * @see #run(Class, String[])
	 * @see #setSources(Set)
	 */
	@SuppressWarnings({ "unchecked", "rawtypes" })
	public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
	 	// 初始化资源加载器
        this.resourceLoader = resourceLoader;
        // 资源加载类不能为 null
		Assert.notNull(primarySources, "PrimarySources must not be null");
        // 初始化加载资源类集合并去重
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
      // 推断应用程序是不是web应用
		this.webApplicationType = WebApplicationType.deduceFromClasspath();
      // 设置初始化器(Initializer)
		setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
      // 设置监听器 
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
      // 推断出主应用入口类
		this.mainApplicationClass = deduceMainApplicationClass();
	}
```



































***

参考文章

* [深入剖析Springboot启动原理的底层源码，再也不怕面试官问了](https://developer.huawei.com/consumer/cn/forum/topic/0202494081937300254)
* [一文搞懂springboot启动原理](https://www.jianshu.com/p/943650ab7dfd)
* [SpringBoot启动流程是怎样的？](https://juejin.cn/post/6895341123816914958#heading-6)
* []()

