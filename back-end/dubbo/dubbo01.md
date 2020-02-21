# 一、基本概念

## 1、分布式系统

​	若干独立计算机（多个服务器）的集合，常规的垂直架构无法应付规模较大的网站。

## 2、分布式演变

​	单一架构

​	垂直结构MVC：多个应用垂直分布

​		页面+业务逻辑没有分离，界面改了，整个应用需要重新发布

​		应用之间不可能完全独立，应用之间有大量交互，不好实现

​	分布式服务架构：RPC远程调用实现逻辑和页面完全分离。

​		可能出现资源浪费，需要引入调度中心

## 3、RPC远程调用

remote procedure call	远程过程调用，是一种进程间通信方式，是一种技术思想，不是规范。它允许程序调用另一个地址空间（通常是共享网络的另一台机器，服务器）的过程或函数。

RPC两个核心模块：通信、序列化

RPC框架：dubbo、gRPC、Thrift、HSF（High Speed Service Framework）

# 二、dubbo

apache dubbo三大核心能力：面向接口的远程调用、只能容错和负载均衡、服务自动注册和发现

## 1、dubbo特性

​	面向接口代理的高性能RPC调用

​			提供高性能的基于代理的远程调用能力，服务以接口为粒度，为开发者屏蔽远程调用细节。

​	只能负载均衡

​			内置多种负载均衡策略，只能感知下游节点健康状态，显著减少调用延迟，提高系统吞吐量。

​	服务自动注册和发现

​			支持多种注册中心服务，服务实例上下线实时感知。

​	运行期流量调度

​			内置条件、脚本等路由策略，通过配置不同的路由策略，轻松实现灰度发布，同机房优先等功能。

​			灰度发布：服务更新，选择部分服务器先使用更新后的代码，慢慢全部服务器更新完。

​	可视化的服务治理和运维

​			随时查询服务元数据、服务健康状态及调用统计。

## 2、dubbo原理模块

provider：服务提供者

consumer：服务消费者

registry：注册中心

container： dubbo框架容器

monitor：监控中心

# 三、远程调用实现

1、将服务提供者注册到注册中心

2、让服务消费之去注册中心订阅服务提供者的服务地址

## 1、pom依赖

~~~xml
<dependencies>
	  <dependency>
	  	<groupId>com.cx.gmall</groupId>
	  	<artifactId>gmall-interface</artifactId>
	  	<version>0.0.1-SNAPSHOT</version>
	  </dependency>
	  <dependency>
	  	<groupId>com.alibaba</groupId>
	  	<artifactId>dubbo</artifactId>
	  	<version>2.6.2</version>
	  </dependency>
	  <!--zookeeper注册中心  dubbo2.6之前用client，2.6之后用zookeeper 
	  	curator是操作注册中心的客户端
	  -->
	  <dependency>
	  	<groupId>org.apache.curator</groupId>
	  	<artifactId>curator-framework</artifactId>
	  	<version>2.12.0</version>
	  </dependency>
  </dependencies>
~~~



## 2、provider.xml

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans        
    		http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        
    		http://dubbo.apache.org/schema/dubbo        
    		http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
 
    <!-- 1、提供方应用信息，用于计算依赖关系	 -->
    <dubbo:application name="user-service-provider"  />
 
    <!--2、使用zookeeper广播注册中心暴露服务地址 	-->
    <!-- <dubbo:registry address="zookeeper://127.0.0.1:2181" /> -->
    <dubbo:registry protocol="zookeeper" address="192.168.0.101:2181"/>
 
    <!-- 3、用dubbo协议在20880端口暴露服务（通信协议和通信端口） 	-->
    <dubbo:protocol name="dubbo" port="20880" />
 
    <!--4、声明需要暴露的服务接口  ref指向接口真正的实现	-->
    <dubbo:service interface="com.cx.gmall.service.UserService" ref="userService" />
 
    <!-- 和本地bean一样实现服务  	-->
    <bean id="userService" class="com.cx.gmall.service.impl.UserServiceImpl" />
</beans>
~~~



## 3、consumer.xml

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans        
    		http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        
    		http://dubbo.apache.org/schema/dubbo        
    		http://dubbo.apache.org/schema/dubbo/dubbo.xsd
    		http://www.springframework.org/schema/context        
    		http://www.springframework.org/schema/context/spring-context.xsd">
    		
    <context:component-scan base-package="com.cx.gmall.service.impl"></context:component-scan>
 
    <!-- 1、提供方应用信息，用于计算依赖关系	 -->
    <dubbo:application name="order-service-sonsumer"  />
 
    <!--2、使用zookeeper广播注册中心暴露服务地址 	-->
    <!-- <dubbo:registry address="zookeeper://127.0.0.1:2181" /> -->
    <dubbo:registry protocol="zookeeper" address="127.0.0.1:2181"/>
 
    <!-- 3、声明需要调用的远程服务的接口，生成远程服务代理 	-->
    <dubbo:reference interface="com.cx.gmall.service.UserService" id="userService"></dubbo:reference>
    
