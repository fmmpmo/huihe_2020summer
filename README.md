# huihe_2020summer
2020假期spring、vue学习

# 1、注解和反射
## 1.注解

Java 注解（Annotation）又称 Java 标注，是 JDK5.0 引入的一种注释机制。

Java 语言中的**类、方法、变量、参数和包**等都可以被标注。

### 内置的注解

**作用在代码的注解是**

- @Override - 检查该方法是否是重写方法。如果发现其父类，或者是引用的接口中并没有该方法时，会报编译错误。

- @Deprecated - 标记过时方法。如果使用该方法，会报编译警告。

- @SuppressWarnings - 指示编译器去忽略注解中声明的警告。

**作用在其他注解的注解(或者说 元注解)是:**

- @Retention - 标识这个注解怎么保存，是只在代码中，还是编入class文件中，或者是在运行时可以通过反射访问。

- @Documented - 标记这些注解是否包含在用户文档中。

- @Target - 标记这个注解应该是哪种 Java 成员。

- @Inherited - 标记这个注解是继承于哪个注解类(默认 注解并没有继承于任何子类)

**从 Java 7 开始，额外添加了 3 个注解:**

- @SafeVarargs - Java 7 开始支持，忽略任何使用参数为泛型变量的方法或构造函数调用产生的警告。

- @FunctionalInterface - Java 8 开始支持，标识一个匿名函数或函数式接口。

- @Repeatable - Java 8 开始支持，标识某注解可以在同一个声明上使用多次。

### 3 个主干类

Annotation.java

```java
 package java.lang.annotation;
 public interface Annotation {

   boolean  equals(Object obj);

   int hashCode();

  String toString();

  Class<? extends Annotation> annotationType();
}
```

ElementType.java

```java
 package java.lang.annotation;

 public enum ElementType {
  TYPE,        /* 类、接口（包括注释类型）或枚举声明  */

  FIELD,        /* 字段声明（包括枚举常量）  */

  METHOD,       /* 方法声明  */

  PARAMETER,      /* 参数声明  */

  CONSTRUCTOR,     /* 构造方法声明 */

  LOCAL_VARIABLE,   /* 局部变量声明  */

  ANNOTATION_TYPE,   /* 注释类型声明  */

  PACKAGE        /* 包声明 */
}
```

RetentionPolicy.java

```java
package java.lang.annotation;
public enum RetentionPolicy {
  SOURCE,    /* Annotation信息仅存在于编译器处理期间，编译器处理完之后就没有该Annotation信息了  */

  CLASS,       /* 编译器将Annotation存储于类对应的.class文件中。默认行为  */

  RUNTIME       /* 编译器将Annotation存储于class文件中，并且可由JVM读入 */
}
```

**每 1 个 Annotation 都与 1 个 RetentionPolicy关联，并且与 1～n 个 ElementType关联**

