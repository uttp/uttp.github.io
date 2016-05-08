---
layout: post
title: 从配置文件看Spring-mvc工程之maven配置文件
tags: java spring mvc 
categories: java
published: true
---

#目录
一、概念
二、settings.xml配置文件
三、pom.xml配置文件
#一、概念
仓库<br/>
私服
#二、settings.xml配置文件
settings.xml文件为maven的全局配置文件，其默认${M2_HOME}/conf/settings.xml，所有用户共同使用此配置文件，如果用户需要配置自己的配置文件则可以把全局配置文件拷贝到用户home目录下的.m2文件夹下面，settings.xml可以配置私服用户名和密码、镜像库等。
<pre><code>
<mirrors>
	<mirror>
		<id>mirror-osc</id>
		<mirrorOf>external:*,!repo-osc-thirdparty,!repo-iss</mirrorOf>
		<url>http://maven.oschina.net/content/groups/public/<url>
	<morror>
<mirrors>
</pre></code>






