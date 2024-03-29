# 定义

Maven是Apache一款开源的项目管理工具。Maven使用项目对象模型(POM-Project Object Model,项目对象模型)的概念，可以通过一小段描述信息来管理项目的构建、报告和文档的软件项目管理工具。在maven中每个项目都相当于一个对象，对象（项目）和对象（项目）之间是有关系的，关系包含了：依赖、继承、聚合，实现Maven项目可以更加方便实现导jar包、拆分项目等效果。

## maven软件坐标

groupId：项目ID，项目所属的组织

artifactId：组件ID，具体项目的唯一标志

version：版本号，迭代开发时标志的产品版本信息

> 标本号的组成：软件名称.主版本号.小版本号.阶段版本号.字母版本号
>
> 主版本号：软件有重大功能的新增和修改
>
> 小版本号：子版本号，小功能的新增和修改
>
> 阶段版本号：BUG修复
>
> 字母版本号：里程碑意义的变动
>
> ALPHA：表示软件正式开启开发，正在实现主要功能，表示内测版本
>
> BETA：表示已经实现了基本功能，消除了一些严重错误，但是依然存在一些bug，表示的是软件开发的公测标本
>
> RC：候选版本，项目已经基本成熟，即将发行
>
> Stable：正在发行的稳定版本
>
> RELEASE / R / GA：正在发行的稳定版本
>
> FINAL：最终版本，也是正式版本中的一种表示方式

# 基础组件

## 仓库 Repository

D:/apache/apache-maven-3.6.3/conf/settings.xml

### 远程仓库/中央仓库

```xml
<mirrors>
  <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>
  </mirror> 
</mirrors>
```

### 本地仓库

```xml
<localRepository>D:/apache/apache-maven-3.6.3/usrlibs</localRepository>
```

将本地jar导入本地仓库

1. 打开cmd或者shell

2. 使用maven命令将本地jar包安装到maven的本地repository中:

   ```shell
   mvn install:install-file -Dfile=D:\apache\driver\tgdb-bolt-driver-3.2.0-20210630203418.jar -DgroupId=com.tgdb -DartifactId=tgdb-bolt-driver -Dversion=3.2.0-20210630203418 -Dpackaging=jar
   ```

3. 在项目的pom.xml文件中中加入相应的依赖maven

   ```xml
   <dependency>
   	<groupId>com.tgdb</groupId>
   	<artifactId>tgdb-bolt-driver</artifactId>
   	<version>3.2.0-20210630203418</version>
   </dependency>
   ```

4. 注意2和3中的groupId、artifactId、version保持一致

### 私有服务器

## 配置

maven在使用过程中会涉及三个配置文件，分别是全局配置文件：settings.xml、用户配置文件：settings.xmlnote、以及项目配置文件：pom.xml，这三个配置文件中如果出现相同的配置文件项，则优先级为：pom.xml > settings.xmlnote > settings.xml，实际项目中用户配置文件一般不使用，所以只需要全局配置文件以及项目配置文件。

### 全局配置

settings.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <localRepository>D:/apache/apache-maven-3.6.3/usrlibs</localRepository>
  <!--是否需要和用户交互，默认为true，一般不修改-->
  <interactiveMode>true</interactiveMode>
  <!--是否启用离线模式，默认为false，在长时间不联网的情况下这个配置十分有用-->
  <offline>false</offline>
  <!--插件的groupId没有提供的情况下，自动搜索groupId，一般不配置-->
  <pluginGroups>
  </pluginGroups>
  <!--用于配置远程仓库所在的服务器在访问时需要的身份认证信息-->
  <servers>
  </servers>
  <!--当maven需要到的依赖jar包不在本地仓库时，就需要去远程仓库下载，这时如果maven的setting.xml中配置了镜像，而且镜像配置的规则匹配到目标仓库时，maven就直接去镜像中配置的仓库地址进行依赖jar的下载。简单而言，mirror可以拦截对远程仓库的请求，改变对目标仓库的下载地址。-->
  <mirrors>
    <mirror>
      <!--镜像ID（多个镜像不能重复）-->
      <id>alimaven</id>
      <!--镜像名称-->
      <name>aliyun maven</name>
      <!--镜像仓库地址-->
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <!--拦截规则-->
      <!--mirrorOf标签为central，任何对于central中央仓库的请求都会被拦截并转发到这个阿里云的maven仓库中。-->
      <mirrorOf>central</mirrorOf>
      <!--为了满足一些复杂的需求，Maven提供了一些镜像拦截规则的配置：-->
      <!--匹配所有的远程仓库：<mirrorOf>*</mirrorOf>-->
      <!--匹配所有的远程仓库，使用localhost和使用file://协议的除外，也就是说匹配所有不在本机上的远程仓库：<mirrorOf>external:*</mirrorOf>-->
      <!--匹配仓库repo1和repo2，使用逗号分隔多个远程仓库：<mirrorOf>repo1,repo2</mirrorOf>-->
      <!--匹配所有的远程仓库，repo1除外，使用感叹号将仓库从匹配中排除：<mirrorOf>*,!repo1</mirrorOf>-->
      <!--注意！镜像仓库会完全屏蔽掉被镜像仓库，即镜像仓库失效后，maven不会再去访问被屏蔽掉的仓库。-->
    </mirror> 
  </mirrors>
  <!--配置连接仓库的代理-->
  <proxies>
  </proxies>
  <!--全局配置项目构建参数的列表，一般情况通过它来完成一些特定环境的定制化操作，例如配置全局的jdk版本也是通过它来完成的-->
  <profiles>
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
  </profiles>
  <!-- 配置远程仓库列表，用于项目开发时多仓库的位置 -->
  <repositories>
  </repositories>
