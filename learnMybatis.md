## 1、环境搭建

### 1.1、搭建数据库

```mysql
CREATE DATABASE `mybatis`;
USE `mybatis`;
CREATE TABLE `user`(
 `id` INT(20) PRIMARY KEY,
 `name` VARCHAR(30) DEFAULT NULL,
 `pwd` VARCHAR(30) DEFAULT NULL
);
INSERT INTO `user` VALUES
(1,"tom","123456"),
(2,"jack","123456"),
(3,"tim","123456");
```

### 1.2、导入依赖

```xml
<dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.25</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.6</version>
</dependency>
```

### 1.3、编写DAO接口

> 还要编写pojo层具体实体类

```java
public interface UserDao {
    List<User> getUserList();
}
```

### 1.4、编写mybatis工具类

```java
public class MybaisUtils {
    private static SqlSessionFactory sqlSessionFactory;
    static {
        try {
            String resource = "mybatis.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    public static SqlSession getSqlSession(){
        return sqlSessionFactory.openSession();
    }
}
```

### 1.5、编写配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimeZone=UTC"/>
                <property name="username" value="root"/>
                <property name="password" value="Tmh010625"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="UserMapper.xml"/>
    </mappers>
</configuration>
```

### 1.6、编写SQL映射xml文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.learn.demon.dao.UserDao">
    <select id="getUserList" resultType="org.learn.demon.pojo.User">
        select * from mybatis.user
    </select>
</mapper>
```

### 1.7、MAVEN额外配置

> 约定大于配置，在resources文件夹外编写xml文件或其他文件时，配置pom.xml

```xml
<build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
        </resources>
</build>
```

## 2、CRUD

### 2.1、namespace

> namespace中的包名要和Dao/mapper接口的包名一致！

### 2.2、select

**语法**：

> 增删改方法中必须提交事务才能执行
>
> ```java
> SqlSession sqlSession = MybaisUtils.getSqlSession();
> sqlSession.commit();
> ```

+ id:就是对应的namespace中的方法名
+ resultType:Sql语句执行的返回值
+ parameterType:参数类型

#### 2.2.1、在接口中写方法

```java
//查询全部用户
List<User> getUserList();
```

#### 2.2.2、在mapper.xml映射文件中编写sql语句

```xml
<select id="getUserList" resultType="org.learn.demon.pojo.User">
        select * from mybatis.user
</select>
```

#### 2.2.3、编写测试方法

```java
@Test
public void test1(){
    SqlSession sqlSession = MybaisUtils.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    List<User> userList = mapper.getUserList();
    for(User user:userList){
        System.out.println(user);
    }
    sqlSession.close();
}
```

### 2.3、insert

#### 2.3.1、在接口中写方法

```java
//insert一个用户
int addUser(User user);
```

#### 2.3.2、在mapper.xml映射文件中编写sql语句

```xml
<insert id="addUser" parameterType="org.learn.demon.pojo.User">
        insert into mybatis.user (id, name, pwd) values (#{id},#{name},#{pwd});
</insert>
```

#### 2.3.3、编写测试方法

```java
@Test
public void test3(){
    SqlSession sqlSession = MybaisUtils.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    mapper.addUser(new User(4, "哈哈", "123123"));
    sqlSession.commit();
    sqlSession.close();
}
```

### 2.4、update

```xml
<update id="updateUser" parameterType="org.learn.demon.pojo.User">
        update mybatis.user set name=#{name},pwd=#{pwd} where id = #{id};
</update>
```

### 2.5、delete

```xml
<delete id="deleteUser" parameterType="int">
        delete from mybatis.user where id = #{id};
</delete>
```

### 2.6、万能map

> Map传递参数，在sql中取出key
>
> 对象传递参数，在sql中取出对象的属性
>
> 只有一个基本类型参数，直接在sql中取出

### 2.7、模糊查询

> 1. java代码执行的时候，传递通配符%%
>
>    ```java
>    List<User> userList = mapper.getUserLike("%李%");
>    ```
>
> 2. 在sql拼接中使用通配符
>
>    ```xml
>    select * from mybatis.user where name like "%"#{value}"%";
>    ```

