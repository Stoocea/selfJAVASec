# Spring
## 1.什么是Spring

他是一个设计层面的框架，极大的简化了开发---java程序员的春天

然后了解其核心是**控制反转（IOC）**和面向切面**（AOP）**，换一句话说 Spring是一个轻量级的**控制反转**（IOC），和面向切面编程的（AOP）的框架

## 2.IOC理论推导

IoC全称Inversion of Control,中文翻译控制反转，这里的反转是指程序主要控制权是谁的手里

之前的业务实现很大程度上是靠程序员一点一点堆上去的，用户想要什么，程序员就多写一点需求，然后一一对应，很是笨重。

而IOC就是要把控制权交给用户，用户想要调啥，我们程序能够指定去干啥，这就是控制反转。

看下面代码例子：

UserMapper接口

```java
public interface UserMapper {
    void getUser();
}
```

UserMapper实现类

```java
package com.stoocea.Dao;

public class UserMapperImpl implements UserMapper{
    @Override
    public void getUser() {
        System.out.println("默认获取用户数据");
    }
}
```

UserService接口

```java
package com.stoocea.Services;

public interface UserService {
 void getUser();
}
```

UserService实现类
```java
package com.stoocea.Services;
import com.stoocea.Dao.UserMapper;
import com.stoocea.Dao.UserMapperImpl;

public class UserServiceImpl implements UserService {
    UserMapper userMapper=new UserMapperImpl();

    @Override
    public void getUser() {
        userMapper.getUser();
    }
}
```
整体逻辑就是按照一般业务来，Dao 层把功能写好，然后用户想要查询的时候由 Service 层去调用 Dao 层对应的功能逻辑----实例化 UserMapperImpl，然后调它的方法

这一整段的逻辑有一个特点：程序是写死的，用户此时只能调这一个功能，假如说我们的 Dao 层多了如下的代码：
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709033886743-024315fb-8a4c-486b-b266-61757ec5a6e0.png#averageHue=%23293138&clientId=u8b065c98-1cab-4&from=paste&height=108&id=ud0b5505c&originHeight=162&originWidth=514&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=29039&status=done&style=none&taskId=u91423969-d139-4024-b113-0daea15d2bc&title=&width=342.6666666666667)
```java
package com.stoocea.Dao;

public class UserMysqlMapperImpl implements UserMapper {
    @Override
    public void getUser() {
        System.out.println("正在调用Mysql数据库的数据");

    }
}

```

```java
package com.stoocea.Dao;

public class UserOracleMapperImpl implements UserMapper{

    @Override
    public void getUser() {
        System.out.println("正在调Oracle数据库的数据");
    }
}
```
那么用户想要具体的调取 Oracle 数据库的数据或者 Mysql 数据库时，Service 层的代码就必须大改
比如说用户想要调 Mysql 时，service 层代码必须这么写
```java
package com.stoocea.Services;
import com.stoocea.Dao.UserMapper;
import com.stoocea.Dao.*;

public class UserServiceImpl implements UserService {
    UserMapper userMapper=new UserMysqlMapperImpl();

    @Override
    public void getUser() {
        userMapper.getUser();
    }
}
```

如果用户想要调用 Oracle ，service 层代码就必须这么写
```java
package com.stoocea.Services;
import com.stoocea.Dao.UserMapper;
import com.stoocea.Dao.*;

public class UserServiceImpl implements UserService {
    UserMapper userMapper=new UserOracleMapperImpl();

    @Override
    public void getUser() {
        userMapper.getUser();
    }
}

```
也就是我们 service 层必须实例化两个不同的 UserMapperImpl，你可能会说那就多建几个 Service 处理的类，但是如果项目有很多这样的操作，那就是重复性的工作冗余
所以我们将控制权交给用户，他想怎么办，程序就跟着变
```java
import com.stoocea.Dao.UserMapper;
import com.stoocea.Dao.*;

public class UserServiceImpl implements UserService {
    UserMapper userMapper;

    public void setUserMapper(UserMapper userMapper) {
        this.userMapper = userMapper;
    }

    @Override
    public void getUser() {
        userMapper.getUser();
    }

}

```
然后测试类（实际调用的地方）就可以这么写
```java
import com.stoocea.Dao.UserMysqlMapperImpl;
import com.stoocea.Services.UserService;
import com.stoocea.Services.UserServiceImpl;
import org.junit.Test;

public class Mytest {

    @Test
    public void testGetUser(){
        UserServiceImpl userService=new UserServiceImpl();
        userService.setUserMapper(new UserMysqlMapperImpl());
        userService.getUser();
    }

}

```
我们通过 set 方法，让用户能够自定义他想用的数据库，我们只需要写好存在的方法和逻辑即可（换一种说法--偷懒？）
IOC 的思想使得程序员不用去管理对象的创建，系统耦合性降低，可以更加专注的在业务的实现上
当然上面的内容仅仅只是 IOC 的原型，很原始，并不是最终形态，还有很多问题没有去解决
这里再解释一下耦合性的问题，耦合意思就是每个类或实体之间的连接，并且互相影响，导致一个出问题，其他就得跟着出问题，这其实对于一个大型项目来说是很致命的，**解耦**这一个方面，就被抬到了很高的位置
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709037453413-7949d9aa-a2a5-477a-a53a-5670140d6438.png#averageHue=%23f1f0f2&clientId=u86e4a55d-2d9e-4&from=paste&height=291&id=u9f596b78&originHeight=437&originWidth=1372&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=309721&status=done&style=none&taskId=ud0ca58ee-363a-4455-afc4-e50349bef54&title=&width=914.6666666666666)
那么如何解耦，以及我们所说的 IOC 思想降低系统耦合性的直观体现，可看上面这张图
经典的 IOC 远远不止我们上面小实验的内容，我们后面的学习内容也是去配置 Spring 的 IOC，然后使用和体悟 IOC，所以接下来，进入 Hello-Spring 环节

## 3.Hello-Spring
### 0x01 环境配置
我个人的 maven 配置的依赖全部如下，采用的是 boogipop 师傅的配置，注意条件：**JDK17 及以上**
```xml
  <dependencies>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-core -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>6.0.4</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-beans -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-beans</artifactId>
      <version>6.0.4</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>6.0.4</version>
    </dependency>

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

```
做完配置，新建项目，整理结构
然后就是写 Spring 的配置文件，这个配置文件再讲的具体一点就是 bean 的配置文件
当然在写 bean 配置文件之前，我们还是得写一个 bean 出来
```java
package com.stoocea.Pojo;

public class Hello {
    private String str;

    public String getStr() {
        return str;
    }

    @Override
    public String toString() {
        return "Hello{" +
                "str='" + str + '\'' +
                '}';
    }

    public void setStr(String str) {
        this.str = str;
    }
}

```