</settings>

```

### 项目配置

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <!-- 1、项目基础信息配置 -->
    <!-- 父项目的坐标。如果项目中没有规定某个元素的值，那么父项目中的对应值即为项目的默认值。 -->
    <parent>
        <!-- 被继承的父项目的组织的唯一标识符 -->
        <groupId>org.springframework.boot</groupId>
        <!-- 被继承的父项目的唯一标识符 -->
        <artifactId>spring-boot-starter-parent</artifactId>
        <!-- 被继承的父项目的版本 -->
        <version>2.4.10</version>
        <!-- 父项目的pom.xml文件的相对路径。默认值为../pom.xml。Maven首先在构建当前项目的地方寻找父项目的pom.xml，其次在文件系统的relativePath位置找，然后在本地仓库，最后在远程仓库寻找父项目的pom。-->
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <!--声明项目描述符遵循哪一个POM模型版本。-->
    <modelVersion>4.0.0</modelVersion>
    <!--项目组织唯一的标识符。在继承关系中子项目会自动继承父项目的groupId所以子项目不需要定义groupId-->
    <groupId>com.duo</groupId>
    <!--项目唯一的标识符-->
    <artifactId>springcloud</artifactId>
    <!--项目产生的构建类型，例如：jar、war、ear、pom。父级项目的packaging必须为pom，packaging默认类型jar类型，如果不做配置，maven会将该项目打成jar包-->
    <packaging>pom</packaging>
    <!--项目当前版本，格式为：主版本.次版本.增量版本-限定版本号-->
    <version>1.0</version>
    <!--项目的名称，Maven产生的文档用-->
    <name>springcloud</name>
    <!--项目主页的URL，Maven产生的文档用-->
    <url>http://www.duo.org</url>
    <!--项目的详细描述，Maven产生的文档用-->
    <description>Parent Project</description>
    
    <!-- 2、项目构建环境配置 -->
    <!--聚合项目和父子项目是两个概念-->
    <!--聚合项目中父级项目中通过modules将子项目包含进来，父级项目在进行compile,package,install,deploy等项目构建的时候，会把所有的子项目都进行相应的构建-->
    <!--父子项目是通过在子项目中添加parent标签来建立项目之间的依赖关系，实现统一的依赖管理及控制插件的版本-->
    <!--一般在生产环境中，会将两种方式结合起来使用-->
    <!--聚合项目中：子项目模块 -->
    <modules>
        <!--声明 eureka_server 为当前项目子模块 ，以后有新的子模块添加也要在此添加-->
        <module>eureka_server</module>
        <module>producer_service</module>
        <module>consumer_service</module>
    </modules>
    <!--构建项目需要的信息-->
    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                </plugin>
            </plugins>
        </pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <version>3.2.0</version>
            </plugin>
        </plugins>
    </build>
    <profiles>
        <profile>
        </profile>
    <profiles>

    <!-- 3、项目仓库管理配置 -->
    <!--一个pom.xml中可以同时配置多个仓库-->
    <repositories>
        <repository>
            <!--远程仓库唯一标识符-->
            <id></id>
            <!--远程仓库名称-->
            <name></name>
             <!--远程仓库URL-->
            <url></url>
            <!--用于定位和排序构建的仓库布局类型-可以是default(默认)或者legacy(遗留)。Maven2为其仓库提供了一个默认的布局。然而Maven1.x有两种不同的布局，可以使用该元素指定布局是default还是legacy。-->
            <layout></layout>
        </repository>
    </repositories>
    <!-- 项目发布仓库管理 -->
    <distributionManagement>
      <!-- 稳定版本仓库地址 -->
      <repository>
        <id>release-rep</id>
        <name></name>
        <url></url>
      </repository>
      <!-- 快照版本仓库地址 -->
      <snapshotRepository>
        <id>snapshot-rep</id>
        <name></name>
        <url></url>
      </snapshotRepository>
    </distributionManagement>
    <!-- 4、项目依赖管理配置 -->
	<!-- maven继承机制， -->
	<!-- 如果配置了parent节点，子模块的依赖从父模块得到了继承，即使在子模块中不配置<dependencies>节点，子模块也引入了它的依赖。 -->
	<!-- 但是上述配置存在一个问题，假设模块A中并不需要fastjson依赖，但由于继承，也同样从父模块中继承了fastjson依赖。换句话说，以后继承自该parent模块的其他子模块也许并不需要parent模块中声明的所有依赖，但是就目前这种配置方式，父模块中的这些依赖都会被无条件的继承，这显然是不合理的，这就需要用到<dependencyManagemnt>节点。 -->
	<!-- <dependencyManagemnt>节点中配置的依赖并不会真正的引入依赖，（即不会下载jar到本地仓库）但是该节点却能够被子模块继承，要在子模块中真正引入依赖，需要将依赖配置在<dependencies>节点下，而这种方式配置的依赖可以省略<version>节点，使用的<version>即是在<dependencyManagement>节点中配置的相应<version>。 -->
	<!-- dependencyManagement里只是声明依赖，并不实现引入，因此子项目需要显示的声明需要用的依赖。如果不在子项目中声明依赖，是不会从父项目中继承下来的；只有在子项目中写了该依赖项，并且没有指定具体版本，才会从父项目中继承该项，并且version和scope都读取自父pom;另外如果子项目中指定了版本号，那么会使用子项目中指定的jar版本。 -->
	<!-- <dependencyManagement>节点的另一个好处是，使得多个子模块中引入的相同依赖版本号能够保持一致。那如何在一个模块中引入另外一个模块的dependencyManagement呢？ -->
	<!-- 假设在某个模块demo-other中也有<dependencyManagement>节点管理着一批依赖，现在想在demo-parent模块中将那些依赖管理全部导入，把demo-other模块中的依赖全部复制到demo-parent模块的<dependencyManagement>节点下是可行但是非常不优雅的做法。比较优雅的做法是用到import依赖范围，即<scope>import</scope>。 -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <!--父项目中的基本依赖，所有的子项目都会继承-->
    <dependencies>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
            <!--maven中scope的范围 编译，测试，运行-->
            <!--1.compile：默认范围，编译测试运行都有效-->
            <!--2.provided：在编译和测试时有效-->
            <!--3.runtime：在测试和运行时有效-->
            <!--4.test:只在测试时有效-->
            <!--5.system:在编译和测试时有效，与本机系统关联，可移植性差-->
			<scope>runtime</scope>
		</dependency>
    </dependencies>

    <!-- 5、项目报表信息配置 -->
    <!--以值替代名称，properties可以在整个POM中使用。-->
    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>2020.0.3</spring-cloud.version>
        <producer-client.version>1.0-RELEASE</producer-client.version>
        <producer-common.version>1.0-RELEASE</producer-common.version>
    </properties>

    <!-- 6、项目部署分发配置 -->
    <!--执行mvn deploy后要发布的位置。有了这些信息就可以将应用部署到远程服务器或者把构件部署到远程仓库-->
    <distributionManagement>
        <!--部署项目产生的构件到远程仓库需要的信息-->
        <repository>
            <uniqueVersion/>
            <id>xxx-maven2</id>
            <name>xxx maven2</name>
            <url>file://basedir/targe/deploy</url>
            <layout>default</layout>
        </repository>
        <!--构件快照部署的位置，如果没有配置该元素，默认部署到repository元素配置的仓库-->
        <snapshotRepository>
            <uniqueVersion/>
            <id>xxx-snapshot-maven2</id>
            <name>xxx snapshot maven2</name>
            <url>scp://svn.xxx.com/xxx:/usr/local/maven-snapshot</url>
            <layout>default</layout>
        </snapshotRepository>
        <!--部署项目的网站需要的信息-->
        <site>
            <!--部署位置的唯一标识符，用来匹配站点和settings.xml文件中的配置-->
            <id>base-site</id>
            <!--部署位置的名称-->
            <name>business api website</name>
            <!--部署位置的URL，按protocol://hostname/path形式-->
            <url>scp://svn.baidu.com/xxx:/var/www/localhost/web</url>
        </site>
    </distributionManagement>
</project>
```

