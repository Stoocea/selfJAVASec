SSM 框架的最后一步（踩坑崩溃中）
![590a4a7060113b9e0985812d2a95daed.jpg](https://cdn.nlark.com/yuque/0/2024/jpeg/36078896/1709365593728-6eac1350-6c67-485b-a398-945db99eee66.jpeg#averageHue=%23fdfefe&clientId=uf6355659-396c-4&from=paste&height=204&id=u5765d8a0&originHeight=306&originWidth=306&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=17117&status=done&style=none&taskId=u32ee944f-8963-4c8e-be84-16007f19ad0&title=&width=204)
## 1 MVC 是什么
首先它并不是一个实体，只是一个设计模式，全称** Model（模型） View（视图） Controller（控制器）**

- Model：需要操纵的数据以及信息，对应我们之前介绍到的 Dao，service
- View：客户能够见到的界面，或者说一层结构，也就是前端的一些东西
- Controller：接收 View 层客户发过来的消息，然后根据其请求去调用相对应的 model 层

那么直观起来就是：**View->Controller->Model**，在 SpringMVC 中就是 JSP->Servlet->beans

## 2.回顾 Servlet
先把环境搭建好，maven 依赖导入
servlet jsp jstl 都是能够想到的相关的依赖
```xml
<dependencies>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>3.8.1</version>
    <scope>test</scope>
  </dependency>
  <dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
  </dependency>
  <!-- https://mvnrepository.com/artifact/javax.servlet.jsp/javax.servlet.jsp-api -->
  <dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>javax.servlet.jsp-api</artifactId>
    <version>2.3.3</version>
  </dependency>
  <dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2  </version>
  </dependency>
  <dependency>
    <groupId>taglibs</groupId>
    <artifactId>standard</artifactId>
    <version>1.1.2</version>
  </dependency>
</dependencies>
```
然后写一个 hello 路由的 servlet，配置一下 webxml，写一个用来回显的 hellojsp
```java
package com.stoocea.Servlet;
import javax.servlet.*;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String method=req.getParameter("method");
        if (method.equals("add")) {
            req.getSession().setAttribute("msg","执行了add方法");
        }
        if(method.equals("delete")){
            req.getSession().setAttribute("msg","执行delete");
        }

        req.getRequestDispatcher("/hello.jsp").forward(req,resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}

```

```xml
<!DOCTYPE web-app PUBLIC
"-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
"http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>


  <servlet>
    <servlet-name>hello</servlet-name>
    <servlet-class>com.stoocea.Servlet.HelloServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>/hello</url-pattern>
  </servlet-mapping>

</web-app>

```

这里记得把 EL 表达式的开关打开
```xml

<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page isELIgnored="false" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
${msg}
</body>
</html>

```
整体的流程大致为：

1. 将 URL 映射到 java 类或者 java 类的方法
2. 封装用户的数据
3. 处理请求，调用相关的业务逻辑 ----封装响应数据
4. 将响应的数据回显到 jsp 或者 html 等表示层的数据

常见的服务器端的 MVC 框架：struts，SpringMVC，ASP.NET 等，所以学 MVC 框架在后续的安全学习中是很有益处的（

## 3 初步认识 SpringMVC
我觉得首先的一点要理解分层的思想，当用户发送请求时，后端服务并不是单一一层来完成数据接收、调用相关业务逻辑、回显这几个步骤的，解耦的重要性一直在 Spring 的体现，便于管理，便于优化和演进。
之前学习 servlet 的时候，用户每有一个不同类型的请求，我们就必须多加一个 servlet，很麻烦，SpringMVC 进一步把这种分发调度的工作抽离，单独隔离出来一层 -----**DispatchServlet**
接下来我们简单实现以下 DispatchServlet 思想下的 web 服务（并不是真实业务情况下）
现在开始正式导入 SpringMVC 相关的包
```xml
<dependencies>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.1.9.RELEASE</version>
  </dependency>
  <dependency>
    <groupId>jakarta.servlet</groupId>
    <artifactId>jakarta.servlet-api</artifactId>
    <version>4.0.3</version>
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
</dependencies>
```

然后配置一下 web.xml 文件，这里我们注册了一个 servlet，是包里面自带的，也就是我们刚才上面理论分析时提到的一个很重要的类   `DispatcherServlet` ，他的详细设置依赖于一个 spring 配置文件，之后会有详细内容，然后给他的路由是分配的所有路由下
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    
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
</web-app>
```

然后是对 `DispatcherServlet`单独的 spring 配置文件，上面三个注册的 bean 是固定的，也就是视图和映射处理器
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--        添加处理映射器-->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
    <!--    添加处理适配器-->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
    <!--    添加视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
        <!--        前缀-->
        <property name="prefix" value="/jsp/"/>
        <!--        后缀-->
        <property name="suffix" value=".jsp"/>
    </bean>
    <!--    Handler-->
    <bean id="/hello" class="com.stoocea.Controller.HelloController"/>
</beans>
```
只需要注意 `<property>`标签，结合下面这个 controller 理解：当我们访问 /hello 路由时，他会来到这层 controller 进行逻辑处理，然后这层 controller 主要是通过 ModelAndView 类来设置模板渲染信息
然后给路由拼接一段 hello 字段 结合上面的前缀后缀就是 /jsp/hello.jsp
```java

import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class HelloController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        ModelAndView mv=new ModelAndView();
        mv.addObject("msg","success");
        mv.setViewName("hello");
        return mv;
    }
}

```
也就是定位到静态资源 jsp 文件夹下的 hello.jsp
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709356202410-4a7439da-0cf0-4a6d-942e-eb35c4f5f710.png#averageHue=%233b4145&clientId=uf6355659-396c-4&from=paste&height=408&id=u1b720357&originHeight=612&originWidth=635&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=106952&status=done&style=none&taskId=u171a6638-20ff-4694-9131-d230d6ab713&title=&width=423.3333333333333)
```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page isELIgnored="false" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
${msg}
</body>
</html>
```

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709356244122-34b096cd-941a-46c6-8a5c-c6d5cceaafc2.png#averageHue=%239f9e9e&clientId=uf6355659-396c-4&from=paste&height=165&id=u28237a09&originHeight=248&originWidth=730&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=16749&status=done&style=none&taskId=u228a0d3e-46d1-499b-b5ca-020a86613df&title=&width=486.6666666666667)

### 0x01 踩坑记录
首先就是版本问题，依赖的版本尽量不要太高
然后是 idea 本身的问题，能够不用 23.1 版本就不用，或者说能够用旧版就用旧版
（踩了两天坑倒是对 idea 怎么构建 web 项目的大致思路有了一个脉络）
解决 SSM 以及 Springboot 这块的问题时，有几个切入点：环境依赖是否版本有问题（过高，过低）
idea 是否存在一些 NC 问题？（这一个是必须要了解的，必须对自己手中的 idea 要有一个全面了解）
## 4.SpingMVC 运行流程
SpringMVC 的设计是围绕着 `DispatcherServlet`来进行的，我们可以看这张图 
![20180522162744760.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709431796433-fc555bfe-d4fa-4203-a47e-aaa6492e306e.png#averageHue=%23f9f9f9&clientId=uac3d13ec-7ba5-4&from=paste&height=454&id=u10aa49ba&originHeight=567&originWidth=1091&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=15961&status=done&style=none&taskId=u9d545fa0-5a0e-4e86-bb43-08da2c5c51b&title=&width=872.8)
总结一句话就是，`DispatchServlet` 会拦截用户的所有请求，然后由它开始，向 `HandlerMapping` 查找匹配的 `handler`，然后去调适配的 `handlerAdapter`，由 `handlerAdapter`去调 `Controller`，`Controller` 会先调用业务逻辑，然后将返回值返回给  `handlerAdapter`，比如 `ModelAndView `，然后 DispatchServlet 接收这些结果数据，发送过 `viewresolver `去处理，dispatchservlet 将其处理结果接收，最终返回给用户
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709432599041-d9c44fd4-4692-4c18-b3e3-e435bf699934.png#averageHue=%23313438&clientId=uac3d13ec-7ba5-4&from=paste&height=333&id=u7b16406e&originHeight=500&originWidth=1645&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=332484&status=done&style=none&taskId=u23a51d39-39cd-42bd-94d3-8901b1078ea&title=&width=1096.6666666666667)
这也就是为什么刚才的 spring 配置文件要配置这些类了，我们真正要去自己实现的，只有实现 controller 去调用业务层的逻辑

## 5 使用注解开发 SpringMVC
web.xml 文件的内容依然不变，但是 springmvc 的配置文件需要修改一下
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:mvc="http://www.springframework.org/schema/mvc"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">


  <context:component-scan base-package="com.stoocea.Controller"/>
  <mvc:default-servlet-handler />
  <mvc:annotation-driven />
  <!--    添加视图解析器-->
  <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
    <!--        前缀-->
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <!--        后缀-->
    <property name="suffix" value=".jsp"/>
  </bean>
  <!--    Handler-->

</beans> 
```
有两个新的注意点 `<mvc:default-servlet-handler />` 它是 mvc 的资源过滤器，因为我们之后的试图解析器会扫描静态资源，有些后缀格式如 mp4 等资源是不能直接解析的，所以必须要有静态资源过滤
然后是`<mvc:annotation-driven />` 它是 mvc 的默认注解引擎，有了它就不用去配置 `HandlerMapping`和`HandlerAdapter` 了，之后注解会帮我们解决这个事情
相应的，Controller 的内容也会有变化
```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/first")
public class HelloController  {

        @RequestMapping("/hello")
        public String hello(Model model){
                model.addAttribute("msg","success MVC");
                return "hello";

        }
}
```
在 SpringMVC 的配置文件中我们已经指定了使用注解引擎，那么 ModelAndView 这类较底层的逻辑我们就不要去实现了，直接用注解

- 路由设置---`@RequestMapping`
- `@Controller` 用来被 mvc 配置文件中的包扫描器识别，认定为 controller

具体的内容返回改为使用 `Model`，且方法 `public String hello(Model model)`的返回值会返回给视图 View，代表上面的  `mv.setViewName("hello");`用以拼接路由，去指定访问 /WEB-`INF/jsp/hello.jsp`的静态资源
然后有一个注意点，我们在类和方法上都加上了 ` @RequestMapping`的注解，类上>方法，所以待会访问的是`first/hello`路由
然后我们开启一下 tomcat，访问一下对应的路由
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709444541049-9ebf9d55-a78f-4c3f-a14e-6f754c7f5c60.png#averageHue=%23979696&clientId=uac3d13ec-7ba5-4&from=paste&height=155&id=u6b98d5f2&originHeight=233&originWidth=896&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=20065&status=done&style=none&taskId=ud655748f-9ee8-4fa5-8faf-f656efc91fc&title=&width=597.3333333333334)
springMVC 的三件套是： `handlerAdapter` `HandlerMapping` `ViewResolver` 但是注解开发身躯了前面两个，SpringMVC 的配置文件只需要写 viewResolver 即可

## 6 Controller 配置总结
依然还是这块代码，这是我们最开始探究 SpringMVC 的运行流程所用到的 controller
```java

import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class HelloController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        ModelAndView mv=new ModelAndView();
        mv.addObject("msg","success");
        mv.setViewName("hello");
        return mv;
    }
}
```
比较的原始，主要是问题就是它一个 controller 只能写一个方法，也就是处理一种请求的可能，而且相绑定的 mvc 配置文件也要多很多内容
第二种就是我们刚才使用注解去开发的 springmvc
```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/first")
public class HelloController  {