然后再去写 bean.xml 文件，**通过 bean 标签的 class 属性来定位该 bean**，然后 bean 标签内部也是可以写东西的，就比如说可以自定义属性值--通过 property 标签实现
通过 bean 标签的 id 来指定实例化之后的实体变量名
```xml
 <?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
        <bean id="hello" class="com.stoocea.Pojo.Hello">
            <property name="str" value="Stoocea"/>
        </bean>
</beans>
```

然后通过测试类去验证一下
```java
import com.stoocea.Pojo.Hello;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import com.stoocea.Pojo.*;
public class Mytest {

    @Test
    public void testHello(){

        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        Hello hello= (Hello) context.getBean("hello");
        System.out.println(hello.toString());
    }
}

```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709038680313-aa578b15-ecc9-468b-a843-1e919c9de40e.png#averageHue=%2331363f&clientId=u86e4a55d-2d9e-4&from=paste&height=99&id=ufa88a58c&originHeight=148&originWidth=622&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=31153&status=done&style=none&taskId=uc5a6c418-c198-49b1-ba8c-7416e57df6e&title=&width=414.6666666666667)

有几个注意点：

1. 当我们在 bean 工厂注册完类之后，该类应该会在旁有标识
2. bean 的属性值设置是通过 set 方法去实现的，如果该类没有 set 方法（也就是当前类不是 bean 类的时候）他就无法进行属性设置，只能获取到该类的实例化对象

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709038877320-61337554-e9c7-4c0e-b150-3fd36c60b3df.png#averageHue=%232e333c&clientId=u86e4a55d-2d9e-4&from=paste&height=482&id=uf32baf05&originHeight=723&originWidth=815&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=188908&status=done&style=none&taskId=ud59eeaa9-a555-4636-a357-b6f4028de35&title=&width=543.3333333333334)

### 0x02 IoC 创建对象
上面已经演示过一个通过 bean xml 文件去配置托管对象了，但那其实只是其中一种方法--通过无参构造去实例化对象。如果要有参构造对象，默认的 IOC 创建对象有如下方式：

#### 1x01 通过索引去有参构造对象
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  https://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="hello" class="com.stoocea.Pojo.Hello">
    <constructor-arg index="0" value="stoocea"/>
  </bean>

</beans>
```

然后写个测试类
```java
import com.stoocea.Pojo.Hello;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import com.stoocea.Pojo.*;
public class Mytest {

    @Test
    public void testHello(){

        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        Hello hello= (Hello) context.getBean("hello");
        System.out.println(hello);
    }
}
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709040620169-163e33cb-5f6e-47c0-93ae-175ee7935990.png#averageHue=%2331363f&clientId=u86e4a55d-2d9e-4&from=paste&height=100&id=ud05cef1d&originHeight=150&originWidth=619&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=32908&status=done&style=none&taskId=u8d9c3840-b389-4fcc-9bdd-c04bc64e92b&title=&width=412.6666666666667)

#### 1x02 通过形参名去有参构造对象
动的只是 beans.xml 文件的内容
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
        <bean id="hello" class="com.stoocea.Pojo.Hello">
            <constructor-arg name="str" value="stoocea"/>
        </bean>

</beans>
```

#### 1x03 通过定位形参的类型去有参构造
这个不太推荐，因为如果出现两个属性值的类型相同就无法定位了
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
        <bean id="hello" class="com.stoocea.Pojo.Hello">
            <constructor-arg type="java.lang.String" value="stoocea"/>
        </bean>

</beans>
```
结果都是一样的，只是形参名去实例化的时候报了个 warning，不影响最终结果
最推荐的是通过形参参数名去赋值，因为之后可能会有直接注入对象的操作，可以更好的区分
有一个很有趣的点，我们新写一个 Pojo 类，有参构造和无参构造都有
```java
package com.stoocea.Pojo;

public class UserT {
    private String flag;

    @Override
    public String toString() {
        return "UserT{" +
        "flag='" + flag + '\'' +
        '}';
    }

    public String getFlag() {
        return flag;
    }

    public void setFlag(String flag) {
        this.flag = flag;
    }

    public UserT(String flag) {
        this.flag = flag;
    }
    public UserT(){
        System.out.println("调用了UserT的无参构造");
    }
}

```
然后 beans.xml 文件装配
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  https://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="hello" class="com.stoocea.Pojo.Hello">
    <constructor-arg type="java.lang.String" value="stoocea"/>
  </bean>
  <bean id="UserT" class="com.stoocea.Pojo.UserT"></bean>
</beans>

```
测试类不变，仅仅只是获取 hello 类，并且只调用 hello 的方法
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709041488016-ba70490d-1af5-41ec-9c4c-51ddd9f18ee8.png#averageHue=%2331363f&clientId=u86e4a55d-2d9e-4&from=paste&height=123&id=u7f9ca53a&originHeight=185&originWidth=817&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=41389&status=done&style=none&taskId=udacad22a-7b5e-4117-a22d-1a19c33c47c&title=&width=544.6666666666666)
发现 UserT 的无参构造也被调用了，也就是说现在的上下文中是存在 UserT 的实例化对象的，Spring 在获取 XML 文件内容，并且根据文件内容进行类实例化的时候，就已经把所有的注册的 bean 都无参实例化了一遍（除了指定有参构造的 bean 以外）

### 0x03 Spring 配置说明
bean 配置的话只有如下的内容
bean beans alias description 不细写，稍微说一下 import，它只有在团队项目中才会用到，因为有可能要导入多个 bean 文件整合，所以必须要有的导入功能
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709041877941-9896fc99-53a0-45a2-86e4-b5132508d73b.png#averageHue=%23242930&clientId=u86e4a55d-2d9e-4&from=paste&height=169&id=u966a8376&originHeight=253&originWidth=826&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=24046&status=done&style=none&taskId=ub79578ae-dddd-453c-aec0-5386a458ddf&title=&width=550.6666666666666)

## 4.DI （依赖注入）

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709042326661-4d62b239-3dc5-4901-a68e-4ed06f7e279b.png#averageHue=%231d2125&clientId=u86e4a55d-2d9e-4&from=paste&height=118&id=u73c4e49c&originHeight=177&originWidth=532&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=14219&status=done&style=none&taskId=u8c7cfaff-acc3-496e-9ca9-7d52f08e1aa&title=&width=354.6666666666667)
依赖注入主要有两种方式，一种是通过构造器注入，一种是通过 Setter 注入（另外一种扩展方式会提到）
构造器注入刚才上面已经学习过了，定位 0x02-1x0？等内容，所以待会重点看 Setter 注入
但是在看之前我们还要记录几个问题：
**1.什么是依赖**
**bean 对象的创建依赖于容器，依赖注入的本质就是 set 注入**
**2.注入什么，往哪注入**
**bean 对象中的所有属性，由容器来注入，其实注入的对象就是 bean，具体到 bean 的每一个属性值**
为了展示全 DI 的方式，我们要先创建一个实验环境

### 0x01 set 方式注入
第一个就是创建一个复杂 bean
```java
import java.util.*;