## mvn命令

使用mvn命令的方式构建项目时候，需要设置JAVA_HOME

### 项目构建命令

```shell
# 使用命令通过模板构建maven项目，maven-archetype-quickstart就是通过idea构建maven项目时选择的archetype
mvn archetype:generate -DgroupId=org.duo -DartifactId=trains-01 -DpackageName=org.duo -DarchetypeArtifactId=maven-archetype-quickstart
#编译生成class文件
mvn compile
#构建war包或者jar包
mvn package
#运行
mvn exec:java -Dexec.mainClass="org.duo.App"
```

### 项目清理/编译/打包

```shell
mvn clean
#编译生成class文件
mvn compile
#构建war包或者jar包
mvn package
#将项目安装到本地仓库中
mvn install
```

### 项目运行/测试/发布

```shell
mvn tomcat:run
mvn test
mvn site
mvn dependency:tree
#将项目安装到本地仓库中
mvn install
#将项目发布到远程仓库，即在pom.xml中配置的distributionManagement中的仓库地址，如果该远程仓库需要验证身份新的话则需要在settings.xml的<servers>增加一个<server>并根据在distributionManagement设置的id来设置远程仓库的身份认证信息
mvn deploy
```

## archetype

archetype项目骨架加载慢的问题：

1. 下载对应版本的archetype-catalog.xml

   https://repo.maven.apache.org/maven2/archetype-catalog.xml

