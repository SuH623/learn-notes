# 1. Maven 高级

## 1.1 分模块开发

当项目规模变大，需要将模块按业务，按功能进行拆分，这样的好处是

* 增强代码的复用性：一些通用的工具类、实体类可以抽取到独立的模块，进行重用
* 便于分工：按业务划分模块可以让开发人员编写代码更为独立，互不干扰
* 对 maven 项目来讲，一些通用的依赖、插件，可以抽取到父模块，简化配置。通过对这一阶段的学习，发现 Spring Boot 的 pom 配置非常简洁，它就是利用了继承，依赖管理等手段来实现此效果

下面就来学习分模块的步骤

### 步骤1 - 创建父模块

与创建普通 maven 工程类似，唯一不同的一点是

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
	...
	
    <packaging>pom</packaging>
	
	...

</project>
```

父模块的作用有两点，后面会展开讲解

* 继承：提供公共的属性、依赖、和插件，供子模块使用
* 聚合：可以统一驱动子模块，例如统一 test、package、install 等

### 步骤2 - 创建子模块

如下图所示，目录关系上【父模块】包含【子模块】
<img src="Maven%E9%AB%98%E7%BA%A7.assets/19.png" style="zoom:67%;" />

如下图所示，目录关系上【父模块】与【子模块】平级，都位于 `G:\传智-Spring` 目录下
<img src="Maven%E9%AB%98%E7%BA%A7.assets/20.png" style="zoom:67%;" />

两种都行，选择一种喜欢的即可

试着创建两个子模块，maven_child1 与 maven_child2，观察 pom 文件，有两点变化：

父模块的 pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
	...
	
    <modules>
        <module>maven_child1</module>
		<module>maven_child2</module>
    </modules>
    
	...

</project>
```

`<modules>` 意思是父模块【聚合】了两个子模块，如果父模块执行一些 maven 命令，例如

* 父模块 compile - 所有被聚合模块都会执行 compile
* 父模块 test - 所有被聚合模块都会执行 test
* 父模块 package - 所有被聚合模块都会执行 package
* 父模块 install - 所有被聚合模块都会执行 install

试着给两个子模块各自添加一个 Service 类，然后执行父模块的 package 命令观察结果

子模块的 pom.xml，以 maven_child1 为例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
	<parent>
        <artifactId>maven_parent</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    
	...

</project>
```

`<parent>` 意思是标注继承自哪个模块，继承可以重用父模块的属性、依赖、和插件，下一节展开讲解

### 步骤3 - 模块间依赖

子模块之间也可以相互依赖，经常把一些公共的功能集中在一个模块里，例如：maven_common 的 pom.xml 文件如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>maven_parent</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>maven_common</artifactId>

</project>
```

其它模块可以依赖之，与平时依赖第三方 jar 类似

```xml
...
<dependency>
	<groupId>org.example</groupId>
	<artifactId>maven_common</artifactId>
	<version>1.0-SNAPSHOT</version>
</dependency>
```



## 1.2 继承与依赖管理

### 1) 继承属性

例如：父模块中指明了模块所用 java 的源码和 class 版本，子模块就无需再次声明

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
	...
	
	<properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>
	
	...

</project>
```

### 2) 自定义属性

如果 pom.xml 文件中像版本号这样的东西重复很多，可以用自定义属性来配置

```xml
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-context</artifactId>
	<version>5.2.9.RELEASE</version>
</dependency>
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-jdbc</artifactId>
	<version>5.2.9.RELEASE</version>
</dependency>
```

可以变成

```xml
<properties>
	...
	<spring.version>5.2.9.RELEASE</spring.version>
</properties>
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>${spring.version}</version>
    </dependency>
</dependencies>
```

### 3) 强制继承依赖

子模块可以继承父模块 `<dependencies>` 内的 GAV（GroupId，ArtifactId，Version）信息

例如，父模块

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
	...
	
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
	
	...

</project>
```

两个子模块的 dependencies

<img src="Maven%E9%AB%98%E7%BA%A7.assets/21.png" style="zoom:67%;" />

强制继承存在的问题是：有的时候我们不希望把父模块中所有依赖都继承过去，例如，父模块声明了一个 javax.servlet-api，但子模块根本不是 web 项目，这种情况下强制继承就太粗暴了

### 4) 继承依赖管理