public class Student {
    private String name;
    private Address address;
    private String[] books;
    private List<String> hobbys;
    private Map<String,String> card;
    private Set<String> games;
    private String wife;
    private Properties info;
//...get set等方法
}
```
Adress 也给他写一下
```java
package com.stoocea.Pojo;

public class Address {
    private String address;

    public void setAddress(String address) {
        this.address = address;
    }

    public String getAddress() {
        return address;
    }
}

```
环境搭建好之后可以去 bean 文件装配了
特别的是 map 和 properties，形式可能不太一样
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
        <bean id="hello" class="com.stoocea.Pojo.Hello">
            <constructor-arg type="java.lang.String" value="stoocea"/>
        </bean>
        <bean id="UserT" class="com.stoocea.Pojo.UserT"></bean>

    <bean id="Address" class="com.stoocea.Pojo.Address"/>
    <bean id="student" class="com.stoocea.Pojo.Student">
            <property name="name" value="stoocea"/>


            <property name="address" ref="Address"/>


        <property name="books">
                <array>
                    <value>stoocea1</value>
                    <value>stoocea2</value>
                    <value>stoocea3</value>
                </array>
            </property> 


        <property name="hobbys">
            <list>
                <value>no</value>
                <value>elsworld</value>
                <value>music</value>
            </list>
        </property>


        <property name="card">
            <map>
                <entry key="身份证" value="1321323131212312"/>
                <entry key="QQ" value="1323312323123123"/>
            </map>
        </property>


        <property name="games">
            <set>
                <value>LOL</value>
                <value>Elsworld</value>
            </set>
        </property>
        
        
        <property name="wife">
            <null/>
        </property>


        <property name="info">
            <props>
                <prop key="学号">123132312</prop>
            </props>
        </property>

    </bean>
</beans>
<
```
然后写个测试类
```java
import com.stoocea.Pojo.Hello;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import com.stoocea.Pojo.*;
public class Mytest {

    @Test
    public void testHello(){

        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        Student student= (Student) context.getBean("student");
        System.out.println(student);
    }
}

```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709079698404-38b5c7f5-5638-4dd2-ba11-2222f998aac8.png#averageHue=%23313740&clientId=ua6b3788e-3553-4&from=paste&height=35&id=u927119ca&originHeight=53&originWidth=1855&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=42716&status=done&style=none&taskId=u5d564540-9a72-48f7-9501-353c8a8e799&title=&width=1236.6666666666667)
#### 1x01 p 命名空间和 c 命名空间注入
使用之前必须导入 xml 约束
```xml
xmlns:p="http://www.springframework.org/schema/p"
xmlns:c="http://www.springframework.org/schema/c"
```
两者还是在 bean 标签中实现的，只不过使用之前必须要有其对应的配置导入
先看我们的目标配置类，UserN 具有 name 字符串和 hobbys 集合，集合里面的每个属性值的类型是 UserT 这个类
```java
package com.stoocea.Pojo;

import java.util.*;
public class UserN {
    private String name;
    private List<UserT> hobbys;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List<UserT> getHobbys() {
        return hobbys;
    }

    public void setHobbys(List<UserT> hobbys) {
        this.hobbys = hobbys;
    }

    @Override
    public String toString() {
        return "UserN{" +
                "name='" + name + '\'' +
                ", hobbys=" + hobbys.toString() +
                '}';
    }
}

```
然后再看 bean 文件是如何装配它的
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:p="http://www.springframework.org/schema/p"    <!--p命名空间使用的前提导入-->
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  https://www.springframework.org/schema/beans/spring-beans.xsd">

  <beans>
    <bean id="UserN" class="com.stoocea.Pojo.UserN" p:name="name" p:hobbys-ref="UserT"/>
  </beans>
```
p 命名空间的意思就是 properties 属性的简写，如果是简单的字符串，或者其他基础类型，可以直接用 p 命名空间解决，如果涉及其他的复杂的 DI，还是通过上面的方法来，当然两者的本质还是通过 set 方式的注入
再来看 c 命名空间，目标类是上面的 UserT
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

  <beans>
    <bean id="UserT" class="com.stoocea.Pojo.UserT" c:flag="stoceaT"/>
  </beans>
```
c 的意思就是 constuctor 的简写，也就是有参构造的简化操作，依然是如果类型很简单，c 标签的有参构造完全足够，复杂还是老老实实 DI
然后 c 命名空间的注入也是有两种的，通过下标，或者直接名称定位
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709081951264-724d3c3c-f612-4a73-8979-3331b23f7a9e.png#averageHue=%2324282f&clientId=ua6b3788e-3553-4&from=paste&height=342&id=u0ac0e20b&originHeight=513&originWidth=789&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=43126&status=done&style=none&taskId=u18d17813-873b-4581-8540-668f1ca9fc0&title=&width=526)
## 
### 0x02 bean 作用域
官方文档说法，我们在配置 bean 的时候，同时能够设置的它的作用域
在 Spring 框架下总计能够设置 6 个作用域，后面四个都是只有在 web 应用中使用
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709086319106-338ab317-084e-42c4-8878-d0dc8827df13.png#averageHue=%231f2429&clientId=u83ae5e54-13ee-4&from=paste&height=405&id=ua04ddec5&originHeight=607&originWidth=1452&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=148676&status=done&style=none&taskId=u4fa3d92e-bb1e-41fb-b3b5-fd88056a774&title=&width=968)

#### 1x01 singleton 单例模式
默认情况下，bean 的作用域是 singleton 单例模式，也就是不论从`ApplicationContext`取出多少个对象，只要是同名对象，都只会生成一个对象
```java
import com.stoocea.Pojo.Hello;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import com.stoocea.Pojo.*;
public class Mytest {

    @Test
    public void testHello(){

        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        UserT userT=  context.getBean("UserT",UserT.class);
        UserT userT2=context.getBean("UserT", UserT.class);
        System.out.println(userT2==userT);
    }
}
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709086630831-66f5a8fe-7f8f-4223-a763-d4e645d69922.png#averageHue=%232e333c&clientId=u83ae5e54-13ee-4&from=paste&height=154&id=ub5d2282e&originHeight=231&originWidth=1911&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=139488&status=done&style=none&taskId=uca9fed42-a50f-41ff-a01b-350ee4c4bbd&title=&width=1274)
结果是 true，说明我们取到的是同一个对象
而单例模式其实就是全局共享作用域，而且单例模式是 bean 的默认模式，我们可以显示出来
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

<bean id="UserT" class="com.stoocea.Pojo.UserT" c:flag="stoceaT" scope="singleton"/>
</beans>
```

