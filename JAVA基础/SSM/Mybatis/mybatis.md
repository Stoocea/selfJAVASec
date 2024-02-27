[https://mybatis.net.cn/](https://mybatis.net.cn/)
官方文档中文留档

## 1.什么是 mybatis

1. MyBatis 是一款优秀的持久层框架，
2. 它支持自定义 SQL、存储过程以及高级映射。
3. MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。
4. MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录

这里查看官方文档的时候，遇到过一些名词：POJO---plain Old Java Object（简称普通 java 对象）    DAO---Data Access Objects（数据持久化对象）
mybatis 就是处在 DAO 层，就是这一点说明跟数据库相关的框架

如何获取 mybatis 呢？

- maven 仓库 [https://mvnrepository.com/artifact/org.mybatis/mybatis/3.5.6](https://mvnrepository.com/artifact/org.mybatis/mybatis/3.5.6)
- github  [https://github.com/mybatis/mybatis-3](https://github.com/mybatis/mybatis-3)

这里我选择用 maven 中的依赖导入 mybatis


## 2. 第一个 mybatis 程序
### 0x01 搭建环境
还是首先先把数据库搭起来，这里我用的小p的
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708239683677-01696a32-ce2d-4c72-833a-4ee80746355a.png#averageHue=%23d8d7d7&clientId=uf4508d2b-7d5c-4&from=paste&height=209&id=ud6f369c6&originHeight=313&originWidth=1236&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=95251&status=done&style=none&taskId=u272c29c0-e5b8-4c9f-9205-e4e35713754&title=&width=824)

把 src 目录删掉，将此时的文件夹当作父工程，之后的每一次测试都是当作模块来做，就不用每次都导重复的包了
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708239811055-ed94563c-ad3f-4789-ac20-1d554d272399.png#averageHue=%23363939&clientId=uf4508d2b-7d5c-4&from=paste&height=366&id=u34392b58&originHeight=549&originWidth=2378&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=323711&status=done&style=none&taskId=uca7d2d9f-64d0-4113-bef4-85b8d1a4ffa&title=&width=1585.3333333333333)


然后是在父工程中导入依赖
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <!--父工程-->
  <groupId>com.stoocea</groupId>
  <artifactId>mybatisTest</artifactId>
  <version>1.0-SNAPSHOT</version>
  <name>Archetype - mybatisTest</name>
  <url>http://maven.apache.org</url>



  <dependencies>

    <!--mysql驱动-->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.30</version>
    </dependency>

    <!--mybatis-->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.6</version>
    </dependency>

    <!-- junit -->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.13.1</version>
    </dependency>

  </dependencies>
</project>
```


### 0x02 编写 mybatis 核心配置文件
然后在我们的第一个子模块里的 resources 目录新建一个 xml 配置文件，默认是叫 mybatis-config.xml
然后内容可以从官方文档截取

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708240533306-0a697104-7979-44c5-b0ef-4ea73dd14a9c.png#averageHue=%2336455d&clientId=uf4508d2b-7d5c-4&from=paste&height=558&id=u36e4978e&originHeight=837&originWidth=2404&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=510974&status=done&style=none&taskId=uf03abd57-07da-4ddc-8bad-31ac37629fb&title=&width=1602.6666666666667)

我自己的配置如下，他主要是先要配置号 jdbc 驱动，以及 jdbc 连接的 url，还有数据库用户名和密码
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
                <property name="driver" value="com.mysql.jdbc.cj.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306?mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>

  
</configuration>
```
url 那一栏多了几个参数，主要是为了 SSL 协议连接还有字符编码的一些参数，避免乱码的问题

- environments 标签是环境配置，加 s 表示有多个环境配置，然后里面的 environment 标签就表示一个测试环境，默认 id 是 development

### 0x03 从 XML 中获取 SqlSessionFactory
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708241121928-744869ee-236a-4015-8646-022018949740.png#averageHue=%23f5f4f3&clientId=uf4508d2b-7d5c-4&from=paste&height=279&id=uef9bd987&originHeight=418&originWidth=2137&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=91153&status=done&style=none&taskId=u63d97635-7954-4410-b88f-028ae615745&title=&width=1424.6666666666667)
我们从官方文档中能够读到 mybatis 中执行 SQL 业务的工具类---SqlSessionFactory，也就是说之后关于 SQL 的操作都是通过它来完成
那么如何去加载这个工具类呢？
我们从官方文档处定位到从 xml 文件构建 SqlSessionFactory 的部分，然后自己写一个 Utils 的 package，主要是用来集成一个工具类
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708241651309-dbd28657-b450-4d41-8e89-7f10c1d25fff.png#averageHue=%23393c3d&clientId=uf4508d2b-7d5c-4&from=paste&height=521&id=u820cd971&originHeight=782&originWidth=2372&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=448818&status=done&style=none&taskId=u348b567f-82f4-4306-b12b-2c0fc6b0bc1&title=&width=1581.3333333333333)
```java
package com.stoocea.Utils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.InputStream;
public class MybatisUtils {
    static {
        try {
            
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (Exception e) {
            e.printStackTrace();
        }

    }

}
```
### 0x04 从 SqlSessionFactory 中获取 SqlSession
我们获取 SqlSessionFactory 的目的就是为了通过他来获取 SqlSession，有一句话能够说明： **SqlSession 提供了在数据库执行 SQL 命令所需的所有方法  **
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708246054080-ebb036b5-325b-471d-b96b-11c221fd4187.png#averageHue=%23f5f3f3&clientId=uf4508d2b-7d5c-4&from=paste&height=239&id=u2abca5ec&originHeight=358&originWidth=2164&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=77887&status=done&style=none&taskId=u66591288-a4b6-490a-8200-7615fc50a29&title=&width=1442.6666666666667)
代码只是相较于上部分做了补充，以及sqlSessionFactory 的作用域给提升到了全局，便于之后的返回值提取
```java
package com.stoocea.Utils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.InputStream;

//构建SqlSessionFactory工具类
public class MybatisUtils {
    private static SqlSessionFactory sqlSessionFactory;

    static {
        try {
            //获取SqlSessionFactory对象
            String resource = "mybatis-config.xml";
            //mybatis中专门用来读取mybatis-config.xml文件内容的读取输入流
            InputStream inputStream = Resources.getResourceAsStream(resource);
            //通过builder来生成sqlSessionFactory
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (Exception e) {
            e.printStackTrace();
        }

    }

    //既然获取到了Factory，现在就是把SqlSession返回出去了，这也是我们构建这个工具类的意义
        public static SqlSession getSqlSeesion(){
                return sqlSessionFactory.openSession();
        }
}
```

### 0x05 探究已映射 SQL 语句
现在我们已经获取了 SqlSession 对象，那么 Sql 语句该如何来执行呢？官方文档中提供了两种方法，一种是通过注解，一种是通过 xml 文件，但是两种方式都是通过 SqlSession 去执行的，这里我们只看 xml 文件的实现
也是通过官方文档来获取 xml 文件内容
其实这个 UserMapper.xml 就代表着我们 UserMapper 接口的实现类，我们之前探究 JDBC 的时候，也是一个 UserDao 和 UserDaoImpl，由UserDaoImpl 来实现 UserDao，并在里面写 JDBC 的执行语句，mybatis 就是把UserDaoImpl 这一步替换成了 UserMapper.xml 来实现
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.stoocea.Dao.UserMapper">
        <select id="getUserList" resultType="com.stoocea.pojo.User">
            select * from mybatis_study.users ;
        </select>
</mapper>
```

Mapper（Dao） 接口
```java
package com.stoocea.Dao;
import com.stoocea.pojo.User;
import java.util.List;

public interface UserMapper {
    List<User> getUserList();

}
```

user 类
```java
package com.stoocea.pojo;

public class User {
    private int id;
    private String username;
    private String password;

    public User(int id, String username, String password) {
        this.id = id;
        this.username = username;
        this.password = password;
    }
    public User() {
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getUsername() {
        return username;
    }

    public String getPassword() {
        return password;
    }
}


```
现在回过头来看 Usermapper.xml 文件的内容，类比着看其实就是完成了 UserMapperImpl 类的事情，去实现 Mapper 接口的功能，省去了 JDBC 驱动加载，连接数据库等一系列操作，只要在标签中定义操作标签即可
这一段 select 标签就代表执行 select 操作一次，id 是 Mapper 接口中定义该操作的方法，resultType 就是我们上面定义的 User 类，注意这里的 User 类中的属性必须是跟我们数据库中的数据类型，名称都必须相同
```xml
<select id="getUserList" resultType="com.stoocea.pojo.User">
  select * from mybatis_study.users ;
</select>
```
然后标签内容就是 select 的 SQL 语句
至此，准备事项已经完毕，包括：

1.  UserMapper 以及它的实现配置 UserMapper.xml
2. 用来获取 SqlSession 的工具类
3. 定义代表待会接收数据的对象其属性必须是跟 数据库中数据的类型，名称都必须相同 
4. mybatis 的核心配置文件，用来与数据库建立连接以及设置 JDBC 驱动，每个 Mapper 的注册

接下来开始写测试类，在 test 包中尽量与原项目结构相同，然后写测试类
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708258783839-068a83f2-f586-4154-999f-5f2a0909a8c8.png#averageHue=%23404749&clientId=ua13889c6-8b35-4&from=paste&height=408&id=u50ef5205&originHeight=612&originWidth=657&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=82474&status=done&style=none&taskId=u0ba33923-9ca4-41f5-a613-3d9ba0e6f22&title=&width=438)

```java
package com.stoocea.Dao;

import com.stoocea.Utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;
import com.stoocea.pojo.User;
import java.util.List;

public class UserMapperTest {
    @Test
    public void test(){
        //第一步先获取SqlSession对象
        SqlSession sqlSession= MybatisUtils.getSqlSeesion();

        //通过SqlSession
        UserMapper userMapper=sqlSession.getMapper(UserMapper.class);
        List<User> userList=userMapper.getUserList();

        for (User user : userList) {
            System.out.println(user);
        }
        sqlSession.close();

    }

}

```
逻辑很清晰，就是获取 SqlSession，然后通过 SqlSession 来获取 UserMapper 接口，此时由于已经和配置文件完成绑定，其功能方法也已经实现
那么接下来就是执行 SQL 语句，然后获取结果集，这里的话由于我们定义的接收数据的类是 User 类，所以结果集也是 User 类型，然后打印输出

会报错
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708258632147-3c73ac77-bab9-4d98-85fd-b5223787e1c7.png#averageHue=%23333434&clientId=ua13889c6-8b35-4&from=paste&height=125&id=ue4c8a1dc&originHeight=187&originWidth=1776&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=78220&status=done&style=none&taskId=u46d1743f-c8f3-417f-996f-3f98183fa71&title=&width=1184)

这里必须注意：配置 mybatis 核心配置文件时，我们并没有将 UserMapper 给注册进去，所以这里重新加上注册标签 <mappers>
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
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306?mybatis_study?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>


    <!-- 每一个MapperXML文件都必须在mybatis的核心配置文件中注册-->
    <mappers>
        <mapper resource="com/stoocea/Dao/UserMapper.xml"/>
    </mappers>
</configuration>
```
还是会报错，说找不到 resource
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708259323783-a6d2f53f-ea09-4ea3-a20d-adfbaabb52b3.png#averageHue=%23303845&clientId=u7ec59013-698f-4&from=paste&height=78&id=u1bb79eeb&originHeight=117&originWidth=1170&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=44386&status=done&style=none&taskId=u40cdf5c7-eae3-42e0-84fa-6c24feb7033&title=&width=780)
这里必须在 pom.xml 加上这么一段
```xml
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
```
为什么呢？我们看一下 build 之后的 class 文件结构，在 test-classes 中我们并没有得到 UserMapper.xml 文件，说明它并没有被导出，也更不可能生效
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708259737331-96bf23b4-7914-41e9-91a9-b12d4bf6f09e.png#averageHue=%23535148&clientId=u7ec59013-698f-4&from=paste&height=395&id=uc88fbe78&originHeight=592&originWidth=694&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=89554&status=done&style=none&taskId=u6caed0d6-2cdd-4862-a54e-3a7e8f0d4dc&title=&width=462.6666666666667)
maven 中约定大于配置，就是在与它的默认约定所有的配置文件都只是在 resources 目录下，我们现在写的 UserMapper.xml 文件是在 java.com.stoocea.Dao 目录下，所以无法编译和导出
所以有两种解决办法：

1. 直接把 UserMapper.xml 拖到编译生成的 Target 目录下的 Test 目录的相同目录下，也就时 UserMapperTest 同一层  不利于持久学习，每次测试都必须手动托？
2. 在 maven pom 文件，子项目或者父项目加上上面 build 标签的内容，增加指定的配置文件目录

这里我们选择第二种
然后再次执行测试类

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708260388702-254ee27e-2c69-4cdb-90eb-a77899cf14c1.png#averageHue=%233c3e3f&clientId=u7ec59013-698f-4&from=paste&height=265&id=uea71ba82&originHeight=398&originWidth=1140&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=104876&status=done&style=none&taskId=u2c215112-c9fd-4c27-86ef-c4cee1d461c&title=&width=760)
成功读取
注意事项：如果师傅们看的是狂神的视频，较高版本下配置 mybatis 驱动的 JDBC 驱动是`com.mysql.cj.jdbc.Driver`，也就是下图的配置

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708260460993-91ed33ff-a292-4da7-958d-f54a32416590.png#averageHue=%23353937&clientId=u7ec59013-698f-4&from=paste&height=69&id=u769bfbd6&originHeight=104&originWidth=994&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=39931&status=done&style=none&taskId=uadac9105-063b-4c7a-90de-07f79ef07ee&title=&width=662.6666666666666)

不然会出现报错的情况


## 3.CURD 增删改查实现
回顾一下 JDBC 实现 CURD 的过程，通过 UserDaoImpl 的各种方法来实现 SQL 的操作，那么 mybatis 的话就是通过 UserMapper.xml 文件中的标签来实现了
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708313261489-c0b17031-cc36-4dfa-a621-d67e1b86ed50.png#averageHue=%235f7450&clientId=u3cd1db1c-3794-4&from=paste&height=276&id=u5bc4ebd6&originHeight=414&originWidth=1134&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=150695&status=done&style=none&taskId=u311b978d-97bb-40a0-a8a8-0d05b604aeb&title=&width=756)
重点分析一下 xml 文件的内容
头文件先不管，我们看 maaper 标签，他有一个 namespace 属性，中文翻译命名空间，之前学 Thinkphp 反序列化链的时候应该还有印象，但是在这里的就只是绑定我们当前 UserMapper.xml 文件要实现的 Mapper 接口，要注意：必须要把包名写全，不然无法定位到

### 0x01 select
首先先看 select 标签，它肯定就代表查询功能，然后再看几个比较重要的参数
```xml
<select id="getUserList" resultType="com.stoocea.pojo.User" parameterType="int">
        select * from users ;
</select>


id：是我们想要调取的Mapper接口的方法名

resultType：是我们SQL查询完毕之后的返回类型，这个基本上不变，就是接收结果类的全类名

parameterType：代表我们接收参数的类型，通常用于where条件时接收
```
现在来实现一个需求，用户输入一个序号，我们返回一个指定序号的 User 信息
那么首先肯定是在 Mapper 接口那定义一个新方法
```java
package com.stoocea.Dao;
import com.stoocea.pojo.User;
import java.util.List;

public interface UserMapper {
    List<User> getUserList();
    
    User getUserbyId(int id);
}
```

然后将对应的 Mapper.xml 文件增加对其的实现 select 语句，一个方法对应一个 SQL 语句，这里也有一些新的知识点，**#{id}**其实就代表参数的占位，里面的 id 字符串可以随意更换，只是代表一个替换
```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.stoocea.Dao.UserMapper">
        <select id="getUserList" resultType="com.stoocea.pojo.User">
            select * from users ;
        </select>

    <select id="getUserbyId" resultType="com.stoocea.pojo.User" parameterType="int">
        select * from Users where id= #{id};
    </select>

</mapper>
```

### 0x02 insert
首先现在 Mapper 接口添加上待会要用的方法，返回类型是 int，插入操作只会返回是否成功的 int 值
```java
package com.stoocea.Dao;
import com.stoocea.pojo.User;
import java.util.List;

public interface UserMapper {
    List<User> getUserList();
    
    User getUserbyId(int id);

    int addUser(User user);

}
```

然后是用 Mapper.xml 去实现 addUser
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.stoocea.Dao.UserMapper">
  <select id="getUserList" resultType="com.stoocea.pojo.User">
    select * from users ;
  </select>

  <select id="getUserbyId" resultType="com.stoocea.pojo.User" parameterType="int">
    select * from Users where id= #{id} ;
  </select>

  <insert id="addUser" parameterType="com.stoocea.pojo.User" >
    insert into mybatis_study.users (id,username,password) values (#{id},#{username},#{password});
  </insert>

</mapper>
```
这里涉及到两个问题，一是我们插入语句肯定是需要三个参数的，但是标签里面每次只能指定一个参数
解决办法是通过传入参数 User 这个类的实体，然后它里面的属性值都是可以通过#{}取出的，只不过必须跟 User 类里面的属性值名称相同，不然无法定位

然后就是继续写测试方法
```java
    public void testaddUser(){
        SqlSession sqlSession=MybatisUtils.getSqlSeesion();
        UserMapper userMapper=sqlSession.getMapper(UserMapper.class);
        userMapper.addUser(new User(3,"stoocea3","1234565"));
        sqlSession.close();
    }
```
测试成功

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708397437767-eb810bac-5f2c-4387-9ffa-19af90704d59.png#averageHue=%23808479&clientId=u80a57b98-a886-4&from=paste&height=192&id=u1dfefd26&originHeight=288&originWidth=785&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=92976&status=done&style=none&taskId=u7daf6696-7e7a-4732-a909-d623b3c5ddc&title=&width=523.3333333333334)
这里还有一种特殊情况，如果版本较低的话，可能会出现提前不成功的情况。因为所有的增删改都属于事务，事务就必须满足”要么一起成功，要么一起失败的原则“，它必须要有提交操作
```java
    public void testaddUser(){
        SqlSession sqlSession=MybatisUtils.getSqlSeesion();
        UserMapper userMapper=sqlSession.getMapper(UserMapper.class);
        userMapper.addUser(new User(3,"stoocea3","1234565"));
        sqlSession.commit();
        sqlSession.close();
    }
```
也就是 sqlSession.commit()，这里是 JDBC 的知识，就不多细讲（不知道为啥我这里成功了）

### 0x03 update
继续在接口处添加 update 的操作
```java
package com.stoocea.Dao;
import com.stoocea.pojo.User;
import java.util.List;

public interface UserMapper {
    List<User> getUserList();
    User getUserbyId(int id);

    int addUser(User user);

    int updateUser(User user);
}

```
添加完毕之后，继续在 mapper.xml 文件里面实现 update
```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.stoocea.Dao.UserMapper">
        <select id="getUserList" resultType="com.stoocea.pojo.User">
            select * from users ;
        </select>

    <select id="getUserbyId" resultType="com.stoocea.pojo.User" parameterType="int">
        select * from Users where id= #{id} ;
    </select>

    <insert id="addUser" parameterType="com.stoocea.pojo.User" >
            insert into mybatis_study.users (id,username,password) values (#{id},#{username},#{password});
    </insert>

    <update id="updateUser" parameterType="com.stoocea.pojo.User">
        update mybatis_study.users set username=#{username} ,password=#{password} where id=#{id}
    </update>
</mapper>
```

然后是写个测试类就行
```java
    @Test
    public void testupdateUser(){
        SqlSession sqlSession=MybatisUtils.getSqlSeesion();
        UserMapper userMapper=sqlSession.getMapper(UserMapper.class);
        userMapper.updateUser(new User(1,"newb","1234567"));
        sqlSession.commit();
        sqlSession.close();

    }
```

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708398060824-7b719ab9-64fb-4006-8a51-1424d42aff1d.png#averageHue=%23868a7b&clientId=u80a57b98-a886-4&from=paste&height=175&id=uc97819b5&originHeight=262&originWidth=695&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=73827&status=done&style=none&taskId=u53fdfe16-4e88-49ae-940f-36a884a810c&title=&width=463.3333333333333)

### 0x04 delete
删除操作就不多说了，增删改操作都是一样的，相当于事务操作，只不过这里的传入参数可以为 int，因为 id 是唯一的
```java
package com.stoocea.Dao;
import com.stoocea.pojo.User;
import java.util.List;

public interface UserMapper {
    List<User> getUserList();
    User getUserbyId(int id);

    int addUser(User user);

    int updateUser(User user);

    int deleteUser(int id);

}
```

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.stoocea.Dao.UserMapper">
        <select id="getUserList" resultType="com.stoocea.pojo.User">
            select * from users ;
        </select>

    <select id="getUserbyId" resultType="com.stoocea.pojo.User" parameterType="int">
        select * from Users where id= #{id} ;
    </select>

    <insert id="addUser" parameterType="com.stoocea.pojo.User" >
            insert into mybatis_study.users (id,username,password) values (#{id},#{username},#{password});
    </insert>

    <update id="updateUser" parameterType="com.stoocea.pojo.User">
        update mybatis_study.users set username=#{username} ,password=#{password} where id=#{id}
    </update>

    <delete id="deleteUser" parameterType="int">
        delete from mybatis_study.users where id=#{id}
    </delete>
</mapper>
```

```java
    @Test
    public void testdeleteUser(){
        SqlSession sqlSession=MybatisUtils.getSqlSeesion();
        UserMapper userMapper=sqlSession.getMapper(UserMapper.class);
        userMapper.deleteUser(3);
        sqlSession.commit();
        sqlSession.close();

    }
```

这里我执行了两次，一次是 id=1 的删除，一次是 id=2 的删除（因为第一次没刷新，我以为出错了）
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708398378604-4ae27862-00ff-4dc7-8297-67334e57efe0.png#averageHue=%23606e61&clientId=u80a57b98-a886-4&from=paste&height=193&id=u773b668d&originHeight=290&originWidth=779&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=95554&status=done&style=none&taskId=uabb22694-a98e-415a-b310-aef13970e53&title=&width=519.3333333333334)


## 4.map 和模糊查询扩展
### 0x01 map 传参
回顾一下我们上面写的 mapperxml 文件，执行 SQL 语句的时候涉及一个参数问题，比如 insert 的时候，如果参数不止我们提到的 3 个，id username password 的话，而是数量更多的参数，我们不可能一个一个填上去，这个时候就需要 map 来优化操作

首先在接口处定义一个新的 map 参数的插入方法
```java
package com.stoocea.Dao;
import com.stoocea.pojo.User;
import java.util.List;
import java.util.Map;

public interface UserMapper {

    ....
    User addUser2(Map<String,Object> map);
    ....
}
```
注意此时返回值依然是 User，但是参数值变成了 Map，键值对形式为 String 和 Object（任意形式的数据）
然后就是 xml 文件对其实现
```xml
    <insert id="addUser2" parameterType="map">
            insert into mybatis_study.users (id,password) values(#{diffid},#{diffpassword})
    </insert>
```
有几处不同，一是 parameterType 改为 map，然后后面的待定系数也变成了随意写的参数，不需要和我们上面定义 parameterType 为 User 类时一样强制全部写出了，这里我只设置了 id 和 password 的传入

然后写一个测试方法
```java
    @Test
    public void testaddUser2(){
        SqlSession sqlSession=MybatisUtils.getSqlSeesion();
        UserMapper userMapper=sqlSession.getMapper(UserMapper.class);
        Map<String,Object> hashMap=new HashMap();

        hashMap.put("diffid",5);
        hashMap.put("diffpassword","123456");

        userMapper.MapaddUser(hashMap);
    }
```
在设置了默认值的情况下，会有如下结果
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708417796363-82634cca-a064-4363-a99f-209fc7b32ae6.png#averageHue=%23868879&clientId=u46f0cd0c-4151-4&from=paste&height=178&id=ud8b14ef4&originHeight=267&originWidth=681&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=74215&status=done&style=none&taskId=udb27843a-79f5-4f21-8443-fc85d36ced4&title=&width=454)
这就是 map 的好处，解决了我上面学的时候的一个疑问，要是我查询语句的时候，and 或者 or 字段要传入两个参数怎么办？这里 map 就很好的解决了，那么看如下 demo ：
```java
User getUserbyIdAndUsername(Map<String,Object> map);
```

```java
    <select id="getUserbyIdAndUsername" parameterType="map" resultType="com.stoocea.pojo.User">
        select * from mybatis_study.users where id=#{diffid} and username=#{diffusername}
    </select>
```

```java
    @Test
    public void getUserbyIdAndUsername(){
        SqlSession sqlSession=MybatisUtils.getSqlSeesion();
        UserMapper userMapper=sqlSession.getMapper(UserMapper.class);
        Map<String,Object> hashMap=new HashMap();

        hashMap.put("diffid",3);
        hashMap.put("diffusername","Archer");

        System.out.println(userMapper.getUserbyIdAndUsername(hashMap));
        sqlSession.close();
    }

```
查询结果无误
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708418470474-c5b24c64-b5b2-4d7a-96f4-886373c3b0a0.png#averageHue=%237d7e71&clientId=u46f0cd0c-4151-4&from=paste&height=129&id=u4cf9c206&originHeight=194&originWidth=775&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=63019&status=done&style=none&taskId=u23d75148-65c7-4d9a-9c18-3a756cbceae&title=&width=516.6666666666666)
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708418463685-a5c0758a-a802-4aa0-a6f4-375e030d2484.png#averageHue=%2357845e&clientId=u46f0cd0c-4151-4&from=paste&height=130&id=u87a9559c&originHeight=195&originWidth=841&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=47646&status=done&style=none&taskId=u104cfbfe-341e-4c17-b91f-895ee5dc53b&title=&width=560.6666666666666)

也就是说我们用 map 传参的话，可以自定参数的数量，也就是任意控制传值，适用于多个参数的传参

### 0x02 模糊查询扩展
这里再学一点的关于模糊查询，java 中的模糊查询要用到通配符 % ，并且通配符的使用能够有效避免 SQL 注入

```java
    List<User> getUserLike(String object);
```

```java
    <select id="getUserLike" resultType="com.stoocea.pojo.User">
        select * from mybatis_study.users where password like "%"#{value}"%"
    </select>
```

```java
    @Test
    public void testlikeSelect(){
        SqlSession sqlSession=MybatisUtils.getSqlSeesion();
        UserMapper userMapper=sqlSession.getMapper(UserMapper.class);
        List<User> userlike=userMapper.getUserLike("1234");
        for (User user : userlike) {
            System.out.println(user);
        }

    }
```


## 5.配置和属性优化
如果单纯只是 mybatis 的增删改查上面的内容就已经够了，具体的精进和理解 mybatis 还是的下面的内容
其实上面我们已经见过两种了，一种是 mybatis 的核心配置文件，还有一种是 mybatis 关于 CURD 操作的 xml 文件
所有的关于 myabatis 配置内容，在官网中能够获取
 configuration（配置） 

- [properties（属性）](https://mybatis.net.cn/configuration.html#properties)
- [settings（设置）](https://mybatis.net.cn/configuration.html#settings)
- [typeAliases（类型别名）](https://mybatis.net.cn/configuration.html#typeAliases)
- [typeHandlers（类型处理器）](https://mybatis.net.cn/configuration.html#typeHandlers)
- [objectFactory（对象工厂）](https://mybatis.net.cn/configuration.html#objectFactory)
- [plugins（插件）](https://mybatis.net.cn/configuration.html#plugins)
- [environments（环境配置）](https://mybatis.net.cn/configuration.html#environments)
   -  environment（环境变量） 
      - transactionManager（事务管理器）
      - dataSource（数据源）
- [databaseIdProvider（数据库厂商标识）](https://mybatis.net.cn/configuration.html#databaseIdProvider)
- [mappers（映射器）](https://mybatis.net.cn/configuration.html#mappers)

上面我们已经了解过了 environments 以及 mappers 的内容

### 0x01  环境配置（environments）
现在来剖析 enviroments 的内容
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708866836094-bba1483b-ac33-43a2-a831-a72d74e095e2.png#averageHue=%232f343c&clientId=u8726b86f-7b61-4&from=paste&height=253&id=u4f31c1ae&originHeight=379&originWidth=1702&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=183629&status=done&style=none&taskId=u21e7550c-a67b-4812-8d79-e190a95a0c0&title=&width=1134.6666666666667)
纯看我们之前用的这一项，enviroments ，复数 s 其实就代表它下面的内容都是关于 enviroment 的内容，具体适用哪一套，default 来指定
比如我们下面的 enviroment 标签，它的 id 是 development，其实也就是当前项目使用的环境配置
这里注意两个标签  ---`<transactionManager type="JDBC"/>`  `<dataSource type="POOLED">`

#### 1x01 事务管理器（transactionManager）
 在 MyBatis 中有两种类型的事务管理器（也就是 type="[JDBC|MANAGED]"，指定 JDBC 就代表它当前采用的是 JDBC 的事务管理器，直接使用 jdbc 的回滚和提交 
MANAGED，个人认为没啥好提的，它默认不做提交或者回滚
如果此时用的是 Spring +mybatis 的组合，则不用去配置事务管理器

#### 1x02 数据源（dataSource）
 dataSource 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象的资源。  
这里主要是注意 dataSource 标签它里面的`type="POOLED"` 属性
他有三个值 UNPOOLED POOLED JNDI 
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708869143489-13416ddc-476d-42c2-9820-44e11d7428fc.png#averageHue=%23f8f6f4&clientId=u8726b86f-7b61-4&from=paste&height=505&id=u2c6c244e&originHeight=758&originWidth=2157&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=189084&status=done&style=none&taskId=u970d33c9-a126-4467-bc76-5dc1f46a74c&title=&width=1438)

分别代表不建立连接池，建立连接池，JNDI 主要为了能够在特殊容器中起作用，而建立连接池主要是为了省时间和资源，你第一次连接之后如果下线了，第二次连接就可以直接连


### 0x02 属性（properties）
这里其实就是学一下 db.properties 的作用
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708870516283-24a74769-484e-4f6e-a5e8-e83061fa4f12.png#averageHue=%23323741&clientId=u8726b86f-7b61-4&from=paste&height=115&id=uc73d7537&originHeight=173&originWidth=1592&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=89664&status=done&style=none&taskId=u57dc7fd9-d5b0-4417-88d4-5851666ef17&title=&width=1061.3333333333333)
这一段内容是关于我们选取驱动和连接数据库时的用户名和密码等配置，有时候并不会这么直白的写在 mybatis-config.xml 文件中，而是通过引入db.properties 文件，然后通过 ${}去赋值 value
演示如下：
db.properties
```properties
driver=com.mysql.cj.jdbc.Driver
username=root
password=123456
url=jdbc:mysql://localhost:3306/mybatis_study?useSSL=true&useUnicode=true&characterEncoding=UTF-8
```

mybatis-config.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="db.properties"/>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                 <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    
    <!-- 每一个MapperXML文件都必须在mybatis的核心配置文件中注册-->
    <mappers>
        <mapper resource="com/stoocea/Dao/UserMapper.xml"/>
    </mappers>
</configuration>
```

这里标签顺序也有讲究，由于` <properties resource="db.properties"/>`相当于引入db.properties 文件的作用，我们必须在 configuration 内容的最前面写上，不然会报错
然后 property 标签中的 value 属性值就可以通过${}取值的形式从 db.propertis 文件中取出来
之后再随便启动一个测试类看看能不能成功查询即可

还有需要注意一点，我们还存在如下形式
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="db.properties">
        <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis_study?useSSL=true&useUnicode=true&characterEncoding=UTF-8"/>
    </properties>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                 <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 每一个MapperXML文件都必须在mybatis的核心配置文件中注册-->
    <mappers>
        <mapper resource="com/stoocea/Dao/UserMapper.xml"/>
    </mappers>
</configuration>
```
也就是在 xml 文件中单独写 property ，然后在需要用到的时候就还是${}获取即可，只不过这里有一个优先级的问题，如果我们是 properties 文件和 xml 文件中的<property>同时存在，并且同名的话，优先采取 properties 文件的内容


### 0x03 类型别名（typeAliases）
别名的作用是什么呢？
 类型别名可为 Java 类型设置一个缩写名字。 它仅用于 XML 配置，意在降低冗余的全限定类名书写。
其实就是可以缩短我们写全类名的时间，用别名来代替全类名
接下来看例子，我们在 mybatis 的配置文件中有 `<typeAliases>`标签，他是用来定义别名的总标签，我们起别名必须用到`<typeAlias>`标签或者 package 标签
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708908930505-5bc92d2f-5c34-49dc-a914-c6df615424e5.png#averageHue=%232b2f38&clientId=ua598202f-b77a-4&from=paste&height=147&id=u548a2e78&originHeight=221&originWidth=1009&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=48433&status=done&style=none&taskId=u8b988162-8ace-4e83-8155-773d0d663cd&title=&width=672.6666666666666)

```xml
    <typeAliases>
        <typeAlias type="com.stoocea.pojo.User" alias="User"/>
    </typeAliases>
```
如果采用`<typeAlias>`来书写别名，那起什么别名都可以，都能被识别得到
我们这里拿查询所有用户为例，resultType 直接用别名
```xml
        <select id="getUserList" resultType="User">
            select * from users ;
        </select>
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708909497417-7ce6d93c-3cd4-48ee-8c7d-36b16d0739b2.png#averageHue=%2330353d&clientId=ua598202f-b77a-4&from=paste&height=141&id=ud993014a&originHeight=211&originWidth=771&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=55554&status=done&style=none&taskId=u3a282f99-fe9a-44ae-ac44-37105523a71&title=&width=514)
查询依旧成功
下面介绍 package 标签，他其实就是将包下的 javabean 扫描，然后自动给他们起上别名为他们类名的首字母小写形式（大写也可以）

```xml
    <typeAliases>
        <package name="com.stoocea.pojo"/>
    </typeAliases>
```
```xml
        <select id="getUserList" resultType="user">
            select * from users ;
        </select>
```
依旧是拿查询所有数据为例
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708909735008-f27cb4c5-168f-4b3a-88bc-e5f7489a8e59.png#averageHue=%232e343c&clientId=ua598202f-b77a-4&from=paste&height=609&id=u0e63412f&originHeight=914&originWidth=1679&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=433769&status=done&style=none&taskId=u94fde085-26c7-4803-bc13-307da6f836d&title=&width=1119.3333333333333)
是可以查询成功的
第一种用于实体类比较少的情况，第二种是实体类比较多的情况
其实也可以在实体类上写一个 `@Alias` 注解，也能够被识别到

### 0x04 设置（settings）
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708910139154-ec806ec9-552a-46f4-8224-8827eccae694.png#averageHue=%23fefefe&clientId=ua598202f-b77a-4&from=paste&height=164&id=ude802bf0&originHeight=246&originWidth=1761&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=17860&status=done&style=none&taskId=u1bfd6e51-bf2d-468f-8808-dbd12c6da88&title=&width=1174)
主要就是记住上面这一段的设置，用来指定日志实现
设置这一块到 Spring 之后就基本不再用了
  

### 0x05 映射器（mappers）
上面的学习排错中已经学习到了，我们每写一个 Dao 层都必须写一个相对应的 mapper 文件用来写 SQL CRUD 的语句和实现 Mapper 接口，但是必须要在 mybatis-config.xml 文件中实现注册
```xml
    <mappers>
        <mapper resource="com/stoocea/Dao/UserMapper.xml"/>
    </mappers>
```

这其实是 mapper 注册的一种方法，我们还有三种方法
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708913196988-0c23b397-0cf2-4a5e-88bf-6574c6e0f897.png#averageHue=%23f3f2f2&clientId=uaa0f3b52-557f-4&from=paste&height=540&id=ub522f926&originHeight=810&originWidth=1488&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=123714&status=done&style=none&taskId=u706cde1f-044d-4430-bdf3-81b731c5c28&title=&width=992)
第二种使用 file 伪协议读取文件的方式完全不推荐
第三种使用映射器接口实现类来定位的话，必须满足如下两个条件：

1. mapperxml 文件必须和 mapper 接口在同一包下
2. mapperxml 文件必须和 mapper 接口同名

第四种直接扫描包内的 mapper.xml，然后自动注册，但是也必须满足第三种方式的条件


### 0x06 生命周期和作用域
直接看官方文档


## 6.结果集映射 resultMap
之前还没学 map 传参类型的时候，去探索过如何解决多参数问题，其中就了解到了 resultmap，但是它并不是来处理传参的，而是处理返回结果
假设如下情况：我们 pojo 实体类如下

```java
public class User {
    private int id;
    private String username;
    private String pwd;
```
然后我们数据库的字段名如下：
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708914675207-43a0c411-4440-49b1-9e79-3992b474d21d.png#averageHue=%23343b45&clientId=uaa0f3b52-557f-4&from=paste&height=95&id=u9dac5a70&originHeight=142&originWidth=611&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=21061&status=done&style=none&taskId=u9ddb27f7-06e4-4b7e-b148-9ddfe9c9f89&title=&width=407.3333333333333)
password 字段名和 pwd 属性值名不同
然后我们执行一个查询所有字段数据的操作
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708915188212-c56e6d92-fbae-40bb-87af-60c766a8d8d6.png#averageHue=%23333942&clientId=uaa0f3b52-557f-4&from=paste&height=100&id=ueebb8a5b&originHeight=150&originWidth=671&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=37456&status=done&style=none&taskId=ub73fbd2d-65d7-4ce0-b389-5f8dbe6155f&title=&width=447.3333333333333)

password 查询为空，这是因为 mybatis 执行完 SQL 查询之后，结果集映射到类中时，只会将字段数据映射到同名属性值上，假设我们 username 属性值改为 name，它结果也是为空，因为它找不到和数据库中的字段 username 同名的属性值，无法映射上去
这里最简单的操作就是在 SQL 中 执行别名操作 `password as pwd`操作，或者直接修改类名或者数据库字段名为同名
resultMap 也能解决这个问题
```xml
<resultMap id="UserMap" type="User">
  <result column="id" property="id"/>
  <result column="username" property="username"/>
  <result column="password" property="pwd"/>
</resultMap>
```

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708916362136-65752f43-96a7-4e9a-a557-ef9cbd03592a.png#averageHue=%232f343d&clientId=uaa0f3b52-557f-4&from=paste&height=221&id=u9b0aa7fc&originHeight=331&originWidth=851&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=95996&status=done&style=none&taskId=u738e7003-18d4-4de5-ab65-befb213689c&title=&width=567.3333333333334)
这里必须和查询标签一起看，当我们指定某一查询语句的 resultMap，并且在 resultMap 标签中也设置了其对应 id 代表时，现在的查询语句返回时的结果集就能够通过我们新增的 resultMap 的内容，把结果一一映射到属性值中，并不会只找同名段了

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708916539691-b82a78df-9a91-442c-90f9-d1e3afbfbeb7.png#averageHue=%2330353d&clientId=uaa0f3b52-557f-4&from=paste&height=143&id=udaf37060&originHeight=215&originWidth=820&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=58415&status=done&style=none&taskId=u86216862-6182-4c1b-addf-0c1778983f7&title=&width=546.6666666666666)
之后还会一对多，多对一的结果集映射情况，这里就学到了再说

## 7.日志工厂
### 0x01 默认日志的实现
日志主要用来解决一个什么问题呢？当我们的 SQL 查询出错，或者是其他错误的时候，报错有的能够帮助我们很快解决问题，有的却没啥用，我需要更加具体的信息来确认我们到底干了哪些事情---日志就很好的解决了这个问题
官方文档中限定了如下日志使用
 SLF4J | LOG4J | LOG4J2 | JDK_LOGGING | COMMONS_LOGGING | STDOUT_LOGGING | NO_LOGGING  
这里肯定重点就是 LOG4J 了
STDOUT_LOGGING 是默认的日志输出
具体的实现是在 mybatis 的核心配置文件中设置
```xml
    <settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
```
name 和 value 的值大小写严格，建议照着官网的复制即可
设置完成之后我们尝试运营一个查询所有字段数据的 test
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708923217003-2a7af7a0-1faf-46fc-ab1a-63cc0efaed10.png#averageHue=%2330353f&clientId=u41627b3a-e7bd-4&from=paste&height=452&id=u6293c631&originHeight=678&originWidth=1650&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=425358&status=done&style=none&taskId=ueb318143-70dd-4369-946c-928cafad45f&title=&width=1100)
其实更加重要的是如下内容：
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708923344440-3fd8258e-da6a-4186-881e-2e2ff0ea2488.png#averageHue=%232f343d&clientId=u41627b3a-e7bd-4&from=paste&height=394&id=u6728b325&originHeight=591&originWidth=1402&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=289112&status=done&style=none&taskId=ua2fdda7f-ca50-43c5-88f4-e922aea3b95&title=&width=934.6666666666666)
它这里记录了我们运行这个 test 程序的所有步骤，从开始到结束

### 0x02 log4J 日志实现
使用 log4j 之前得导包，不然找不到其相关的内容
然而 log4j 由于漏洞过多，这里就不强行去跟 log4j 的内容，我们采用 log4j2 的内容
```xml
<dependency>
  <groupId>org.apache.logging.log4j</groupId>
  <artifactId>log4j-slf4j-impl</artifactId>
  <version>2.19.0</version>
</dependency>
<dependency>
  <groupId>org.apache.logging.log4j</groupId>
  <artifactId>log4j-core</artifactId>
  <version>2.19.0</version>
</dependency>
```

导完包之后还得写 log4j2 的 xml 配置文件
```properties
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="TRACE">

    <!-- 配置日志信息输出目的地  Appenders-->
    <Appenders>
        <!-- 输出到控制台 -->
        <Console name="Console" target="SYSTEM_OUT">
            <!--配置日志信息的格式 -->
            <PatternLayout
                    pattern="%d{HH:mm:ss} [%t] %-5level %logger{36} - %msg%n" />
            "/>
        </Console>

        <!-- 输出到文件，其中有一个append属性，默认为true，即不清空该文件原来的信息，采用添加的方式，若设为false，则会先清空原来的信息，再添加 -->
        <File name="MyFile" fileName="./logs/info.log" append="true">
            <PatternLayout>
                <!--配置日志信息的格式 -->
                <pattern>%d{HH:mm:ss} [%t] %-5level %logger{36} - %msg%n</pattern>
            </PatternLayout>
        </File>

    </Appenders>


    <!-- 定义logger,只有定义了logger并引入了appender,appender才会有效 -->
    <Loggers>
        <!-- 将业务dao接口所在的包填写进去,并用在控制台和文件中输出 -->
        <logger name="com.stoocea.dao.UserMapper" level="TRACE"
                additivity="false">
            <AppenderRef ref="Console" />
            <AppenderRef ref="MyFile" />
        </logger>

        <Root level="info">
            <AppenderRef ref="Console" />
            <AppenderRef ref="MyFile" />
        </Root>
    </Loggers>
</Configuration>
```
由于其内容限制，这里我拿 boogipop 师傅的 xml 文件内容稍加修改适配我自己的项目结构
然后我们可以在任意一个测试类中使用 log4j2 的功能
```java
package com.stoocea.Dao;

import com.stoocea.Utils.MybatisUtils;
import com.stoocea.pojo.User;
import org.apache.ibatis.session.SqlSession;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.junit.Test;

import java.util.List;

public class UserMapperTest {
    static Logger logger= LogManager.getLogger();
    @Test
    public void testResultMap(){
        SqlSession sqlSession= MybatisUtils.getSqlSeesion();
        UserMapper userMapper=sqlSession.getMapper(UserMapper.class);
        List<User> userList=userMapper.getUserList();
        logger.info("=====info信息开始=====");
        for (User user : userList) {
            System.out.println(user);
        }
        sqlSession.close();
    }
}

```
大致输出如下：
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708927101435-c8c7b848-48f5-400b-946b-d992b95df46d.png#averageHue=%23313740&clientId=u41627b3a-e7bd-4&from=paste&height=430&id=ue6ee8a34&originHeight=645&originWidth=1869&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=550913&status=done&style=none&taskId=u38996456-9626-4371-ab95-f3b357e9d9f&title=&width=1246)

## 8.分页
### 0x01 用 limit 实现分页
分页其实就是限制 SQL 查询回显结果，减少数据处理量，然后这里我们不从 SQL 的层面进行这个操作，我们通过 mybatis 程序实现
```java
    @Test
    public void testSelectbylimit(){
        SqlSession sqlSession=MybatisUtils.getSqlSeesion();
        UserMapper userMapper=sqlSession.getMapper(UserMapper.class);
        HashMap hashMap=new HashMap<String,Object>();
        hashMap.put("startIndex",0);
        hashMap.put("pageSize",4);
        List<User> userList=userMapper.getUserbylimit(hashMap);
        for (User user : userList) {
            System.out.println(user);
        }

        sqlSession.close();
    }
```

```xml
<select id="getUserbylimit" parameterType="map"   resultMap="UserMap">
  select * from mybatis_study.users limit #{startIndex},#{pageSize};
</select>
```
本质还是在写 SQL，并没有说通过新的一个处理对象来进行操作

### 0x02 用 RowBounds 对象来实现分页
```xml
    <select id="getUserbyRowBounds" resultMap="UserMap">
        select * from users;
    </select>
```

```java
@Test
public void testselectbyrowbounds(){
    SqlSession sqlSession=MybatisUtils.getSqlSeesion();
    RowBounds rowBounds=new RowBounds(1,2);
    List<User> userlist=sqlSession.selectList("com.stoocea.Dao.UserMapper.getUserbyRowBounds",null,rowBounds);

    for (User user : userlist) {
        System.out.println(user);
    }
    sqlSession.close();
}

```
查询结果如下
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708932150658-ebfc7793-3160-4b95-8b8c-2ec79aeb6ccd.png#averageHue=%232f343d&clientId=u41627b3a-e7bd-4&from=paste&height=253&id=u445e4dfb&originHeight=379&originWidth=1342&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=153631&status=done&style=none&taskId=u48b741b0-19fd-4a92-b5f7-70f7a9e4281&title=&width=894.6666666666666)

## 9.注解开发
在了解注解开发之前，我们要首先理解面向接口编程
其实面向接口编程这个概念是对于整体公司的一个项目来定的，架构师负责接口的构建，把需求都写好，然后其他人去实现这些功能，根本的目的是为了：解耦，提高复用性，保持扩展等
其他的概念就不提了，直接进入注解开发
mybatis 的注解开发主要体现在 Mapper 接口中的实现
```java
    @Select("select * from users")
    List<User> getUserNew();
```
这个时候就没有必要去写 Mapper.xml 文件了，mybatis 的注解开发作用就是为了简化 xml 文件去实现方法
然后直接测试类

```java
    public void testselectbyNew(){
        SqlSession sqlSession=MybatisUtils.getSqlSeesion();
        UserMapper userMapper=sqlSession.getMapper(UserMapper.class);
        List<User> userlist=userMapper.getUserNew();
        for (User user : userlist) {
            System.out.println(user);
        }


        sqlSession.close();
    }
```
发现确实是打印了，但是我们的 password 字段怎么办？我们接收结果的 User 类中只有 pwd 字段，而正常查询的结果只会找相同的字符串属性值去映射进去，这里就必须要用到 xml 文件，并且采用结果映射集去解决
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708940096847-83fc57b4-923b-4b55-8b12-95dd639c35d2.png#averageHue=%232f343d&clientId=u41627b3a-e7bd-4&from=paste&height=365&id=u058f5afd&originHeight=547&originWidth=1320&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=224934&status=done&style=none&taskId=uefb41268-748e-46b6-9a80-9e6cebc104f&title=&width=880)

所以官方文档也说了，注解开发适用于简单的 sql 查询，不适用于复杂的查询环境，各取所需即可

## 10.mybatis 执行流程剖析
其实学到现在自己心里也有个底了
![](https://cdn.nlark.com/yuque/0/2024/jpeg/36078896/1708948808517-50a5f334-9aa6-4a4e-84b9-7c974247269c.jpeg)

SqlSessionFactoryBuilder->inputStream->得到xml配置->sqlSessionFactory->transaction事务管理->excutor执行器->得到Sqlsession  


## 11.Lombok 简化
啥是 lombok 呢？它是一个插件，然后可以帮助我们自动补齐 getter  setter 等重复性代码操作，注意这里的补齐是通过写注解实现的，并不是自动在类中代码补齐

先导入 maven 依赖
```xml
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <version>1.18.24</version>
</dependency>
```

然后我们随便拿一个实体类来测试一下
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708953252650-39925db8-8c4f-46ba-b164-73396ce6554d.png#averageHue=%232d323a&clientId=u701ce8ce-daa4-4&from=paste&height=605&id=ue311e627&originHeight=907&originWidth=1846&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=446868&status=done&style=none&taskId=ued198819-c5c3-4c43-a214-95d4a3b0b4b&title=&width=1230.6666666666667)


## 12.复杂查询环境搭建
这里我直接用了 boogipop 师傅的 SQL 文件
```plsql
CREATE TABLE `teacher` (
  `id` INT(10) NOT NULL,
  `name` VARCHAR(30) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8

INSERT INTO teacher(`id`, `name`) VALUES (1, 'Boogipop'); 

CREATE TABLE `student` (
  `id` INT(10) NOT NULL,
  `name` VARCHAR(30) DEFAULT NULL,
  `tid` INT(10) DEFAULT NULL,
  PRIMARY KEY (`id`),
  CONSTRAINT `fktid` FOREIGN KEY (`tid`) REFERENCES `teacher` (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8
INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('1', 'Kino', '1'); 
INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('2', 'Jack', '1'); 
INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('3', 'Benk', '1'); 
INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('4', 'Siri', '1'); 
INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('5', 'Benk', '1');
```
创建成功之后我们来到 student 表，tid 列名查看一下是否成功相关老师的数据，如下图能够显示出来就没啥问题
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708956518256-6d38731f-994d-4ab2-8cd5-be9e93baca19.png#averageHue=%23f4f2f1&clientId=u701ce8ce-daa4-4&from=paste&height=197&id=uee6e59e2&originHeight=295&originWidth=722&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=38541&status=done&style=none&taskId=u0ac3b915-a472-4a95-89d8-2c71fd38214&title=&width=481.3333333333333)
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708956640446-8e5fa436-1f92-4e5d-917e-a334a5e951fb.png#averageHue=%232d353f&clientId=u701ce8ce-daa4-4&from=paste&height=304&id=u0f8ceaa7&originHeight=456&originWidth=546&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=60039&status=done&style=none&taskId=uc14b8c7a-6738-49d4-a742-0914580753c&title=&width=364)
然后就是整个项目的内容，我们依然采取之前学习时的内容，就不重复，只不过要新增两个接口以及对应的 mapper.xml 文件，一个是 student ，一个是 teacher
节约时间，之前学到的方法我们都用上
这里的 student 类由于绑定了 teacher 关联，所以还得实例化一个 Teacher 类，Teacher 类内容在下面
```java

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Student {
    private String name;
    private int id;

    private Teacher teacher;
    
}

```

```java
package com.stoocea.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Teacher {
    private int id;
    private String name;
}
```
都用上 lombok
然后是两者的 mapper.xml 实现文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.stoocea.Dao.StudentMapper">

</mapper>
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.stoocea.Dao.TeacherMapper">
    <select id="getAllTeacher" resultType="teacher">
        select * from teacher;
    </select>
</mapper>
```

最后是 mybatis 核心配置文件中对两者 mapper 进行注册
```xml
    <mappers>
        <mapper resource="com/stoocea/Dao/UserMapper.xml"/>

        <mapper class="com.stoocea.Dao.TeacherMapper"/>
        <mapper class="com.stoocea.Dao.StudentMapper"/>
    </mappers>
```


然后更新测试方法
```java
    @Test
    public void testGetAllTeacher(){
        SqlSession sqlSession=MybatisUtils.getSqlSeesion();
        TeacherMapper teacherMapper=sqlSession.getMapper(TeacherMapper.class);
        List<Teacher> teacherlist=teacherMapper.getAllTeacher();

        for (Teacher teacher : teacherlist) {
            System.out.println(teacher);
        }
    }
```
测试通过
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708958523170-be8418ea-b7be-485b-8ec6-0ae85a8a504a.png#averageHue=%232e333c&clientId=u701ce8ce-daa4-4&from=paste&height=195&id=u9cb39f8f&originHeight=292&originWidth=1397&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=137147&status=done&style=none&taskId=u67eaf6f3-3258-48e9-826d-06e993b2afc&title=&width=931.3333333333334)

### 0x01 多对一查询
#### 1x01 子查询嵌套

现在有这么一个需求----"查询所有学生的信息，并且附带查询出每个学生所属老师的信息"

涉及复杂的查询情况，我们就不用注解进行查询了，这里配置 student 的 xml 文件
```plsql
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.stoocea.Dao.StudentMapper">

    <select id="getStudent" resultMap="StudentTeacher">
        select * from student;
    </select>
    <resultMap id="StudentTeacher" type="Student">
        <result property="id" column="id"/>
        <result property="name" column="name"/>
        <association property="teacher" column="tid" javaType="Teacher" select="getTeacher"/>
    </resultMap>

    <select id="getTeacher" resultType="Teacher">
        select * from teacher where id=#{id};
    </select>
</mapper>
```
我们将需求拆分为两个部分：

1. 查询到所有学生的信息
2. 通过查询到的学生信息中的 tid，去 teacher 表中查询 teacher 的信息

所以我们有两个查询 select 标签，一个是查询到所有的 student 信息，一个是查询到所有 id 为 tid 的老师的信息
但是我们如何将查询到的学生的 tid 传入 teacher 的查询语句呢？换个方式问，我们如何解决学生中属性名并且类型都不同的 teacher 属性与数据库字段 tid 不相同的情况？
第二种问法很自然想到 resultMap
```plsql
    <resultMap id="StudentTeacher" type="Student">
        <result property="id" column="id"/>
        <result property="name" column="name"/>
        <association property="teacher" column="tid" javaType="Teacher" select="getTeacher"/>
    </resultMap>
```
主要就是 assciation 标签了，他专门针对于我们的属性值为对象时使用 property 和 column 都不动，依然是我们属性值名和数据库列名，javaType，就是指定我们当前 assciation 标签指定的返回对象的类型，由于用了别名，直接写 teacher
然后就是 select，association 标签的原意就是将 查询出来的 Column 结果给到 select 的查询语句中
这也解释了为什么我们还有第二种查询： select 出所有的老师信息

再尝试 test 一下
```java
    @Test
    public void testgetStudent(){
        SqlSession sqlSession=MybatisUtils.getSqlSeesion();
        StudentMapper studentMapper=sqlSession.getMapper(StudentMapper.class);

        List<Student> studentlist=studentMapper.getStudent();
        for (Student student : studentlist) {
            System.out.println(student);
        }
    }
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708961210528-0937a35b-1f11-4be5-bdcb-b0057d3b7844.png#averageHue=%232f353e&clientId=u701ce8ce-daa4-4&from=paste&height=225&id=uae28a17c&originHeight=338&originWidth=1032&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=125976&status=done&style=none&taskId=u59179527-2920-4182-ab9a-44da952c2a2&title=&width=688)
#### 1x02 结果嵌套查询
其实就是两表联立查询
```plsql
select s.id sid,s.name sname,s.tid tid,t.name tname from student s join teacher t where s.tid=t.id
```
这一段 sql 查询语句建立了一张 student 和 teacher 的联合表，此时 student 的 id  name tid 在新表中分别为 sid sname tid 然后老师的 name 为 tname
所以我们是从这样一张新表中查询数据，结果集也要有所改变
下列结果集的 column 列的信息都是新表的列然后 property 属性还是不变，只是我们最后的 teacher 属性，由于他是对象，我们单独列一项出来 association 标签，并且指定好类型，查询他的 name 信息，column 为两表联合之后的新列
```xml
        <resultMap id="StudentTeacher2" type="Student">
            <result property="tid" column="tid"/>
            <result property="name" column="sname"/>
            <result property="id" column="sid"/>
            <association property="teacher" javaType="Teacher">
                <result column="tname" property="name"/>
            </association>
        </resultMap>
```
查询语句：
```xml
    <select id="getStudent2" resultMap="StudentTeacher2">
        select s.id sid,s.name sname,s.tid tid,t.name tname from student s join teacher t where s.tid=t.id
    </select>
```

### 0x02 一对多查询
一对多的思路是差不多的，还是按两种模式走，要么就是子查询作为条件，要么就是按结果嵌套查询
但是子查询比较绕，我这里只写结果集处理的方法了
```xml
<select id="getAllTeacherAndS" resultMap="TeacherAndStudent">
  select t.id tid,t.name tname,s.name sname from student s join teacher t where s.tid=t.id
</select>

<resultMap id="TeacherAndStudent" type="teacher">
  <result property="id" column="tid"/>
  <result column="tname" property="name"/>
  <collection property="students" ofType="Student">
    <result column="id" property="sid"/>
    <result property="name" column="sname"/>
    <result column="tid" property="tid"/>
  </collection>
</resultMap>
```
要注意一点就是我们 teacher 中的学生属性是 List<Student>类型，是一个列表集合，我们要采用 collection 标签，然后里面的类型不用 javaType 去指定，而是用 ofType 特殊属性
```java
@Test 
public void testgetAllTeacherwithStudent(){
    SqlSession sqlSession=MybatisUtils.getSqlSeesion();
    TeacherMapper teacherMapper=sqlSession.getMapper(TeacherMapper.class);
    for (Teacher allTeacherAnd : teacherMapper.getAllTeacherAndS()) {
        System.out.println(allTeacherAnd);
    }
}
```
查询结果如下，由于没有修改学生的 teacher 属性值，还是老项目，所以学生的 teacher 属性为 null 
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709002678240-b3909ed8-95f1-4beb-bd2c-dcb7576fbdf2.png#averageHue=%23313740&clientId=u011e6a75-96b8-4&from=paste&height=32&id=u59691c16&originHeight=48&originWidth=1891&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=40193&status=done&style=none&taskId=ufddbc688-d93d-4118-b801-91c82b7474f&title=&width=1260.6666666666667)


## 13 动态 SQL
什么是动态 SQL?  就是根据不同的情况来生成不同的 SQL 语句
### 0x01 动态 SQL 环境搭建
先建表 
```plsql
REATE TABLE `blog`(
  `id` VARCHAR(50) NOT NULL COMMENT '博客id',
  `title` VARCHAR(100) NOT NULL COMMENT '博客标题',
  `author` VARCHAR(30) NOT NULL COMMENT '博客作者',
  `create_time` DATETIME NOT NULL COMMENT '创建时间',
  `views` INT(30) NOT NULL COMMENT '浏览量'
)ENGINE=INNODB DEFAULT CHARSET=utf8
```
然后再新建项目
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709013539716-dafb377e-6fb0-483d-b967-01b41f5f8a61.png#averageHue=%232a3036&clientId=u011e6a75-96b8-4&from=paste&height=313&id=uffc46c0e&originHeight=470&originWidth=550&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=62639&status=done&style=none&taskId=u0b73cea7-0534-4b53-92dd-537ad8a6b60&title=&width=366.6666666666667)
有些文件不变，可以直接复制之前的项目内容
还有几个关键的类：

- 两个工具类：UUID 获取工具类    SqlSession 获取工具类
- Blog 对象
- Blog 接口 xml 文件

```java
package com.stoocea.Utils;

import java.util.UUID;

public class IDUtils {
    public static String getID(){
        return UUID.randomUUID().toString();

    }
}

```

```java
package com.stoocea.Utils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.InputStream;

//构建SqlSessionFactory工具类
public class MybatisUtils {
    private static SqlSessionFactory sqlSessionFactory;

    static {
        try {
            //获取SqlSessionFactory对象
            String resource = "mybatis-config.xml";
            //mybatis中专门用来读取mybatis-config.xml文件内容的读取输入流
            InputStream inputStream = Resources.getResourceAsStream(resource);
            //通过builder来生成sqlSessionFactory
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (Exception e) {
            e.printStackTrace();
        }

    }

    //既然获取到了Factory，现在就是把SqlSession返回出去了，这也是我们构建这个工具类的意义
    public static SqlSession getSqlSeesion(){
        return sqlSessionFactory.openSession();
    }
}

```


```java
package com.stoocea.Dao;

import com.stoocea.Pojo.Blog;
public interface BlogMapper {
    boolean AddBlog(Blog blog);
}

```

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.stoocea.Dao.BlogMapper">

        <insert id="AddBlog" parameterType="blog">
            insert into blog(id,title,author,create_time,views) values (#{id},#{title},#{author},#{create_time},#{views})
        </insert>

</mapper>
```
我们根据这里的插入语句先重复插入几段数据
```java
package com.stoocea.Dao;

import com.stoocea.Utils.IDUtils;
import com.stoocea.Utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;
import com.stoocea.Pojo.*;

import java.util.Date;

public class Mytest {
    @Test
    public void testAddBlog(){
        Blog blog=new Blog();
        String uuid=  new IDUtils().getID();
        blog.setId(uuid);
        blog.setTitle("howFoc1s");
        blog.setAuthor("Archer");
        blog.setCreate_time(new Date());
        blog.setViews(100);
        SqlSession sqlSession= MybatisUtils.getSqlSeesion();
        BlogMapper blogMapper=sqlSession.getMapper(BlogMapper.class);
        blogMapper.AddBlog(blog);
        sqlSession.commit();
        sqlSession.close();
    }
}

```
多插入几组
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709016420501-e39eb107-b6e6-407e-aa41-8ad10d85293d.png#averageHue=%23343943&clientId=u011e6a75-96b8-4&from=paste&height=167&id=u5d8b547b&originHeight=251&originWidth=1389&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=134500&status=done&style=none&taskId=u1ef9b06a-0a98-4a44-b2f4-bd4d224b82b&title=&width=926)

### 0x02 动态 SQL 之 if 与常用标签
现在有一个需求，当我给你 title 时，你要能够查询出所有指定 title 行的数据，如果不传，你仍然能够给我查询出数据
动态 SQL 的编辑都是在 mapper 的 xml 文件中实现的
```xml
<select id="SelectBlog" resultType="Blog" parameterType="map">
  select * from blog
  <if test="title !=null">
    where title=#{title}
  </if>
  <if test="author != null">
    and author=#{author}
  </if>
</select>
```
if 标签的作用就是通过 test 的判断条件，如果传入参数满足 test 中的条件，则自动添加标签内的 sql 语句，如果不满足就不拼接

再看测试类，由于我们传的是 map 参数，他会自动检测 map 中的键值，这里我们没有传任何的东西，导致他两个 if 都不会进入，所以直接 select *
```java
    @Test
    public void testSelectBlog(){
        SqlSession sqlSession=MybatisUtils.getSqlSeesion();
        BlogMapper blogMapper=sqlSession.getMapper(BlogMapper.class);
        HashMap map=new HashMap();
        List<Blog> blogList=blogMapper.SelectBlog(map);
        for (Blog blog : blogList) {
            System.out.println(blog);
        }
    }
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709017380528-5f315b6d-6f87-44a1-8ea8-41d0dac4eabd.png#averageHue=%23313740&clientId=u011e6a75-96b8-4&from=paste&height=103&id=u1542464f&originHeight=154&originWidth=1881&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=141445&status=done&style=none&taskId=uc9059be4-4e27-4454-b930-a55c9cb4fea&title=&width=1254)


假如说我们 map put 一个条件 title 键值对为 howFoc3s
```java
    @Test
    public void testSelectBlog(){
        SqlSession sqlSession=MybatisUtils.getSqlSeesion();
        BlogMapper blogMapper=sqlSession.getMapper(BlogMapper.class);
        HashMap map=new HashMap();
        map.put("title","howFoc3s");
        List<Blog> blogList=blogMapper.SelectBlog(map);
        for (Blog blog : blogList) {
            System.out.println(blog);
        }
    }
```
查询结果如下
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709017514295-46600b98-ff73-495a-bf8b-56f93356ca03.png#averageHue=%232f353e&clientId=u011e6a75-96b8-4&from=paste&height=63&id=u3022688a&originHeight=94&originWidth=1849&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=68157&status=done&style=none&taskId=uc5d0d4f2-de4f-4808-97a8-caf02248efb&title=&width=1232.6666666666667)
然后就是 choose、when、otherwise 分别是 if ，else if，else 的意思

再介绍一个 set 标签，update 数据的时候使用，里面可插 if 等
```java
<update id="updateblog" parameterType="Map">
        update blog
        <set>
            <if test="title!=null">
                title=#{title},
            </if>
            <if test="author!=null">
                author=#{author},
            </if>
            <if test="views!=null">
                views=#{views}
            </if>
            <where>
                <choose>
                    <when test="id!=null">
                        id=#{id}
                    </when>
                    <otherwise>
                        id="1"
                    </otherwise>
                </choose>
            </where>
        </set>
    </update>
```

后续的标签就不做过多介绍了，mybatis 就此，后续进入 spring