**自定义注解**

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@interface myAnnotation {
    String value() default "";
}
```

### 作用

Annotation 是一个辅助类，它在 Junit、Mybatis、Spring 等工具框架中被广泛使用。

1）编译检查  -- @Override、@Deprecated、@SuppressWarnings

2) 在反射中使用 Annotation

3) 根据 Annotation 生成帮助文档 -@Documented

## 2.反射

**[JAVA反射机制](https://baike.baidu.com/item/JAVA反射机制/6015990)是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能称为java语言的反射机制。**

**反射被视为动态语言的关键**

**静态语言（强类型语言）**

静态语言是在编译时变量的数据类型即可确定的语言，多数静态类型语言要求在使用变量之前必须声明数据类型。 
例如：C++、Java、Delphi、C#等。

**动态语言（弱类型语言）**

动态语言是在运行时确定数据类型的语言。变量使用之前不需要类型声明，通常变量的类型是被赋值的那个值的类型。 
例如PHP/ASP/Ruby/Python/Perl/ABAP/SQL/JavaScript/Unix Shell等等。

**像这样的Python语句的赋值操作和eval函数，静态语言就做不到**

![](img\1.png)

**Java不是动态语言，但Java具有一定的动态性，表现在以下几个方面:**

- **反射机制；**

- 动态字节码操作；（动态(运行中)改变某个类的结构（添加/删除/修改 新的类/属性/方法））

- 动态编译；(通过Runtime调用javac, 通过JavaCompiler动态编译)

- 执行其他脚本代码；(例如 通过js引擎可以执行javascript代码)



### 核心类

`java.lang.reflect`包反射核心类有核心类`Class`、`Constructor`、`Method`、`Field`、`Parameter`

#### Class

​		**加载完某个类之后，在堆内存的方法区中就产生了对应的Class类型的对象（一个类只有一个Class对象），这个Class对象就包含了完整的类的结构信息。可以通过这个对象看到类的结构。**

1. 在Object类中定义了public final Class getClass()，该方法会被所有的子类继承。
2. 一个Class对象对应的是一个加载到JVM中的一个.class文件；
3. 一个加载的类在JVM中，只有一个Class实例；
4. 通过Class可以完整地得到一个类中的所有加载的结构

#####  获取Class实例

```java
/*
若已知具体的类，通过类的class属性获取，该方法最为安全可靠，程序性能最高。  Class clazz = 类名.class;
*/
Class c1 = User.class; 
/*
已知某个类的实例，通过调用该实例的getClass()方法获取Class对象。  Class clazz = 实例.getClass();
*/
Class c2 = new User().getClass();
/*
已知一个类的全类名，且该类在类路径下，可通过Class类的静态方法forName()获取，需要处理ClassNotFoundException异常。 Class clazz = Class.forName("包名.类名");
全限定类名：包名+类名
*/
Class c3 = Class.forName("com.huihe.entity.User");
/*
内置基本数据类型的TYPE属性 如 Integer.TYPE
*/
System.out.println(c1 == c2 && c2 == c3); //true 一个类只有一个Class对象
```

##### 什么类型有Class对象

```java
Class c1 = Object.class; //类 class java.lang.Object
Class c2 = Comparable.class; // 接口 interface java.lang.Comparable
Class c3 = String[].class; //一维数组 class [Ljava.lang.String;
Class c4 = int[][].class; //二维数组 class [[I
Class c5 = ElementType.class; //枚举 class java.lang.annotation.ElementType
Class c6 = Override.class; // 注解 interface java.lang.Override
Class c7 = int.class; // 基本数据类型 int
Class c8 = void.class; // void void
```

##### 常用API

```java
Class<User> c = (Class<User>) Class.forName("com.huihe.entity.User");//获取Class对象
String name = c.getName();//全限定类名
String simpleName = c.getSimpleName();//类名
Field username = c.getDeclaredField("username");//某个字段
Field[] declaredFields = c.getDeclaredFields();//所有私有字段
Method getUsername = c.getDeclaredMethod("getUsername");//某个方法
Method[] declaredMethods = c.getDeclaredMethods();//所有方法
Constructor<User> declaredConstructor = c.getDeclaredConstructor();//某个构造
Constructor<?>[] declaredConstructors = c.getDeclaredConstructors();//所有构造
User user = c.newInstance(); //通过Class调用无参构造生成对象
User user1 = declaredConstructor.newInstance(); //通过无参构造器对象构造对象
User user2 = (User) declaredConstructors[1].newInstance("1","2"); //通过全参构造器对象构造对象
username.setAccessible(true); //跳过权限检测
System.out.println(username.get(user2)); //调用属性对象的方法，查看在某个实体中的值
System.out.println(getUsername.invoke(user2)); //方法对象的方法，激活某个实体的方法
System.out.println(Arrays.toString(c.getAnnotations())); //查看类上的注解
System.out.println(Arrays.toString(username.getAnnotations())); //查看属性上的注解
System.out.println(Arrays.toString(getUsername.getAnnotations())); //查看方法上的注解
for(Annotation annotation :c.getAnnotations()){ //遍历类上注解
    if (annotation.annotationType().equals(MyAnnotation.class)){ //如果有我的自定义注解
        User instance = c.newInstance(); //构造一个user对象
    }
}
```

## 3.例子

### 3.1、基于注解动态实例化对象

#### 3.1.1 注解

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface MyAnnotation {
    String value() default "";
}
```

#### 3.1.2 类

```java
package com.huihe.bean;

import com.huihe.annotation.MyAnnotation;

public class User {
    //Flied
    @MyAnnotation("hxj")
    private String username;

    @MyAnnotation("123")
    private String password;

    public User(){}
    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    //Method
    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public String toString() {
        return "User{" +
                "username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}
```

#### 3.1.3 测试程序