#### 1x02 prototype 原型模式
原型模式就是跟 singleton 模式相反的一个模式，我们从 context 中取出的所有对象都是独立的个体
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

<bean id="UserT" class="com.stoocea.Pojo.UserT" c:flag="stoceaT" scope="prototype"/>
</beans>
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709088295602-a698fb0f-1446-406e-bfa2-ba50fc62c35a.png#averageHue=%232e323b&clientId=u83ae5e54-13ee-4&from=paste&height=164&id=uf063220e&originHeight=246&originWidth=1937&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=146733&status=done&style=none&taskId=ud9b9f7cd-ef06-4af0-af08-aba853e8df3&title=&width=1291.3333333333333)

## 5 自动装配 bean
### 0x01 简单实现以及环境搭建
Spring 去装配 bean 有三种方式，第一种通过 xml 文件去装配我们已经学习了，第二种是通过 java 代码去实现，第三种就是 Spring 自动装配 bean
具体实现如下
```xml
    <bean id="cat" class="com.stoocea.Pojo.Cat"/>
    <bean id="dog" class="com.stoocea.Pojo.Dog"/>
    <bean id="person" class="com.stoocea.Pojo.Person" autowire="byName">
        <property name="name" value="stoocea"/>
    </bean>
```


```java
package com.stoocea.Pojo;

public class Person {
    private String name;
    private Cat cat;
    private Dog dog;

//下面是set和get以及toString
}

```

单单以 Dog 为例，cat 类的实现是一样的
```java
package com.stoocea.Pojo;

public class Dog {
    public void shout(){
        System.out.println("wang");
    }
}

```

```java
@Test
public void testPerson(){
    ApplicationContext context=new ClassPathXmlApplicationContext("beans.xml");
    Person person=context.getBean("person", Person.class);
    person.getDog().shout();
    person.getCat().shout();
}

```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709089335496-b5cc1905-ef08-40dd-8b6d-bcc3d65fefb9.png#averageHue=%232d323a&clientId=u83ae5e54-13ee-4&from=paste&height=197&id=EDufh&originHeight=296&originWidth=1863&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=160881&status=done&style=none&taskId=ue5043f1b-29ee-4ec7-8c99-3cdeda7b9d0&title=&width=1242)

其实自动装配还有很多的方式区实现，只不过最精确最适用的是 byName，因为他会去 Spring 容器去找同名的 bean，但如果属性值是对象类型的话，byType 还是能找到的
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709089461622-27003d87-10a6-42dd-9443-bece44d83b07.png#averageHue=%232a2f37&clientId=u83ae5e54-13ee-4&from=paste&height=204&id=uc3b86da7&originHeight=306&originWidth=1500&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=109334&status=done&style=none&taskId=u2aa43233-1c27-47ae-912a-0ac3bbb189a&title=&width=1000)

### 0x02 注解实现自动装配
首先看一段文件
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709091762398-8090db44-9d8d-4650-9426-f05f7bd4685e.png#averageHue=%232e343c&clientId=u34eb92a5-9bb8-4&from=paste&height=278&id=ua164d6b1&originHeight=417&originWidth=1348&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=153618&status=done&style=none&taskId=u492a94ab-b067-4255-b174-1c83ddd924b&title=&width=898.6666666666666)
既然它提供了这么多方法去实现，肯定是每个方法都是各自有用处的，也各有各的好坏
在注解实现自动装配之前，我们首先得在 beanxml 文件中打上约束，官方文档有具体的内容，这里是我自己的约束
```xml
xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:c="http://www.springframework.org/schema/c"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd"
```
重点就是一个 `xmlns:context="[http://www.springframework.org/schema/context"](http://www.springframework.org/schema/context")` 
以及两个：
`[http://www.springframework.org/schema/context](http://www.springframework.org/schema/context)` 
`[https://www.springframework.org/schema/context/spring-context.xsd](https://www.springframework.org/schema/context/spring-context.xsd)`

导入完约束之后，还需要配置注解的支持，其实就是在打完约束之后，再在 beans 标签内加上一段
```xml
<context:annotation-config/>
```

#### 1x01 @Autowired 注解
这个时候我们再修改一下 xml，只留下最基本的对象注册
```xml
    <bean id="cat" class="com.stoocea.Pojo.Cat"/>
    <bean id="dog" class="com.stoocea.Pojo.Dog"/>
    <bean id="person" class="com.stoocea.Pojo.Person"/>
```
然后在 Person 类中写一下自动配置的注解 `@Autowired`
```java

import org.springframework.beans.factory.annotation.Autowired;

public class Person {

    private String name;

    @Autowired
    private Cat cat;
    @Autowired
    private Dog dog;

    @Override
    public String toString() {
        return "Peroson{" +
        "name='" + name + '\'' +
        ", cat=" + cat +
        ", dog=" + dog +
        '}';
    }

//set和get方法以及toString方法的内容......
}
```
会发现他在类中确实有了被装配的标志，然后去 test 测试一下
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709096729030-7112fc84-0f04-48f9-84de-1ce6a541c4c5.png#averageHue=%232f333c&clientId=u34eb92a5-9bb8-4&from=paste&height=265&id=u93cb60a7&originHeight=398&originWidth=947&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=109551&status=done&style=none&taskId=uee2a7dba-c878-49d7-b992-b47595223a7&title=&width=631.3333333333334)
依旧成功
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709096808956-a61faf07-b307-49f9-955e-6203dd292d78.png#averageHue=%232e333c&clientId=u34eb92a5-9bb8-4&from=paste&height=497&id=u6ac28de4&originHeight=745&originWidth=1800&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=357709&status=done&style=none&taskId=u2aa50468-0937-4ee2-847b-0e988075b69&title=&width=1200)

