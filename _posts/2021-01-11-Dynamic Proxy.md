---
title: "Java Dynamic Proxy"
last_modified_at: 2021-01-11T16:05:02-05:00
categories:
  - Blog
tags:
  - AOP
  - Dynamic Proxy
# link: https://foresx.github.io/blog
header:
  overlay_image: /assets/images/banner.jpg
  overlay_filter: 0.5

---

## Dynamic Proxy

![UML图](/assets/images/dynamic_proxy.png)

代理分为静态代理和动态代理两种.

### 静态代理

1. 静态代理是我们自己手动去构建 proxy object 且必须实现接口方法.
2. 静态代理在编译时就已经实现,编译完成后代理类是一个实际的class文件.

两种实现方式:

1. 继承
通过继承来 override 父类中的方法.
缺点因为 java 是单继承的,所以针对每一个target object 都需要生成一个 proxy object 作为子类.导致类的数量增加.
2. 聚合
通过把 target project 耦合到 proxy project 中,重新实现 target object 的方法.
相比于上一种方法,可以将多个 target object 耦合到 proxy object, 只用一个 proxy object 作为多个 target object 的代理.

总结,静态代理需要手动编写代码,并且需要新建 proxy class 来进行实现.这样会造成类的爆炸,代码的复杂性上升,维护起来麻烦.

### 动态代理

1. java 中的动态代理主要分为两类.
   - jdk(>=1.3) 提供的,需要 target object 必须实现了接口才能进行代理.
这是因为 jdk proxy 需要通过接口信息去构建生成代理对象.并且jdk proxy 生成的对象extends Proxy, 所以 jdk proxy 只支持对于接口实现类的动态代理.
   - CGLIB 是通过 ASM 字节码处理框架, 转换字节码并且生成 target object 的子类从而实现对target object 的扩展(子类就是我们的 proxy object)
2. 动态代理是在运行时动态生成的，即编译完成后没有实际的class文件，而是在运行时动态生成类字节码，并加载到JVM中

动态代理的本质就是在程序运行时进行对proxy object 类进行生成. 即通过输入的内容计算出需要动态生成的proxy object 的内容形成class 字节码.然后通过反射对字节码进行实例化返回给需要使用的地方.

### JDK Proxy

前面介绍了一些 jdk proxy 的基础知识.

先定义好我们的 target object. (记住 jdk proxy target object 必须是接口实现类)

```java
public interface SingService {

  void sing();

  class SingServiceImpl implements SingService {

    @Override
    public void sing() {
      System.out.println("I'm singing in the rain!");
    }
  }
}
```

接下来定义我们的 InvocationHandler 实现类以及动态创建出我们的 proxy object.

```java
package com.command.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class ProxyDemo {

  public static void main(String[] args) {
    SingService singServiceImpl = new SingService.SingServiceImpl();
    SingServiceProxy invocationHandler = new SingServiceProxy(singServiceImpl);

    // 通过 Proxy.newProxyInstance 创建出我们的动态代理类.
    // classLoader, 实现的接口, invocationHandler
    SingService singServiceProxy = (SingService) Proxy
        .newProxyInstance(singServiceImpl.getClass().getClassLoader(),
            new Class[]{SingService.class}, invocationHandler);

    singServiceProxy.sing();
  }

  public static class SingServiceProxy implements InvocationHandler {

    private final Object targetObject;

    public SingServiceProxy(Object targetObject) {
      super();
      this.targetObject = targetObject;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
      // 可以根据方法的信息去选择如何加强各种方法
      if (method.getName().equals("sing")) {
        System.out.println("Prepare my mic!");
        Object result = method.invoke(targetObject, args);
        System.out.println("Clapping!");
        return result;
      } else {
        return method.invoke(targetObject, args);
      }

    }
  }
}
```

#### 如何保存生成的 proxy 类

先找到这个变量 ProxyGenerator 中 saveGeneratedFiles 是从哪个环境变量中赋值的.
启动的时候先设置好环境变量.System.setProperty("*", "true");
这样一般通常都会在 com.sun.proxy 下生成一个 Proxy$*.class 类.

> jdk8 and before: System.setProperty("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");
 After jdk8: System.setProperty("jdk.proxy.ProxyGenerator.saveGeneratedFiles", "true");
 If there is an abnormal situation: you can directly search for the class ProxyGenerator in jdk according to the following methods! ! !

example:

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package com.sun.proxy;

import com.command.proxy.SingService;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

public final class $Proxy0 extends Proxy implements SingService {
  private static Method m1;
  private static Method m2;
  private static Method m3;
  private static Method m0;

  public $Proxy0(InvocationHandler var1) throws  {
    super(var1);
  }

  public final boolean equals(Object var1) throws  {
    try {
      return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
    } catch (RuntimeException | Error var3) {
      throw var3;
    } catch (Throwable var4) {
      throw new UndeclaredThrowableException(var4);
    }
  }

  public final String toString() throws  {
    try {
      return (String)super.h.invoke(this, m2, (Object[])null);
    } catch (RuntimeException | Error var2) {
      throw var2;
    } catch (Throwable var3) {
      throw new UndeclaredThrowableException(var3);
    }
  }

  public final void sing() throws  {
    try {
      super.h.invoke(this, m3, (Object[])null);
    } catch (RuntimeException | Error var2) {
      throw var2;
    } catch (Throwable var3) {
      throw new UndeclaredThrowableException(var3);
    }
  }

  public final int hashCode() throws  {
    try {
      return (Integer)super.h.invoke(this, m0, (Object[])null);
    } catch (RuntimeException | Error var2) {
      throw var2;
    } catch (Throwable var3) {
      throw new UndeclaredThrowableException(var3);
    }
  }

