# 10 代理模式

代理模式是AOP的**底层**

代理模式分为静态代理和动态代理：

## 10.1 静态代理

角色：

* 抽象角色：接口或者抽象类
* 真实角色：被代理的角色
* 代理角色：代理真实角色，基本操作
* 客户：访问代理对象

**代码实现**

Rent.java 即抽象角色

```java
//抽象角色：租房
public interface Rent {
   public void rent();
}
```

Host.java 即真实角色

```java
//真实角色: 房东，房东要出租房子
public class Host implements Rent{
   public void rent() {
       System.out.println("房屋出租");
  }
}
```

Proxy.java 即代理角色

```java
//代理角色：中介
public class Proxy implements Rent {

   private Host host;
   public Proxy() { }
   public Proxy(Host host) {
       this.host = host;
  }

   //租房
   public void rent(){
       seeHouse();
       host.rent();
       fare();
  }
   //看房
   public void seeHouse(){
       System.out.println("带房客看房");
  }
   //收中介费
   public void fare(){
       System.out.println("收中介费");
  }
}
```

Client.java 即客户

```java
//客户类，一般客户都会去找代理！
public class Client {
   public static void main(String[] args) {
       //房东要租房
       Host host = new Host();
       //中介帮助房东
       Proxy proxy = new Proxy(host);

       //你去找中介！
       proxy.rent();
  }
}
```

分析：在这个过程中，你直接接触的就是中介，就如同现实生活中的样子，你看不到房东，但是你依旧租到了房东的房子通过代理，这就是所谓的代理模式，程序源自于生活，所以学编程的人，一般能够更加抽象的看待生活中发生的事情。

**静态代理的好处:**

- 可以使得我们的真实角色更加纯粹 . 不再去关注一些公共的事情 .
- 公共的业务由代理来完成 . 实现了业务的分工 ,
- 公共业务发生扩展时变得更加集中和方便 .

缺点 :

- 类多了 , 多了代理类 , 工作量变大了 . 开发效率降低 .

我们想要静态代理的好处，又不想要静态代理的缺点，所以 , 就有了动态代理 !

## 10.2 动态代理

java动态代理机制中有两个重要的类和接口`InvocationHandler`（接口）和`Proxy`（类），这一个类`Proxy`和接口`InvocationHandler`是我们实现动态代理的核心。

### InvocationHandler

每一个动态代理类的调用处理程序都必须实现InvocationHandler接口，并且每个代理类的实例都关联到了实现该接口的动态代理类调用处理程序中，当我们通过动态代理对象调用一个方法时候，这个方法的调用就会被转发到实现InvocationHandler接口类的invoke方法来调用，看如下invoke方法：

```java
Object invoke(Object var1, Method var2, Object[] var3) throws Throwable;
```

### Proxy

用于创建一个代理对象的类，常用的方法是newProxyInstance方法

```java
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)
```

* loader：一个`classloader`对象，定义了由哪个`classloader`对象对生成的代理类进行加载。
* interfaces：一个interface对象数组，表示我们将要给我们的代理对象提供一组什么样的接口，如果我们提供了这样一个接口对象数组，那么也就是声明了代理类实现了这些接口，代理类就可以调用接口中声明的所有方法。
* h：一个`InvocationHandler`对象，表示的是当动态代理对象调用方法的时候会关联到哪一个`InvocationHandler`对象上，并最终由其调用。

### 用法

抽象接口

```java
public interface People {

    public String work();
}
```

真实对象

```java
public class Teacher implements People{

    @Override
    public String work() {
        System.out.println("老师教书育人...");
        return "教书";
    }
}
```

代理类

```java
package reflect;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class WorkHandler implements InvocationHandler{

    //代理类中的真实对象  
    private Object obj;

    public WorkHandler() {
        // TODO Auto-generated constructor stub
    }
    //构造函数，给我们的真实对象赋值
    public WorkHandler(Object obj) {
        this.obj = obj;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //在真实的对象执行之前我们可以添加自己的操作
        System.out.println("before invoke。。。");
        Object invoke = method.invoke(obj, args);
        //在真实的对象执行之后我们可以添加自己的操作
        System.out.println("after invoke。。。");
        return invoke;
    }
}
```

客户端

```java
package reflect;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;

public class Test {

    public static void main(String[] args) {
        //要代理的真实对象
        People people = new Teacher();
        //代理对象的调用处理程序，我们将要代理的真实对象传入代理对象的调用处理的构造函数中，最终代理对象的调用处理程序会调用真实对象的方法
        InvocationHandler handler = new WorkHandler(people);
        /**
         * 通过Proxy类的newProxyInstance方法创建代理对象，我们来看下方法中的参数
         * 第一个参数：people.getClass().getClassLoader()，使用handler对象的classloader对象来加载我们的代理对象
         * 第二个参数：people.getClass().getInterfaces()，这里为代理类提供的接口是真实对象实现的接口，这样代理对象就能像真实对象一样调用接口中的所有方法
         * 第三个参数：handler，我们将代理对象关联到上面的InvocationHandler对象上
         */
        People proxy = (People)Proxy.newProxyInstance(handler.getClass().getClassLoader(), people.getClass().getInterfaces(), handler);
        //System.out.println(proxy.toString());
        System.out.println(proxy.work());
    }
}
```

# 11 AOP

```xml
<!-- https://mvnrepository.com/artifact/org.aspectj/aspectjweaver -->
<dependency>
   <groupId>org.aspectj</groupId>
   <artifactId>aspectjweaver</artifactId>
   <version>1.9.4</version>
</dependency>
```

使用Spring实现AOP