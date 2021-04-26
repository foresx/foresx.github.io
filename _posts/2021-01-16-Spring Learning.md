---
title: "Spring Basic Learning"
last_modified_at: 2021-01-11T16:05:02-05:00
categories:
  - Blog
tags:
  - IOC
  - Spring
# link: https://foresx.github.io/blog
header:
  overlay_image: /assets/images/banner.jpeg
  overlay_filter: 0.5

---

## Spring IOC 相关内容

IOC Inversion of Control.

现在有三种方式来使用 spring:

xml: 现在一般都是没有源码的时候使用
javaconfig
annotation

### Spring Lifecycle

#### Spring Bean Lifecycle Callbacks

1. 实现 initializingBean(afterBeanPropertiesSet), DisposableBean(destroy)接口的方法
通过实现spring beans 里定义的接口来将方法插入到 spring bean 的生命周期中去.

2. init-method, destroy-method
这个一般用户 xml 文件定义 bean 中, bean 中可以关联 init-method, destroy-method 属性,来指定要在 bean 的生命周期中去调用 bean 中的方法. 在 beans 标签中可以配置所有 bean 的默认 default-init-method and default-destroy-method.

3. @PostConstruct, @PreDestroy(侵入性最小)
同上, 可以定义一个 bean 来进行使用,然后加上这两个注解即可在 bean 的生命周期中去调用这个方法.

综上,这三组都是用来做我们对 spring bean 的自定义,在 bean 的生命周期中嵌入我们想要做的功能.
> PS: spring 的 aop 生成会是在所有 bean 初始化完成以后(包括了我们的自定义方法) 才会开始进行. 如果你在 init 方法上用 aop 的话,并不会造成问题,但是会很奇怪,把我们的 target bean 的生成 和 aop 耦合在了一起.

---

三个方法的调用顺序不同:
init 方法上:

1. annotation 定义的方法先被调用.

2. 实现接口的方法再被调用.

3. xml 定义的方法最后被调用.

destroy 方法上同上.

#### Spring Bean Startup and Shutdown callbacks(通过这个去影响 bean 的启动顺序以及 bean 对上下文的影响)

```java
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}

public interface LifecycleProcessor extends Lifecycle {

    void onRefresh();

    void onClose();
}

public interface Phased {
    // 通过这个可以知道启动顺序, 数字越大启动越慢,销毁越快. 数字是负数的话,会在 spring 标准组件之前去启动,在最后进行关闭
    int getPhase();
}


public interface SmartLifecycle extends Lifecycle, Phased {
    // 是否会自动启动
    boolean isAutoStartup();

    void stop(Runnable callback);
}
```

可以实现DefaultLifecycleProcessor去进行对timeoutPerShutdownPhase进行自定义.

#### Spring register shutdown callback

```java
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");

        // add a shutdown hook for the above context...
        ctx.registerShutdownHook();

        // app runs here...

        // main method exits, hook is called prior to the app shutting down...
    }
}
```

#### Spring ApplicationContextAware and BeanNameAware

1. 实现 ApplicationContext 接口

2. 依赖注入 ApplicationContext(可以带上类型)

基于 ApplicationContext, BeanNameAware 可以让我根据 context 中的信息动态自定义beanName

> The callback is invoked after population of normal bean(属性赋值) properties but before an initialization callback such as InitializingBean, afterPropertiesSet, or a custom init-method.

### Spring Dependencies

1. depend-on 手动设置初始化顺序

2. 懒加载; xml: lazy-init; @Lazy

3. 自动装配.(Autowiring 的局限性以及excluding a bean from Autowiring)

4. Method Injection(look-up 解决单例类中有一个 prototype 的类)

### ClassPath Scanning and Managed Components

1. ComponentScan includeFilter and excludeFilter.

2. @Primary, @Qualifier(注解继承,自定义注解), @Profile(优先级).

3. spring context indexer 可以在编译的时候建立 bean 的索引从而提升启动速度.

### Bean

1. bean 的作用域(singleton, prototype)