        @RequestMapping("/hello")
        public String hello(Model model){
                model.addAttribute("msg","success MVC");
                return "hello";

        }
}
```
不仅配置简单，而且一个 controller 里面就能够写很多路由访问处理逻辑，很具有 spring 注解开发的特色
但是注意：
配置文件这块，必须要打上注解支持以及注解扫描器去定位到我们的 controller
```xml
<context:component-scan base-package="com.stoocea.Controller"/>
<mvc:default-servlet-handler />
<mvc:annotation-driven />
```

## 7 RestFul 风格
主要是安全，隐藏参数，让我们很难猜到后台到底干了那些事情
其实说是学习这种风格，不如说是学习注解
```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class TestController {


    @RequestMapping("/test/{a}/{b}")
    public String test(@PathVariable String a, @PathVariable String b, Model model) {
        String c=a + b;
        model.addAttribute("msg",c);
        return "hello";
    }

}
```
`@PathVariable`的作用就是直接取路由作为值，然后进行处理，当然有着类型的限定
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709449797116-69cb3469-c4de-466e-bf44-fab76724920a.png#averageHue=%23a8a7a7&clientId=uac3d13ec-7ba5-4&from=paste&height=167&id=uf2fe8e38&originHeight=250&originWidth=971&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=22545&status=done&style=none&taskId=ucd44777d-f2af-4032-8320-cc34567bca3&title=&width=647.3333333333334)

RequestMapping 还有一个限定的属性值 `method = `他是用来限定请求方式的，这样不仅做到一些安全的设定，也能够实现同一路由却有不同的请求方式的结果
```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@Controller
public class TestController {


