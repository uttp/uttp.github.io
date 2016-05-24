---
layout: post
title: 从配置文件看Spring-mvc工程之maven配置文件
tags: java spring mvc 
categories: java
published: true
---

#目录
一、概念<br/>
二、settings.xml配置文件<br/>
三、pom.xml配置文件<br/>

#一、概念
仓库<br/>
私服
#二、settings.xml配置文件
　　settings.xml文件为maven的全局配置文件，其默认${M2_HOME}/conf/settings.xml，所有用户共同使用此配置文件，如果用户需要配置自己的配置文件则可以把全局配置文件拷贝到用户home目录下的.m2文件夹下面，settings.xml可以配置私服用户名和密码、镜像库等。
~~~java
<mirrors>
	<mirror>
		<id>mirror-osc</id>
		<mirrorOf>external:*,!repo-osc-thirdparty,!repo-iss</mirrorOf>
		<url>http://maven.oschina.net/content/groups/public/<url>
	<morror>
<mirrors>
~~~
上段是settings.xml中镜像的配置文件，镜像是maven仓库的映射，通过镜像来代理其它仓库。一个镜像一般有三个属性：id、mirrorOf和url，id是镜像的唯一标识符，mirrorOf为镜像代理仓库的id，url为镜像地址。*表示代理所有的仓库，external:*表示代理除了本地仓库以外的所有仓库，!repo表示代理的仓库中除去repo仓库。上面那段配置文件表示的是除了映射本地仓库、repo-osc-thirdparty仓库和repo-iss仓库外，其它的仓库都映射到url所指定的仓库地址。
~~~java
<servers>
	<server>
		<id>sanpshots</id>
		<username>username1</username>
		<password>pass1</password>
	</server>

	<server>
		<id>repo</id>
		<username>username2</username>
		<password>pass2</password>
	</server>
</servers>
~~~
这段配置文件是用于配置从私服上下载包和部署包到私服需要验证的用户名和密码，其id和仓库id 一致。
#三、pom.xml文件
其基本属性如groupId、artifactId、packaging、properties、dependies等常见属性不再说明。这里主要看repository、build、distributionManagement和profile属性。
~~~java
<repository>
	<id>central</id>
	<releases>
		<enabled>true</enabled>
	</releases>
	<snapshots>
		<enabled>true</enabled>
	</snapshots>		
	<url>http://repo1.maven.org/maven2</url>
</repository>
~~~
repository属性表明了仓库的id、仓库中下载的版本类型和仓库地址
~~~java
<build>
   <outputDirectory>target/classes</outputDirectory>
   <sourceDirectory>src/main/java</sourceDirectory>
   <testSourceDirectory>src/test/java</testSourceDirectory>
   <plugins>
      <plugin>
	<groupId>org.codehaus.mojo</groupId>
        <artifactId>build-helper-maven-plugin</artifactId>
        <version>1.9.1</version>
        <executions>
           <execution>
              <phase>generate-sources</phase>
              <goals>
                 <goal>add-source</goal>
              </goals>
              <configuration>
                 <sources>
                    <source>src/core/java</source>
                 </sources>
              </configuration>
           </execution>
        </executions>
     </plugin>
     <plugin>
        <groupId>org.apache.tomcat.maven</groupId>
        <artifactId>tomcat7-maven-plugin</artifactId>
        <version>2.1</version>
        <configuration>
             <url>http://localhost:8081/manager/text</url>
             <!--<server>tomcat</server>—>
             <username>tomcat</username>
             <password>tomcat</password>
             <path>/</path>
        </configuration>
    </plugin>
   </plugins>
   <resources>
      <resource>
         <directory>src/main/resources</directory>
      </resource>
      <resource>
         <directory>src/core/resources</directory>
      </resource>
   </resources>
</build>
~~~
outputDirecotry属性规定了mvn compile编译项目编译时目标文件的位置，sourceDirectory规定了java编译源文件的位置，testSourceDirectory规定了测试源文件的位置，resources规定了资源文件的位置，资源文件在编译后都会放到目标文件的根目录下，在spring-mvc配置文件中classpath与配置的resource对应。一般maven默认的源文件是在src/main/java中，破坏这种默认的目录结构使有些人难以看懂，但是有些项目就是配置了多个源文件的位置，它是利用plugin属性了，配置builder-helper-maven-plugin插件可以，如上配置文件中配置了src/core/java也为源文件的位置，这样在编译时也会编译src/core/java文件到目标文件，如下图对应上面配置文件生编译后生成的目录结构</br>
![CompileCatalogStructure][CompileCatalogStructure]</br>
在预发布环境中实现分支一键部署到指定的服务器，我觉得其中一种部署脚本实现可以是这样的：一、拉取分支；二、修改pom.xml文件中的tomcat7-maven-plugin插件的url属性，使其对应指定的工程部署的端口号（一个预发布环境有多个tomcat服务器）；三、运行 mvn clean tom
cat7:redeploy命令。其中重要的一环是tomcat7-maven-plugin插件的配置，url配置属性指定了已经开启了的tomcat服务器的管理页，通过管理页可以部署war包到对应的服务器，username属性和password属性是登录管理页面的验证（另一种配置是在settings.xml文件中配置），其配置应该和tomcat服务器conf/tomcat-users.xml中的配置对应，如下图是部署成功后显示的log信息</br>
![mvnDeploySuccess] [mvnDeploySuccess]</br>
~~~java
<distributionManagement>
   <repository>
      <id>repo</id>
      <name>Netease Maven Repository</name>
      <url>http://mvn.hz.netease.com/artifactory/libs-releases</url>
   </repository>

   <snapshotRepository>
      <id>snapshots</id>
      <name>Netease Maven Repository</name>
      <url>http://mvn.hz.netease.com/artifactory/libs-snapshots</url>
   </snapshotRepository>
</distributionManagement>
~~~
mvn deploy命令是部署项目生成的构建到远程仓库，distributionManagement是配置远程仓库的基本信息，远程仓库分为release版本和snapshot版本，对应的仓库认证信息配置在settings.xml文件对应的（id相同）server中。
~~~java
<profile>
   <id>local</id>
   <activation>
      <activeByDefault>true</activeByDefault>
   </activation>
   <build>
      <resources>
         <resource>
            <directory>src/main/resources/local</directory>
         </resource>
      </resources>
   </build>
</profile>
<profile>
   <id>online</id>
   <build>
      <resources>
         <resource>
            <directory>src/main/resources/online</directory>
         </resource>
      </resources>
   </build>
</profile>
~~~
在本地、预发布和线上的环境配置很多不一样，把它们之间配置的差异性放在不同的资源文件中，在不同环境时进行切换，profile实现了这一功能,通过mvn package -P profileName可以指定打包使用的profile。


[CompileCatalogStructure]: {{"/spring-mvc_compile_catalog_structure.png" | prepend: site.imgrepo }}

[mvnDeploySuccess]: {{"/spring-mvc_mvn_deploy_success.png" =100x100 | prepend: site.imgrepo }}