`@Autowired`的实现甚至都不需要的 set 方法，只需要在 IOC 容器中存在
`@Autowired`其实靠的就是 byname 的方式在 IOC 容器寻找同名 bean，然后进行装配，只不过它的 byname 方法比较智能，即使 IOC 容器中的目标 bean 的 id 不跟属性值相同，也能够实现装配，但是不是如果不是唯一的 bean 就不行了
所以`@Autowired`这个时候一般是和另外一个注解联合使用---- `_@Qualifier_`_，_它用来指定自动装配时，具体到底是哪一个 bean
假如我们此时同类的 bean1 有两个 dog2222 和 dog222 
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709097568606-dca82f43-3f74-430f-9c3b-c0970a730155.png#averageHue=%232f343d&clientId=u34eb92a5-9bb8-4&from=paste&height=119&id=u7b618bd2&originHeight=178&originWidth=1070&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=67832&status=done&style=none&taskId=u96fe2acf-4670-4787-9aae-bc763b82a63&title=&width=713.3333333333334)

然后 Qualifier 指定是装配 id 为 dog222 的 bean 
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709097592406-e4c368c7-5854-48bb-b85c-d4ba0ebaa8ae.png#averageHue=%232e333c&clientId=u34eb92a5-9bb8-4&from=paste&height=138&id=u6c54796e&originHeight=207&originWidth=567&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=33238&status=done&style=none&taskId=u09936415-ff1c-4605-9ec3-5b23307b8a8&title=&width=378)
成功执行
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709097629122-c840791f-8259-444c-a69e-255bd2535109.png#averageHue=%232e333c&clientId=u34eb92a5-9bb8-4&from=paste&height=468&id=ue0df7bfc&originHeight=702&originWidth=1624&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=289933&status=done&style=none&taskId=u91e7aed1-ed41-4472-b0c7-27b123ea670&title=&width=1082.6666666666667)
但其实 java 自带一个注解更加智能，它会先在 IOC 容器中 byname 搜索，然后再 bytype 搜索
一般要使用的话也是直接写在属性名上面，或者 set 方法的上面，但是 JDK 高版本就没有了，本身@Autowired 也够用了，并且也有Qualifier 辅助其使用
```java
@Resource
public void setPeople(People people) {
    this.people = people;
}
```

## 6 Spring 注解开发
注解开发的内容已经接触过了，上面的注解自动装配属性就是其中一种，但其实 Spring 的注解还有很多的内容

### 0x01 环境搭建
新建一个 moudle，然后项目结构不变，beanxml 文件包括进来
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.stoocea"/>
    <context:annotation-config/>

</beans>
```
这里多了一行内容 `<context:component-scan base-package="com.stoocea.Pojo"/>` component---装配扫描器，他会往指定的目录中扫描注解，只要有`@Component`注解在上的类，就会被识别，装入 IOC 容器
```java
package com.stoocea.Pojo;

import org.springframework.stereotype.Component;

@Component
public class User {
    private String name;
}

```
但是现在还没有对其属性值进行装配，这里我们配合另一个注解 `@Value`进行实现
```java
package com.stoocea.Pojo;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class User {
    @Value("stoocea")
    private String name;
//set和get，以及toString....
}
```
Component 注解中有一个参数：
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709101659292-9cdfe16f-f08d-4f1d-ae55-6b8eeda78056.png#averageHue=%233a4049&clientId=ubaa01bb3-6a62-4&from=paste&height=74&id=u5e8c7365&originHeight=111&originWidth=485&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=24920&status=done&style=none&taskId=ue679315b-5120-4ec5-9d1b-30c264bf382&title=&width=323.3333333333333)
value 用来起 IOC 容器中的 id
当然这要是简单类型肯定随便玩，真正复杂起来，肯定还是 xml 文件的 DI，所以注解和 DI 要结合着使用
根据之后的 SpringMVC 三层分层，`@Component`注解衍生出了各自层的注解

- Dao 层：`_@Repository_`
- Controller 层： `@Controller`
- Service 层：`@Service`

四者的作用都是一样的，将对应的类注册进 IOC 容器

### 0x02 使用 JavaConfig 实现配置
Spring 高版本之后的 bean 管理都是通过 javaconfig 类来实现，xml 文件几乎可以不用了
如何指定 JavaConfig 类呢，通过`@Configuration`注解实现
```java
package com.stoocea.Config;


import com.stoocea.Pojo.User;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class StooceaConfig {

    @Bean
    public User getUser(){
        return new User();
    }
}
```
`@Configuration`其实相当于之前配置文件中的`<beans>`标签，而 JavaConfig 类中的`@Bean`注解下面所对应的内容就是相当于`<bean>`，而`@Bean`方法的名字就相当于 IOC 容器中的 id 名

```java
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import com.stoocea.Pojo.*;
import com.stoocea.Config.StooceaConfig;
import javax.management.modelmbean.XMLParseException;

public class Mytest {
    @Test
    public void testUser() {
        ApplicationContext context=new AnnotationConfigApplicationContext(StooceaConfig.class);
        User user=context.getBean("getUser", User.class);
        System.out.println(user);
    }

}
```
也能够正常获取到，并且我们全程没有使用配置文件
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709103117079-fd80aff9-8929-4fb4-b524-f489831749ae.png#averageHue=%233a3f49&clientId=ubaa01bb3-6a62-4&from=paste&height=97&id=u3eb8c03e&originHeight=146&originWidth=549&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=34871&status=done&style=none&taskId=ua1470990-821c-49fb-bc96-000107ecdd6&title=&width=366)

这里我们再多做一个例子
JavaConfig 中注册两个 bean User 和chinesepeople 进 IOC 容器
```java
package com.stoocea.Config;


import com.stoocea.Pojo.ChinesePeople;
import com.stoocea.Pojo.User;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Component;

@Configuration
@ComponentScan("com.stoocea")
public class StooceaConfig {

    @Bean
    public User User(){
        return new User();
    }

    @Bean
    public ChinesePeople chinesepeople(){
        return new ChinesePeople();
    }

}

```
再看一下两者的具体内容：
定义了一个 Chinesepeople 类，然后在 User 类中利用自动专配 `@Autowired`和`@Qualifier`将 people 属性赋值为 chinesepeople 类
```java
package com.stoocea.Pojo;

import org.springframework.stereotype.Component;

@Component
public class ChinesePeople implements People{
    public void say(){
        System.out.println("i am chinese man");
    }
}

```
```java
package com.stoocea.Pojo;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class User {
    @Value("stoocea")
    private String name;

    @Autowired
    @Qualifier(value = "chinesepeople")
    private People people;
    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                '}';
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
    public void say(){
        people.say();
    }

}

