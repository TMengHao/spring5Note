## 一、Servlet规范介绍

#### 1.Servlet规范来自于JAVAEE规范中的一种

#### 2.作用：

> 1. 在Servlet规范中，指定动态资源文件开发步骤
> 2. 在Servlet规范中，指定Http服务器调用动态资源文件规则
> 3. 在Servlet规范中，指定Http服务器管理动态资源文件实例对象规则

## 二、Servlet接口实现类

1. Servlet接口来自于Servlet规范下一个接口，这个接口存在Http服务器提供jar包
2. Tomcat服务器下Lib文件有一个servlet-api.jar存放的Servlet接口（java.servlet.Servlet接口）
3. Servlet规范中任务，Http服务器能调用的动态资源文件必须是一个Servlet接口实现类

## 三、Servlet接口实现类开发步骤

> 第一步：创建一个JAVA类继承与HttpServlet父类，使之成为一个Servlet接口实现类
>
> 第二步：重写httpServlet父类两个方法，doGet或doPost