    @RequestMapping(name="/test/{a}/{b}",method = RequestMethod.POST)
    public String test(@PathVariable String a, @PathVariable String b, Model model) {
         String c=a + b;
         model.addAttribute("msg",c);
         return "hello";
    }

}
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709449917734-400c27fe-d8fe-4764-a30a-cab3fc4755d8.png#averageHue=%23b2b1b0&clientId=uac3d13ec-7ba5-4&from=paste&height=229&id=u68444d48&originHeight=343&originWidth=1113&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=44372&status=done&style=none&taskId=ue33d1ab2-6fb4-49de-b0e8-ba1cc95a79a&title=&width=742)

但其实一般 RequestMapping 不这么写，太长了，很慢
```java

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
public class TestController {


    //@RequestMapping(name="/test/{a}/{b}",method = RequestMethod.POST)
    @GetMapping("/test/{a}/{b}")
    public String test(@PathVariable String a, @PathVariable String b, Model model) {
         String c=a + b;
         model.addAttribute("msg",c);
         return "hello";
    }

}

```

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709450343804-ac7cddc8-1de4-412f-9230-b5ce8f31050b.png#averageHue=%23898888&clientId=uac3d13ec-7ba5-4&from=paste&height=141&id=isCcE&originHeight=211&originWidth=903&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=20242&status=done&style=none&taskId=ua35b8ff6-c963-43cf-910a-f0dfb454059&title=&width=602)
有哪种方法的限定，就有某种方法的 Mapping
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709450426519-65573853-600e-4dc5-8cf9-fa2fe3747a3f.png#averageHue=%233a3f47&clientId=uac3d13ec-7ba5-4&from=paste&height=322&id=ua888bfd6&originHeight=483&originWidth=697&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=107733&status=done&style=none&taskId=u8a3beb47-1806-465f-85d1-5365d318857&title=&width=464.6666666666667)

