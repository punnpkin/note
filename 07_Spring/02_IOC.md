# 02 IOC理论

> 对象创建的控制权反转给用户
>
> **控制反转IoC(Inversion of Control)，是一种设计思想，DI(依赖注入)是实现IoC的一种方法**，也有人认为DI只是IoC的另一种说法。没有IoC的程序中 , 我们使用面向对象编程 , 对象的创建与对象间的依赖关系完全硬编码在程序中，对象的创建由程序自己控制，控制反转后将对象的创建转移给第三方，个人认为所谓控制反转就是：**获得依赖对象的方式反转**了。

**程序**不再管理对象的创建，而是接收对象：

```java
public class UserServiceImpl implements UserService {

    private UserDao userDao;

    // 利用set动态注入
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public void getUser() {
        userDao.getUser();
    }
}
```

使用set，使得对象可以让**用户**创建：

```java
public class MyTest {

    public static void main(String[] args) {
        // 业务层，不接触DAO层
        UserServiceImpl userService = new UserServiceImpl();
		// 用户创建对象
        userService.setUserDao(new UserDaoMysqlImpl());
        // 用户创建对象
        userService.setUserDao(new UserDaoImpl());

        userService.getUser();
    }
}
```

---

控制反转是一种通过描述（XML或注解）并通过第三方去生产或获取特定对象的方式。在Spring中实现控制反转的是IoC容器，其实现方法是依赖注入（Dependency Injection,DI）。

# 03 XML

## 3.1 代码

### 1. Hello类

```java
public class Hello {
    private String str;

    @Override
    public String toString() {
        return "Hello{" +
                "str='" + str + '\'' +
                '}';
    }

    public String getStr() {
        return str;
    }

    public void setStr(String str) {
        this.str = str;
    }
}
```

### 2. xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="hello" class="com.yang.Hello">
        <property name="str" value="Spring" />
    </bean>
</beans>
```

### 3. Test

```java
public class MyTest {
    public static void main(String[] args) {
        // 获取上下文context对象
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        // 对象都在Spring中管理，现在只需要取出对象
        Hello hello = (Hello) context.getBean("hello");
        System.out.println(hello.toString());

    }
}
```

## 3.2 解释

问题：

- Hello 对象是谁创建的 ?  

  hello 对象是由Spring创建的

- Hello 对象的属性值是怎么设置的 ? 

  hello 对象的属性是由Spring容器(xml文件)设置的，通过更改配置文件，可以创建不同的对象

- 对象如何使用？

  从context中取出来

这个过程就叫控制反转 :

- 控制 : 谁来控制对象的创建 , 传统应用程序的对象是由程序本身控制创建的 , 使用Spring后 , 对象是由**Spring来创建**的
- 反转 : 程序本身不创建对象 , 而变成被动的接收对象 .

依赖注入 : 就是**利用set方法**来进行注入的.

IOC是一种编程思想，由**主动的编程变成被动的接收**

可以通过`newClassPathXmlApplicationContext`去浏览一下底层源码 .

# 04 对象是如何创建的

配置文件加载的时候，容器管理的对象就初始化了

## 4.1 无参构造

```xml
<bean id="hello" class="com.yang.Hello">
    <property name="str" value="Spring" />
</bean>
```

```java
public static void main(String[] args) {
    // 获取上下文context对象
    ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
    // 对象都在Spring中管理，现在只需要取出对象
    Hello hello = (Hello) context.getBean("hello");
    System.out.println(hello.toString());
}
```

在`Hello hello = (Hello) context.getBean("hello");`的时候执行了无参构造方法创建对象。

## 4.2 有参构造

下标，名字，类型

```xml
<!-- 第一种根据index参数下标设置 -->
<bean id="userT" class="com.kuang.pojo.UserT">
   <!-- index指构造方法 , 下标从0开始 -->
   <constructor-arg index="0" value="kuangshen2"/>
</bean>
<!-- 第二种根据参数名字设置 -->
<bean id="userT" class="com.kuang.pojo.UserT">
   <!-- name指参数名 -->
   <constructor-arg name="name" value="kuangshen2"/>
</bean>
<!-- 第三种根据参数类型设置 -->
<bean id="userT" class="com.kuang.pojo.UserT">
   <constructor-arg type="java.lang.String" value="kuangshen2"/>
</bean>
```

# 05 Spring 配置

## 5.1 alias

```xml
<alias name="hello" alias="myHello" />
```

## 5.2 Bean

```xml
<!--
id: 对象名
class：bean对象的限定名
name：别名，可以同时取多个别名
-->
<bean id="hello1" class="com.yang.Hello" name="hello2,hello3,hello4">
    <property name="str" value="Spring" />