```java
@Test
public void test3() throws ClassNotFoundException, IllegalAccessException, InstantiationException {
    Class<?> c = Class.forName("com.huihe.bean.User");
    Field[] fields = c.getDeclaredFields();
    Object instance = c.newInstance();
    for(Field field : fields){
        MyAnnotation annotation = field.getAnnotation(MyAnnotation.class);
        field.setAccessible(true);
        System.out.println(annotation.value());
        field.set(instance, annotation.value());
    }
    System.out.println(instance);
}
```

### 3.2、基于配置文件动态实例化对象

#### 3.2.1 bean.properties

```properties
class=com.huihe.bean.User
username=hxj
password=123
```

#### 3.2.2 类 

同3.1.2

#### 3.2.3 测试程序

```java
@Test
public void test4() throws IOException, ClassNotFoundException, IllegalAccessException, InstantiationException {
    Properties properties = new Properties();
    InputStream inputStream = this.getClass().getResourceAsStream("/bean.properties");
    properties.load(inputStream);
    String clz = properties.getProperty("class", "");
    Class<?> c = Class.forName(clz);
    Object instance = c.newInstance();
    for(Field field : c.getDeclaredFields()){
        field.setAccessible(true);
        field.set(instance, properties.getProperty(field.getName(),""));
    }
    System.out.println(instance);
}
```

# Spring框架概述

### 优点：

1. 方便解耦，开发（Spring就是一个大工厂，可以将所有对象的创建和依赖关系维护都交给spring管理）**IoC**
2. spring支持**aop**编程（spring提供面向切面编程，可以很方便的实现对程序进行权限拦截和运行监控等功能）
3. 声明式事务的支持（通过配置就完成对事务的支持，不需要手动编程）
4. 方便程序的测试，spring 对junit支持，可以通过注解方便的测试spring 程序
5. 方便集成各种优秀的框架（Hibernate 、Mybatis、Redis、Ehcache、Quartz、RabbitMQ、Shiro 等）
6. 降低javaEE API的使用难度（Spring 对javaEE开发中非常难用的一些API 例如JDBC, javaMail, 远程调用等，都提供了封装，是这些API应用难度大大降低）

**Spring是一个轻量级控制反转(IoC)和面向切面(AOP)的容器框架。**

### 缺点：

1. 大量反射，运行效率低了。
2. spring像一个胶水，将框架黏在一起，后面拆分的话就不容易拆分了。
3. 配置繁琐(spring boot简化了许多配置，”约定优于配置“)

### 组成模块：

Spring框架包含组织为约20个模块的功能。这些模块分为核心容器IoC，数据访问/集成，Web，AOP（面向切面的编程），检测，消息传递和测试，如下图所示。

![](img\spring-overview.png)

如果使用[Maven](https://maven.apache.org/)进行依赖管理，您甚至无需显式提供日志记录依赖。例如，要创建应用程序上下文并使用依赖项注入来配置应用程序，您的Maven依赖项将如下所示：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>4.3.28.RELEASE</version>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

# 2.Spring IoC容器

​	**控制反转**（Inversion of Control，缩写为**IoC**），是[面向对象编程](https://baike.baidu.com/item/面向对象编程)中的一种设计原则，可以用来减低计算机代码之间的[耦合度](https://baike.baidu.com/item/耦合度)。其中最常见的方式叫做**依赖注入**（Dependency Injection，简称**DI**），还有一种方式叫“依赖查找”（Dependency Lookup）。通过控制反转，对象在被创建的时候，由一个调控系统内所有对象的外界实体将其所依赖的对象的引用传递给它。也可以说，依赖被注入到对象中。IoC/DI

spring实现:通过反射获取**set方法**或者**构造方法**，注入所要依赖的对象。 set注入/构造注入

下图是Spring工作方式的高级视图。您的应用程序类与配置元数据结合在一起，以便在`ApplicationContext`创建和初始化之后，您便拥有了完整配置的可执行系统或应用程序。

![](img\container-magic.png)



### 1.配置文件一般格式：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

<!--    <bean id="dao" class="com.huihe.dao.impl.OracleDao">-->
<!--    </bean>-->
<!--   静态工厂  -->
<!--    <bean id="dao" class="com.huihe.factory.DaoFactory" factory-method="getInstance">-->
<!--    </bean>-->
	<!--   实例工厂  -->
    <bean id="factory" class="com.huihe.factory.DaoFactory">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="dao" factory-bean="factory" factory-method="getInstance">
        <!-- collaborators and configuration for this bean go here -->
    </bean>
</beans>
```

2.读取配置文件的Bean

```java
ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
dao = (Dao) context.getBean("dao");
dao.insert();
```

