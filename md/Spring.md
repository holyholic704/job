## Spring

开源的轻量级框架，主要用来简化 Java 应用的开发，并通过POJO为基础的编程模型促进良好的编程习惯

### 优点

* 轻量：Spring 是轻量的，基本的版本大约 2MB
* 方便解耦，简化开发：可以将对象依赖关系的维护交给 Spring 管理
* IoC 控制反转：对象的创建由 Spring 完成，并将创建好的对象注入给使用者
* AOP 编程的支持：可以将一些日志，事务等操作从业务逻辑的代码中抽取出来，提高代码的复用性
* 声明式事务的支持：只需要通过配置就可以完成对事务的管理，而无需手动编程
* 方便集成各种优秀框架：其内部提供了对很多优秀框架的直接支持
* 非侵入式：Spring 的 API 不会在业务逻辑的代码中出现，可以很方便的移植到其他框架上
* 异常处理：Spring 提供方便的 API 把具体技术相关的异常，如由 JDBC，Hibernate 抛出的异常，转化为一致的 unchecked 异常

### 缺点
* Spring 像一个胶水，将框架黏在一起，后面拆分的话就不容易拆分了
* Spring 的 JSP 代码过多，控制器过于灵活，缺乏一个公用的控制器，不适合分布式

### 三大核心思想

依赖注入、控制反转、面向切面编程

## Spring 项目

### 编写 Spring 程序

1. 导入 JAR 包：spring-context
2. 编写配置文件
3. 创建接口和实现类

### 接口与实现类

```java
public interface PersonService {
    void say();
}

public class PersonServiceImpl implements PersonService {
    public void say() {
        System.out.println("fuck you");
    }
}
```

### 配置文件

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">
	
    <!-- id不能重复，class中只能是类，不能是接口-->
    <bean id="personService" class="com.test.service.impl.personServiceImpl"/>

</beans>
```

### 测试类

```java
@Test
public void oldMehod() {
    // 之前需要自己手动去创建对象
    PersonService ps = new PersonServiceImpl();
    ps.say();
}