2. 将配置文件放入本地仓库对应的目录中：

   D:\apache\maven\repository\org\apache\maven\archetype\archetype-catalog\3.2.1

3. 修改Intellij-IDEA

   File==>Settings=>Build==>Build Tools==>Maven==>Runner==>VM Options:-DarchetypeCatalog=local

将项目定义成archetype：

1. 进入自定义的maven项目中
2. 执行：mvn archetype:create-from-project，表示通过已存在的项目来构建骨架
3. 在构建过程中如果报mvn.bat找不到，就在MAVEN_HOME的bin目录下复制一份mvn.cmd重命名成mvn.bat
4. 添加到本地仓库：mvn clean install
5. 添加到自定义骨架：Intellij-IDEA==>File==>New Project==>Maven==>Create from archetype==>Add  Archetype(输入：groupId、artifactId、version)

# 项目管理

## Maven依赖范围管理

1. compile 编译、运行、测试、打包都依赖的jar包，如开发项目时对spring-core的依赖就是compile范围。默认的依赖范围就是compile
2. provided 只有在编译和运行时有效，打包时不会包含这样的jar包，如servlet-api容器相关的依赖
3. system 本地jar包，作用范围和provided一致，但是必须配合systemPath指定本地依赖的路径才能使用
4. runtime 只有在运行时有效，但是打包时会将对应的jar包包含进来，如jdbc驱动在编译时是不需要参与的，但是运行时是需要具体的第三方实现的jar包的
5. test 只有在测试的时候有效，在编译和运行以及打包时都不会使用，如测试使用的junit依赖就是test范围

## 父子项目

1. 父项目的打包方式必须是pom，即：<packaging>pom</packaging>

   ```xml
   <packaging>pom</packaging>
   ```

2. 子项目：

   ```xml
   <!--继承关系：继承一个项目-->
   <parent>
       <groupId>org.duo</groupId>
       <artifactId>WebApp</artifactId>
       <version>1.0-SNAPSHOT</version>
       <relativePath>../springcloud/pom.xml</relativePath>
   </parent>
   <!--继承关系中，子项目会自动继承父项目的groupId-->
   <!--因此子项目只需要定义artifactId、version和name即可-->
   <artifactId>Sub-Web</artifactId>
   <version>1.0-SNAPSHOT</version>
   <name>Sub-Web</name>
   ```

