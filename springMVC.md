## 一、特点

1. 轻量级，简单易学
2. 高效，基于请求响应的MVC框架
3. 与spring兼容性好，无缝结合
4. 约定优于配置
5. 功能强大：restful,数据验证，格式化，本地化，主题等
6. 简洁灵活

## 二、执行流程

> 1. DispatcherServlet表示前置控制器，是整个SpringMVC的控制中心，用户发出请求，DispatcherServlet接收请求并拦截请求
>
> > 1. 假设URL为：http://localhost:8080/SpringMVC/hello
> > 2. 如上URL拆分为三部分
> > 3. http://localhost:8080服务器域名
> > 4. SpringMVC部署在服务器上的web站点
> > 5. hello表示控制器
> > 6. 通过分析，url表示为：请求位于服务器localhost:8080上的SpringMVC站点的hello控制器
>
> 2. HandlerMapping为处理器映射，DispatcherServlet调用HandlerMapping，HandlerMapping根据请求url查找Handler
> 3. HandlerExecution表示具体的Handler，其主要作用是根据url查找控制器
> 4. HandlerExecution将解析后的信息传递给DispatcherServlet
> 5. HandlerAdapter表示处理适配器，其按照特定的规则去执行Handler
> 6. Handler让具体的Controller执行
> 7. Controller将具体的执行信息返回给HandlerAdapter
> 8. HandlerAdapter将视图逻辑名或模型传递给DispatcherServlet
> 9. DispatcherServlet调用视图解析器（ViewResolver）来解析HandlerAdapter传递的逻辑视图名
> 10. 视图解析器将解析的逻辑视图名传给DispatcherServlet
> 11. DispatcherServlet根据视图解析器解析的视图结果，调用具体的视图
> 12. 最终视图呈现给用户

## 三、基于XML实现

#### 1.引入相关依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.3.0</version>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>2.5</version>
</dependency>
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>jsp-api</artifactId>
    <version>2.2</version>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>
```

#### 2.创建类

```java
public class HelloController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
        ModelAndView mv = new ModelAndView();
        //业务代码
        String result = "你好，我是MVC";
        mv.addObject("msg",result);
        //视图跳转
        mv.setViewName("test");
        return mv;
    }
}
```

#### 3.配置web.xml文件

```xml
<!--配置DispatcherServlet,这个是sptingmvc的核心-->
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--DispatcherServlet要绑定的spring配置文件-->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc-servlet.xml</param-value>
    </init-param>
    <!--启动级别：1-->
    <load-on-startup>1</load-on-startup>