@Test
public void newMehod(){
    // 读取配置文件
    ApplicationContext context = new ClassPathXmlApplicationContext("application-context.xml");
    // 从Spring获取对象
    PersonService ps = (PersonService) context.getBean("personService");
    ps.say();
}
```

## Spring 模块

![spring](../md.assets/spring.png)









## IoC

控制反转（IoC，Inversion of Control）控制反转是一种 **思想**。是指创建对象的控制权的转移，把对象的创建和管理交给外部容器完成，即 IoC 容器

### IoC 的作用

IoC 思想最核心的地方在于，资源不由使用资源的双方管理，而由不使用资源的第三方管理

* 资源集中管理，实现资源的可配置和易管理
* 降低了使用资源双方的依赖程度，即降低了耦合度

### IoC 容器

#### BeanFactory 接口

IoC 容器的基本实现，Spring 里面最底层的接口，包含了各种 bean 的定义，读取 bean 配置文档，管理 bean 的加载、实例化，控制 bean 的生命周期，维护 bean 之间的依赖关系

```java
@Test
public void test() {
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
    // 使用绝对路径
    reader.loadBeanDefinitions(new FileSystemResource("D:/test/src/main/resources/application-context.xml"));

    PersonService ps = (PersonService) factory.getBean("personService");
    ps.say();
}
```

#### ApplicationContext 接口

BeanFactory 的子接口，除了提供 BeanFactory 所具有的功能外，提供了更多的高级特性

- 继承 MessageSource，因此支持国际化
  - 根据用户的语言设置显示相应的语言和提示
- 统一的资源文件访问方式
- 提供在监听器中注册 bean 的事件
- 同时加载多个文件
- 载入多个（有继承关系）上下文，便得每一个上下文都专注于一个特定的层次，比如应用的 web 层

```java
@Test
public void test() {
    // 从类路径下加载配置文件 
    ApplicationContext context = new ClassPathXmlApplicationContext("application-context.xml");
    // 从文件系统中加载配置文件 
    ApplicationContext context=new FileSystemXmlApplicationContext("D:/test/src/main/resources/application-context.xml");
    PersonService ps = (PersonService) factory.getBean("personService");
    ps.say();
}
```

#### BeanFactory 和 ApplicationContext 的区别

* BeanFactroy 是以 **延迟加载** 形式来注入 bean 的。在读取配置文件之后不会创建里面 bean 的对象，即只有在使用到某个 bean 时，才会进行加载实例化，但这样就难以发现一些可能存在的配置问题。如果 bean 的某一个属性没有注入，会到第一次使用 getBean 方法才会抛出异常
  * 应用启动的时候占用资源很少，对资源要求较高的应用，或者需要在一些配置较差的机器中运行程序，比较有优势，但事务的管理、AOP 功能将会失效

* ApplicationContext 是在容器启动时，**一次性创建了所有的 bean 对象**。在容器启动时，就可以发现一些可能存在的配置问题，有利于检查所依赖属性是否注入。ApplicationContext 启动后就预载入所有的单实例 bean，确保当你需要的时候，可以立马使用
  * 所有的 bean 在启动的时候都加载，系统运行的速度快
  * 在启动的时候加载了所有的 bean，可以在系统启动的时候，尽早的发现系统中的配置问题，建议web应用，在启动的时候就把所有的 bean 都加载了，把费时的操作放到系统启动中完成
  * 唯一的不足是占内存空间，当应程序配置 bean 较多时，程序启动较慢

* BeanFactory 通常以编程的方式被创建，ApplicationContext 还能以声明的方式创建，如使用 ContextLoader

* BeanFactory 和 ApplicationContext 都支持 BeanPostProcessor、BeanFactoryPostProcessor，但两者之间的区别是：BeanFactory 需要手动注册，而 ApplicationContext 则是自动注册

## Bean

bean 是被实例的，组装的以及被 Spring 容器管理的 Java 对象

### Bean 的装配

IoC 容器将 bean 对象创建好并传递给使用者的过程叫做 bean 的装配

* 默认的方式，IoC 容器会 **调用 bean 的无参构造方法** 来创建对象，所以要保证这些 bean 有无参构造方法

```xml
<bean id="personService" class="com.test.service.impl.personServiceImpl"/>
```

* 实例工厂，需要自己定义一个工厂类，在该类中的方法都是非静态的

```java
public class MyBeanFactory {
    public PersonService create() {
        return new PersonServiceImpl();
    }
}
```

```xml
<bean id="factory" class="com.test.util.MyBeanFactory"/>
<bean id="personService" factory-bean="factory" factory-method="create"/>
```

* 静态工厂，需要自己定义一个工厂类，在该类中的方法都是 static 修饰的

```java
public class MyBeanFactory {
    public static PersonService create() {
        return new PersonServiceImpl();
    }
}
```

```xml
<bean id="personService" class="com.test.util.MyBeanFactory" factory-method="create"/>
```

### Bean 的作用域

使用 IoC 容器为我们创建对象的时候，可以设定该对象的作用域，**默认是单例的**

```xml
<bean id="test" scope="prototype" class="com.test.scopeTest"/>
```

![20190518204607](../md.assets/20190518204607.png)

* request、session、globalSession 作用域，只有在 Web 应用中使用 Spring 时才有效



### Bean 生命周期











## DI

ioc是一种思想，有一些实现方式，其中较为常用的一种是依赖注入(Dependency Injection，简称DI)，依赖注入是指程序运行过程中，若需要调用另一个对象协助时，无须在代码中创建被调用者，而是依赖于外部容器，由外部容器创建后传递给程序。
依赖注入让 Spring 的Bean之间以配置文件的方式组织在一起，而不是以硬编码的方式耦合在一起。

**依赖注入(DI，Dependency Injection)**

ioc是一种思想，有一些实现方式，其中较为常用的一种是依赖注入。**依赖注入是指程序运行过程中，若需要调用另一个对象协助时，无须在代码中创建被调用者，而是依赖于外部容器，由外部容器创建后传递给程序。**依赖注入让Spring 的Bean之间以配置文件的方式组织在一起，而不是以硬编码的方式耦合在一起。在之前的第一个程序中实际上就是使用了依赖注入这种方式获取的StudentServiceImpl对象。

























 

## Spring 中都用到了哪些设计模式

**工厂模式**

BeanFactory就是简单工厂模式的体现，用来创建对象的实例；

**单例模式**

Bean默认为单例模式。

**代理模式**

Spring的AOP功能用到了JDK的动态代理和CGLIB字节码生成技术；

**模板方法**

用来解决代码重复的问题。比如. RestTemplate, JmsTemplate, JpaTemplate。

**观察者模式**

定义对象键一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知被制动更新，如Spring中listener的实现--ApplicationListener。