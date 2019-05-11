# 什么是Tomcat

Tomcat简单的说就是一个运行JAVA的网络服务器，底层是Socket的一个程序，它也是JSP和Serlvet的一个容器。

------

# 为什么我们需要用到Tomcat

如果你学过html，css，你会知道你写的页面只能自己访问，**别人不能远程访问你写的页面**，Tomcat就是**提供能够让别人访问自己写的页面的一个程序**



Tomcat 的目录结构

* bin：存放 Tomcat 的启动、停止等批处理脚本和 Shell 脚本
* conf：存放关于 Tomcat 的相关配置文件
  * Catalina：存放针对每个虚拟机的 Context 配置
  * context.xml：定义所有 Web 应用所需要加载的 Context 配置，如果 Web 应用指定了自己的 context.xml，该文件配置会被覆盖
  * catalina.properties：Tomcat 环境变量配置
  * catalina.policy：当 Tomcat 在安全模式下运行时，此文件为默认的安全策略配置
  * logging.properties：Tomcat 日志配置文件
  * server.xml：Tomcat 服务器核心配置文件，用于配置 Tomcat 的链接器、监听端口、虚拟主机等
  * tomcat-user.xml：配置用户名密码和相关权限
  * web.xml：Tomcat 中所有应用默认的部署描述文件
* lib：Tomcat 服务器依赖库目录，用于存放 Tomcat 服务器运行环境依赖 JAR 包
* logs：存放日志文件
* temp：存放运行时产生的临时文件
* webapps：默认的 Web 应用部署目录
* work：工作目录，Web 应用 JSP 代码生成和编译的临时目录



## 总体架构

### Server

服务器的主要功能，接受客户端发来的请求数据并解析，完成相关业务处理，然后把处理结果作为响应返回给请求客户端。通常使用 Socket 监听服务器指定端口来实现该功能，通过 start() 方法启动服务器，打开 Socket 链接，监听服务器端口，并负责在接收到客户端请求时进行处理并返回响应，通过 stop() 方法来停止服务器并释放资源

![QQ浏览器截图20190509144017](tomcat/QQ浏览器截图20190509144017.png)

### Connetor 和 Container

将请求监听与请求出路放在一起扩展性很差，所以需要将网络协议与请求处理从概念上分离。一个 Server 可以包含多个 Connector 和 Container。Connector 负责开启 Socket 并监听客户端请求、返回响应数据；Container 负责具体的请求处理。Connector 和 Container 分别拥有自己的 start() 方法和 stop() 方法来加载和释放自己维护的资源。

一个 Server 包含多个 Service（它们相互独立，只是共享一个JVM和系统类库），一个Service 负责维护多个 Connector 和一个 Container，这样来自 Connector 的请求只能由它所属 Service 维护的 Container 处理

![QQ浏览器截图20190509144418](tomcat/QQ浏览器截图20190509144418.png)

在 Tomcat 中，为了与 Tomcat 中的组件命名一致，将 Container 重命名为 Engine，用来表示整个 Servlet 引擎，注意，Engine 表示整个 Servlet 引擎，而非 Servlet 容器，表示整个 Servlet 容器的是 Server。引擎只负责请求的处理，不需要考虑请求链接、协议等的处理

![QQ浏览器截图20190509144631](tomcat/QQ浏览器截图20190509144631.png)

### Container 设计

可以使用 Context 来表示一个 Web 应用，一个 Engine 可以包含多个 Context。Context 也拥有 start() 和 stop() 方法，用以在启动时加载资源以及在停止时释放资源。采用这种方式设计，将加载和卸载资源的过程分解到每个组件当中，使组件充分解耦，提高服务器的可扩展性和可维护性

![QQ浏览器截图20190509144752](tomcat/QQ浏览器截图20190509144752.png)

当一台主机要提供多个域名的服务，可以将每个域名视为一个虚拟的主机，在每个虚拟主机下包含多个 Web 应用。可以用 Host 表示虚拟主机的概念，一个 Host 可以包含多个 Context

![QQ浏览器截图20190509144905](tomcat/QQ浏览器截图20190509144905.png)

在一个 Web 应用中，可以包含多个 Servlet 实例来处理来自不同链接的请求。在 Tomcat 中，用 Wrapper 表示 Servlet 定义。我们使用 Container 来表示容器，Container 可以添加并维护子容器，因此 Engine、Host、Context、Wrapper 均继承自 Container。它们之间是弱依赖的关系，即它们之间的关系是通过 Container 的父子容器的概念体现的，不过 Service 持有的是 Engine 接口

![QQ浏览器截图20190509145029](tomcat/QQ浏览器截图20190509145029.png)

Tomcat 的 Container 可以表示不同的概念级别：Servlet 引擎、虚拟主机、Web应用和 Servlet，那么我们就可以 将不同级别的容器作为处理客户端请求的组件，这具体由我们提供的服务器的复杂度决定。假使我们以嵌入式的方式启动 Tomcat，且运行极其简单的请求处理，不必支持多 Web 应用的场景，可以只在 Service 中维护一个简化版的 Engine