```java
@GetMapping
@POSTMapping
@DELETEMapping
......
```

## 8 转发和重定向
spring 中的试图解析器底层调用的是 servlet 的应用，转发和重定向都能够通过 servlet 中熟悉的操作去实现

```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
public class TestController {

    //@RequestMapping(name="/test/{a}/{b}",method = RequestMethod.POST)
    @GetMapping("/test/{a}/{b}")
    public String test(@PathVariable String a, @PathVariable String b, Model model) {
         String c=a + b;
         model.addAttribute("msg",c);
         return "redirect:/index.jsp";
    }

}
```

```java

<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<body>

<h1>喜报！你被重定向了！</h1>
<h2>Hello World!</h2>
</body>
</html>
```
我们依然是去访问 /test/1/2,发现被重定向
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709451987595-0eff35a2-b439-4ed0-a7d3-5e2f84ab1ce6.png#averageHue=%23a9a9a8&clientId=uac3d13ec-7ba5-4&from=paste&height=200&id=ub527160e&originHeight=300&originWidth=975&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=33367&status=done&style=none&taskId=u3cfdc6b4-8f6e-4f2d-bbe9-03990924d37&title=&width=650)
当我们限定 redirect 的时候，在 servlet 中他就会识别出来是重定向的操作，就不去拼接试图解析器 viewHandler 中的前后缀了，直接就访问到指定的目录下

## 10 接收请求和数据回显
两种情况，当我们接收前端数据为普通类型，字符串，int 等
当我们接收前端数据为一个对象时，比如说下面这个 User
```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private int id;
    private String name;
    private  String password;

}
```
然后写一下后端的接收参数
```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
public class TestController {

    //@RequestMapping(name="/test/{a}/{b}",method = RequestMethod.POST)
    @GetMapping("/test/{a}/{b}")
    public String test(@PathVariable String a, @PathVariable String b, Model model) {
         String c=a + b;
         model.addAttribute("msg",c);
         return "redirect:/index.jsp";
    }

    @GetMapping("/t1")
    public String test3(@RequestParam("username") String name,Model model){
        model.addAttribute("msg",name);
        return "hello";
    }


}
```

访问 t1 路由，get 传参一个 username 参数
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709453062388-d52a6327-0ba7-4424-89a3-f03f08e174f5.png#averageHue=%23a7a6a6&clientId=u868e15f1-17c5-4&from=paste&height=162&id=u075133d6&originHeight=243&originWidth=950&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=20392&status=done&style=none&taskId=uf7be545d-64b6-47e5-956b-372ab7bc0c7&title=&width=633.3333333333334)
其实还是在学习注解的使用， `@RequestParam`的意义在于我们能够快速了解当前路由接收参数是什么，然后前端的该如和传输也是他来定
然后是传递一个对象改如何处理
```java
import com.stoocea.Pojo.User;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
public class TestController {

    //@RequestMapping(name="/test/{a}/{b}",method = RequestMethod.POST)
    @GetMapping("/test/{a}/{b}")
    public String test(@PathVariable String a, @PathVariable String b, Model model) {
         String c=a + b;
         model.addAttribute("msg",c);
         return "redirect:/index.jsp";
    }

    @GetMapping("/t1")
    public String test3(@RequestParam("username") String name,Model model){
        model.addAttribute("msg",name);
        return "hello";
    }

    @GetMapping("/t3")
    public String test4(User user,Model model){
        model.addAttribute("msg",user);
        return "hello";
    }
    
}
```
访问 t3 路由，也是直接打印出 t3，但是这里我没有重写 tostring 方法，所以直接打印全类名和 hash 值了
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709453261347-7d4673dc-a162-43a8-bf61-70e6d344af88.png#averageHue=%23717070&clientId=u868e15f1-17c5-4&from=paste&height=125&id=ub685ee2d&originHeight=187&originWidth=673&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=19661&status=done&style=none&taskId=ueefc434c-b892-41dd-8978-9e84be21b4f&title=&width=448.6666666666667)


那我们接下来看下一个例子，同时重写一下 toString 方法，把属性值都打印出来
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709453456158-b5eb0efa-f532-4826-889b-f39218b66c3a.png#averageHue=%233a3f48&clientId=u868e15f1-17c5-4&from=paste&height=225&id=ubd65a4a3&originHeight=338&originWidth=631&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=83648&status=done&style=none&taskId=u0e5fd826-e632-47b6-9759-0d373286cff&title=&width=420.6666666666667)

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709453387772-fe5a71ed-9db9-4c6a-ba89-70c52c9259fd.png#averageHue=%23989897&clientId=u868e15f1-17c5-4&from=paste&height=159&id=u17a7ae7a&originHeight=239&originWidth=1143&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=28804&status=done&style=none&taskId=u8de23d4b-af7b-4bbe-88d4-403563b8fa7&title=&width=762)

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709453400459-5769f8f2-ede3-45ce-a64a-a96687a78c68.png#averageHue=%23979696&clientId=u868e15f1-17c5-4&from=paste&height=158&id=u676fe8d6&originHeight=237&originWidth=1158&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=29183&status=done&style=none&taskId=ub7e86bfd-d57e-4564-8964-7ae9a9ec016&title=&width=772)
传参的时候，参数名还是得跟属性值相同才能接收的到