为了解决上述问题，使用依赖继承管理：即父模块中使用 `<dependencyManagement>` 来声明依赖，子模块通过 `<dependency>` 选择自己实际需要的坐标，只需声明 GroupId 与 ArtifactId，而依赖的版本由父模块统一控制

#### 例1 - 依赖管理

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
	...
	
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>javax.servlet</groupId>
                <artifactId>javax.servlet-api</artifactId>
                <version>3.1.0</version>
                <scope>provided</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
	
	...

</project>
```

这时候子模块并没有直接将依赖继承过来，而是需要自己声明【我需要】哪个依赖，例：

maven_child1 模块只需要 javax.servlet-api 依赖，则在自己的 pom.xml 中配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
	...

    <artifactId>maven_child1</artifactId>

    <dependencies>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
        </dependency>  
    </dependencies>

</project>
```

注意版本号没有了，看似虽然只简单了一点点，但意义在于版本号由父模块统一控制了！

> 当然，如果子模块自己指定了 version，还会以自己指定的 version 为准，但这样一来就丧失了统一控制版本的初衷

#### 例2 - 控制间接依赖版本

父模块中配置了一个 dubbo 依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
	...
	
    <dependencyManagement>
        <dependencies>
            ...
			<dependency>
                <groupId>org.apache.dubbo</groupId>
                <artifactId>dubbo</artifactId>
                <version>2.7.8</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
	
	...

</project>
```

maven_child2 子模块使用了 dubbo 依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	
    ...

    <dependencies>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo</artifactId>
        </dependency>
    </dependencies>

</project>
```

通过下图可以看到
<img src="Maven%E9%AB%98%E7%BA%A7.assets/22.png" style="zoom:67%;" />

虽然仅是引入了一个 maven 依赖，但 dubbo 本身运行所需的 spring-context 等依赖也被间接加入进来，现在的问题是它间接加入的 spring 相关的 jar 包版本有点低，而我们希望用更高的 spring 的版本怎么办呢？

解决方法是，在父模块中加入 spring-context 的高版本依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
	...
	
    <dependencyManagement>
        <dependencies>
            ...
			<dependency>
                <groupId>org.apache.dubbo</groupId>
                <artifactId>dubbo</artifactId>
                <version>2.7.8</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context</artifactId>
                <version>5.2.9.RELEASE</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
	
	...

</project>
```

刷新后，发现子模块间接依赖的 spring-context 的版本受到父模块的控制：

<img src="Maven%E9%AB%98%E7%BA%A7.assets/image-20211130152229490.png" alt="image-20211130152229490" style="zoom:67%;" />

#### 例3 - 排除依赖

父模块中声明了 zookeeper 和 logback 依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
	...
	
    <dependencyManagement>
        <dependencies>
            ...
			<dependency>
                <groupId>org.apache.zookeeper</groupId>
                <artifactId>zookeeper</artifactId>
                <version>3.6.2</version>
            </dependency>
			<dependency>
                <groupId>ch.qos.logback</groupId>
                <artifactId>logback-classic</artifactId>
                <version>1.2.3</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
	
	...

</project>
```

maven_child2 子模块使用了 zookeeper 和 logback 依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	
    ...

    <dependencies>
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
        </dependency>
    </dependencies>

</project>
```

我们的期望是使用 logback 作为日志输出，但 zookeeper 自带了 log4j 日志的依赖，两个日志是有冲突的，如下图所示，此时并不能用就近原则来解决此问题
<img src="Maven%E9%AB%98%E7%BA%A7.assets/24.png" style="zoom:67%;" />

可以使用依赖排除，排除掉不想要的即可

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
	...
	
    <dependencyManagement>
        <dependencies>
            ...
			<dependency>
                <groupId>org.apache.zookeeper</groupId>
                <artifactId>zookeeper</artifactId>
                <version>3.6.2</version>
				<exclusions>
                    <exclusion>
                        <groupId>org.slf4j</groupId>
                        <artifactId>slf4j-log4j12</artifactId>
                    </exclusion>
                    <exclusion>
                        <groupId>log4j</groupId>
                        <artifactId>log4j</artifactId>
                    </exclusion>
                </exclusions>
            </dependency>
			<dependency>
                <groupId>ch.qos.logback</groupId>
                <artifactId>logback-classic</artifactId>
                <version>1.2.3</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
	
	...

</project>
```