</beans>
~~~

## 4、启动类

~~~java
public class UserServiceProviderMainApplicaton {
	public static void main(String[] args) throws IOException {
		ClassPathXmlApplicationContext ioc = new ClassPathXmlApplicationContext("provider.xml");
		ioc.start();
		
		//提供阻塞方法
		System.in.read();
	}
}
~~~

# 四、dubbo整合springboot

## 1、导入依赖

~~~xml
		<dependency>
			<groupId>com.cx.gmall</groupId>
			<artifactId>gmall-interface</artifactId>
			<version>0.0.1-SNAPSHOT</version>
		</dependency>
		<dependency>
			<groupId>com.alibaba.boot</groupId>
			<artifactId>dubbo-spring-boot-starter</artifactId>
			<version>0.2.0</version>
		</dependency>
~~~

## 2、dubbo配置

application.properties	替换provider.xml文件中dubbo的配置,消费者服务相似配置

~~~properties
dubbo.application.name=user-service-provider
dubbo.registry.address=127.0.0.1:2181
dubbo.registry.protocol=zookeeper

dubbo.protocol.name=dubbo
dubbo.protocol.port=20880
~~~

## 3、暴露服务和远程调用接口

~~~java
@Service	//dubbo注解：暴露服务
@Component
public class UserServiceImpl implements UserService {

	public List<UserAddress> getUserAddressById(String userId) {
		UserAddress userAddress1 = new UserAddress(1, "安徽安庆", "1", "cx", "13515605954", "Y");
		UserAddress userAddress2 = new UserAddress(2, "安徽合肥", "1", "cx", "13515605953", "N");
		return Arrays.asList(userAddress1,userAddress2);
	}
}
~~~

~~~java
@Service("orderService")
public class OrderServiceImpl implements OrderService{
	
	//@Autowired
	@Reference
	UserService userService;

	public List<UserAddress> initOrder(String userId) {
		System.out.println(userId);
		//1、查询用户的收货地址
		List<UserAddress> userAddressList = userService.getUserAddressById("1");
		return userAddressList;
	}
}
~~~

## 4、开启基于注解的dubbo功能

~~~java
@EnableDubbo	//开启基于注解的dubbo功能
@SpringBootApplication
public class BootUserServiceProviderApplication {

	public static void main(String[] args) {
		SpringApplication.run(BootUserServiceProviderApplication.class, args);
	}
}
~~~



# 五、dubbo可配置项

## 1、启动时检查

​			服务消费者	一般设置check=false

## 2、超时和配置覆盖关系

​			服务消费者	timeout=xxx	单位毫秒	默认1000毫秒

​			服务提供者也可以设置



​			方法级优先，接口次之，全局配置再次之

​			如果级别一样，则消费方优先，提供方次之

## 3、重试次数

​			retries=x	不包含第一次。当有多个相同服务，会在多个服务之间样一共重试x次。

​			幂等可以设置重试次数（查询、删除），非幂等不能设置重复次数（新增、修改）

## 4、多版本，灰度发布

​			version=x.x.x		版本迭代，部分服务提供者升级，再将所有消费者升级，最后将剩余服务提供者升级

# 六、高可用

## 1、zookeeper宕机和dubbo直连

​			zookeeper宕机，还可以 消费dubbo暴露的服务。通过设计，减少系统不能提供服务的时间。还可以直接使用dubbo直连，在消费者引用的注解后面添加远程引用的位置（url=127.0.0.1:20881）

​		原因：

​			监控中心宕掉后不影响使用，只是丢失部分采样数据

​			注册中心对等集群，任意一台宕掉后，将自动切换到另一台

​			注册中心全部宕掉后，服务提供者和消费者仍能通过本地缓存通讯

​			服务提供者全部宕掉后，消费者将无法使用，并无限次重连等待提供者恢复

## 2、集群下dubbo负载均衡配置

​		loadbalance：默认random	随机调用，可在暴露服务或消费时设置

​		weight：权重

​			1、Random Load Balance	基于权重的随机负载均衡机制

​			2、RoundRobin Load Balance	基于权重的轮询负载均衡机制

​			3、LeastActive Load Balance	最少活跃次数-负载均衡策略

​			4、ConsistentHash Load Balance	一致性hash-负载均衡策略

## 3、服务降级Hystrix

​		屏蔽远程调用，不发起调用， 直接返回null

​		容错：远程调用失败时，返回空对象

# 七、dubbo原理

## 1、RPC原理	

RPC框架的目标是将2-8步都封装起来，这些对用户透明，不可见。

​	服务消费方（client）以本地调用方式调用服务

​	client stub（消费方地代理对象）找到服务地址，并肩消息发送到服务端

​	server stub（服务方代理对象）收到消息并解码

​	server stub根据解码的结果调用本地服务

​	本地服务之星并将结果返回给server stub

​	server stub减结果打包并发送给消费方

 	client stub接收到消息并解码

​	消费方得到最终结果

## 2、netty原理

## 3、dubbo原理