## 11 JSON
### 0x01 Jackson
#### 1x01 Jackson 的基本使用
JSON 数据是什么就不多提了，用于给前后端传递数据的一种纯文本格式
先把 jackson 的依赖导入
```xml
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>
  <version>2.9.8</version>
</dependency>
```
然后写个用来测试 pojo 类和测试方法
```java
package com.stoocea.Pojo;



public class User {
    private int id;
    private String name;
    private  String password;

    public User() {
    }

    public User(int id, String name, String password) {
        this.id = id;
        this.name = name;
        this.password = password;
    }



    public int getId() {
        return id;
    }

    @Override
    public String toString() {
        return "User{" +
        "id=" + id +
        ", name='" + name + '\'' +
        ", password='" + password + '\'' +
        '}';
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

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }



}

```

```java
    @GetMapping("/j1")
    @ResponseBody
    public String jsontest1() throws Exception{
        User user=new User(1,"stoocea","asdbsdadasd");
        ObjectMapper objectMapper=new ObjectMapper();
        String result=objectMapper.writeValueAsString(user);
        return result;
    }
```
这里的`@ResponseBody`注解使我们当前这个方法的返回结果不会去走视图解析器 viewHandler 的逻辑，而直接 return 结果到 web 页面上，也就是说我们访问/j1 路由得到的结果直接就是 result
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709455900241-edb31445-af25-4e47-87e0-aa1aa0cd724b.png#averageHue=%23848483&clientId=u313c9f22-f48e-4&from=paste&height=137&id=u3d186dc6&originHeight=205&originWidth=718&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=20109&status=done&style=none&taskId=u9a742eed-bd64-4c7c-a159-ff2d5565d54&title=&width=478.6666666666667)

然后 json 字符串的解决中文乱码的问题

有一种方法是通过 `@RequestMapping` 里面的 produces 参数，设置返回包的格式，然后设置 charset 为 utf-8
```java
    @RequestMapping(value = "/j1",produces = "application/json;charset=utf-8")
    @ResponseBody
    public String jsontest1() throws Exception{
        User user=new User(1,"stoocea","你好你好");
        ObjectMapper objectMapper=new ObjectMapper();
        String result=objectMapper.writeValueAsString(user);
        return result;
    }
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709456072582-793fafe2-4844-4b81-bfe5-9fe030ad07b3.png#averageHue=%231f3c55&clientId=u313c9f22-f48e-4&from=paste&height=251&id=u2b1a9500&originHeight=376&originWidth=791&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=25607&status=done&style=none&taskId=u6d4c3298-7438-406f-beb1-e0f621c8ed3&title=&width=527.3333333333334)
mvc 有一种以逸待劳的方法，在 Spring 配置文件中加入如下内容
```java
<mvc:annotation-driven>
    <mvc:message-converters register-defaults="true">
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">
            <property name="supportedMediaTypes">
                <list>
                    <value>text/html;charset=UTF-8</value>
                    <value>application/json;charset=UTF-8</value>
                    <value>text/plain;charset=UTF-8</value>
                    <value>application/xml;charset=UTF-8</value>
                </list>
            </property>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```
或者说是直接在 controller 类上加上 `@RestController`注解，也就是说给所有的方法前都加上一段 `@ResponseBody`只返回结果，不进入视图解析器的逻辑
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709467586076-2a0c740c-8183-4398-bdae-69fe655610e0.png#averageHue=%239b9a9a&clientId=u313c9f22-f48e-4&from=paste&height=169&id=ubfc122ca&originHeight=253&originWidth=663&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=20040&status=done&style=none&taskId=u70476239-becc-4706-807d-0b6aca39f36&title=&width=442)
如果我们有多个对象需要传递，可以采用 List 的形式
```java
@RequestMapping(name="/j1",produces="text/html;charset=utf-8")
    @ResponseBody
    public String jsontest1() throws Exception{
        ObjectMapper objectMapper=new ObjectMapper();
        User user1=new User(1,"极","dadad");
        User user2=new User(1,"极","dadad");
        User user3=new User(1,"极","dadad");
        User user4=new User(1,"极","dadad");
        List<User> userList=new ArrayList<User>();
        userList.add(user1);
        userList.add(user2);
        userList.add(user3);
        userList.add(user4);
        String result=objectMapper.writeValueAsString(userList);
        return result;
    }
```

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709467875308-3689aea6-d798-47c7-8cfb-d5eb15d80f26.png#averageHue=%23d1d0cf&clientId=u313c9f22-f48e-4&from=paste&height=626&id=uccfad065&originHeight=939&originWidth=2529&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=188924&status=done&style=none&taskId=u950b8acb-1e7d-4cb4-afcc-da47c153b27&title=&width=1686)
其实可以写一个关于 JSON 的工具类---jsonUtils，这里主要是对时间戳等格式进行格式化
```java
package com.stoocea.Utils;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.SimpleTimeZone;

