# 一、springboot入门

## 1、springboot介绍

springboot用来简化spring应用开发，约定大于配置，去繁从简。是整个spring技术站的一个大整合，j2ee的一站式开发解决方案。

springboot 	-->	 j2ee一站式解决方案

springCloud	-->	分布式整体解决方案

优点：

​	快速创建独立运行的审批人能够项目以及与主流框架集成

​	使用嵌入式的servlet容器，应用无需打成war包，打成ar包就行

​	starters自动依赖与版本控制

​	大量的自动配置，简化开发，也可修改默认值

​	无序配置xml，无代码生成，开箱即用

​	准生产环境的运行时应用监控

​	与云计算的天然集成

## 2、微服务

microservices是一种架构风格，一个应用应该是一组小型服务；可以通过HTTP的方式进行互通。

每一个功能元素最终都是一个可独立替换和独立升级的软件单元

## 3、环境设置

maven配置，在settings文件中添加如下配置

~~~xml
<profile>
		<id>jdk-1.8</id>
		<activation>
			<activeByDefault>true</activeByDefault>
			<jdk>1.8</jdk>
		</activation>
		<properties>
			<maven.compiler.source>1.8</maven.compiler.source>
			<maven.compiler.target>1.8</maven.compiler.target>
			<maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
		</properties>
	</profile>
~~~

idea设置

​	maven全局设置

## 4、springboot  hello

### 4.1 导入springboot相关的依赖

~~~xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
~~~

### 4.2编写主程序

~~~java
package com.cx;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication  //标注一个主程序类，说明是一个 springboot应用
public class HelloMainApplication {
    //启动spring应用
    public static void main(String[] args) {
        SpringApplication.run(HelloMainApplication.class,args);
    }
}
~~~

### 4.3 写controller

~~~java
package com.cx.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class HelloController {

    @ResponseBody
    @RequestMapping("/hello")
    public String hello(){
        return "hello world";
    }
}
~~~

### 4.4 使用jar包启动服务

pom添加依赖

~~~xml
<!--该插件可以将应用打包成一个可执行的jar包-->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
~~~

打成jar包后使用java -jar命令启动服务。打包时自动把tomcat的jar包也打进去了。

## 5、hello	探究

### 5.1 pom.xml

父项目

~~~xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
	<version>1.5.9.RELEASE</version>
</parent>

它的父项目
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>1.5.9.RELEASE</version>
    <relativePath>../../spring-boot-dependencies</relativePath>
</parent>
~~~

这是springboot的版本仲裁中心



starters启动器

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
~~~

spring-boot-starter：springboot场景启动器，上面的依赖帮我们导入了web模块正常运行所以来的组件。

springboot将所有的功能场景都抽取出来，做成一个个的starters（启动器），只需在项目中引入相关场景的启动器，改场景的所有依赖的jar包都会导入进来。

### 5.2 主程序类。入口

@SpringBootApplication