![QQ浏览器截图20190509145711](tomcat/QQ浏览器截图20190509145711.png)

此外，Tomcat 的 Container 有一个很重要的功能，就是后台处理。在很多情况下，我们的 Container 需要执行一些异步处理，而且是定期执行，如每隔30秒执行一次，Tomcat 对于 Web应用文件变更的扫描就是通过该机制实现的。Tomcat 针对后台处理，在 Container 上定义了 backgroundProcess() 方法，并且其基础抽象类（ContainerBase）确保在启动组件的同时，异步启动后台处理。因此，在绝大多数情况下，各个容器组件仅需要实现 Container 的 backgroundProcess() 方法即可，不必考虑创建异步线程

### LifeCycle

可以发现所有的组件均存在启动、停止等生命周期方法，拥有生命周期管理的特性。可以基于生命周期管理特性的组件抽象一个 LifeCycle 通用接口，该接口定义了生命周期管理的核心方法。

- init()：初始化组件
- start()：启动组件
- stop()：停止组件
- destory()：销毁组件

该接口还支持组件状态以及状态之间的转换，支持添加事件监听器（LifecycleListencr），用于监听组件的状态变化。这样就可以采用一致的机制来初始化、启动、停止以及销毁各个组件

![QQ浏览器截图20190509145914](tomcat/QQ浏览器截图20190509145914.png)

LifeCycle 接口状态图，每个周期可能对应数个状态的转换

![QQ浏览器截图20190509150026](tomcat/QQ浏览器截图20190509150026.png)

不是每个状态都会触发周期事件，也不是所有生命事件均存在对应状态

![QQ浏览器截图20190509150750](tomcat/QQ浏览器截图20190509150750.png)

### Pipeline 和 Value

Tomcat 采用了职责链模式来实现客户端请求的处理，Tomcat 中每个 Container 组件通过执行一个职责链来完成具体的请求处理，增强了组件的灵活性和可扩展性。Tomcat 定义了 Pipeline（管道）和 Valve（阀）两个接口，前者用于构造职责链，后者代表职责链上的每个处理器

![QQ浏览器截图20190509150933](tomcat/QQ浏览器截图20190509150933.png)

Pipeline 中维护了一个基础的 Valve，它始终位于 Pipeline 的末端，即最后执行，封装了具体的请求处理和输出响应的过程。通过 addValve() 方法，可以为 Pipeline 添加其他的 Valve。后添加 的 Valve 位于基础 Valve 之前，并按照添加顺序执行。 Pipeline 通过获得首个Valve来启动整个链条的执行

Tomcat 容器灵活之处在于每个层级的容器（Engine、Host、Context、Wrapper）均有对应的基础Valve 实现，同时维护一个 Pipeline 实例，这样就可以在任何层级的容器上针对请求处理进行扩展。由于Tomcat 的每个层级均通过 Pipeline 和 Valve 进行请求处理，所以很容易将一些通用的 Valve 实现根据需要添加到任何层级的容器上

![QQ浏览器截图20190509151035](tomcat/QQ浏览器截图20190509151035.png)

### Connector 设计

要想与 Container 配合实现一个完整的服务器功能，Connector 至少要完成如下几项功能，才能保证将接收到的客户端请求交由与请求地址匹配的容器处理

- 监听服务器端口，读取来自客户端的请求
- 将请求数据按照指定协议进行解析
- 根据请求地址匹配正确的容器进行处理
- 将响应返回客户端

在 Tomcat 中，ProtocolHandler 表示一个协议处理器，针对不同协议和 IO方式，提供不同的实现。ProtocolHandler 包含一个 Endpoint 用于监听 Socket 监听，该接口按照 IO 方式进行分类实现，还包含一个 Processor 用于按照指定协议读取数据，并将请求交由容器处理

![QQ浏览器截图20190509151529](tomcat/QQ浏览器截图20190509151529.png)

Tomcat 并没有 Endpoint 接口，只有 AbstractEndpoint 抽象类

在 Connector 启动时，Endpoint 会启动线程来监听服务器端口，并在接受到请求后调用 Processor 进行数据读取。当 Processor 读取客户端请求后，需要按照请求地址映射到具体的容器进行处理，这个过程称为请求映射。由于 Tomcat 各个组件采用通用的生命周期管理，而且可以通过管理工具具进行状态变更，因此请求映射除考虑映射规则的实现外，还要考虑容器组件的注册与销毁

Tomcat 通过 Mapper 和 MapperListener 两个类实现上述功能。前者用于维护容器映射信息，同时按照映射规则（ServIet规范定义）查找容器。后者实现了 ContainerListener 和 LifecycIeListener，用于在容器组件状态发生变更时，注册或者取消对应的容器映射信息。为了实现上述功能，MapperListener 实现了 Lifecycle 接口，当其启动时（在service启动时启动），会自动作为监听器注册到各个容器组件上，同时将已创建的容器注册到 Mapper

Tomcat 7 之前，Mapper 由 Connector 维护，Tomcat 8 之后改由 Service 维护