```

最后测试一下
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709103737877-24128c61-2a8a-4ff3-a419-00c629f02c01.png#averageHue=%23393f48&clientId=ubaa01bb3-6a62-4&from=paste&height=486&id=u9ba6530a&originHeight=729&originWidth=1567&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=402160&status=done&style=none&taskId=ua3cd6680-9633-478c-ad65-0c61ee44020&title=&width=1044.6666666666667)
其实 User 类和 chinesepeople 类中也可以不写 Component 注解，javaconfig 中就已经将两者给注册了
但其实两者还是有区别的，如果直接在类上加注解 `@Component`,该 bean 的作用域是 prototype，而在 javaconfig 中的`@Bean`注解作用的类，它的作用域默认是 singleton，也就是全局单一


## 7 静态代理和动态代理
这里的动态代理内容师傅们如果不介意可以看我这篇文章 [https://www.yuque.com/5tooc3a/jas/ssz3gd93odxikn0g?singleDoc](https://www.yuque.com/5tooc3a/jas/ssz3gd93odxikn0g?singleDoc#)
这里我再自己总结记录写一下：
不论是静态代理还是动态代理，他们都是一种设计模式，重点是其思想：在不改变原有功能的代码前提下，我们将每个功能点中的共同部分--**比说日期写进日志，与对应的端口建立 TCP 连接等操作**给抽离出来，装进代理类，每当我们调用功能类中的这些方法时，代理类能够在恰当的时间点去执行这些部分

### 0x0S 动态代理
（因为某些深刻的教诲和回忆） 细写和感受一下动态代理
 静态代理，动态代理，他们的服务角色都是一样的：

1. 抽象对象
2. 真实对象

动态代理的类不是我们静态写好的，他是动态生成的，这也是为什么会有静态代理和动态代理的区分
动态代理分为两大类：**基于接口的动态代理  基于类的动态代理**
现在来尝试一个实验
总接口
```java
package com.stoocea.demo1;

public interface Rent {
    void rent();
}

```


某一功能实现类
```java
public class Host implements Rent {
    @Override
    public void rent() {
        System.out.println("正在test。。。。");
    }
}
```

invocationHandler
```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class TestInvocationHandler implements InvocationHandler {
    Rent rent;

    public TestInvocationHandler(Rent rent) {
        this.rent = rent;
    }

    public TestInvocationHandler() {
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object result=method.invoke(rent,args);
        System.out.println("正在经由动态代理实现过程");
        return null;
    }
}
```
最后是测试类，也就是客户端调用
```java
package com.stoocea.demo1;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;

public class Client {
    public static void main(String[] args) {
        Rent host=new Host();
        InvocationHandler hostInvocationHandler=new TestInvocationHandler(host);
        Rent host1=(Rent) Proxy.newProxyInstance(host.getClass().getClassLoader(),host.getClass().getInterfaces(),hostInvocationHandler);
        host1.rent();

    }
}

```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709126456266-7f4ebcb8-831d-4273-a581-368a00b80ad9.png#averageHue=%233d4048&clientId=u21e1b06b-b436-4&from=paste&height=108&id=ua107462b&originHeight=162&originWidth=663&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=29391&status=done&style=none&taskId=ud899493a-76ef-46af-9d90-39b6521f7ea&title=&width=442)
最终我们返回的功能类是`Proxy`调取静态方法 `newProxyInstance`获取到的，之前写的功能类 Host（也就是房东雇主）调用它的方法时，会自动编码跳转到`hostInvocationHandler`的 invoke 中进行逻辑处理，自己的逻辑就不会走了，动态代理的实现都是通过反射实现的，不论是调用雇主的原方法，还是创建代理类，都是通过反射

理解完代理模式，我们进入 AOP 的学习

## 8 AOP（面向切面编程）
先看全英文 Aspect 	Oriented Programming，就是直译过来的，它原理是通过预编译，或者运行动态代理实现程序功能的**统一和维护，**这里的统一和维护就是我们动态代理所讲的功能重复点，我们把它抽离出来。或者是一个新功能，我们需要加到每一个功能生成前，生成中，生成后等
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709126885743-3b4bd708-18be-4bf8-841b-907432583712.png#averageHue=%23f1efee&clientId=u21e1b06b-b436-4&from=paste&height=273&id=u8cf46928&originHeight=409&originWidth=1337&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=335584&status=done&style=none&taskId=ue69a722a-9127-4410-bb41-37cfb8ca146&title=&width=891.3333333333334)
有些很绕的概念就直接贴图了，能理解其真实面目是什么即可

### 0x01 通过 Spring 实现 AOP
首先搭建环境，实现一个 JDBC 业务
```java
package com.stoocea.Dao;

public interface CURD {
    void add();
    void delete();
    void update();
    void select();
}


```

```java
package com.stoocea.Dao;

public class SimpleImpl implements CURD{

    @Override
    public void add() {
        System.out.println("添加了数据");
    }

    @Override
    public void delete() {
        System.out.println("删除了数据");
    }

    @Override
    public void update() {
        System.out.println("更新了数据");
    }

    @Override
    public void select() {
        System.out.println("查询数据");
    }
}


```

现在的需求是要在所有的 CURD 方法之前和方法执行之后，分别打印 before after 字符串，且不修改原始代码
那这里就是定义 AOP 中的常用类了 `AfterReturningAdvice``MethodBeforeAdvice`
两者都是接口，且他们的最大的父类都是 `Advice`，然后下面分了
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709128483080-d240f994-84c1-48aa-92a6-7f09aee6c252.png#averageHue=%233f3227&clientId=u9fbe2d8a-7ae6-4&from=paste&height=21&id=u021ba442&originHeight=31&originWidth=658&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=8843&status=done&style=none&taskId=ubdae22b4-adc7-4d1e-8515-57a43a8ecae&title=&width=438.6666666666667)
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709128492992-a9ace238-00f6-480f-a85b-1f2ac06ac648.png#averageHue=%23547fcf&clientId=u9fbe2d8a-7ae6-4&from=paste&height=25&id=ub0eefc61&originHeight=37&originWidth=605&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=8642&status=done&style=none&taskId=u84d0b00a-9b57-41bb-90e0-e0ecd708170&title=&width=403.3333333333333)
当然还有更多，只不过这两者是上面常用类的父类
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709128526316-de01c548-69a6-49c1-9091-6a7de8003d07.png#averageHue=%23383e47&clientId=u9fbe2d8a-7ae6-4&from=paste&height=341&id=u4bcb286d&originHeight=512&originWidth=1896&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=352887&status=done&style=none&taskId=uabe56908-3533-4f01-892d-51cab49852b&title=&width=1264)
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709128532951-cb35292f-90b5-43c3-b23c-0b36273ae9a5.png#averageHue=%23383d46&clientId=u9fbe2d8a-7ae6-4&from=paste&height=318&id=uf7de4b77&originHeight=477&originWidth=1413&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=255415&status=done&style=none&taskId=u37022575-304b-4cb1-8b05-4346bac5dc0&title=&width=942)
两者都只定义了一个方法，并且他们的参数都是 3 个
MethodBeforeAdvice：

- Method-目标方法
- args-该方法的参数
- target-目标方法的类

AfterReturningAdvice：

- returnValue：方法作用完之后的返回值
- method： 目标方法
- target：目标方法的类

所以我们可以直接取出
```java
package com.stoocea.log;