> ***注意***
>
> * 以上操作均建议在父模块进行配置，虽然在子模块做也可以，但通用性就差了
> * Spring Boot 里经常使用排除依赖配合条件装配的技巧，切换内嵌 web 容器实现、日志实现、数据源实现等等

### 5) boot 依赖管理

学到现在，回过头来再看看 spring boot 应用的 pom.xml 文件

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.5</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <!-- ... -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

现在大家应该能看懂，它是采用了 maven 继承的思想，继承了 spring-boot-starter-parent 这个依赖，有如下特点

1. spring-boot-starter、spring-boot-starter-test 等依赖的版本通过**依赖管理**的方式来统一管理了版本
2. spring-boot-starter 等以 starter 结尾的依赖使用了**传递依赖**的方式，集成了大量相关的依赖

<img src="Maven%E9%AB%98%E7%BA%A7.assets/image-20211102193208269.png" alt="image-20211102193208269" style="zoom:67%;" />

以上的两个做法，都大大简化了依赖管理。

## 1.3 插件

maven 提供了很多有用的插件，介绍一些较为有用的

### 1) maven-compiler-plugin

它可以控制项目中

- `<source>` 控制 java 源码的版本
  - 3.8.0 以上版本默认 java 源码版本为 1.6，否则默认 java 版本为 1.5
- `<target>` 控制 class 的版本
  - 3.8.0 以上版本默认 class 版本为 1.6，否则默认 class 版本为 1.5
- `<encoding>` 控制源码字符集
- `<parameters>` 设置为 true ，可以在编译 class 时生成方法的参数名，否则方法参数名默认为 arg0，arg1

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	
    ...

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                    <encoding>UTF-8</encoding>
                    <parameters>true</parameters>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

以上配置均有简化形式

```xml
<properties>
	<maven.compiler.source>8</maven.compiler.source>
	<maven.compiler.target>8</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.parameters>true</maven.compiler.parameters>
</properties>
```



### 2) maven-surefire-plugin

它可以控制项目的单元测试

平时我们可以点击 idea 工具条来跳过测试
<img src="F:/%25E5%259F%25BA%25E7%25A1%2580%25E6%25A1%2586%25E6%259E%25B6%25E8%25B5%2584%25E6%2596%2599/java%25E8%25AF%25BE%25E7%25A8%258B/%25E6%25BB%25A1%25E5%258F%2594%25E6%25A1%2586%25E6%259E%25B6/old/%25E8%25B5%2584%25E6%2596%2599/ppt/day06/img/25.png" style="zoom:67%;" />

它的实际原理是在执行 mvn 命令时添加了 `-DskipTests=true` 这个虚拟机参数

如果需要更详细的控制，见下面的例子

例1：跳过所有测试

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	
    ...

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.22.2</version>
                <configuration>
                    <skipTests>true</skipTests>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

例2：包含 TestService 开头的这些测试类，默认只会包含 Test 开头、Test 结尾、TestCase 结尾的测试类

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	
    ...

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.12.4</version>
                <configuration>
                    <includes>**/TestService*.java</includes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

