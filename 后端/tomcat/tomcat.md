# 一、tomcat基础

## 1.1、web概念

软件架构：

​	c/s	：客户端/服务器

​	b/s	：浏览器/服务器

浏览器只能解析静态资源，服务器动态资源被访问后，需要在服务器端转换成静态资源，在返回给浏览器

网络通信三要素：

​	IP、端口、协议（跪地上给你数据传输的规则）

​	基础协议：

​			1、tcp：安全协议。三次握手，速度较慢

​			2、udp：不安全协议，速度快

## 1.2、常见的web服务器

1.2.1、概念

​	服务器：接受用户请求，处理请求，作出响应

​	web服务器软件：在web服务器软件中，可以部署web项目，让用户通过浏览器访问这些项目

1.2.2、常见web服务器软件

webLogic、webSphere、JBOSS、Tomcat

1.3、常见问题

​	启动乱码：修改logging.properties文件： java.util.logging.ConsoleHandler.encoding = UTF-8 ，改为GBK；

​	启动一闪而过：JAVA_HOME不要加分号 ；

​	tomcat首页404： classpath=.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;（.;一定不能少，因为它代表当前路径)  

二、tomcat架构

## 2.1、http工作原理

HTTP协议是浏览器和服务器之间的数据传输协议，HTTP基于TCP/IP协议来传输数据，HTTP协议不涉及数据包传输，主要规定了客户端和服务器之间的通信格式。

![http原理](D:%5Cstudy%5Cnote%5Ctomcat%5Cimages%5Chttp%E5%8E%9F%E7%90%86.PNG)



服务器主要工作：

​	接受请求建立连接

​	解析http格式的数据包

​	执行请求，生成http格式的数据包

## 2.2、http服务请求处理

![http服务请求处理](D:%5Cstudy%5Cnote%5Ctomcat%5Cimages%5Chttp%E6%9C%8D%E5%8A%A1%E8%AF%B7%E6%B1%82%E5%A4%84%E7%90%86.PNG)

## 2.3、servlet工作流程

![servlet工作流程](D:%5Cstudy%5Cnote%5Ctomcat%5Cimages%5Cservlet%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B.PNG)

## 2.4、tomcat整体架构

tomcat要实现两个核心功能

1）：处理socket请求，负责网络哦字节流与Rerquest和Response对象的转化

2）：加载和管理servlet，以及具体处理Request请求

因此tomcat设计了两个核心组件 连接器（Connector）和容器 （Container），连接器负责对外交流，容器负责内部处理

![tomcat架构](D:%5Cstudy%5Cnote%5Ctomcat%5Cimages%5Ctomcat%E6%9E%B6%E6%9E%84.PNG)



## 2.5、连接器	Coyote

Coyote是tomcat的连接器框架的名称，是tomcat提供的供客户端访问的外部接口。

Coyote封装了底层的网络通信（socket请求和相应处理），为Catalina容器提供了统一的接口，使Catalina容器与具体的请求协议和IO操作方式完全解耦，

Coyote作为独立的模块，只负责具体的协议和IO的相关操作，与Servlet规范实现没有直接联系，因此即便是Request和response对象，也并未实现Servlet规范对应的接口，而是在Catalina中将他们进一步封装为ServletRequest和ServletResponse。

![Coyote和Catalina交互](D:%5Cstudy%5Cnote%5Ctomcat%5Cimages%5CCoyote%E5%92%8CCatalina%E4%BA%A4%E4%BA%92.PNG)

### 2.5.1、IO模型与协议

自8.5/9.0版本起，tomcat移除了对BIO的支持

| 传输层IO模型 | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| NIO          | 非阻塞IO，才用java NIO实现                                   |
| NIO2         | 异步IO，才用java7最新的NIO2类库实现                          |
| APR          | 才用Apaceh可移植运行库实现，是C/C++编写的本地库，需单独安装APR库 |

| 应用层协议 | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| HTTP/1.1   | 大部分web才用的访问协议                                      |
| AJP        | 用于和web服务器集成（如Apache），以实现对静态资源的优化和集群部署，当前支持AJP/1.3 |
| HTTP/2     | 大幅度提升了web性能，自8.5版本后支持                         |