优点：合理有效的复用依赖jar包，子项目互相独立，更加便于敏捷开发和独立管理。

缺陷：如果子项目比较多的情况下，项目之间的系统集成性能较差，进行系统部署时，需要对每一个项目打包，最后再将所有的打包结果手工整理到一起

## 项目聚合

项目聚合是maven中针对多个项目进行统一打包的一种管理方式，在项目聚合关系中，项目之间的整理性较高，便于系统集成和维护；但是在聚合项目关系中，子项目和聚合项目之间，它们的依赖是独立管理的，不利于依赖的复用，所以一般让子项目继承聚合项目。

1. 聚合项目的顶级

   ```xml
   <groupId>com.duo</groupId>
   <artifactId>springcloud</artifactId>
   <version>1.0</version>
   <name>springcloud</name>
   <!--在聚合项目中，顶级项目不一定是子项目的父项目，所以顶级项目的打包方式不一定是pom，但是在实际的项目开发过程中一般会将项目聚合和父子项目结合起来使用，所以一般会把聚合项目的顶级项目设置为子模块的父项目-->
   <packaging>pom</packaging>
   <!--聚合项目中：子项目模块，在对顶级项目打包时，会同时对所有的子项目进行打包-->
   <modules>
       <!--声明 eureka_server 为当前项目 子模块 ，以后有新的子模块添加也要在此添加-->
       <module>eureka_server</module>
       <module>producer_service</module>
       <module>consumer_service</module>
   </modules>
   ```

2. 子项目

   在聚合项目中子项目是以模块的形式创建的：Intellij-IDEA==>File==>New==>Module，并且在创建过程中以可以选择是否将顶级项目设置为该模块的父项目。

   ```xml
   <!-- 在聚合项目中，顶级项目跟子模块是没有继承关系的，只是在实际开发过程中继承关系有利于依赖jar包的复用，所以一般会将顶级项目设置为子模块的父项目 -->
   <parent>
       <!--父工程 gropuId-->
       <groupId>com.duo</groupId>
       <!--父工程 artifactId-->
       <artifactId>springcloud</artifactId>
       <!--父工程 版本-->
       <version>1.0</version>
       <!--在聚合项目中relativePath可以省略-->
       <!-- <relativePath>../pom.xml</relativePath>-->
   </parent>
   <!--在继承关系中，子项目会自动继承父项目的groupId，所以子项目不需要定义groupId-->
   <artifactId>consumer_service</artifactId> 
   ```


## Maven构建JaveWEB项目

1. Intellij-IDEA==>File==>New==>Project==>Maven(不使用骨架创建项目，即不勾选Create from archtype)

2. 在main目录下创建webapp目录，然后再在webapp下创建WEB-INF，最后在WEB-INF目录下创建web.xml：

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
            version="3.1">
       <welcome-file-list>
           <welcome-file>/index.jsp</welcome-file>
       </welcome-file-list>
   </web-app>
   ```

3. 将项目转换成web项目：Intellij-IDEA==>File==>Project Structure...==>Modules==>Add==>Web

   - 修改Deployment Descriptors中web.xml的路径
   - 修改Web Resource Directories中的web目录，即WEB-INF目录的父目录：webapp目录
   - 点击Create Artifact创建artifact

4. 在webapp目录下创建index.jsp

   ```jsp
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <html>
   <head>
       <title>Title</title>
   </head>
   <body>
   <H1>我的WEB项目</H1>
   </body>
   </html>
   ```

5. 修改pom文件

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <groupId>org.duo</groupId>
       <artifactId>WebApp</artifactId>
       <version>1.0-SNAPSHOT</version>
       <packaging>war</packaging>
       <properties>
           <maven.compiler.source>8</maven.compiler.source>
           <maven.compiler.target>8</maven.compiler.target>
       </properties>
       <dependencies>
           <dependency>
               <groupId>javax.servlet</groupId>
               <artifactId>javax.servlet-api</artifactId>
               <version>3.1.0</version>
               <scope>provided</scope>
           </dependency>
       </dependencies>
   </project>
   ```

6. 创建servlet

   ```java
   package org.duo;
   
   import javax.servlet.annotation.WebServlet;
   import javax.servlet.http.HttpServlet;
   import javax.servlet.http.HttpServletRequest;
   import javax.servlet.http.HttpServletResponse;
   import java.io.IOException;
   
   @WebServlet(value = "/hello")
   public class HelloServlet extends HttpServlet {
   
       @Override
       protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
           resp.getWriter().println("hello maven web!");
       }
   }
   ```

