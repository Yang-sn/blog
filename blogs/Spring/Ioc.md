---
title: Ioc
date: 2021-05-07
categories:
  - Spring
tags:
  - ioc
  - 容器
  - 依赖注入
---

## Ioc 是什么？

---

Ioc (Inversion of Control)，俗称**控制反转**，是一种设计思想。Ioc 是一种通过**描述**来生成或者获取对象的技术。Spring 只是实现了 Ioc 思想的框架之一。

控制反转分为两部分，一部分是控制，一部分是反转。我们需要理解谁控制谁，控制的是什么?为什么是反转？正转又是什么?

> 谁控制谁，控制什么： 传统 Java SE 程序设计，我们直接在对象内部通过 new 进行创建对象，是程序主动去创建依赖对象；而 IOC 是有专门一个容器来创建这些对象，即由 IOc 容器来控制对象的创建而不再显式地使用 new；谁控制谁？当然是 IOC 容器控制了对象；控制什么？那就是主要控制了外部资源获取和生命周期（不只是对象也包括文件等）。

> 为何是反转，哪些方面反转了： 有反转就有正转，传统应用程序是由我们自己在对象中主动控制去直接获取依赖对象，也就是正转；而反转则是由容器来帮忙创建及注入依赖对象；为何是反转？因为由容器帮我们查找及注入依赖对象，对象只是被动的接受依赖对象，所以是反转；哪些方面反转了？依赖对象的获取被反转了。

## DI 是什么？

DI 是**Dependency Injection**的缩写，俗称依赖注入。上面说到**Ioc**是一种控制反转的思想，**依赖注入则是实现控制反转的手段与方式**。依赖注入又是分为依赖和注入两部分。

> 什么是依赖？ 所谓依赖，说直白点就是：A 用了 B，那 A 就依赖 B。换成程序世界的说法，如果 A 类里面出现了 B 类有关的代码（删除 B 类，编译 A 类会报错），那 A 就依赖 B。比如说一个班级的组成依赖多个老师和同学，如果没有相应的老师和同学，那么班级也就不复存在了。这时我们可以说班级类就依赖老师类和同学类。

注入大概有三种方式：构造器注入，方法注入和属性注入。

```java
    public class ClassPojo {
        private  String name;
        //加入注解  属性注入
        private  List<Student> studentList;
        private  List<Teacher> teacherList;
        //构造器注入 对象实例化的时候就注入依赖
        public ClassPojo(List<Student> studentList, List<Teacher> teacherList) {
            this.studentList = studentList;
            this.teacherList = teacherList;
        }
        // 调用set 方法注入
        public void setStudentList(List<Student> studentList) {
            this.studentList = studentList;
        }

        public void setTeacherList(List<Teacher> teacherList) {
            this.teacherList = teacherList;
        }
    }
```

## Spring 中的 Ioc

Spring Ioc 是一个管理**Bean**的容器，它要求所有的容器都要实现一个顶层容器接口**BeanFactory**。我们可以看下这个接口的源码(删掉了全部的英文注释，加入了中文注释)：

```java
package org.springframework.beans.factory;

import org.springframework.beans.BeansException;

public interface BeanFactory {

	//前缀
	String FACTORY_BEAN_PREFIX = "&";

    //多个获取bean的方法
	Object getBean(String name) throws BeansException;


	<T> T getBean(String name, Class<T> requiredType) throws BeansException;

	<T> T getBean(Class<T> requiredType) throws BeansException;

	Object getBean(String name, Object... args) throws BeansException;

	//是否包含bean
	boolean containsBean(String name);

	//是否单例  Spring默认是单例
	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
    //是否原型
	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;

	//是否类型匹配
	boolean isTypeMatch(String name, Class<?> targetType) throws NoSuchBeanDefinitionException;
    //获取bean的类型
	Class<?> getType(String name) throws NoSuchBeanDefinitionException;

	//获取bean的别名
	String[] getAliases(String name);

}

```

由于 BeanFactory 的功能较为简单，在它的基础上 Spring 设计了一个更为高级的接口 **ApplicationContext**。而 ApplicationContext 类则拓展了（继承）消息国际化接口**MessageSource**、环境可配置接口**EnvironmentCapable**、应用事件发布接口**ApplicationEventPublisher**、资源模式解析接口**ResourcePatternResolver**。在 Spring 的体系中**BeanFactory**和**ApplicationContext**是最为重要的两个接口。我们使用的大部分 Spring Ioc 容器都是 ApplicationContext 的实现类。

## 相关注解

@**Bean**：用于装配 Bean 到 Ioc 容器中，一般用于方法上，默认把标注的**方法名**作为 Bean 名称保存到容器中，也可以直接通过 name 属性指定对应的 Bean 名称。常和@Configuration 配合使用来加载外部依赖的配置 Bean。

@**Component**:一般用于类上，标明这个类将被 Ioc 装配，默认会把**类名首字母小写**作为 Bean 名称。也可以通过 name 属性指定名称。

@**componentScan**:批量装配 Bean 的方式一般用于类上，标明将采用何种策略去扫描装配 Bean，默认只会扫描**当前类所在的包及其子包**。可以通过配置来修改规则，比如：@ComponentScan（basePackages={com.**.**},excludeFilters = {@Filter(classes = xxx.classs)},includeFilters = {@Filter(classes = xxx.class)}）根据配置和过滤规则，去加载指定的 Bean。还有一个**lazyInit**属性来开启延**迟依赖注入**，不让容器初始化的时候就进行实例化和依赖注入。

@**Autowired**:依赖注入的常用注解之一。一般用于属性上，也可以用于**方法、方法**的参数中。是**根据类型**去匹配对应的 Bean，如果找到的 Bean 不唯一，会**根据属性的名称和 Bean 的名称**去匹配，找不到则抛出异常。默认必须找到对应的 Bean，可以通过配置 required 属性来关闭。

@**Primary**: 当使用@Autowired 按照类型进行注入的时候，如果存在多个，可以通过给某个 bean 加上这个注解，那么会优先进行注入，当多个这个类型的 bean 都标注了这个注解，这个时候依然不清楚应该注入哪个 bean，可以使用@**Qualifier**来配置 name 来确定最终装配哪个 bean。