</bean>
```

## 5.3 import

import用于团队开发使用，多个xml之间相互导入的问题

```
applicationContext.XML
```



# 06 依赖注入

如果一个类A 的功能实现需要借助于类B，那么就称类B是类A的**依赖**，如果在类A的内部去实例化类B，那么两者之间会出现较高的**耦合**，一旦类B出现了问题，类A也需要进行改造，如果这样的情况较多，每个类之间都有很多依赖，那么就会出现牵一发而动全身的情况，**程序会极难维护**，并且很容易出现问题。

要解决这个问题，就要把A类对B类的控制权抽离出来，交给一个第三方去做，把控制权反转给第三方，就称作**控制反转（IOC Inversion Of Control）**。

**控制反转是一种思想**，是能够解决问题的一种可能的结果，而**依赖注入（Dependency Injection）**就是其最典型的实现方法。由第三方（**我们称作IOC容器**）来控制依赖，把他通过**构造函数、属性或者工厂模式**等方法，注入到类A内，这样就极大程度的对类A和类B进行了**解耦**。

## 6.1 构造依赖注入

上文

## 6.2 Set依赖注入

环境搭建：

1. 复杂类型

   ```java
   public class Address {
       private String address;
   
       public String getAddress() {
           return address;
       }
   
       public void setAddress(String address) {
           this.address = address;
       }
   }
   ```

2. 测试对象

   ```java
   public class Student {
   
       private String name;
       private Address address;
       private String[] books;
       private List<String> hobbys;
       private Map<String,String> card;
       private Set<String> games;
       private Properties info;
       private String wife;
   }
   ```

3. beans.xml

   ```xml
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <bean id="address" class="com.yang.Address" />
   
       <bean id="student" class="com.yang.Student">
           <!--普通-->
           <property name="name" value="yang" />
           <!--Bean-->
           <property name="address" ref="address" />
           <!--数组-->
           <property name="books">
               <array>
                   <value>book1</value>
                   <value>book2</value>
                   <value>book3</value>
               </array>
           </property>
           <!--List-->
           <property name="hobbys">
               <list>
                   <value>hobby1</value>
                   <value>hobby2</value>
                   <value>hobby3</value>
               </list>
           </property>
   
           <!--Map-->
           <property name="card">
               <map>
                   <entry key="id" value="123456"></entry>
                   <entry key="bank" value="1234567"></entry>
               </map>
           </property>
   
           <!--Set-->
           <property name="games">
               <set>
                   <value>Gta5</value>
                   <value>LOL</value>
               </set>
           </property>
   
           <!--Null-->
           <property name="wife">
               <null />
           </property>
   
           <!--property-->
           <property name="info">
               <props>
                   <prop key="driver">1</prop>
                   <prop key="url">2</prop>
                   <prop key="username">root</prop>
                   <prop key="passwd">1234556</prop>
               </props>
           </property>
       </bean>
   </beans>
   ```

4. 测试类

   ```java
   public class MyTest {
       public static void main(String[] args) {
           ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
   
           Student student = (Student) context.getBean("student");
           System.out.println(student.getName());
       }
   }
   ```

## 6.3 拓展注入

p空间，set注入

```xml
<bean id="user" class="com.yang.User" p:age="18" p:name="yang"></bean>
```

c空间，构造器注入

```xml
<bean id="user2" class="com.yang.User" c:age="18" c:name="wang"></bean>
```



# 07 bean作用域

在Spring中，那些组成应用程序的主体及由Spring IoC容器所管理的对象，**被称之为bean**。

作用域有：

## 7.1 Singleton

当一个bean的作用域为Singleton，那么Spring IoC容器中只会存在一个共享的bean实例，并且所有对bean的请求，只要id与该bean定义相匹配，则只会返回bean的同一实例。Singleton是单例类型，就是在**创建起容器时就同时自动创建了一个bean的对象**，不管你是否使用，他都存在了，每次获取到的对象都是同一个对象。注意，Singleton作用域是Spring中的缺省作用域。

```xml
 <bean id="ServiceImpl" class="cn.csdn.service.ServiceImpl" scope="singleton">
```

## 7.2 Prototype

当一个bean的作用域为Prototype，表示一个bean定义对应多个对象实例。Prototype作用域的bean会导致在每次对该bean请求（将其注入到另一个bean中，或者以程序的方式调用容器的getBean()方法）时都会创建一个新的bean实例。Prototype是原型类型，它在我们创建容器的时候并没有实例化，而是当我们**获取bean的时候才会去创建一个对象**，而且我们每次获取到的对象都不是同一个对象。根据经验，对有状态的bean应该使用prototype作用域，而对无状态的bean则应该使用singleton作用域。

```xml
<bean id="account" class="com.foo.DefaultAccount" scope="prototype"/>  
<!--或者-->
<bean id="account" class="com.foo.DefaultAccount" singleton="false"/>
```

## 7.3 Request

当一个bean的作用域为Request，表示在一次HTTP请求中，一个bean定义对应一个实例；即每个HTTP请求都会有各自的bean实例，它们依据某个bean定义创建而成。该作用域仅在基于web的Spring ApplicationContext情形下有效。考虑下面bean定义：

```xml
<bean id="loginAction" class=cn.csdn.LoginAction" scope="request"/>
```

针对每次HTTP请求，Spring容器会根据loginAction bean的定义创建一个全新的LoginAction bean实例，且该loginAction bean实例仅在当前HTTP request内有效，因此可以根据需要放心的更改所建实例的内部状态，而其他请求中根据loginAction bean定义创建的实例，将不会看到这些特定于某个请求的状态变化。当处理请求结束，request作用域的bean实例将被销毁。

## 7.4 Session

当一个bean的作用域为Session，表示在一个HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。考虑下面bean定义：

```xml
<bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/>
```

针对某个HTTP Session，Spring容器会根据userPreferences bean定义创建一个全新的userPreferences bean实例，且该userPreferences bean仅在当前HTTP Session内有效。与request作用域一样，可以根据需要放心的更改所创建实例的内部状态，而别的HTTP Session中根据userPreferences创建的实例，将不会看到这些特定于某个HTTP Session的状态变化。当HTTP Session最终被废弃的时候，在该HTTP Session作用域内的bean也会被废弃掉。