7. 项目启动：Intellij-IDEA==>Edit Configurations

   - Server==>Configure...==>Tomcat Home:设置tomcat home目录
   - Deployment==>add==>Artifact...==>WebApp

# Maven打包

## maven-jar-plugin

```xml
<build>  
    <plugins>  
        <plugin>  
            <groupId>org.apache.maven.plugins</groupId>  
            <artifactId>maven-jar-plugin</artifactId>  
            <version>2.6</version>  
            <configuration>  
                <archive>  
                    <manifest>
                        <!-- 会在MANIFEST.MF加上Class-Path项并配置依赖包 -->
                        <addClasspath>true</addClasspath>
                        <!-- 指定依赖包所在目录 -->
                        <classpathPrefix>lib/</classpathPrefix>
                        <!--指定MANIFEST.MF中的Main-Class-->
                        <mainClass>com.xxg.Main</mainClass>
                    </manifest>  
                </archive>  
            </configuration>  
        </plugin>  
        <plugin>  
            <groupId>org.apache.maven.plugins</groupId>  
            <artifactId>maven-dependency-plugin</artifactId>  
            <version>2.10</version>  
            <executions>  
                <execution>  
                    <id>copy-dependencies</id>  
                    <phase>package</phase>  
                    <goals>  
                        <goal>copy-dependencies</goal>  
                    </goals>  
                    <configuration>  
                        <outputDirectory>${project.build.directory}/lib</outputDirectory> 
                    </configuration>  
                </execution>  
            </executions>  
        </plugin>  
  
    </plugins>  
</build>  
```

maven-jar-plugin用于生成META-INF/MANIFEST.MF文件的部分内容，例如下面是一个通过maven-jar-plugin插件生成的MANIFEST.MF文件片段：

```ini
Class-Path: lib/commons-logging-1.2.jar lib/commons-io-2.4.jar  
Main-Class: com.xxg.Main  
```

只是生成MANIFEST.MF文件还不够，maven-dependency-plugin插件用于将依赖包拷贝到outputDirectory指定的位置，即lib目录下。配置完成后，通过mvn package指令打包，会在target目录下生成jar包，并将依赖包拷贝到target/lib目录下。指定了Main-Class，有了依赖包，那么就可以直接通过java -jar xxx.jar运行jar包。这种方式生成jar包有个缺点，就是生成的jar包太多不便于管理。

## maven-assembly-plugin

```xml
<build>  
    <plugins>  
        <plugin>  
            <groupId>org.apache.maven.plugins</groupId>  
            <artifactId>maven-assembly-plugin</artifactId>  
            <version>2.5.5</version>  
            <configuration>  
                <archive>  
                    <manifest>  
                        <mainClass>com.xxg.Main</mainClass>  
                    </manifest>  
                </archive>  
                <descriptorRefs>  
                    <descriptorRef>jar-with-dependencies</descriptorRef>  
                </descriptorRefs>  
            </configuration>  
        </plugin>  
    </plugins>  
</build>  
```

打包方式：mvn package assembly:single；打包后会在target目录下生成一个xxx-jar-with-dependencies.jar文件，这个文件不但包含了自己项目中的代码和资源，还包含了所有依赖包的内容。所以可以直接通过java -jar来运行。此外还可以直接通过mvn package来打包，无需assembly:single，不过需要加上一些配置：

```xml
<build>  
    <plugins>
        <plugin>  
            <groupId>org.apache.maven.plugins</groupId>  
            <artifactId>maven-assembly-plugin</artifactId>  
            <version>2.5.5</version>  
            <configuration>  
                <archive>  
                    <manifest>  
                        <mainClass>com.xxg.Main</mainClass>  
                    </manifest>  
                </archive>  
                <descriptorRefs>  
                    <descriptorRef>jar-with-dependencies</descriptorRef>  
                </descriptorRefs>  
            </configuration>  
            <executions>  
                <execution>  
                    <id>make-assembly</id>  
                    <phase>package</phase>  
                    <goals>  
                        <goal>single</goal>  
                    </goals>  
                </execution>  
            </executions>  
        </plugin>
    </plugins>  
</build>  
```