Tomcat为了实现支持 多种IO模型和应用层协议，一个容器可能对接多个连接器，连接器和容器组装起来叫做一个service组件，Tomcat内部可能有多个Service组件，通过在Tomcat中配置多个Service，可以实现通过不同的端口号来访问同一台机器上部署的不同应用。

### 2.5.2、连接器组件

![连接器组件](D:%5Cstudy%5Cnote%5Ctomcat%5Cimages%5C%E8%BF%9E%E6%8E%A5%E5%99%A8%E7%BB%84%E4%BB%B6.PNG)

连接器中各个组件的作用：

EndPoint

1）：EndPoint：Coyote通信端点，即通信监听的解扣，是具体的Socket接受和发送处理器，是对传输层的抽象，因此EndPoint用来实现TCP/IP协议

2）：Tomcat并没有EndPoint接口，威海市提供了一个抽象类AbstractEndPoint，里面定义了两个类：Acceptor和SocketProcessor。Acceptor用于监听Socket请求，SocketProcessor用于处理Socket请求，它实现Runnable接口，在run方法里调用协议处理组件Processor进行处理，为了提高性能，SocketProcessor被提交到线程池来执行。这个线程池叫做执行器（Executor）。

Processor

Processor：Coyote协议处理接口，如果说EndPoint是用来实现TCP协议，那么Processor就是用来实现HTTP协议的，Processor接收到来自EndPoint的Socket，读取字节流解析成Tomcat的Request和Response对象，并通过Adapter将其提交到容器处理，Processor对应应用层协议的抽象。

Adapter

Adapter：ProtocolHandler接口负责解析并生成Tomcat Request对象，但是这个对象不是标准的ServletRequeset对象，不能用来作为参数调用容器，Tomcat于是引入CoyoteAdapter，这是适配器模式的经典应用，连接器调用CoyoteAdapter的Service方法，传入的是Tomcat Request对象，CoyoteAdapter将Request转成ServletRequest，在调用容器的Service方法。

## 2.6、容器--Catalina

Tomcat是一个由一系列可配置的组件构成的web容器，而Catalina是tocmat的Servlet容器。

Catalina是servlet容器实现，包含安全、会话、集群、管理等各方面。

### 2.6.1、Catalina地位

![结构图](D:%5Cstudy%5Cnote%5Ctomcat%5Cimages%5C%E7%BB%93%E6%9E%84%E5%9B%BE.PNG)

### 2.6.2、Catalina结构

![catlina结构](D:%5Cstudy%5Cnote%5Ctomcat%5Cimages%5Ccatlina%E7%BB%93%E6%9E%84.PNG)

catalina各个组件的作用：

| 组件      | 作用                                                         |
| --------- | ------------------------------------------------------------ |
| Catalina  | 负责解析tomcat的配置文件，以此来创建服务Server组件，并根据命令对其进行管理 |
| Server    | 负责组装并启动Servlet引擎，tomcat连接器                      |
| Service   | 是Server内部的组件，一个Server可以包含多个Service，并将若干个Connector组件绑定到一个Container上 |
| Connector | 连接器，处理与客户端的通信                                   |
| Container | 容器，负责处理用户的servlet请求                              |

### 2.6.3、container结构

tomcat设计了四中种器，Engine、Host、Context、Wrapper

![container结构](D:%5Cstudy%5Cnote%5Ctomcat%5Cimages%5Ccontainer%E7%BB%93%E6%9E%84.PNG)

各个组件的含义：

| 容器    | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| Engine  | 着呢个Catalina的Servlet引擎，用来管理多个虚拟站点，一个Service最多只能有一个Engine |
| Host    | 表示一个主机，或者说一个站点，可以给Tomcat配置多个虚拟主机站点 |
| Context | 表示一个web引用程序                                          |
| Wrapper | 表示一个servlet，不包含自容器                                |

![容器继承树](D:%5Cstudy%5Cnote%5Ctomcat%5Cimages%5C%E5%AE%B9%E5%99%A8%E7%BB%A7%E6%89%BF%E6%A0%91.PNG)

## 2.7、tomcat启动流程

<img src="D:%5Cstudy%5Cnote%5Ctomcat%5Cimages%5Ctomcat%E5%90%AF%E5%8A%A8%E6%B5%81%E7%A8%8B%E5%9B%BE.PNG" alt="tomcat启动流程图"  />