例3：排除 TestService 开头的这些测试类，默认会排除所有内部类

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	
    ...

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.12.4</version>
                <configuration>
                    <excludes>**/TestService*.java</excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```



### 3) maven-resources-plugin

它的主要作用有三点

- 一是将 `src/main/resources` 下的资源（`*.xml`, `*.properties` 等）拷贝到 classes 目录，以便后续运行时能够找到
- 二是拷贝资源文件时，指定源文件的字符编码
- 三是拷贝资源文件时，替换占位符为属性值

如果不配置，在 maven 构建时，会提示警告错误

```
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ maven_child1 ---
[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
```

意思是，源文件的编码没有设置，使用平台编码 utf-8，这个编码是 idea 默认的，如果你在 windows 黑窗口下运行 `mvn compile`，则提示的警告信息不同

```
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ maven_child1 ---
[WARNING] File encoding has not been set, using platform encoding GBK, i.e. build is platform dependent!
[WARNING] Using platform encoding (GBK actually) to copy filtered resources, i.e. build is platform dependent!
```

而此时平台编码是 gbk 的

为了避免潜在的问题，建议显式设置此编码，这样警告信息就没有了！

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	
	...

    <properties>
        <project.build.sourceEncoding>utf-8</project.build.sourceEncoding>
    </properties>
	
	...
    
</project>
```

现在希望将文件内 ${} 占位符替换为 maven 中定义的【属性】，例如：
maven_child1 模块的 src/main/resources/my.properties 内容如下

```properties
host=${host}
```

可以用下面的 pom 配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
	...

    <properties>
        <project.build.sourceEncoding>utf-8</project.build.sourceEncoding>
        <host>127.0.0.1</host>
    </properties>

    <build>
		<plugins>
        	<plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <version>3.2.0</version>
                <configuration>
                    <resources>
                        <resource>
                            <!--将src/main/resources中的内容拷贝到classes目录中-->
                            <directory>src/main/resources</directory>
                            <!--开启占位符过滤-->
                            <filtering>true</filtering>
                        </resource>
                    </resources>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

`<properties>` 中的子标签的键值，可以

- 替换 pom.xml 文件本身的 ${}，这个在给大家的案例、作业中已经大量体现
- 在 maven-resources-plugin 的资源拷贝且 `<filtering>true</filtering>` 时，替换应用资源文件中的 ${}

执行完成后，可以查看 target/classes 下的 my.properties，发现其内容已经变为

```properties
host=127.0.0.1
```



### 4) spring-boot-maven-plugin

spring-boot-maven-plugin 用来生成可运行的 jar 或 war 包，如果用官网的 Spring Boot 骨架：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.4.2</version>
		<relativePath/> <!-- lookup parent from repository -->
  	</parent>
	
	...

    <build>
        <plugins>
            
			...
			
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

如果用了 aliyun 的 Spring Boot 骨架，因为没有用继承的方式，配置会稍显复杂

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">

	...

    <build>
        <plugins>
            
			...
			
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.3.4.RELEASE</version>
                <configuration>
                    <mainClass>com.itheima.SpringCaseBoot01Application</mainClass>
                </configuration>
                <executions>
                    <execution>
                        <id>repackage</id>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```

- mainClass 可以省略，默认会将编译时第一个含有 main 方法的类作为 mainClass😉

其目录结构如下

```cmd
├─BOOT-INF
│  ├─classes
│  │  └─com
│  │      └─itheima
│  │          ├─config
│  │          └─demo
│  │              ├─bean
│  │              ├─dao
│  │              └─service
│  └─lib
├─META-INF
│  └─maven
│      └─com.itheima
│          └─demo
└─org
    └─springframework
        └─boot
            └─loader
                ├─archive
                ├─data
                ├─jar
                ├─jarmode
                └─util
```

- BOOT-INF/classes - 自己编写的 class
- BOOT-INF/lib - 所有依赖的 jar
- META-INF/MANIFEST.MF - jar 包启动信息
- org/springframework/boot/loader - jar 启动及相关类

#### jar

Spring Boot 项目运行 maven package 命令后（会附加执行插件的 repackage 命令），所有依赖 jar 都会被包含进去，习惯称作 fat jar 或 uber jar

> windows 可以用 tree 命令查看解压后的 jar 目录结构

#### war

只需要在创建项目时，在向导内选择打包方式为 war
<img src="F:/%25E5%259F%25BA%25E7%25A1%2580%25E6%25A1%2586%25E6%259E%25B6%25E8%25B5%2584%25E6%2596%2599/java%25E8%25AF%25BE%25E7%25A8%258B/%25E6%25BB%25A1%25E5%258F%2594%25E6%25A1%2586%25E6%259E%25B6/old/%25E8%25B5%2584%25E6%2596%2599/ppt/day06/img/40.png" style="zoom:67%;" />

要修改的有这么几处

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    
	...
	
	<dependencies>
        ...
		
		<!-- 1. 如果希望添加 jsp 支持，则加入-->
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-jasper</artifactId>
        </dependency>
    </dependencies>

    <build>
		<!-- 2. 决定最终 war 包名称，对 jar 包同样适用 -->
        <finalName>spring_case_boot_02</finalName>
        <plugins>
			<!-- 3. 忽略 web.xml -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>2.6</version>
                <configuration>
                    <failOnMissingWebXml>false</failOnMissingWebXml>
                </configuration>
            </plugin>
            
			...
        </plugins>
    </build>

</project>
```

注意：

- war 包也支持以命令行方式运行，与 jar 运行方式类似
- war 包也可以部署到外部 tomcat 环境下，但这时内嵌 tomcat 的相关配置如 server.port 会失效！



## 1.4 profile

为了实现不同环境的构建需求，maven 提供了 profile 的支持，下面是一个不同环境下拷贝资源时为 host 提供不同值的例子

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">    
	...
    <build>		
        ...		
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>

    <profiles>
        <profile>
            <id>development</id>
            <properties>
                <host>localhost</host>
            </properties>
        </profile>

        <profile>
            <id>production</id>
            <properties>
                <host>www.itheima.com</host>
            </properties>
        </profile>
    </profiles>
</project>
```

构建时选择 profile 有多种方式，常见方式列举如下

方式1 - idea 中勾选相应的 profile
<img src="Maven%E9%AB%98%E7%BA%A7.assets/26.png" style="zoom:67%;" />
它的原理是在执行 mvn 命令时添加 `-P production` 参数

当选择 production profile 后，my.properties 拷贝后的内容变为

```properties
host=www.itheima.com
```

当选择 development profile 后，my.properties 拷贝后的内容变为

```properties
host=localhost
```



方式2 - 设置默认激活

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
	...

    <build>
		
        ...
		
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>

    <profiles>
        <profile>
            <id>development</id>
            <properties>
                <host>localhost</host>
            </properties>
			<activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>

        <profile>
            <id>production</id>
            <properties>
                <host>www.itheima.com</host>
            </properties>
        </profile>
    </profiles>
</project>
```



## 1.5 仓库与镜像

### 下载加速

我们都知道，maven 的仓库分为本地仓库和远程仓库。默认的远程仓库叫做中央仓库，地址为：`https://repo.maven.apache.org/maven2/`，它的下载速度比较慢，不做任何配置时，下载一个 dubbo 的 jar 包，就是从中央仓库下载的

<img src="Maven%E9%AB%98%E7%BA%A7.assets/29.png" style="zoom: 67%;" />

可以为项目配置多个仓库地址，下载时会由上到下依次尝试，哪个仓库成功了，就停止尝试，因此可以把速度快的仓库配置在靠上位置

> 注意：其实配多个仓库的更主要原因是，某些 jar 包只在 A 仓库有，在 B 仓库没有，这时就得配置多个仓库了

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
	...

    <repositories>
        <repository>
            <id>aliyun</id>
            <url>http://maven.aliyun.com/nexus/content/groups/public</url>
        </repository>
        <repository>
            <id>central</id>
            <url>https://repo.maven.apache.org/maven2/</url>
        </repository>
    </repositories>
	
	...
	
</project>
```

观察一下 jar 的下载地址

<img src="Maven%E9%AB%98%E7%BA%A7.assets/30.png" style="zoom:67%;" />

那么我们之前配置的 `<mirror>` 又是干嘛的，与 `<repository>` 的区别是什么呢？

* 首先它是配置在 settings.xml 中，`<repository>` 一般配置在 pom.xml 中，位置不同
* 其次它的 mirror 的目的很单纯：就是为了加速，用镜像地址覆盖掉原始仓库的地址

用法示例：接下来配置镜像的 `<mirrorOf>` 为 central，在 settings.xml 中

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">

  ...
	
  <mirrors>     
     <mirror>
        <id>aliyun</id>
        <mirrorOf>central</mirrorOf>
        <name>aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror> 
  </mirrors>

  ...
	
</settings>
```

就会覆盖掉 pom.xml 中真正的 `<id>` 为 central 的仓库，取而代之，即使配置了 id 为 central 的仓库，实际也会走 mirror 的下载

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
	...

    <repositories>
        <repository>
            <id>central</id>
            <url>https://repo.maven.apache.org/maven2/</url>
        </repository>
    </repositories>
	
	...
	
</project>
```

注意：

* 最好重启 idea 让 mirror 生效
* 如果镜像的 `<mirrorOf>` 为 `*` 号，则所有仓库的下载都会走 mirror

<img src="Maven%E9%AB%98%E7%BA%A7.assets/31.png" style="zoom:67%;" />



### 私服 deploy

在公司开发时，自己开发出来的 jar 包不会放在大家都能访问的中央仓库上，公司内部都会搭建自己的私服，私服的搭建【不是重点】，我们主要学习如何将开发好的 jar 包 deploy（通俗的说法就是上传）到私服上

私服采用了 nexus 公司提供的免费软件来搭建，下载地址为：https://www.sonatype.com/nexus/repository-oss-download

#### 搭建步骤

如果自己尝试搭建，建议选择 2.x 这个旧的版本，因为更小、更简单

以 2.x 为例，下载完毕后
第一步，解压缩，会得到下面两个文件夹和如下重要目录结构

```
nexus-2.14.20-02
	|-bin
		|-jsw
			|-windows-x86-64
				|-console-nexus.bat   以黑窗口方式运行
				|-install-nexus.bat	  安装nexus为windows服务
				|-start-nexus.bat     启动nexus的windows服务
				|-stop-nexus.bat      停止nexus的windows服务
				|-uninstall-nexus.bat 卸载nexus的windows服务
			|-conf
				|-wrapper.conf
sonatype-work
```

第二步（可选），如果 java 的环境变量配置的不是 1.8 版本，修改上图中 wrapper.conf，将

```
wrapper.java.command=java
```

修改为

```
wrapper.java.command=C:\Program Files\Java\jdk1.8.0_212\bin\java
```

其中 `C:\Program Files\Java\jdk1.8.0_212\bin` 根据你 jdk 安装的实际情况改动

第三步，如果 java 的环境变量配置的是 1.8 版本，直接运行 console-nexus.bat 即可，想停止服务，按 `Ctrl+C` 或者直接关闭黑窗口即可

> 不建议装为 windows 服务，这个软件将来也应该运行在公司内网服务器上，而不是你的个人电脑上

第四步，打开浏览器

```
http://localhost:8081/nexus
```

点击右上角的 Log In 链接，输入用户名 `admin` 和密码 `admin123`

先认识一下管理界面
![](Maven%E9%AB%98%E7%BA%A7.assets/32.png)

我们要做的就是记录正式版（Releases）和快照版（Snapshots）的仓库地址，以后我们自己开发打包的 jar 就会向这两个地址去上传，同时也可以供其它公司同事下载

#### deploy 步骤

第一步，配置仓库地址，用于 deploy 上传

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
	...

    <distributionManagement>
        <repository>
            <id>releases</id>
            <url>http://localhost:8081/nexus/content/repositories/releases/</url>
        </repository>
        <snapshotRepository>
            <id>snapshots</id>
            <url>http://localhost:8081/nexus/content/repositories/snapshots/</url>
        </snapshotRepository>
    </distributionManagement>
	
	...
	
</project>
```

* releases 对应正式版的仓库
* snapshots 对应快照版的仓库

第二步，因为上传 jar 需要认证，还要在 settings.xml 中配置认证信息

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">

  ...
	
  <servers>
    <server>
      <id>releases</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
    <server>
      <id>snapshots</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
  </servers>

  ...
	
</settings>
```

其中 `<server>` 的 id 标签值要与 `<repository>` 以及 `<snapshotRepository>` 的 id 标签值一致，用户名密码这里为了简单就采用了私服管理员的，以后公司会分配个人账号和密码

第三步，运行 mvn deploy 命令，即可将对应模块的 jar 上传至私服，注意我们的 jar 的 version 是 snapshot 结尾的，最后会上传至 snapshots 对应的库

<img src="Maven%E9%AB%98%E7%BA%A7.assets/34.png" style="zoom:67%;" />

<img src="Maven%E9%AB%98%E7%BA%A7.assets/33.png" style="zoom:67%;" />

第四步，其它开发人员如果要从私服下载 jar 包，按普通方式配置仓库即可

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
	...

    <repositories>
        <!-- 其它第三方的仓库 -->
		
		<!-- 本公司的私服仓库地址 -->
        <repository>
            <id>releases</id>
            <url>http://localhost:8081/nexus/content/repositories/releases/</url>
        </repository>
        <repository>
            <id>snapshots</id>
            <url>http://localhost:8081/nexus/content/repositories/snapshots/</url>
        </repository>
    </repositories>
	
	...
	
</project>
```

还可以简化配置一个仓库 `http://localhost:8081/nexus/content/groups/public/` 它对应的仓库称为仓库组，相当于私服中多个仓库的统一入口，但仅能用于下载