---
layout: post
title: SpringBoot运行原理
date: 2021-04-09
Author: Los
tags: [springboot]
comments: false
---

先看@SpringBootApplication
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
        @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
...
}

````
主要关注的几个注解如下
@SpringBootConfiguration：标记当前类为配置类
@EnableAutoConfiguration：开启自动配置
@ComponentScan：扫描主类所在的同级包以及下级包里的Bean
关键是@EnableAutoConfiguration
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
   String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";
   Class<?>[] exclude() default {};
   String[] excludeName() default {};
}
```
最关键的要属@Import(EnableAutoConfigurationImportSelector.class)，借助EnableAutoConfigurationImportSelector，@EnableAutoConfiguration可以帮助SpringBoot应用将所有符合条件的@Configuration配置都加载到当前SpringBoot创建并使用的IoC容器:通过@Import(AutoConfigurationImportSelector.class)导入的配置功能，
AutoConfigurationImportSelector中的方法getCandidateConfigurations，得到待配置的class的类名集合,这个集合就是所有需要进行自动配置的类，而是是否配置的关键在于META-INF/spring.factories文件中是否存在该配置信息
```java
  protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,
            AnnotationAttributes attributes) {
        List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
                getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
        Assert.notEmpty(configurations,
                "No auto configuration classes found in META-INF/spring.factories. If you "
                        + "are using a custom packaging, make sure that file is correct.");
        return configurations;
    }
```
打开，如下图可以看到所有需要配置的类全路径都在文件中，每行一个配置，多个类名逗号分隔,而\表示忽略换行![在这里插入图片描述](https://img-blog.csdn.net/20180625171312420?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0F4ZWxhMzBX/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



![在这里插入图片描述](https://img-blog.csdnimg.cn/20201113165102963.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqY2phdmE=,size_16,color_FFFFFF,t_70#pic_center)

整个流程如上图所示

总结图:![在这里插入图片描述](https://img-blog.csdnimg.cn/2020111316535889.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqY2phdmE=,size_16,color_FFFFFF,t_70#pic_center)

SpringBoot自动化配置关键组件关系图
mybatis-spring-boot-starter、spring-boot-starter-web等组件的META-INF文件下均含有spring.factories文件，自动配置模块中，SpringFactoriesLoader收集到文件中的类全名并返回一个类全名的数组，返回的类全名通过反射被实例化，就形成了具体的工厂实例，工厂实例来生成组件具体需要的bean。
————————————————
版权声明：本文为CSDN博主「牧竹子」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/zjcjava/article/details/84028222