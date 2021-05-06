---
title: Spring Boot
date: 2021-05-06 13:21:08
tags: 
- Java
- 后端
---

### 微服务阶段

#### 什么是微服务

微服务是一种架构风格，它要求我们在开发一个应用是，这个应用必须构建成一系列小服务的组合；可以通过HTTP的方式进行互通。

#### 单体应用架构

单体应用架构是指，将一个应用中的所有应用服务都封装在一个应用中

* 这样做的好处是，易于开发和测试；也十分方便部署；当需要扩展时，只需要将war复制多份，然后放到多个服务器上，再做个负载均衡
* 缺点是，哪怕修改了一个非常小的地方，都需要停掉整个服务，重新打包、部署这个应用war包。特别是对于一个大型应用个，我们不可能把所有应用都放在一个应用里面，如何维护、如何分工合作都是问题

#### 微服务架构

all in one 的架构模式，我们把所有的功能单元放在一个应用里面。然后我们把整个应用部署到服务器上。如果负载能力不行，我们将整个应用进行水平复制，进行扩展，然后再负载均衡

所谓微服务架构就是打破之前all in one的架构方式，把每个功能元素独立出来。把独立出来的功能元素的动态组合，需要的功能元素才进行组合，需要多一些事可以整合多个功能元素。所以微服务架构是对功能元素进行复制，而没有对整个应用进行复制

这样做的好处是：

* 节省了调用资源
* 每个功能元件

#### 如何构建微服务

一个大型系统的微服务架构，就像是一个复杂交织的神经网络，每一个神经元就是一个功能元素，它们各自完成自己的功能，然后通过HTTP互相请求调用。比如一个电商系统，查缓存、连数据库、浏览页面、结账等服务都是一个个独立的功能程序，都被微化了，它们作为一个个微服务共同构建了一个庞大的系统。如果修改其中的一个功能，只需要升级其中一个功能服务单元即可

***

### 第一个SpringBoot程序

官方提供了一个快速生成的网站，IDEA集成这个网站

* 可以在官网直接下载后导入idea开发

* 直接使用idea创建一个springboot项目

* Maven-plugins有的版本会出现问题，解决办法：http://maven.apache.org/plugins/index.html

  查看出现问题的plugin的可用版本，然后手动在pom.xml中进行version设置

更改项目端口号

```properties
#更改端口号
server.port=8081
```

***

### 原理初探

#### 自动配置

**pom.xml**

* spring-boot-dependencies：核心依赖在父工程中

  ps:如果想要更改Maven的plugin的默认版本可以在父类中进行更改

* 我们在写或者引入一些SpringBoot依赖的时候，不需要指定版本，因为有这些版本仓库

**启动器**

* ~~~xml
  <!--启动器-->
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter</artifactId>
          </dependency>
  ~~~

* 说白了就是SpringBoot的启动场景

* 比如Spring-boot-starter-web，他就会帮我们自动导入web环境所有的依赖

* SpringBoot会将所有的功能场景都变成一个个启动器

* 我们要使用什么功能就只需要找到对应的启动器

**主程序**

```java
//@SpringBootApplication:标注这个类是一个SpringBoot的应用
@SpringBootApplication
public class Springboot01HelloworldApplication {
    public static void main(String[] args) {
        //将SpringBoot应用启动
        SpringApplication.run(Springboot01HelloworldApplication.class, args);
    }
}
```

* 注解

  * ```java
    @SpringBootConfiguration:SpringBoot的配置
    	@Configuration: Spring配置类
            	@Component :说明这也是一个Spring的组件
    
    @EnableAutoConfiguration:自动配置
        @AutoConfigurationPackage:自动配置包
            @Import(AutoConfigurationPackages.Registrar.class):导入选择器"包注册"
        @Import(AutoConfigurationImportSelector.class):自动导入选择
    //获取所有配置
    List<String> configurations = getCandidateConfigurations(annotationMetadata,attributes);
    ```

结论：SpringBoot所有的自动配置都是在启动的时候扫描并加载：spring.factories所有的自动配置类都在这里面，但是不一定生效，要判断条件是否成立，只要导入了对应的start,就油对应的启动器了，有了启动器，我们自动装配就会生效，然后配置成功！

1. springboot在启动的时候，从类路径下/META-INF/spring.factories获取指定的值；
2. 将这些自动配置的类导入容器，自动配置类就会生效，帮我们进行自动配置
3. 以前我们需要自动配置的东西，现在SpringBoot帮我们做了
4. 整合javaEE，解决方案和自动配置的东西都在spring-boot-autoconfigure-2.2.0.RELEASE.jar这个包下
5. 它会把所有需要导入的组件，以类名的方式返回，这些组件就会被添加到容器
6. 容器中也会存在非常多的xxxAutoConfiguration的文件，就是这些文件给容器中导入了这个场景所需要的所有组件，并自动配置
7. 有了自动配置类就免去了我们配置的时间