import org.springframework.aop.AfterReturningAdvice;

import java.lang.reflect.Method;

public class After implements AfterReturningAdvice {

    @Override
    public void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable {
        System.out.println("After.....");
    }
}

```
```java
package com.stoocea.log;

import org.springframework.aop.MethodBeforeAdvice;

import java.lang.reflect.Method;

public class Before implements MethodBeforeAdvice {
    @Override
    public void before(Method method, Object[] args, Object target) throws Throwable {
        System.out.println("Before......");
    }
}



```
然后通过 Spring 去配置一下 AOP
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:p="http://www.springframework.org/schema/p" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
     http://www.springframework.org/schema/context
     http://www.springframework.org/schema/context/spring-context-4.1.xsd
     http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.1.xsd">

    <bean id="after" class="com.stoocea.log.After"/>
    <bean id="before" class="com.stoocea.log.Before"/>
    <bean id="simple" class="com.stoocea.Dao.SimpleImpl"/>

<!--开始配置AOP-->

    <!--打上AOP的约束-->
    <aop:config>
            <aop:pointcut id="pointcut1" expression="execution(* com.stoocea.Dao.SimpleImpl.*(..))"/>
            <aop:advisor advice-ref="before" pointcut-ref="pointcut1"/>
            <aop:advisor advice-ref="after" pointcut-ref="pointcut1"/>
    </aop:config>
</beans>
```
这里要注意打切面的三项，一是必须要有切入点，也就是我们想要切入的类，以及想要切入的方法
然后就是切了之后该干嘛了，这里分为了两种方法，一种是通过环绕切入，也就是我们演示的这个
通过 IOC 给处理类实例化后，通过 `advice-ref="before"`  `advice-ref="after"`来自定是切入前，还是切入后装配
另一种方法就是通过切面（大白话就是一个用来定义 before 方法和 after 方法的类），然后在切面内实现 before 或者 after
```java
package com.stoocea.log;

public class DiyAop {
    public void before(){
        System.out.println("before ，，，，，");
    }

    public void after(){
        System.out.println("after.......");     
    }

}
```
通过切面去实现 AOP 我觉得比较有结构感，相比较原始的来说
```xml

  <!--开始配置AOP-->

  <!--打上AOP的约束-->
  <aop:config>
    <aop:aspect ref="diy">
      <aop:pointcut id="point" expression="execution(* com.stoocea.Dao.SimpleImpl.*(..))"/>
      <aop:before method="before" pointcut-ref="point"/>
      <aop:after method="after" pointcut-ref="point"/>
    </aop:aspect>

  </aop:config>
</beans>
```

### 0x02 通过注解去实现 AOP
我们还是以切面为例，首先是通过注解去实现一个切面类，我们就拿刚才的 DiyAop 切面类为例，现在打上`@Aspect`的注解
```java
package com.stoocea.log;


import org.aspectj.lang.annotation.Aspect;

@Aspect
public class DiyAop {
    public void before(){
        System.out.println("before ，，，，，");
    }

    public void after(){
        System.out.println("after.......");
    }

}

```
然后在 Spring 配置文件中注册这个切面类，我这里全都注册了
```java
    <bean id="after" class="com.stoocea.log.After"/>
    <bean id="before" class="com.stoocea.log.Before"/>
    <bean id="simple" class="com.stoocea.Dao.SimpleImpl"/>
    <bean id="diy" class="com.stoocea.log.DiyAop"/>
```
然后开启注解扫描
```java
<aop:aspectj-autoproxy/>
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709170673101-dfc7204d-be86-490a-95d4-3587c8da0aac.png#averageHue=%23393e47&clientId=u91f4515e-7315-4&from=paste&height=236&id=u0015dd04&originHeight=354&originWidth=993&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=127833&status=done&style=none&taskId=ueb156de0-1318-446e-8ab2-663e59f6901&title=&width=662)
然后测试类
```java
import com.stoocea.Dao.CURD;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Mytest {
    public static void main(String[] args) {
        ApplicationContext context=new ClassPathXmlApplicationContext("beans.xml");
        CURD curdImpl= context.getBean("simple", CURD.class);
        curdImpl.add();
    }
}
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709170806726-24115935-f4cf-406e-92bc-d6f3f691d4e0.png#averageHue=%233b3e46&clientId=u91f4515e-7315-4&from=paste&height=88&id=u66048ef6&originHeight=132&originWidth=793&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=24449&status=done&style=none&taskId=u3deb54f2-3be7-4f7c-b85f-068091ecdcf&title=&width=528.6666666666666)
我们刚才学到的用环绕切面去实现 AOP，也能够通过注解实现，作用点还是`DiyAop` 
```java
import org.aspectj.apache.bcel.classfile.Signature;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class DiyAop {

    @Before("execution(* com.stoocea.Dao.SimpleImpl.*(..))")
    public void before(){
        System.out.println("before ，，，，，");
    }


    @After("execution(* com.stoocea.Dao.SimpleImpl.*(..))")
    public void after(){
        System.out.println("after.......");
    }

    @Around("execution(* com.stoocea.Dao.SimpleImpl.*(..))")
    public void around(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("环绕前");

        Object proceed=pjp.proceed();

        System.out.println("环绕后");

    }

}
```
这里的 `ProceedingJoinPoint`它代表当前的作用点，我们能够通过它来获取到很多信息，这里就不展示了
然后依然是测试类看看
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709171264481-d96ca032-a71a-48db-b7d6-5d648e3de3d8.png#averageHue=%233c3e46&clientId=u91f4515e-7315-4&from=paste&height=139&id=u57b54683&originHeight=208&originWidth=517&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=27506&status=done&style=none&taskId=uc27a42e6-6b0c-4389-9d7c-81c23833342&title=&width=344.6666666666667)
说明先是环绕的执行，然后等切入点开始执行的时候，切面的 before 和 after 开始运作，最后整个流程结束之后再进环绕面执行环绕后

## 9 Spring 整合 Mybatis
Spring 的最后一步，顺便回顾一下 mybatis 的知识