其中<phase>package</phase>、<goal>single</goal>即表示在执行package打包时，执行assembly:single，所以可以直接使用mvn package打包。不过，如果项目中用到spring Framework，用这种方式打出来的包运行时会出错。可以使用maven-shade-plugin或者spring-boot-maven-plugin解决：

## maven-shade-plugin

maven-plugin-shade 插件提供了两个能力：

1. 把整个项目（包含它的依赖）都打包到一个 "uber-jar" 中

   uber-jar 也叫做 fat-jar 或者 jar-with-dependencies，意思就是包含依赖的 jar。

2. shade - 即重命名某些依赖的包

   shade 意为遮挡，在此处可以理解为对依赖的 jar 包的重定向（主要通过重命名的方式）

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.2.4</version>
            <configuration>
                <!-- 此处按需编写更具体的配置 -->
            </configuration>
            <executions>
                <execution>
                    <!-- 和 package 阶段绑定 -->
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### includes

使用 <includes> 排除不需要的依赖，示例如下：

```xml
<configuration>
    <artifactSet>
        <excludes>
            <exclude>classworlds:classworlds</exclude>
            <exclude>junit:junit</exclude>
            <exclude>jmock:*</exclude>
            <exclude>*:xml-apis</exclude>
            <exclude>org.apache.maven:lib:tests</exclude>
            <exclude>log4j:log4j:jar:</exclude>
        </excludes>
    </artifactSet>
</configuration>
```

使用 `<filters>` 结合 `<includes>` & `<excludes>` 标签可实现更灵活的依赖选择，示例如下：

```xml
<configuration>
    <filters>
        <filter>
            <artifact>junit:junit</artifact>
            <includes>
                <include>junit/framework/**</include>
                <include>org/junit/**</include>
            </includes>
            <excludes>
                <exclude>org/junit/experimental/**</exclude>
                <exclude>org/junit/runners/**</exclude>
            </excludes>
        </filter>
        <filter>
            <artifact>*:*</artifact>
            <excludes>
                <exclude>META-INF/*.SF</exclude>
                <exclude>META-INF/*.DSA</exclude>
                <exclude>META-INF/*.RSA</exclude>
            </excludes>
        </filter>
    </filters>
</configuration>
```

除了可以通过自定义的 filters 来过滤依赖，此插件还支持自动移除项目中没有使用到的依赖，以此来最小化 jar 包的体积，只需要添加一项配置即可。示例如下：

```xml
<configuration>
    <minimizeJar>true</minimizeJar>
</configuration>
```

### 重定位class文件 

如果最终的 jar 包被其他的项目所依赖的话，直接地引用此 jar 包中的类可能会导致类加载冲突，这是因为 classpath 中可能存在重复的 class 文件。为了解决这个问题，我们可以使用 shade 提供的重定位功能，把部分类移动到一个全新的包中。示例如下：

```xml
<configuration>
    <relocations>
        <relocation>
            <pattern>org.codehaus.plexus.util</pattern>
            <shadedPattern>org.shaded.plexus.util</shadedPattern>
            <excludes>
                <exclude>org.codehaus.plexus.util.xml.Xpp3Dom</exclude>
                <exclude>org.codehaus.plexus.util.xml.pull.*</exclude>
            </excludes>
        </relocation>
    </relocations>
</configuration>
```

涉及标签：

- `<pattern>`：原始包名
- `<shadedPattern>`：重命名后的包名
- `<excludes>`：原始包内不需要重定位的类，类名支持通配符

例如，在上述示例中，我们把 org.codehaus.plexus.util 包内的所有子包及 class 文件（除了 ~.xml.Xpp3Dom 和 ~.xml.pull 包下的所有 class 文件）重定位到了 org.shaded.plexus.util 包内。当然，如果包内的大部分类我们都不需要，一个个排除就显得很繁琐了。此时我们也可以使用 `<includes>` 标签来指定我们仅需要的类，示例如下：

```xml
<project>
    ...
    <relocation>
        <pattern>org.codehaus.plexus.util</pattern>
        <shadedPattern>org.shaded.plexus.util</shadedPattern>
        <includes>
            <include>org.codehaud.plexus.util.io.*</include>
        </includes>
    </relocation>
    ...
</project>
```

### 生成可执行jar包

使用 maven-plugin-shade 后，最终生成的 jar 包可以包含所有项目所需要的依赖。如果想生成可执行的jar则只需要指定 `<mainClass>` 启动类就可以了。示例如下：