</servlet>
	<!--
    springmvc中， /   /*
    /:只匹配所有的请求，不会去匹配jsp页面
    /*:匹配所有的请求，包括jsp页面
    -->
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

#### 4.配置springmvc-servlet.xml文件

```xml
<!--处理器映射器-->
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
<!--处理器适配器-->
<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
<!--处理器解析器-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>
<!--创建对象-->
<bean id="/hello" class="org.learn.demon.HelloController"/>
```

#### 5.创建jsp文件

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
${msg}
</body>
</html>
```

#### 6.配置Tomcat服务器

> **添加服务器之后，到工程结构，浏览tomcat打包的war包中有没有lib文件，没有就创建一个，把所有需要的jar包放进去**

![](noteImages\tomcat配置1.JPG)

![](noteImages\tomcat配置2.JPG)

#### 7.运行结果

浏览器输入/hello显示：你好，我是MVC界面

## 四、基于注解实现

#### 1.导入依赖

> 同三、->1.依赖配置相同

#### 2.编写web.xml配置文件

```xml
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc-servlet.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

#### 3.编写spring配置文件

> 1. mvc:default-servlet-handler：使springmvc不处理静态资源  .css .js .mp4 .html .mp3
> 2. mvc:annotation-driven：支持mvc注解驱动，spirng中一般采用@RequestMapping注解完成映射关系，要想使注解生效，就需要配置DefaultAnnotationHandlerMapping和一个AnnotationMethodHandlerAdapter实例，这两个实例分别在类级别和方法级别处理，而annotation-driven可以帮我们自动完成上面的配置
> 3. 配置视图解析器，prefix为前缀，suffix为后缀

```xml
<context:component-scan base-package="org.learn.demon"/>
<mvc:default-servlet-handler/>
<mvc:annotation-driven/>
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

#### 4.创建对应的控制类

```xml
@Controller
public class HelloController {
    @RequestMapping("/h1")
    public String hello1(Model model){
        //向模型中添加属性msg与值，可以在jsp页面中取出并渲染
        model.addAttribute("msg","Hello 你好");
        return "hello";
    }
}
```

#### 5.完善前端视图和controller之间的关系

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
${msg}
</body>
</html>
```

#### 6.运行调试

## 五、注解说明

#### 1.@Controller

> 被这个注解的类中的所有的方法，如果返回值是String，并且有具体页面可以跳转，那么就会被视图解析器解析。
>
> ---注意：返回的字符串如果没有组成页面跳转拼接成的正确的URL，那么就会报错404

#### 2.@RequestMapping

> 用于映射url到控制器类或一个特定的处理程序方法，可用于类或方法上。在类上，表示类中的所有请求的方法都是以该地址作为父路径

##### 1.method方法属性值

> RequestMethod.GET
>
> RequestMethod.POST
>
> RequestMethod.DELETE......
>
> 可以点击method方法进入详细查看

以上所有的method都可以直接写为一个单独的注解（例如：@PostMapping(value="url地址") 或 @GetMapping(value="url地址") 等等）

##### 2.@PathVariable

> 让方法参数中的值对应绑定到一个URL模板变量上
>
> 加入该注解后，方法中的变量可以使URL地址发生改变，RequestMapping中value中的值就可以写为类似"/login/{a}/{b}"的方式

## 六、详解

#### 1.RESTFUL风格

> 通过不同的请求方式来实现不同的效果（同一个URL地址，不同的请求方式结果不同）

#### 2.转发和重定向

> return里面默认写字符串即为转发，字符串里面写上redirect:页面路径即可重定向页面（例:"redirect:/index.jsp"）

#### 3.接收请求参数及数据回显

> 1. 接收前端用户传递的参数，判断参数的名字，假设名字直接在方法上，可以直接使用
> 2. 假设传递的是一个对象User，匹配User对象中的字段名；如果字段名一致则接收，否则，匹配不到

#### 4.数据回显方式区别

> Model：只有几个方法适用于存储数据
>
> ModelMap：继承了LinkedMap，除了实现了自身的一些方法，同样继承了LinkedMap的方法和特性
>
> ModelAndView：在存储数据的同时，进行设置返回的逻辑视图，进行控制层的跳转

#### 5.乱码问题

在web.xml文件中配置spring过滤器

> 注意这里映射需要设置/*，因为需要匹配jsp页面

```xml
<filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

或者tomcat服务器中conf文件夹下server.xml文件中添加编码格式

```xml
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" 
           URIEncoding="utf-8"/>
```

## 七、JSON

### 一、概念

#### 1.什么是json

> 1. JSON(JavaScript Object Notation,JS对象标记)是一种轻量级的数据交换格式
> 2. 采用完全独立于编程语言的文本格式来存储和表示数据
> 3. 简洁和清晰的层次结构使得JSON成为理想的数据交换语言
> 4. 易于人阅读和编写，同时也易于机器解析和生成，并有效的提升网络传输效率

#### 2.语法

> 1. 对象表示为键值对，数据由逗号分隔
> 2. 花括号保存对象
> 3. 方括号保存数组

#### 3.配置依赖

> maven中添加jackson依赖（或者fast-json也可以）

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.12.3</version>
</dependency>
```

#### 4.创建类

> 添加@ResponseBody注解，返回json字符串，创建ObjectMapper对象，接收json字符串

```java
@RequestMapping(value = "/t1")
    @ResponseBody
    public String test1() throws JsonProcessingException {
        ObjectMapper mapper = new ObjectMapper();
        User user = new User("路飞",1,20);
        String str = mapper.writeValueAsString(user);
        return str;
    }
```

#### 5.乱码处理

> 在spirng配置文件中配置

```xml
<mvc:annotation-driven>
    <mvc:message-converters register-defaults="true">
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">
            <constructor-arg value="UTF-8"/>
        </bean>
        <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
            <property name="objectMapper">
                <bean class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
                    <property name="failOnEmptyBeans" value="false"/>
                </bean>
            </property>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```

#### 6.@RestController和@ResponseBody注解

> 1. @RestController作用在类上，代替@Controller，使用该注解默认该类返回的都是json字符串，不会被视图解析器解析
> 2. @ResponseBody作用在方法上，配合@Controller使用，被该注解注解的方法返回json字符串，不会被视图解析器解析
