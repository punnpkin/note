# rest-service

[spring](https://spring.io/guides/gs/rest-service)

## 代码

```java
// Greeting.java
package com.example.restservice;

public class Greeting {

    private final long id;
    private final String content;

    public Greeting(long id, String content) {
        this.id = id;
        this.content = content;
    }

    public long getId() {
        return id;
    }

    public String getContent() {
        return content;
    }
}
```

```java
//GreetingController.java
package com.example.restservice;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.atomic.AtomicLong;

@RestController // 这个注解将类标注为控制器，每个方法都将返回对象
public class GreetingController {
    private static final String template = "Hello, %s!";
    // 原子long，多用于多线程环境下的counter
    private final AtomicLong counter = new AtomicLong();
	
    // /greeting下get方法的映射
    @GetMapping("/greeting")
    public Greeting greeting(
        // 请求参数设定
        @RequestParam(value = "name", defaultValue = "World") String name){
        // 返回Greeting对象，spring’s HTTP message converter 会转为json
        return new Greeting(counter.incrementAndGet(), String.format(template,name));
    }
}
```

```java
// RestserviceApplication.java
@SpringBootApplication
public class RestserviceApplication {

    public static void main(String[] args) {
        SpringApplication.run(RestserviceApplication.class, args);
    }

}
```

`@SpringBootApplication`会添加以下方法：

* `Configuration`：Tags the class as a source of bean definitions for the application context.
* `EnableAutoConfiguration`：告诉Spring Boot根据类路径设置，其他bean和各种属性设置开始添加bean。

* `@ComponentScan`: Tells Spring to look for other components, configurations, and services in the `com/example` package, letting it find the controllers.

The `main()` method uses Spring Boot’s `SpringApplication.run()` method to launch an application.