```xml
<project>
    ...
    <configuration>
        <transformers>
            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                <mainClass>org.sonatype.haven.HavenCli</mainClass>
            </transformer>
        </transformers>
    </configuration>
    ...
</project>
```

jar包中默认会包含一个MANIFEST.MF文件，里面描述了一些jar包的信息。使用java自带的jar命令打包的时候可以指定MANIFEST.MF，其中也可以指定Main-Class来使得jar包可运行。那么使用shade来指定和直接在MANIFEST.MF文件中指定有什么区别呢？答案是没有区别，仔细观察可发现<mainClass>标签的父标签是<transformer>有一个implementation属性，其值为"~.ManifestResourceTransformer"，意思是Manifest资源文件转换器。上述示例只自指定了启动类，因此shade会自动生成一个包含Main-Class的MANIFEST.MF文件，然后在打jar包时指定这个文件。如果想要完全定制MANIFEST.MF文件则可以使用<manifestEntries>标签，示例如下：

```xml
<project>
    ...
    <configuration>
        <transformers>
            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                <manifestEntries>
                    <Main-Class>org.sonatype.haven.ExodusCli</Main-Class>
                    <Build-Number>123</Build-Number>
                </manifestEntries>
            </transformer>
        </transformers>
    </configuration>
    ...
</project>
```

## spring-boot-maven-plugin

maven-shade-plugin、spring-boot-maven-plugin这两种plugin打出的可执行的jar包，都是叫fat-jar(uber-jar)，区别如下：

maven-shade-plugin打的fat-jar，所有的jar包全部展开了，只有class文件（它可以将多个JAR文件合并为一个JAR文件，并避免在运行时出现冲突。它通过重命名和打包类来解决冲突，从而使得项目能够轻松部署和运行。）。而spring-boot-maven-plugin打的包是jar in jar,重写了类加载器,类似war包

```xml
<build>
    <!-- 生成的jar包或者war包的名称 -->
    <finalName>springboot</finalName>
    <plugins>
        <!-- 解决 Description Resource Path Location Type Dynamic Web Module 3.0
            requires Java 1.6 or newer -->
        <!-- 这个maven插件同时解决了， maven update project JDK变回1.5 问题 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
        <!-- 打成jar包部署或者直接使用java -jar命令的时候，提示了xxxxxx.jar中没有主清单属性： 主清单属性即：META-INF，其中META-INF文件夹下有一个MANIFEST.MF文件，该文件指明了程序的入口以及版本信息等内容 -->
        <!-- 解决办法：在pom中添加一个SpringBoot的构建的插件，然后重新运行 mvn install即可。 -->
        <!-- 即如果将SpringBoot打成jar运行，则需要添加一个spring-boot-maven-plugin到pom.xml中 -->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <!-- SpringBoot集成Jsp 以jar包的方式运行，必须指定spring-boot-maven-plugin的版本为1.4.2，其他版本不兼容 -->
            <!-- <version>1.4.2.RELEASE</version> -->
            <!--在不使用spring-boot-start-parent构建springboot应用的时候必须加入repackage配置后，maven打包时才会把第三方jar包一起打入 -->
            <!-- 代表maven打包时会将外部引入的jar包（比如在根目录下或resource文件下新加外部jar包）打包到项目jar，在服务器上项目才能运行，不加此配置，本地可以运行，因为本地可以再lib下找到外部包，但是服务器上jar中是没有的。 -->
            <!-- 并且外部引入的jar的scope必须设置为system -->
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
            <!-- executable设置为true表示在jar中添加启动脚本，制作在linux环境下可执行的jar。即：在linux环境下，通过chmod a+x后可以通过./app.jar的方式运行，但是这种打包方式将无法使用jar -xvf解压jar包（可以使用unzip命令进行解压） -->
            <!-- executable设置为false的话，则是指定普通的可执行jar，通过java -jar命令运行，并且也可以通过jar -xvf解压jar包 -->
            <!-- 仅在打算直接执行时才启用此选项，一般情况下都设置为false -->
            <configuration>
                <includeSystemScope>true</includeSystemScope>
                <executable>false</executable>
            </configuration>
        </plugin>
        <!-- war包打包插件 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-war-plugin</artifactId>
            <configuration>
                <failOnMissingWebXml>false</failOnMissingWebXml>
            </configuration>
        </plugin>
    </plugins>
</build>
```

