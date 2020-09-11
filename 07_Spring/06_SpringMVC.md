# SpringMVC

1. 轻量级
2. 高效，基于请求响应的MVC框架
3. 和Spring结合好
4. 约定优于配置
5. 功能强大：Restful，数据验证，格式化...
6. 简洁灵活

DispatcherServlet的作用是将请求分发到不同的地方

1. `DispatcherServlet`表示前置控制器，是整个控制中心，用户发出的的**请求会被拦截**

   - url：`http://localhost:8080:/SpringMVC/hello`

   - localhost:8080是服务器域名
   - SpringMVC部署在服务器上的web站点
   - hello表示控制器

2. HandlerMapping为**处理器映射**

   `DispatcherServlet`调用HanderMapping，HandlerMaping根据请求url查找Handler

3. HandlerExecution表示具体的Handler，其主要作用是根据url查找控制器

4. HandlerExecution将解析后的信息传递给`DispatcherServlet`

5. HandlerAdapter表示处理器适配器，其按照待定的规则执行Handler

6. Handler让具体的Controller执行

7. Controller将具体的执行信息返回给HandlerAdapter
8. HandlerAdapter将视图逻辑名或模型传递给DispatcherServlet
9. DIspatcherServlet调用视图解析器来，解析HandlerAdapter传递的逻辑视图
10. 视图解析器将逻辑视图名传给DispatcherServlet
11. `DispatcherServlet`根据视图解析器的视图结果
12. 最终视图呈现给用户