### 0x01 环境搭建
```java
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.example</groupId>
  <artifactId>SpringRe0</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>pom</packaging>
  <name>Archetype - SpringRe0</name>
  <url>http://maven.apache.org</url>
  <modules>
    <module>Spring-01</module>
    <module>Spring-02-HelloSpring</module>
    <module>Spring-03-Anno</module>
    <module>Spring-04-AOPpre</module>
    <module>Spring-05-Conmybatis</module>
  </modules>


  <dependencies>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-core -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>6.0.4</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-beans -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-beans</artifactId>
      <version>6.0.4</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>6.0.4</version>
    </dependency>

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

    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjrt</artifactId>
      <version>1.9.19</version>
    </dependency>
  </dependencies>


</project>

```
```properties
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis_study?useSSL=true&seUnicode=true&characterEncoding=UTF-8
username=root
password=123456
```
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <properties resource="db.properties">
    <property name="useless" value="useless"/>
  </properties>
  <settings>
    <setting name="logImpl" value="STDOUT_LOGGING"/>
  </settings>
  <typeAliases>
    <package name="com.stoocea.Pojo"/>
  </typeAliases>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"></transactionManager>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"></property>
        <property name="url" value="${url}"></property>
        <property name="username" value="${username}"></property>
        <property name="password" value="${password}"></property>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="com/stoocea/Mapper/UserMapper.xml"/>
  </mappers>
</configuration>
```
UserMapper 接口
```java
package com.stoocea.Mapper;
import com.stoocea.Pojo.*;

import java.util.*;

public interface UserMapper {
    List<User> getuserlist();
    //根据ID返回数据
    User getuserbyname(String name);
    //添加用户
    Boolean adduser(User user);
    //删除用户
    Boolean deluser(int id);
    //更新用户
    Boolean cguser(User user);
    //添加用户Map
    Boolean mapadduser(Map<String,Object> map);
    //模糊查询
    User getuserbylike(Map<String,Object> map);
}

```
UserMapper xml 文件
```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.stoocea.Mapper.UserMapper">
    <select id="getuserlist" resultType="User">
        select * from users
    </select>
    <!--补充一个getuserbyid的select标签-->
    <select id="getuserbyname" resultType="User" parameterType="String">
        select * from users where username=#{name}
    </select>
    <insert id="adduser" parameterType="User">
        insert into mybatis.users(id,username,pwd) values(#{id},#{name},#{pwd});
    </insert>
    <delete id="deluser" parameterType="int">
        delete from users where id=#{id}
    </delete>
    <update id="cguser" parameterType="User">
        update users set username=#{name},pwd=#{pwd} where id=#{id}
    </update>
    <insert id="mapadduser" parameterType="Map">
        insert into users(id,username) values(#{id},#{name})
    </insert>
    <select id="getuserbylike" parameterType="Map" resultType="User">
        select * from users where username like concat("%",#{name},"%")
    </select>
</mapper>
```

获取 mybatis  sqlssession 的工具类
```java
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

public class MybatisUtil {
    private static SqlSessionFactory sqlSessionFactory;    //提升作用域
    static {
        try {
            //获取sqlsessionFactory对象 这一步是必须的
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
    //既然有了 SqlSessionFactory，顾名思义，我们可以从中获得 SqlSession 的实例。SqlSession 提供了在数据库执行 SQL 命令所需的所有方法
    public static SqlSession getSqlsession(){
        return sqlSessionFactory.openSession(true);
    }
}
```
User 类就不贴了
最后是测试类
```java
import com.stoocea.Mapper.UserMapper;
import com.stoocea.Pojo.*;
import com.stoocea.Utils.MybatisUtil;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.junit.Test;

import java.util.*;

public class Mytest {

    @Test
    public void testMybatis(){
        SqlSession sqlSession= MybatisUtil.getSqlsession();
        UserMapper userMapper=sqlSession.getMapper(UserMapper.class);
        List<User> userList=userMapper.getuserlist();
        for (User user : userList) {
            System.out.println(user);
        }
    }
}

```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709174628928-22b8a4ae-c648-493e-b244-02a82af46d78.png#averageHue=%233e434c&clientId=u5673df07-82fd-4&from=paste&height=108&id=u22311661&originHeight=162&originWidth=762&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=73801&status=done&style=none&taskId=u39dde4f4-6e26-4b53-bcb7-cd963809a8b&title=&width=508)
能查询结果就没啥问题了
### 0x02 Spring 整合 Mybatis
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
  <!--    注意这里-->
  <!--<context:component-scan base-package="com.stoocea.mapper"/?-->
  <!--Datasource:使用spring的数据源替换mybatis的配置,选一个连接池 c3p0，jdbc，druid
  这里我们使用spring提供的jdbc：org.springframework.jdbc.datasource
  -->
  <context:property-placeholder location="db.properties"/>
  <bean id="datasource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="${driver}"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mybatis_study?useSSL=true&amp;seUnicode=true&amp;characterEncoding=UTF-8"/>
    <property name="username" value="root"/>
    <property name="password" value="123456"/>
  </bean>
  <!-- 获取sqlsessionfactory-->
  <bean id="sqlsessionfactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!--绑定mybatis配置文件-->
    <property name="dataSource" ref="datasource"/>
    <property name="configLocation" value="classpath:mybatis-config.xml"/>
    <property name="mapperLocations" value="classpath:com/stoocea/Mapper/*.xml"/>
  </bean>

  <bean class="org.mybatis.spring.SqlSessionTemplate" id="sqlsession">
    <!--只能使用构造器创建，因为没有set方法-->
    <constructor-arg index="0" ref="sqlsessionfactory"/>
  </bean>
  <bean id="UserMapper" class="com.stoocea.Mapper.UserMapperImpl">
    <property name="sqlSessionTemplate" ref="sqlsession"/>
  </bean>
</beans>
```
```java
package com.stoocea.Mapper;

import com.stoocea.Pojo.User;
import org.mybatis.spring.SqlSessionTemplate;

import java.util.List;
import java.util.Map;

public class UserMapperImpl implements UserMapper{
    private SqlSessionTemplate sqlSessionTemplate;

    public void setSqlSessionTemplate(SqlSessionTemplate sqlSessionTemplate) {
        this.sqlSessionTemplate = sqlSessionTemplate;
    }


    @Override
    public List<User> getuserlist() {
        UserMapper userMapper=sqlSessionTemplate.getMapper(UserMapper.class);
        return userMapper.getuserlist();
    }

    @Override
    public User getuserbyname(String name) {
        return null;
    }

    @Override
    public Boolean adduser(User user) {
        return null;
    }

    @Override
    public Boolean deluser(int id) {
        return null;
    }

    @Override
    public Boolean cguser(User user) {
        return null;
    }

    @Override
    public Boolean mapadduser(Map<String, Object> map) {
        return null;
    }

    @Override
    public User getuserbylike(Map<String, Object> map) {
        return null;
    }
}

```
