# Servlet

Servlet运行在Servlet容器中，其生命周期由容器来管理。Servlet的生命周期通过javax.servlet.Servlet接口中的init()、service()和destroy()方法来表示

**生命周期**

Servlet的生命周期包含了下面4个阶段：

1. 加载和实例化

2. 初始化

3. 请求处理

4. 服务终止

```java
public interface Servlet {
	
    // 初始化方法
    public void init(ServletConfig config) throws ServletException;
    
    // 获取配置
    public ServletConfig getServletConfig();
    
    // 提供服务的方法
    public void service(ServletRequest req, ServletResponse res)
	throws ServletException, IOException;
	
    // 获取servlet的信息
    public String getServletInfo();
	
    // 关闭时执行
    public void destroy();
}
```

**工作流程**

Web服务器在与客户端交互时Servlet的工作过程是:

1.     在客户端对web服务器发出请求
2.     web服务器接收到请求后将其发送给Servlet
3.     Servlet容器为此产生一个实例对象并调用ServletAPI中相应的方法来对客户端HTTP请求进行处理，然后将处理的响应结果返回给WEB服务器
4.     web服务器将从Servlet实例对象中收到的响应结构发送回客户端

**Service方法**

Servlet实例通过ServletRequest对象得到客户端的相关信息和请求信息，在对请求进行处理后，调用ServletResponse对象的方法设置响应信息。

1. service()方法的职责

   service()方法为Servlet的核心方法，客户端的业务逻辑应该在该方法内执行，典型的服务方法的开发流程为：解析客户端请求-〉执行业务逻辑-〉输出响应页面到客户端

2. service()方法与线程

   为了提高效率，Servlet规范要求一个Servlet实例必须能够同时服务于多个客户端请求，即service()方法运行在多线程的环境下，Servlet开发者必须保证该方法的线程安全性。

**实现抽象类**

1. 接收处理请求
2. 给出响应信息



# cookie

```java
Cookie[] cookies = req.getCookies(); // 获取cookie
cookie.getName();
cookie.getValue();

Cookie cookie = new Cookie("time", System.currentTimeMillis()+"");
cookie.setMaxAge(1000);
resp.addCookie(cookie);
```

cookie的一些点：

* 一个cookie只能保存一个信息
* 一个web站点可以发送多个cookie，最多20个cookie
* cookie限制4kb
* 300cookie
* 删除cookie
  * 不设置有效期，关闭失效
  * 设置有效期0，（同一个域名下不同链接）

* 编码

  ```java
  Cookie cookie = new Cookie("name", URLEncoder.encode("杨楠", "utf-8"));
  out.write(URLDecoder.decode(cookie.getValue(),"utf-8"))
  ```

# Session

Session

* 服务器会给每一个用户（浏览器）创建一个Session对象的
* 一个Session独占一个浏览器，浏览器没有关闭，这个Session就存在
* 用户登录之后，整个网站都可以访问

Session和Cookie的区别

* Cookie把用户的数据写给了用户的浏览器，浏览器保存
* Session把用户的数据写入到用户独占Session中，服务端保存

* Session对象由服务器创建

使用场景

* 保存登录信息
* 购物车信息
* 经常使用的数据在Session中

# JavaBean

实体类

* 无参构造
* 属性私有
* get/set方法

一般用于和数据库的字段做映射，ORM对象关系映射

* 数据库表对应类
* 字段对应类的属性
* **行**记录对应类的对象