~~~java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
    @Filter(type = FilterType.CUSTOM,classes = {TypeExcludeFilter.class}),
    @Filter(type = FilterType.CUSTOM,classes = {AutoConfigurationExcludeFilter.class})}
)
public @interface SpringBootApplication {
~~~

@SpringBootConfiguration：表示这是一个springboot的配置类

​		@Configuration：spring底层注解，表明这是一个配置类。配置类也是容器中的组件。

@EnableAutoConfiguration：开启自动配置功能

~~~java
@AutoConfigurationPackage
@Import({EnableAutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
~~~

@AutoConfigurationPackage：自动配置包

​		@Import({Registrar.class})：

​		**将主配置类的所在包及其子包里面所有的组件扫描到spring容器中**

@Import({EnableAutoConfigurationImportSelector.class})



## 6、使用spring initializer快速创建springboot项目

创建时选择需要的模快，想到会自动联网创建springboot项目；

默认生成的项目：

​	主程序一斤生成好了，我们只需要写逻辑部分

​	resources文件夹中目录结构

​			static：保存所有的静态资源；js、css、html

​			templates：保存所有的模板页面；（springboot默认 jar包使用嵌入式的tomcat，默认不支持jsp页面）；乐意使用模板引擎（freemarker、thymeleaf）；

​			application.properties：应用的配置文件，可以修改一些默认配置。

# 二、springboot配置

## 1、配置文件

springboot使用一个全局的配置文件，配置文件名是固定的；

​	application.properties

​	application.yml

YAML：标记语言，以数据为中心，比json、xml等更适合做配置文件

## 2、yml语法

K:(空格)v：表示一对键值对（空格必须有）；

以空格的缩进来控制层级关系，只要是左对齐的一列数据，都是同一个层级的；

属性和值也是大小写敏感的；

字符串不用加双引号，加双引号的内容不会被转义，加单引号的内容会被转义

~~~yml
#对象、map
friends1:
  name: cx
  age:  20

friends2:  {name: cx,age: 20}
#数组
pets1:
  - cat
  - dog
  - monkey
pets2: [cat,dog,monkey]
~~~

## 3、配置文件值的获取

yml配置文件

~~~yml
person:
  name: cx
  age: 20
  map: {a: 1,b: 2}
  lists:
    - a1
    - a1
  dog:
    name: dog
    age: 2
~~~

~~~xml
<!--配置文件处理器，配置文件进行绑定就会有提示-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
~~~



javabean

~~~java
package com.cx.bean;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.Map;

/**
 * 将配置文件中配置的每一个属性的值，映射到这个组件中
 * @ConfigurationProperties：告诉springboot，将本类中的所有属性与配置文件中配置进行绑定
 *      prefix：配置文件中的哪个下面的属性一一映射
 * @Component：只有这个组件是容器中的组件，才能使用容器提供的@ConfigurationProperties功能
 */
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;
    private Integer age;
    private Map<String,String> map;
    private List<Object> lists;
    private Dog dog;

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", map=" + map +
                ", lists=" + lists +
                ", dog=" + dog +
                '}';
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Map<String, String> getMap() {
        return map;
    }

    public void setMap(Map<String, String> map) {
        this.map = map;
    }

    public List<Object> getLists() {
        return lists;
    }

    public void setLists(List<Object> lists) {
        this.lists = lists;
    }

    public Dog getDog() {
        return dog;
    }

    public void setDog(Dog dog) {
        this.dog = dog;
    }
}

~~~

使用自动注入@AutoWired即可使用

## 4、注解

@ConfigurationProperties：读取全局配置文件

@PropertySource：读取指定的配置文件

```java
@PropertySource(value = {"classpath:person.properties"})
```

@ImportResource：导入spring的配置文件，让配置文件中的内容生效。该注解标注在配置类上

```java
@ImportResource(locations = {"classpath:bean.xml"})
@SpringBootApplication  //标注一个主程序类，说明是一个 springboot应用
public class HelloMainApplication {
    //启动spring应用
    public static void main(String[] args) {
        SpringApplication.run(HelloMainApplication.class,args);
    }
}
```

springboot不推荐写配置文件，推荐写配置类的方式：@Configuration、@Bean

## 5、Profile

用于对个环境时，不同环境的参数的设置

### 1、多profile文件

​	在编写主配置文件的时候，文件名可以是：applciation-{profile}.properties/yml

​	默认使用applciation.properties的配置

### 2、yml支持对文档快方式

~~~yml
server:
  port: 8081

spring:
  profiles:
    active: prod
---
server:
  port: 8081
spring:
  profiles: prod

---
spring:
  profiles: dev
server:
  port: 8082
~~~



### 3、激活制定的profile

​	1、在配置文件中指定：spring.profiles.active=prod

​	2、命令行

​			java -jar xxx.jar --spring.profiles.active=prod

​	3、虚拟机参数

​			-Dspring.profiles.active=prod		



## 6、配置文件加载位置

./config/	: 项目根目录下的config

./	: 项目根目录下

classpath:./config/	：类路径下的config

classpath:/

优先级由高到低，高优先级的配置会覆盖低优先级的配置。这四个会互补。

也可以使用spring.config.location=xxx 来设置配置文件的新位置



## 7、自动配置原



# 三、springboot与日志

springboot底层使用的是：slf4j+logback；默认info级别，也就是root级别

## 1、整合日志步骤：

​	1、排除原有日志实现

​	2、添加替换原有日志实现的jar包

​	3、导入slf4j真正的实现

## 2、日志级别

trance	 debug	 info	warn	error

3、

logging.file=springbootceshi.log	logging.file:指定日志输出文件

logger.path=/spring/log	logger.path指定日志输出到那个文件夹下，默认文件名为spring,log

# 四、springboot与web开发

## 一、简介

使用springboot应用

1、创建springboot，选中我么你需要的模块

2、springboot已经将这些场景配置好了，只需要在配置文件进行少量配置就可以运行起来

3、编写逻辑代码

## 二、对静态资源的映射

1、所有/webjars/**，都去classpath:META-INF/resources/webjars/找资源；

webjars：以jar包方式引入的静态资源文件

2、以	/**	访问当前项目的任何资源（静态资源文件夹）

```java
"classpath:/META-INF/resources/"
"classpath:/resources/"
"classpath:/static/"
"classpath:/public/"
"/"
```

## 三、模板引擎

JSP、Velocity、Freemaker、Thymeleaf

springboot推荐是用Thymeleaf：语法更简单，功能更强大

### 1、引入Thymeleaf

~~~xml
<properties>
    <java.version>1.8</java.version>
    <thymeleaf.version>3.0.9.RELEASE</thymeleaf.version>
    <!--布局功能的支持程序thymeleaf3 需使用layout2-->
    <thymeleaf-layout-dialect.version>2.2.2</thymeleaf-layout-dialect.version>
</properties>
<!--引入thymeleaf 2.16版本 版本太低-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
~~~

### 2、thymeleaf语法

~~~java
public class ThymeleafProperties {
    private static final Charset DEFAULT_ENCODING;
    public static final String DEFAULT_PREFIX = "classpath:/templates/";
    public static final String DEFAULT_SUFFIX = ".html";
    private boolean checkTemplate = true;
    private boolean checkTemplateLocation = true;
    private String prefix = "classpath:/templates/";
    private String suffix = ".html";
    //前后缀，只要把html页面放在该路径下，thymeleaf就会自动渲染
~~~

使用：

​	1、导入thymeleaf的名称空间

~~~xml
<html lang="en" xmlns:th="http://www.thymeleaf.org">
~~~

​	2、使用thymeleaf语法

~~~html
<body>
    <h1>成功</h1>
    <div th:text="${hello}">欢迎信息</div>
    <hr/>
    <!--[[]]:相当于th:text (转义)     [()]:相当于th:utext (不转义)-->
    <div th:each="user:${users}">[[${user}]]</div>
</body>
~~~

~~~java
	@RequestMapping("/success")
    public String success(Map<String,Object> map){
        map.put("hello","<h1>你好</h1>");
        map.put("users", Arrays.asList("张","钱"));
        return "success";
    }
~~~

## 四、springMVC自动配置

https://docs.spring.io/spring-boot/docs/2.2.4.RELEASE/reference/htmlsingle/#boot-features-developing-web-applications

## 五、如何修改springboot的默认配置

模式：

1、springboot在自动配置的很多组件的时候，显卡内容器中有没有用户自己配置的，如果有就用户配置的，没有才用做的功能配置的。

2、