public class JsonUtils {

    public String getJson(Object o) throws JsonProcessingException {
        return getJson(o,"yyyy-MM-dd HH-mm-ss");
    }

    public String getJson(Object o,String Dateformat) throws JsonProcessingException {
        ObjectMapper objectMapper=new ObjectMapper();
        objectMapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS,false);
        SimpleDateFormat sdf=new SimpleDateFormat(Dateformat);
        objectMapper.setDateFormat(sdf);
        return objectMapper.writeValueAsString(o);
    }
}

```

```java
    @RequestMapping("/j3")
    @ResponseBody
    public String json3() throws Exception{
        Date date=new Date();
        return new JsonUtils().getJson(date);
    }
```

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709469431379-c2615810-75f2-45cb-af44-9da39382de87.png#averageHue=%23a4a4a3&clientId=u313c9f22-f48e-4&from=paste&height=159&id=u647e95cc&originHeight=238&originWidth=827&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=21909&status=done&style=none&taskId=ud522f1ec-05fa-414c-9379-435c2c8d671&title=&width=551.3333333333334)

### 0x02 fastjson
这里再过一遍基础，按捺分析源码调链子的冲动。。。。。
先导入依赖，用 2.0 版本的 fastjson
```xml
<dependency>
  <groupId>com.alibaba.fastjson2</groupId>
  <artifactId>fastjson2</artifactId>
  <version>2.0.7</version>
</dependency>
```
然后就可以开始直接用了
```java
@RequestMapping("/j3")
@ResponseBody
public String json3() throws Exception{
Date date=new Date();
return JSON.toJSONString(date);
}

```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709471558112-89fedd12-b675-4065-833d-12804e4f267c.png#averageHue=%23aeadac&clientId=ue040ba73-270e-4&from=paste&height=187&id=u6979d958&originHeight=280&originWidth=692&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=20544&status=done&style=none&taskId=u55d7b1c9-87d2-41a3-bdfd-584b9c6aa6d&title=&width=461.3333333333333)
够 fast 了吧
fastjson 使用起来还是挺方便的，并且里面的利用方法都是挺清晰的
包括：
java 对象转 JSON 对象 ，JSON 字符串对象转 java 对象 
java 对象转 JSON 字符串，JSON 字符串转 java 对象
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709471706752-d51714f0-debd-4ffa-85ca-9f52fa7f5f89.png#averageHue=%23383e46&clientId=ue040ba73-270e-4&from=paste&height=499&id=uf7c48cf5&originHeight=1077&originWidth=877&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=563769&status=done&style=none&taskId=u39fa1e10-3f08-456f-8c30-073ae4ec1d8&title=&width=406.66668701171875)


## 12 SSM 整合
Spring SpringMVC Mybatis 的一次联合运用复习
### 0x01 环境搭建

#### 1x01 Mapper 层（Dao 层）
Mapper 层主要就是实现 CRUD ，关于数据库的配置，连接，操作等
依赖
```xml
<dependencies>
  <!--数据库驱动  -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.47</version>
    </dependency>
  <!--数据库连接池  -->
    <dependency>
      <groupId>com.mchange</groupId>
      <artifactId>c3p0</artifactId>
      <version>0.9.5.2</version>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
    </dependency>
  <!--mvc  -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.1.9.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>5.1.9.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>jakarta.servlet</groupId>
      <artifactId>jakarta.servlet-api</artifactId>
      <version>4.0.3</version>
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
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.9.8</version>
    </dependency>
    <dependency>
      <groupId>com.alibaba.fastjson2</groupId>
      <artifactId>fastjson2</artifactId>
      <version>2.0.7</version>
    </dependency>

  <!--mybatis  -->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.2</version>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>2.0.2</version>
    </dependency>
  </dependencies>
```

然后还有静态资源导出的问题
```xml
  <build>
    <resources>
      <resource>
        <directory>src/main/java</directory>
        <includes>
          <include>**/*.properties</include>
          <include>**/*.xml</include>
        </includes>
        <filtering>false</filtering>
      </resource>
      <resource>
        <directory>src/main/resources</directory>
        <includes>
          <include>**/*.properties</include>
          <include>**/*.xml</include>
        </includes>
        <filtering>false</filtering>
      </resource>
    </resources>
    <finalName>SSMFirst</finalName>
  </build>
```


数据库的搭建
```plsql
CREATE DATABASE ssmbuild;
USE ssmbuild;
CREATE TABLE `books`(
  `bookID` INT NOT NULL AUTO_INCREMENT COMMENT '书id',
  `bookName` VARCHAR(100) NOT NULL COMMENT '书名',
  `bookCounts` INT NOT NULL COMMENT '数量',
  `detail` VARCHAR(200) NOT NULL COMMENT '描述',
  KEY `bookID`(`bookID`)
)ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO `books`(`bookID`,`bookName`,`bookCounts`,`detail`)VALUES
(1,'Java',1,'从入门到放弃'),
(2,'MySQL',10,'从删库到跑路'),
(3,'Linux',5,'从进门到进牢');
```
Mapper 接口，xml 文件
```java
package com.stoocea.Mapper;

