---
title: "Spring boot basic learning"
last_modified_at: 2021-01-31T16:05:02-05:00
categories:
  - Blog
tags:
  - Spring
  - SpringBoot
# link: https://foresx.github.io/blog
header:
  overlay_image: /assets/images/banner.jpeg
  overlay_filter: 0.5

---

### Spring Boot 基础内容

spring boot starter parent 控制版本
spring boot starter 各种类库的启动器

java8 以上, 自带 Tomcat 容器

#### plugin

Maven plugin
Gradle plugin

### 自动配置项

debug=true, 打印出 conditional 的计算结果 report
server.servlet.context-path rest 风格前缀
logging.level + package name or class name: level

### Spring boot 怎么做到无配置启动的呢?

1. Servlet 3.0 以及 Spring 5 的支持. java config 和 Servlet handle types
2. Conditional 的设计

### Spring MVC Servlet 如何启动的

1. SpringServletContainerInitializer 通过这个接口实现了 ServletContainerInitializer, 并且通过 spi 的方式(javax.servlet.ServletContainerInitializer中定义)调用其onStartup方法.
2. 然后SpringServletContainerInitializer 通过@HandlesTypes(WebApplicationInitializer.class) 将classpath 下这个接口的实现类注入到方法参数中从而调用他们的 onStartUp 方法.
3. AbstractDispatcherServletInitializer 在这个实现类中 DispatchServlet 被初始化并且生成 handler等内容;
```java
protected void initStrategies(ApplicationContext context) {
		// 通过这里来完成文件上传下载的支持, multipartResolver bean 的名字在这里写死了
		initMultipartResolver(context);
		// 国际化的支持
		initLocaleResolver(context);
		// 主题支持
		initThemeResolver(context);
		// 对我们 controller(handler mapping) 的映射, 核心
		initHandlerMappings(context);
		initHandlerAdapters(context);
		initHandlerExceptionResolvers(context);
		initRequestToViewNameTranslator(context);
		initViewResolvers(context);
		initFlashMapManager(context);
	}
```
4. 这样一个 Spring MVC 的 Servlet 就被初始化好了并且被实现了派发的功能