  static {
    try {
      m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
      m2 = Class.forName("java.lang.Object").getMethod("toString");
      m3 = Class.forName("com.command.proxy.SingService").getMethod("sing");
      m0 = Class.forName("java.lang.Object").getMethod("hashCode");
    } catch (NoSuchMethodException var2) {
      throw new NoSuchMethodError(var2.getMessage());
    } catch (ClassNotFoundException var3) {
      throw new NoClassDefFoundError(var3.getMessage());
    }
  }
}
```

从上面我们可以看出来,我们的方法也就是 m3, 是通过我们实现 invocationHandler 接口的类调用的,从而通过反射去调用到target object 的方法.
Java dynamic proxy底层基于Proxy/InvocationHandler相关类和反射技术，其自动生成代理类的逻辑是生成一个代理接口的实现类（这也是只能做接口动态代理的原因），对该类中方法调用都指向到了我们自己定义的xxxInvocationHandler，在xxxInvocationHandler中再通过反射调用真正类的方法。

#### Jdk dynamic proxy 源码分析

首先我们看一下 Proxy.newProxyInstance() 这个方法.三个参数, target class loader, target class interface, invocationHandler.

```java
@CallerSensitive
    public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
    {
        Objects.requireNonNull(h);

        final Class<?>[] intfs = interfaces.clone();
        final SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
        }

        /*
         * Look up or generate the designated proxy class.
         */
        Class<?> cl = getProxyClass0(loader, intfs);

        /*
         * Invoke its constructor with the designated invocation handler.
         */
        try {
            if (sm != null) {
                checkNewProxyPermission(Reflection.getCallerClass(), cl);
            }
            // 通过反射获取代理类的构造器
            final Constructor<?> cons = cl.getConstructor(constructorParams);
            final InvocationHandler ih = h;
            if (!Modifier.isPublic(cl.getModifiers())) {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        cons.setAccessible(true);
                        return null;
                    }
                });
            }
            // 通过反射构建代理的实例
            return cons.newInstance(new Object[]{h});
        } catch (IllegalAccessException|InstantiationException e) {
            throw new InternalError(e.toString(), e);
        } catch (InvocationTargetException e) {
            Throwable t = e.getCause();
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            } else {
                throw new InternalError(t.toString(), t);
            }
        } catch (NoSuchMethodException e) {
            throw new InternalError(e.toString(), e);
        }
    }
```

其中最重要的三个方法:

```java
//生成代理类的类信息
Class<?> cl = getProxyClass0(loader, intfs);
//通过反射获取代理类的构造器
final Constructor<?> cons = cl.getConstructor(constructorParams);
//使用反射拿到构造器对象后创建一个代理对象，并返回
return cons.newInstance(new Object[]{h});
```

getProxyClass0 从 proxyClassCache 中调用 get 方法.

```java
    /**
     * a cache of proxy classes
     */
    private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
        proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());
```

所以实际上是调用的 ProxyClassFactory 中的 apply 方法.

在这个方法中主要做了几件事情:

- 一些规则检测，如类加载器是否有效，Class是否为接口，接口数组中的接口是否有重复,决定生成 proxy class 的位置
- 拼接代理需要的一些信息，如类名$Proxy+递增数字
- ProxyGenerator.generateProxyClass(proxyName, interfaces, accessFlags). 代理类的byte[]生成(在这里我们可以看到我们熟悉的saveGeneratedFiles)
- 加载byte[]到内存，并返回Class

```text
  第三步中,两个重要的方法:
  ProxyGenerator var3 = new ProxyGenerator(var0, var1, var2);
  final byte[] var4 = var3.generateClassFile();
其中 generateClassFile 方法的主要处理:

- Object对象中的方法特殊处理
- 循环添加方法，及Method对象
- 循环检测返回值类型
- 静态代码块，初始化代码的生成
- class标准文件头的拼接
- 最终把所有的数据转换成byte数组返回
```

[了解更多](https://docs.oracle.com/javase/8/docs/technotes/guides/reflection/proxy.html)

### CGLIB(Code generation library) Proxy

CGLIB 层级以及应用项目.

![CGLIB](/assets/images/cglib.png)

CGLIB 有两种方式来实现.

1. 实现 MethodInterceptor 接口
2. Enhancer 生成.

### CGLIB vs JDK Proxy

1. 两者的主要区别还是 cglib 是基于父类来生成 proxy object. 所以当类上存在 final 关键字的时候以及方法存在 final 关键字修饰的时候, cglib 无法生效. 而 jdk proxy 是基于反射实现的,并且需要接口信息去生成对象以及extends java.lang.reflect.Proxy 所以target object 必须是接口的实现类才能够生成 proxy object.

2. 使用CGLib实现动态代理，CGLib底层采用ASM字节码生成框架，使用字节码技术生成代理类，在jdk6之前比使用Java反射效率要高。唯一需要注意的是，CGLib不能对声明为final的方法进行代理， 因为CGLib原理是动态生成被代理类的子类。在jdk6、jdk7、jdk8逐步对JDK动态代理优化之后，在调用次数较少的情况下，JDK代理效率高于CGLIB代理效率，只有当进行大量调用的时候，jdk6和jdk7比CGLIB代理效率低一点，但是到jdk8的时候，jdk代理效率高于CGLIB代理，总之，每一次jdk版本升级，jdk代理效率都得到提升，而CGLIB代理消息确有点跟不上步伐。

### 在 Spring AOP 中的应用

1. Spring AOP 默认使用 jdk proxy. 在无法使用 jdk proxy 时会去使用 cglib 进行代理.

2. 也可以强制使用 cglib 生成代理对象

annotation:

```java
@EnableAspectJAutoProxy 中的 proxyTargetClass 设置成 true 即可
```

xml-based:

```java
<aop:aspectj-autoproxy proxy-target-class="true"/>
```