JavaConfig @Configuration @Bean

**SpringApplication**

这个类主要做了以下四件事情：

1. 推断应用的类型是普通的项目还是web项目
2. 查找并加载所有可用初始化器，设置到initializers属性中
3. 找出所有的应用程序监听器，设置到listeners属性中
4. 推断并设置main方法的定义类，找到运行的主类

关于SpringBoot，谈谈你的理解：

* 自动装配
* run()
* 全面接管SpringMVC的配置！实操！

***

### SpringBoot配置

SpringBoot使用一个全局的配置文件，配置文件名称是固定的

* application.properties
  * 语法结构：key=value
* application.yml
  * 语法结构：key: value

**配置文件的作用：**修改SpringBoot自动配置的默认值，因为SpringBoot在底层都给我们自动配置好了

具体查看yaml的官方解释

~~~yaml
#注入到我们的配置类中
#普通的key-value
name: ggssh

#对象
student:
  name: ggssh
  age: 12

#行内写法
student: {name: ggssh,age: 3}

#数组
pets:
  - cat
  - dog
  - pig

pets: [cat,dog,pig]
~~~

yaml可以直接给实体类赋值

![image-20210123225045716](https://ggssh.oss-cn-beijing.aliyuncs.com/mdimg/image-20210123225045716.png)

按照官方要求进行配置后可以消除爆红

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
~~~

```java
/*
@ConfigurationProperties作用:
将配置文件中配置的每一个属性的值，映射到这个组件中
告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定
参数prefix="person":将配置文件中的person下面的所有属性一一对应

只有这个组件是容器中的组件，才能使用容器提供的@ConfigurationProperties功能
 */
@Component//注册bean
@ConfigurationProperties(prefix = "person")

//javaConfig绑定配置文件的值，可以采取这些方式
//加载指定的配置文件
//@PropertySource(value = "classpath:qinjiang.properties")
public class person {
    //SPEL表达式取出配置文件的值
    @Value("${name}")
    private String name;
    private Integer age;
    private Boolean happy;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

可以考虑在配置文件中进行全局配置

**SpringBoot推荐使用yaml**，学习使用yaml

|                | @ConfigurationProperties | @Value     |
| :------------- | :----------------------- | :--------- |
| 功能           | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定       | 支持                     | 不支持     |
| SpEL           | 不支持                   | 支持       |
| JSR303数据校验 | 支持                     | 不支持     |
| 复杂类型封装   | 支持                     | 不支持     |

* cp只需要写一次即可，value则需要每个字段都添加
* 松散绑定：比如yml中写的last-name，这个和lastname是一样的，后面跟着的字母默认是大写的，这就是松散绑定
* JSR303数据校验，这就是我们可以在字段时增加一层过滤器验证，可以保证数据的合法性
* 复杂类型封装，yml中可以封装对象，使用@value就不支持

结论：

* 配置yml和properties都可以获取到值，强烈推荐yml
* 如果我们在某个业务中，只需要获取配置文件中的某个值，可以使用一下@value
* 如果说我们专门编写了一个JavaBean和配置文件进行映射，就直接使用@configurationProperties

***

#### JSR303数据校验

spring-boot中可以用@validated来校验数据，如果数据异常则会同意抛出异常，方便异常中心统一处理

在pom.xml中添加依赖，以后依赖再出现问题的话就进入父目录中将依赖版本进行修改，进入Maven Repository网站中查看稳定的依赖，记得加RELEASE

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

![img](https://upload-images.jianshu.io/upload_images/3145530-8ae74d19e6c65b4c?imageMogr2/auto-orient/strip|imageView2/2/w/654/format/webp)

**Hibernate Validator 附加的 constraint**

![img](https://upload-images.jianshu.io/upload_images/3145530-10035c6af8e90a7c?imageMogr2/auto-orient/strip|imageView2/2/w/432/format/webp)

```yaml
@NotNull(message="名字不能为空")
private String username;
@Max(value=120,message="年龄最大不能超过120")
private int age;
@Email(message="邮箱格式错误")
private String email;

空检查
@Null 验证对象是否为null
@NotNull 验证对象是否不为null,无法查验长度为0的字符串
@NotBlank
@NotEmpty

Boolean检查
@AssertTrue 验证Boolean对象是否为True
@AssertTrue

长度检查
@Size(min=,max=)
@Length(min=,max=)

日期检查
@Past 验证Date和Calendar对象是否在当前时间之前
@Future
@Pattern 验证String对象是否符合正则表达式的规则
```

具体可以查看 https://developer.ibm.com/zh/languages/java/articles/j-lo-jsr303/

***

#### 多环境配置及配置文件位置

1. file:./config/
2. file:./
3. classpath:/config/
4. classpath:/

同意路径下配置文件优先级：properties>yaml>yml

```properties
#springboot的多环境配置配置:可以选择激活哪一个配置文件
spring.profiles.active=test
server.port=8080
```

```yaml
server:
  port: 8081
spring:
  profiles:
    active: test
---
server:
  port: 8082
spring:
  config:
    activate:
      on-profile: dev
---
server:
  port: 8083
spring:
  config:
    activate:
      on-profile: test
```

***

#### 自动配置原理再理解

![image-20210124220719359](https://ggssh.oss-cn-beijing.aliyuncs.com/mdimg/image-20210124220719359.png)

```properties
#配置文件能够配置的东西，都存在一个固有的规律
#xxxAutoConfiguration:默认值  xxxProperties 和  配置文件绑定，我们就可以使用自定义配置
```

* SpringBoot启动会加载大量的自动配置类
* 我们看我们需要的功能有没有在SpringBoot默认写好的自动配置类当中
* 我们再来看这个自动配置类中到底配置了哪些组件(只要我们要用的组件存在其中，我们就不需要再手动配置了)
* 给容器中自动配置类添加组件是，会从properties类中获取某些属性，我们只需要在配置文件中指定这些属性的值即可
  * xxxxAutoConfiguration:自动配置类
  * xxxxProperties:封装配置文件中相关属性

可以通过 debug=true 来查看哪些配置类生效，哪些配置类没有生效

***

### SpringBoot Web开发

jar:webapp! 

自动装配

SpringBoot帮我们配置了什么，我们能不能进行修改？能修改哪些东西，能不能扩展

* xxxxAutoConfiguration:向容器中自动配置组件
* xxxxProperties:自动配置类，装配配置文件中自定义的一些内容

要解决的问题：

* 导入静态资源
* 首页
* jsp，模板引擎thymeleaf
* 装配扩展SpringMVC
* 增删改查
* 拦截器
* 国际化

#### 静态资源

什么是webjars:

**用maven构建 项目时，resources 目录就是默认的classpath**

总结：

1. 在SpringBoot中我们可以使用以下方式处理静态资源
   * webjars      localhost:8080/webjars/
   * public, static, /**,resources         localhost:8080/
2. 优先级:  resources>static(默认)>public

#### 首页如何定制

前端交给我们的页面是html页面。如果是我们以前开发，我们需要把他们转成jsp页面，jsp好处就是当我们查出一些数据转发到jsp页面之后，我们可以用jsp轻松实现数据的显示以及交互。jsp支持非常强大的功能，包括能写java代码，但是我们现在的这种情况，SpringBoot这个项目首先是一jar的方式，不是war，第二我们使用的是嵌入式的Tomcat，所以默认是不支持jsp的

在不支持jsp的情况下，如果我们直接使用纯静态页面的方式，会给我们的开发带来非常大的麻烦，SpringBoot推荐使用模板引擎

其实jsp就是一个模板引擎，还有用的比较多的freemarker，包括SpringBoot给我们推荐的Thymeleaf，但是思想都是一样的。

#### 模板引擎

结论：只要需要使用thymeleaf，只需要导入对应的依赖就行了。我们将html放在我们的templates目录下即可

~~~java
public static final String DEFAULT_PREFIX = "classpath:/templates/";
public static final String DEFAULT_SUFFIX = ".html";
~~~

![image-20210125200609472](https://ggssh.oss-cn-beijing.aliyuncs.com/mdimg/image-20210125200609472.png)

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>Insert title here</title>
</head>
<body>
<!--所有的html元素都可以被thymeleaf替换接管：  th:元素名  -->
<p th:text="${msg}"></p>
<p th:utext="${msg}"></p>

<hr>
<h3 th:each="user:${users}" th:text="${user}"></h3>
</body>
</html>
```

#### 装配扩展SpringMVC

```java
//如果你想diy一些定制化的功能，只要写这个组件，然后将它交给SpringBoot。SpringBoot就会给我们自动装配
//扩展 SpringMVC dispatchservlet
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {

    //public interface ViewResolver 实现了视图解析器接口的类，我们就可以把它看做视图解析器

    @Bean
    public ViewResolver myViewResolver(){
        return new MyViewResolver();
    }

    //自定义了一个自己的视图解析器MyViewResolver
    public static class MyViewResolver implements ViewResolver {
        @Override
        public View resolveViewName(String s, Locale locale) throws Exception {
            return null;
        }
    }
}
```

在SpringBoot中，有非常多的xxxxConfiguration帮助我们进行扩展配置，只要看见这个东西就要注意

```java
//如果我们要扩展SpringMVC 官方建议我们这样做
@Configuration
@EnableWebMvc //导入一个类:DelegatingwebMvcConfiguration:从容器中获取所有的webmvcconfig
public class MyMvcConfig implements WebMvcConfigurer {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/kuang").setViewName("test");
    }
}
```

要学的还有很多……