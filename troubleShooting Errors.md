#### bean不存在

1. 查看bean是否注入成功！
2. Junit单元测试，查看代码能否查询结果！
3. 问题一定不在我们底层了，是spring出现问题
4. 查看spring配置文件，springmvc整合的时候没有调用我们service层的bean:
   1. applicationContext.xml没有注入bean
   2. web.xml查看配置文件，发现问题，配置的是spring-mvc.xml，没有service的bean，所以空指针异常