 # 第一个Mybatis程序

## 程序结构

![](https://whs1999.oss-cn-shenzhen.aliyuncs.com/20200402181841.png)

 ## 导jar包

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>MyBatis-Study</artifactId>
        <groupId>com.rui</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>mybatis-01</artifactId>

    <!--在build中配置resources，来防止我们资源导出失败的问题-->
    <build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>

</project>
```

## 建造工具类

![](https://whs1999.oss-cn-shenzhen.aliyuncs.com/20200402180722.png)

```java
package com.rui.utils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

//sqlSessionFactory--->SessionFactory
public class MyBatisUtils {
    private static SqlSessionFactory sqlSessionFactory;
    static {
        try{
            //使用mybatis第一步、获取sqlSessionFactory对象
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        }catch(IOException e) {
            e.printStackTrace();
        }
    }
    //既然有了 SqlSessionFactory，顾名思义，我们就可以从中获得 SqlSession 的实例了。
    // SqlSession 完全包含了面向数据库执行 SQL 命令所需的所有方法。
    // 你可以通过 SqlSession 实例来直接执行已映射的 SQL 语句。
    public static SqlSession getSqlSession(){
        return sqlSessionFactory.openSession();
    }
}
```



![](https://whs1999.oss-cn-shenzhen.aliyuncs.com/20200402182425.png)

![](https://whs1999.oss-cn-shenzhen.aliyuncs.com/20200402182915.png)

## 编写配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--configuration核心配置文件-->
<configuration>
    <!--environments配置环境组-->
    <!--default默认环境-->
    <environments default="development">
        <!--environment单个环境-->
        <environment id="development">
            <!--transactionManager配置事务管理器-->
            <transactionManager type="JDBC"/>
            <!--配置连接池-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=utf8"/>
                <property name="username" value="root"/>
                <property name="password" value="Cc105481"/>
            </dataSource>
        </environment>
    </environments>
    <!--每一个Mapper.XML都需要在mybatis核心配置文件中注册-->
    <mappers>
        <mapper resource="com/rui/dao/UserMapper.xml"/>
    </mappers>
</configuration>
```

## 写实体类

```java
package com.rui.pojo;

public class User {
    private int id;
    private String name;
    private String pwd;

    public User() {
    }

    public User(int id, String name, String pwd) {
        this.id = id;
        this.name = name;
        this.pwd = pwd;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPwd() {
        return pwd;
    }

    public void setPwd(String pwd) {
        this.pwd = pwd;
    }

    @Override
    public String toString() {
        return "user{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", pwd='" + pwd + '\'' +
                '}';
    }
}
```

## 写接口

```java
package com.rui.dao;

import com.rui.pojo.User;

import java.util.List;
import java.util.Map;

public interface UserMapper {
    //查询全部用户
    List<User> getUserList();

    //根据id查询用户
    User getUserById(int id);

    //万能Map
    int addUser2(Map<String,Object> map);

    //insert一个用户
    int addUser(User user);

    //修改用户
    int updateUser(User user);

    //删除一个用户
    int deleteUser(int id);
}
```

## 写 Mapper 文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace=绑定一个对应的mapper接口-->
<mapper namespace="com.rui.dao.UserMapper">
    <!--select查询语句-->
    <select id="getUserList" resultType="com.rui.pojo.User">
       /*定义sql*/
       select * from mybatis.user
   </select>

    <select id="getUserById" resultType="com.rui.pojo.User" parameterType="int">
       /*定义sql*/
       select * from mybatis.user where id = #{id};
   </select>

    <!--对象中的属性可以直接取出来-->
    <insert id="addUser" parameterType="com.rui.pojo.User">
        insert into mybatis.user (id, name, pwd) values (#{id},#{name},#{pwd});
    </insert>

    <!--对象中的属性，可以直接取出来 parameterType=传递map中的key-->
    <insert id="addUser2" parameterType="map">
        insert into mybatis.user (id, name, pwd) values (#{userId},#{userName},#{password});
    </insert>

    <update id="updateUser" parameterType="com.rui.pojo.User">
        update mybatis.user set name = #{name},pwd=#{pwd} where id=#{id} ;
    </update>

    <delete id="deleteUser" parameterType="int">
        delete from mybatis.user where id = #{id};
    </delete>
</mapper>
```

## 写test

```java
package com.rui;

import com.rui.dao.UserMapper;
import com.rui.pojo.User;
import com.rui.utils.MyBatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.HashMap;
import java.util.List;

public class UserDaoTest {
    @Test
    public void test(){
        //第一步：获得SqlSession对象
        SqlSession sqlSession = MyBatisUtils.getSqlSession();
        try{
            //方式一：getMapper
            UserMapper mapper = sqlSession.getMapper(UserMapper.class);
            List<User> userList = mapper.getUserList();

            //方式二：不推荐
            //List<User> userList = sqlSession.selectList("com.rui.dao.UserDao.getUserList");

            for (User user : userList) {
                System.out.println(user);
            }

        }
        finally {
            //关闭SqlSession
            sqlSession.close();
        }
    }

    @Test
    public void getUserById(){
        SqlSession sqlSession = MyBatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        User user = mapper.getUserById(1);
        System.out.println(user);
        sqlSession.close();
    }
    //曾删改需要提交事务
    @Test
    public void addUser(){
        SqlSession sqlSession = MyBatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        int res = mapper.addUser(new User(4, "哈哈", "123333"));
        if (res>0){
            System.out.println("插入成功");
        }
        //提交事务
        sqlSession.commit();
        sqlSession.close();
    }

    //万能map
    @Test
    public void addUser2(){
        SqlSession sqlSession = MyBatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        HashMap<String, Object> map = new HashMap<>();
        map.put("userId",4);
        map.put("userName","王五");
        map.put("password","23333");
        mapper.addUser2(map);
        //提交事务
        sqlSession.commit();
        sqlSession.close();
    }

    @Test
    public void updateUser(){
        SqlSession sqlSession = MyBatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        mapper.updateUser(new User(4,"呵呵","123123"));
        sqlSession.commit();
        sqlSession.close();
    }

    @Test
    public void deleteUser(){
        SqlSession sqlSession = MyBatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        mapper.deleteUser(4);
        sqlSession.commit();
        sqlSession.close();
    }
}
```



## 避免5个问题

![](https://whs1999.oss-cn-shenzhen.aliyuncs.com/20200402180115.png)

