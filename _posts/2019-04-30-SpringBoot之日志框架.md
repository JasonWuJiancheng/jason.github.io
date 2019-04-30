---
layout: post
title: 'SpringBoot之日志框架'
subtitle: '简析SpringBoot日志框架的选择和使用'
date: 2019-04-30
categories: 技术
cover: ''
tags: java spring
---

# SpringBoot之日志框架

<!-- TOC -->

- [SpringBoot之日志框架](#springboot之日志框架)
    - [一、为何需要使用日志框架？](#一为何需要使用日志框架)
    - [二、框架选择](#二框架选择)
        - [1、选择](#1选择)
        - [2、日志门面](#2日志门面)
    - [三、关于SLF4J](#三关于slf4j)
        - [1、如何使用slf4j](#1如何使用slf4j)
        - [2、统一日志记录](#2统一日志记录)
    - [四、日志依赖](#四日志依赖)
    - [五、使用日志](#五使用日志)
        - [1、默认配置](#1默认配置)
        - [2、指定配置](#2指定配置)
    - [五、切换框架](#五切换框架)

<!-- /TOC -->

## 一、为何需要使用日志框架？

当你需要开发一个大型系统时：

1. 为了调试方便，在代码中写了很多System.out.println("")；语句，在控制台输出，便于观察程序的执行情况；
2. 老板审查时发现，代码中的这些语句并不需要，让你删去这些语句，你用了一下午时间完成了...
3. 后来，老板认为，这个提示蛮好，让你重新加回，并希望能够写在文件中，你又用了一下午时间，希望他能满意...
4. 工程经历了一次架构更新，老板想让你更改输出的格式以及信息，于是...

可见，==我们需要一个方便使用的日志框架==。

## 二、框架选择

**市面上的日志框架有：**

JUL、JCL、Jboss-logging、logback、log4j、log4j2、slf4j....等等

**他们大体上分为两类：**

| 日志门面  （日志的抽象层）                                   | 日志实现                                             |
| ------------------------------------------------------------ | ---------------------------------------------------- |
| ~~JCL（Jakarta Commons Logging）~~    **SLF4j（Simple  Logging Facade for Java）**    ~~jboss-logging~~ | Log4j  JUL（java.util.logging）  Log4j2  **Logback** |

### 1、选择

我们在实际开发中，只需要在**左边选一个门面，右边选一个实现**即可。

我们采用排除法选择：

1. 门面：
   - JCL：apache旗下Jakarta项目组产品，最后一次更新在2014年，已经跟不上潮流了；
   - jboss-logging：小众产品，特殊需要时再使用；
2. 实现：
   - Log4j：日志效率较低（约为Logback的1/10），且作者为了解决该问题，重新开发了一套框架Logback。==**事实上SLF4j、Log4j、Logback均出自于同一人之手**；==
   - JUL：java自带的日志框架，为了不让log4j占领市场推出的；
   - Log4j2：功能强大，且效率高，是假借Log4j之名由Apache推出的全新版本，但因为太新，和其他框架适配不好；

再者，**由于SLF4J与Logback由一人编写，所以能够实现所有功能，完美适配**。

所以，我们选择：

**日志门面：SLF4J；日志实现：Logback；**

而恰巧，**SpringBoot默认也用的是SLF4J+Logback组合**。

下面，针对不了解门面模式的，再做具体叙述：

### 2、日志门面

所谓日志门面，指的是日志的一个**抽象层**，一个**统一的接口层**。

门面模式，其核心为**外部与一个子系统的通信必须通过一个统一的外观对象进行，使得子系统更易于使用**。具体原理如下图所示：

![日志门面](https://jasonwujiancheng.github.io/screenshot/003/框架选择-01.png)

门面模式的核心为Facade即门面对象，门面对象核心为几个点：

- 知道所有子角色的功能和责任
- 将客户端发来的请求委派到子系统中，没有实际业务逻辑
- 不参与子系统内业务逻辑的实现

简单来说，**门面只是一个日志标准，并不是日志系统的具体实现**，即：

- 提供日志接口
- 提供获取具体日志对象的方法

## 三、关于SLF4J

具体的内容可以参照[SLF4J官方网站](https://www.slf4j.org)上的描述,这里我仅对实现方法做简单描述。

### 1、如何使用slf4j

在开发时，日志记录方法的调用，不应该直接调用日志的实现类，而是调用日志抽象层里面的方法；

我们只需要在系统中**导入slf4j的jar和  logback的实现jar**。

一个简单的例子：

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

官方图示：

![结构图示](https://jasonwujiancheng.github.io/screenshot/003/concrete-bindings.png)

接下来从左到右来解释一下：

> **浅蓝色：抽象接口；深蓝色：日志实现类；绿色：适配器；灰色：没有实现slf4j接口的实现类；**
>
> 1. unbound：当没有绑定实现类的时候，slf4j会将日志记录丢掉（即输出到/dev/null下）；
> 2. bound to logback-classic：当绑定logback实现类时，会调用它的classic与core包生成日志；
> 3. bound to log4j：（因为log4j出现时没有预料到slf4J的出现）当绑定log4j时，需要引入一个适配包（slf4j-log）进行桥接，；
> 4. bound to util：同3，也需要一个适配包（slf4j-jdk)；
> 5. bound to simple:同2；
> 6. bound to no-operation：同2，但是不对日志做操作；

大家可以看见，与logback的连接更为自然，而与其它日志框架的连接，在于**适配包（庆幸的是slf4j都帮你写好了）**。

**==关于日志文件的配置：==**每一个日志的实现框架都有自己的配置文件。**使用slf4j以后，配置文件还是做成日志实现框架自己本身的配置文件；**

### 2、统一日志记录

> 由于我们做SpringBoot项目时默认使用SLF4J+Logback框架，但是我们都知道，各个框架都有自己默认的日志记录，如：Spring（commons-logging）、Hibernate（jboss-logging）...

能不能做到，**即使是别的框架存在，我们也能统一用slf4j输出**？

答案是肯定的。我们来看官方给的解决方案：

![统一方式](https://jasonwujiancheng.github.io/screenshot/003/legacy.png)

接下来我来从左到右、从上到下解释一下这张图：

> 1. 如果我们**项目中存在不同框架，且底层用logback实现：**那我们就需要用中间包来代替原来的实现（要先删去原来的包）；
> 2. 如果我们**项目中存在不同框架，且底层用其他框架实现：**那我们就需要移除默认的logback实现，并引入相应的中间包；

总的来说，三步操作：

1. **将系统中其他日志框架先排除出去；**
2. **用中间包来替换原有的日志框架；**
3. **导入slf4j其他的实现**

## 四、日志依赖

我们新建一个SpringBoot的web项目，并且定位到pom文件，可以看见，

SpringBoot使用它来做日志功能：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```

在idea中，pom文件上右键选择DIagrams->show Dependencies可以看见依赖关系，如下图：

springboot2.x：

![依赖关系](https://jasonwujiancheng.github.io/screenshot/003/dependencies.png)wo

springboot1.x：

![依赖关系2](https://jasonwujiancheng.github.io/screenshot/003/dependencies2.png)

我们看见：

1. SpringBoot底层也是使用slf4j+logback的方式进行日志记录；
2. SpringBoot也把其他的日志都替换成了slf4j；

可以在依赖中看一下中间包：

![中间包](https://jasonwujiancheng.github.io/screenshot/003/中间包.png)

我们发现，包名都没有更改，依然为log4j默认的包名，事实是这样吗？

进入查看（此处springboot2.0以上版本会有些许不同）：

jcl-over-slf4j：新建了slf4j的日志工厂

```java
public abstract class LogFactory {
    static String UNSUPPORTED_OPERATION_IN_JCL_OVER_SLF4J = "http://www.slf4j.org/codes.html#unsupported_operation_in_jcl_over_slf4j";
    //新建了slf4j的日志工厂
    static LogFactory logFactory = new SLF4JLogFactory();
```

log4j-over-slf4j：应用slf4j的工厂方法

```java
public class LogManager {
    public LogManager() {
    }

    //应用slf4j的工厂方法
    public static Logger getRootLogger() {
        return Log4jLoggerFactory.getLogger("ROOT");
    }

    public static Logger getLogger(String name) {
        return Log4jLoggerFactory.getLogger(name);
    }

    public static Logger getLogger(Class clazz) {
        return Log4jLoggerFactory.getLogger(clazz.getName());
    }
```

原来，所谓**中间包，也就是改写了实现的替换包。**

## 五、使用日志

SpringBoot帮我们配置好了日志。

### 1、默认配置

我们可以测试一下：

```java
//记录器
    Logger logger = LoggerFactory.getLogger(getClass());
    @Test
    public void contextLoads() {
        //日志级别 由低到高为：
        //可以调整需要输出的日志级别；日志就只会在指定级别及以上生效；
        logger.trace("这是trace日志...");
        logger.debug("这是debug日志...");
        //springboot默认给我们使用的是info级别的(可以在properties里面配置）
        logger.info("这是info日志...");
        logger.warn("这是warn日志...");
        logger.error("这是error日志...");

    }
```

会在控制台得到如下输出：

```
2019-04-30 [ main ] - [ TRACE ] [ c.j.springboot.SpringBoot03LoggingApplicationTests : 19 ] - 这是trace日志...
2019-04-30 [ main ] - [ DEBUG ] [ c.j.springboot.SpringBoot03LoggingApplicationTests : 20 ] - 这是debug日志...
2019-04-30 [ main ] - [ INFO  ] [ c.j.springboot.SpringBoot03LoggingApplicationTests : 22 ] - 这是info日志...
2019-04-30 [ main ] - [ WARN  ] [ c.j.springboot.SpringBoot03LoggingApplicationTests : 23 ] - 这是warn日志...
2019-04-30 [ main ] - [ ERROR ] [ c.j.springboot.SpringBoot03LoggingApplicationTests : 24 ] - 这是error日志...
```

你可以在application.properties文件中修改默认配置：

```properties
# 指定包日志输出级别
logging.level.com.jason=trace

# 指定日志记录文件(默认在当前项目下生成）
# logging.file=springboot.log

# 指定日志生成目录(在磁盘根目录下创建spring文件夹及子文件夹log，使用spring.log作为默认文件）
# 与logging.file互斥
logging.path=/spring/log

# 在控制台输入日志的格式
logging.pattern.console=%d{yyyy-MM-dd} [ %thread ] - [ %-5level ] [ %logger{50} : %line ] - %msg%n
# 在文件中输出日志的格式
logging.pattern.file=%d{yyyy-MM-dd} [ %thread ] === %-5level === %logger{50} : %line === %msg%n
```

当然，你也可以在resources中编写你自己的配置文件。

关于输出格式，就不赘述了，可以自己了解，这里只举个例子：

```
日志输出格式：
		%d表示日期时间，
		%thread表示线程名，
		%-5level：级别从左显示5个字符宽度
		%logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
		%msg：日志消息，
		%n是换行符
    -->
    %d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n
```

### 2、指定配置

给类路径下放上每个日志框架自己的配置文件即可；此时SpringBoot就不使用其默认的配置了。

| Logging System          | Customization                                                |
| ----------------------- | ------------------------------------------------------------ |
| Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml` or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`                          |
| JDK (Java Util Logging) | `logging.properties`                                         |

以Logback为例，如果使用`logback.xml`配置，则直接被框架识别；

如果使用`logback-spring.xml`，则日志框架就不直接加载日志的配置项，由SpringBoot解析日志配置，可以使用SpringBoot的高级Profile功能，如：

```xml
<layout class="ch.qos.logback.classic.PatternLayout">
    <!-- 配合xxx-spring.xml来使用的高级功能 -->
    <springProfile name="dev">
        <!-- 指定dev环境下日志输出格式 -->
        <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level ---> %logger{50} - %msg%n</pattern>
    </springProfile>
    <springProfile name="!dev">
        <!-- 指定非dev环境下日志输出格式 -->
        <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level ==== %logger{50} - %msg%n</pattern>
    </springProfile>
</layout>
```

如果使用logback.xml作为日志配置文件，还要使用profile功能，会有以下错误：

 `no applicable action for [springProfile]`

## 五、切换框架

可以按照slf4j的日志适配图，进行相关的切换；

**1、slf4j+log4j的方式：**

此处注意！

**==一定一定要除去logback的默认实现包（logback-classic）与log4j中间包（log4j-over-slf4j）！==**

然后再导入官方图中给出的slf4j-log4j12。

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
    <exclusion>
      <artifactId>logback-classic</artifactId>
      <groupId>ch.qos.logback</groupId>
    </exclusion>
    <exclusion>
      <artifactId>log4j-over-slf4j</artifactId>
      <groupId>org.slf4j</groupId>
    </exclusion>
  </exclusions>
</dependency>

<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
</dependency>
```

**2、log4j2的方式：**

此处注意！

**==一定一定要除去springboot默认的日志依赖（spring-boot-starter-logging）！==**

然后再导入log4j2的依赖。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>spring-boot-starter-logging</artifactId>
            <groupId>org.springframework.boot</groupId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