import com.stoocea.Pojo.Books;

import java.util.List;

public interface BookMapper {

    int addBook(Books books);
    int deleteBookById(int id);

    int updateBook(Books books);

    Books queryBookById(int id);

    List<Books> queryAllBook();
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.stoocea.Mapper.BookMapper">
    <select id="SelectAllBooks" resultType="Books">
        select * from books
    </select>
    <select id="SelectBookById" resultType="Books" parameterType="int">
        select * from books where id=#{id}
    </select>
    <insert id="AddBook" parameterType="Books">
        insert into books(bookName,bookCounts,detail) values (#{bookName},#{bookCounts},#{detail})
    </insert>
    <update id="UpdateBook" parameterType="map">
        update books set bookName=#{bookname},bookCounts=#{bookcounts},detail=#{detail} where bookID=#{id}
    </update>
    <delete id="deleteBookById" parameterType="int">
        delete from books where bookID=#{id}
    </delete>
</mapper>
```
最终还是要在 service 层给他实现出来的
```java
package com.stoocea.Service;
import com.stoocea.Pojo.Books;

import java.util.List;

public interface MapperService {
    int addBook(Books books);
    int deleteBookById(int id);

    int updateBook(Books books);

    Books queryBookById(int id);

    List<Books> queryAllBook();
}
```
```java
package com.stoocea.Service;

import com.stoocea.Pojo.Books;

import java.awt.print.Book;
import java.util.List;
import com.stoocea.Mapper.BookMapper;
public class BookServiceImpl implements MapperService {
    private BookMapper bookMapper;

    public void setBookMapper(BookMapper bookMapper) {
        this.bookMapper = bookMapper;
    }

    @Override
    public int addBook(Books books) {
        return bookMapper.addBook(books);
    }

    @Override
    public int deleteBookById(int id) {
        return bookMapper.deleteBookById(id);
    }

    @Override
    public int updateBook(Books books) {
        return bookMapper.updateBook(books);
    }

    @Override
    public Books queryBookById(int id) {
        return bookMapper.queryBookById(id);
    }

    @Override
    public List<Books> queryAllBook() {
        return bookMapper.queryAllBook();
    }
}

```


#### 1x02 Spring 层
Spring 层操作的主要对象就是 bean，然后就是整合 mybatis
我们回顾一下 Spring 是如何整合 mybatis 的：

- 获取数据库配置内容，准备连接数据库
- 连接数据池（这里就需要数据库的配置文件了）
- 注册实现 sqlSession,这里其实分为了两部操作，一步是获取到 sqlSessionFactory，一步是通过动态注册 Mapper 到 spring 容器中，SqlSessionFactory 又能够获取到Mapper 接口，进而获取到 sqlsession

注意这里我还是用的 Spring 自带的数据连接池，c3p0 的池子我连不上
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

<!--    关联数据库配置文件，也就是连接数据库的默认配置-->
    <context:property-placeholder location="database.properties"/>

    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
    </bean>

<!--    注册sqlSessionFactory-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">

<!--        连接数据池-->
        <property name="dataSource" ref="dataSource"/>
<!--        读取mybatis的配置文件-->
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
    </bean>



<!--    加载mapper包中的所有接口，通过sqlSessionFactory获取sqlSession对象，然后创建所有的dao接口并存储在Spring容器中-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        <property name="basePackage" value="com.stoocea.Mapper"/>
    </bean>
    <bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="transactionManager">
        <!--注入数据源-->
        <property name="dataSource" ref="dataSource"/>
    </bean>
</beans>
```

然后注册一下 service 层，管理起来方便一点
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <context:component-scan base-package="com.stoocea.Service"/>
    <bean id="BookServiceImpl" class="com.stoocea.Service.BookServiceImpl">
        <property name="bookMapper" ref="bookMapper"/>
    </bean>
</beans>
```


#### 1x03 springMVC 层
springmvc 最重要的东西就是 dispatchservlet，我们要在 web.xml 文件中去标记它，但它的内容又是通过配置文件来定义的，所以我们还必须要有一个 springmvc 的配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">


    <context:component-scan base-package="com.stoocea.Controller"/>
    <mvc:default-servlet-handler />
    <mvc:annotation-driven />
    <!--    添加视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
        <!--        前缀-->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <!--        后缀-->
        <property name="suffix" value=".jsp"/>
    </bean>
    <!--    Handler-->

</beans>
```

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>

  <!--乱码解决  -->
  <filter>
    <filter-name>encoding</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>encoding</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>


  <!--dispatchservlet的实现  -->
  <servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:spring-mvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>

</web-app>
```


### 0x02 测试 查询所有书籍功能
然后我们可以写一个测试类来看看是否可以正常请求
```java
import com.stoocea.Pojo.Books;
import com.stoocea.Service.BookServiceImpl;
import com.stoocea.Service.MapperService;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class mytest {
    @Test
    public void test(){
        ApplicationContext context =new ClassPathXmlApplicationContext("applicationContext.xml");
        MapperService bookService=context.getBean("BookServiceImpl", MapperService.class);
        for (Books books : bookService.queryAllBook()) {
            System.out.println(books);
        }
    }
}

```
正常的测试还是没问题，说明我们的思路和配置没错
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709482027729-d22afc9c-284e-4eff-9aa2-0080dd9558c8.png#averageHue=%233d434c&clientId=u3d0d3ad1-7789-4&from=paste&height=294&id=u1d4fc438&originHeight=441&originWidth=1718&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=383852&status=done&style=none&taskId=u38af1c7c-7b5e-4e5d-ab02-5daaa8cc859&title=&width=1145.3333333333333)
我们之前配置 web.xml 的时候，由于单学 SpringMVC，导致配置文件一直是 spring-mvc，项目所有的 bean 都是在 spring-mvc xml 文件中，所以 web.xml 中关于 dispatchservlet 的关联文件只写了 spring-mvc，实际上应该写上总的 `applicationContext.xml`
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709520412378-22d81c4e-f6f5-4aa3-b624-babe4e01d6f9.png#averageHue=%2340464f&clientId=u0115491d-06a4-4&from=paste&height=265&id=u1a44bc4d&originHeight=397&originWidth=1515&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=252163&status=done&style=none&taskId=u27cb58fc-7539-4e4e-8884-3ed63392e24&title=&width=1010)

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709520481165-399417b9-e134-4d2b-bd52-311646d6316e.png#averageHue=%23c4c3c3&clientId=u0115491d-06a4-4&from=paste&height=337&id=u6e055792&originHeight=506&originWidth=2559&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=96458&status=done&style=none&taskId=u2946ae0e-1907-4d0e-9868-f6ea8ecfbc2&title=&width=1706)

#### 1x01 排错注意事项
这里 jsp 文件中的 EL 表达式不能生效，必须得加上头 `<%@page isELIgnored="false" %>`
这里涉及一个取查询对象属性的操作
去掉前端的修饰，我们看`<%@ taglib prefix="c" uri="[http://java.sun.com/jsp/jstl/core"](http://java.sun.com/jsp/jstl/core") %>`
```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@page isELIgnored="false" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
<link href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
<title>书籍展示</title>
</head>
<body>
div class="container">
<div class="row clearfix">
    <div class="col-md-12">
        <div class="page-header">
            <h1>
                <small>书籍列表 ———— 显示所有数据</small>
            </h1>
        </div>
    </div>
</div>

<div  class="row clearfix">
    <div class="col-md-12 column">
        <table class="table table-hover table-striped">
            <thead>
            <tr>
                <th>书籍编号</th>
                <th>书籍名称</th>
                <th>书籍数量</th>
                <th>书籍详情</th>
            </tr>
            </thead>
            <tbody>
            <c:forEach var="book" items="${list}">
                <tr>
                    <td>${book.bookID}</td>
                    <td>${book.bookName}</td>
                    <td>${book.bookCounts}</td>
                    <td>${book.detail}</td>
                </tr>
            </c:forEach>
            </tbody>
        </table>
    </div>
</div>
</div>
</body>	
</html>

```
然后 c 标签 foreach 遍历我们刚才后端返回出来的 list 数组对象

### 0x03 测试增加书籍功能
首先是一些 jsp 的内容，重点关注 <input>标签的 name 属性值必须和类中的属性值以及数据库列名相同，不然会报错为空，找不到属性值
```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <%@ page isELIgnored="false" %>
    <html>
      <head>
        <link href="https://cdn.staticfile.org/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
        <title>添加书籍</title>
      </head>
      <body>
        <div>
          <div class="container">
            <div class="panel panel-default">
              <div class="panel-heading">
                <h3 class="panel-title">新建部门</h3>
              </div>
              <div class="panel-body">
                <form action="${pageContext.request.contextPath}/books/addBook" method="post">
                  <div class="form-group">
                    <label >书籍名称</label>
                    <input type="text" class="form-control" placeholder="书籍名称" name="bookName" required="required">
                  </div>
                  <div class="form-group">
                    <label >书籍数量</label>
                    <input type="text" class="form-control" placeholder="书籍数量" name="bookCounts" required="required">
                  </div>
                  <div class="form-group">
                    <label >书籍详情</label>
                    <input type="text" class="form-control" placeholder="书籍详情" name="detail" required="required">
                  </div>
                  <button type="submit" class="btn btn-primary">提交</button>
                </form>
              </div>
            </div>
          </div>
        </div>
      </body>
    </html>

```

```java
@RequestMapping("/toaddbook")
public String toaddPaper(){
    return "addbook";
}

@RequestMapping("/addBook")
public String addBook(Books books){
mapperService.addBook(books);
//直接return的话要考虑拼接以及最外面一层的路由，过于麻烦，直接重定向走即可
return "redirect:/books/allBook";
}
```
新增两个 controller，一个是用来从 index 界面跳转到 addbook ，然后 addbook 提交表单内容给 addBook，进入该路由的逻辑，执行 add 方法
之后的项目内容就不写了，进入 SpringMVC 拦截器