Tomcat 通过适配器模式实现了 Connector 与 Mapper、Container 的解耦。Tomcat 默认的 Connector 实现（Coyote）对应的适配器为 CoyoteAdapter。也就是说，如果你希望使用 Tomcat 的链接器方案，但是又想脱离 Servlet 容器（虽然这种情况几乎不可能出现，但是从架构可扩展性的角度来讲，还是值得讨论一下），此时只需要实现我们自己的 Adapter 即可。当然，我们还需要按照 Container 的定义开发我们自己的容器实现（不一定遵从 ServIet 规范）

![QQ浏览器截图20190509153652](tomcat/QQ浏览器截图20190509153652.png)

### Executor

Tomcat 提供 Executor 接口来表示一个可以在组件间共享的线程池，该接口继承自 LifeCycle 接口，可按照通用的组件进行管理。在 Tomcat 中 Executor 由 Service 维护，因此同一个 Service 中的组件可以共享一个线程池。如果没有定义任何线程池，相关组件，如 Endpoint，会自动创建线程池，此时，线程池不再共享。在 Tomcat 中，Endpoint 会启动一组线程来监听 Socket 端口，当接收到客户端请求后，会创建请求处理对象，并交由线程池处理，由此支持并发处理的客户端请求

![QQ浏览器截图20190509154429](tomcat/QQ浏览器截图20190509154429.png)

### Bootstrap 和 Catalina

Tomcat 通过类 catalina 提供了一个 Shell 程序，用于解析 server.xml 创建各个组件，同时负责启动、停止应用服务器。Tomcat 使用 Digester 解析 XML 文件，包括 server.xml 以及 web.xml 等。Tomcat 提供了 Bootstrap 作为应用服务器启动人口。Bootstrap 负责创建 Catalina 实例，根据执行参数调用 Catalina 相关方法完成针对应用服务器的操作（启动、停止）。

为什么 Tomcat 不直接通过 Catalina 启动，而是又提供了 Bootstrap 呢？你可以查看一下 Tomcat 的发布包目录，Bootstrap 并不位于 Tomcat 的依赖库目录下（$CATALINA_HOME/Iib),而是直接在 $CATALINA_HOME/bin 目录下。Bootstrap 与 Tomcat 应用服务器完全松耦合（通过反射调用CataIina实例），它可以直接依赖 JRE 运行并为 Tomcat 应用服务器创建共享类加载器，用于构造 CataIina 实例以及整个 Tomcat 服务器

完整的服务器设计

![QQ浏览器截图20190509155904](tomcat/QQ浏览器截图20190509155904.png)

上述是 Tomcat 标准的启动方式但是正如我们所说，既然 Server 及其子组件代表了应用服务器本身，那么我们就可以不通过 Bootstrap 和 Catalina 来启动服务器。Tomcat 提供了一个同名 org.apache.cataIina.startup.Tomcat，使用它我们可以将 Tomcat 服务器嵌入到我们的应用系统中并进行启动。也可以自己编写代码来启动 Server，也可以自定义其他配置方式启动，如 YAML。这就是 Tomcat 灵活的架构设计带给我们的便利，也是我们设计中间件产品的架构关注点之一

![QQ浏览器截图20190509160200](tomcat/QQ浏览器截图20190509160200.png)

## Tomcat 启动

Tomcat 的启动非常标准化，统一按照生命周期管理接口 LifeCylce 的定义进行启动。首先调用 init() 方法进行组件的逐级初始化，然后再调用 start() 方法进行启动，每次调用均伴随着生命周期状态变更事件的触发。每一级组件除完成自身的处理外，还要负责调用子组件相应的生命周期管理方法，组件与组件之间是松耦合的设计，因此可以很容易通过配置进行修改和替换

![QQ浏览器截图20190509160315](tomcat/QQ浏览器截图20190509160315.png)

## 请求处理

从本质上讲，应用服务器的请求处理开始于监听的 Socket 端口接收到数据，结束于将服务器
处理结果写入 Socket 输出流
在这个处理过程中，应用服务器需要将请求按照既定协议进行读取，并封装为与具体通信方案无关的请求对象。然后根据请求映射规则定位到具体的处理单元（在 Java 应用服务器中，多数是某个 web 应用下的一个 Servlet）进行处理。当然，如果我们的应用不是基于简单的 Servlet API，而是基于当前成熟的MVC框架，如 Struts、 SpringMVC，那么在多数情况下请求将进一步匹配到 Servlet 下的一个控制器，这部分已经不属于应用服务器的处理范畴，而是由具体的 MVC 框架进行匹配。当 Servlet 或者控制器的业务处理结束后，处理结果将被写入一个与通信方案无关的响应对象。最后，该响应对象将按照既定协议写入输出流。

![QQ浏览器截图20190509161513](tomcat/QQ浏览器截图20190509161513.png)

## 类加载器

### J2SE 标准类加载器

JVM 提供了 3 个类加载器，它们以一种父子树的方式创建，同时使用委派模式确保应用程序可通过自身的类加载器（System）加载所有可见的 Java 类

![QQ浏览器截图20190509162408](tomcat/QQ浏览器截图20190509162408.png)









 





