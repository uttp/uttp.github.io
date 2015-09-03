---
layout: post
title: 负载均衡
tags: 负载均衡
categories: 负载均衡
published: true
---

#目录
一、NDS负载均衡</br>
二、四层负载均衡</br>
三、七层负载均衡</br>
四、缓存服务器</br>
#一、DNS负载均衡
(1)DNS简介</br>
DNS(domain name system，域名系统),主要是用于对域名和ip之间的映射，当在浏览器输入域名之后，域名解析会经过以下过程：</br>
1)浏览器缓存。查询浏览器缓存看是否有与域名对应的ip，如果有直接返回</br>
2)系统缓存。查看hosts文件看是否有待解析域名与匹配的ip地址，如果有直接返回，否则，查询本地dns缓存，如果有直接返回</br>
3)路由器缓存。查看路由器是否有对应的域名到ip地址的缓存，如果有则直接返回</br>
以上都称为客户端缓存</br>
4)本地DNS服务器。查看本地网络配置的DNS服务器缓存，如果有直接返回，否则，查看运营商的DNS服务器缓存，如果存在直接返回</br>
5)根域名服务器。本地DNS服务器里面如果没有对应的ip，则根据本地DNS服务器设置进行，一般情况下是查询根域名服务器</br>
6)顶级域名服务器。根域名服务器会返回顶级域名服务器地址给本地DNS服务器，本地DNS服务器取查找顶级域名服务器，如果在顶级域名服务器缓存有对应的解析，则直接返回</br>
7)主域名服务器。顶级域名服务器解析不了，就会返回主域名服务器，主域名服务器会去查找对应解析，找到后返回到本地DNS服务器缓存</br>
下图DNS解析过程：</br>
![dnsr] [dnsr]   </br>
</br>
浏览器输入DNS后解析过程:</br>
![bdDNSR] [bdDNSR] </br>
<br>
(2)DNS负载均衡</br>
一个域名可以对应多个ip地址，这样可以在主域名服务器设置多个ip地址来进行负载均衡</br>
DNS负载均衡有以下优点:</br>
1)DNS负载均衡实现简单，只需把负载均衡的任务交给DNS服务器就可以了</br>
2)支持地理位置的域名解析，即将域名解析成距离用户地理位置最近的服务器地址</br>
当然DNS负载均衡也有缺点</br>
1)DNS为多级解析，当某一台服务器挂掉后，DNS缓存还存在，导致获取不到服务器的信息</br>
2)DNS采用的是轮询的策略，不能按照服务器负载来实现；DNS为多级解析，这可能导致来自同一区域的访问都会集中到一台服务器上</br>
#二、四层负载均衡
根据报文中的ip地址和端口，再加上负载均衡设备设置的服务器选择方式，选择内部服务器与请求客户端建立连接，然后发送数据，典型的又LVS和F5</br>
(1)LVS负载均衡服务器</br>
LVS由三部分组成：最前端的负载均衡，中间的服务器集群，和最底端的数据共享层</br>
1)IP负载均衡技术</br>
LVS前端负载均衡服务器虚拟出一个IPVS，请求根据VIP到达负载均衡服务器，然后再由负载均衡服务器从Real Service列表中选取服务器进行相应</br>
① VS/NAT(virtual service via network address translation)</br>
过程是这样的，请求到达了负载均衡服务器后，根据负载均衡算法选择真实的服务器，把VIP和对应的端口号改为，真实服务器对应的ip和端口号。真实服务器处理了请求后，返回给负载均衡服务器，把ip地址和对口号修改为负载均衡所对应的ip地址和端口号</br>
这个过程中我们会发现一个问题就是每次响应请求都要经过负载均衡服务器，这样就导致负载均衡服务器的压力过大</br>
![vsnat] [vsnat] </br>
② VS/TUN(virtual service via IP Tunneling) </br>
过程是这样的，请求到达负载均衡服务器后，选择真实的服务器，对报文由ip隧道技术进行封装，到达真实的服务器，处理请求，返回给用户</br>
在返回请求应答的过程中不需要经过负载均衡服务器，这样就减小了负载均衡服务器的压力；而且真实服务器可以在不同的网段<br>
![vstun] [vstun]  </br>
③ VS/DR(virtual service vis Direct Routing) </br>
连接调度和管理与VS/NAT和VS/TUN一样，报文转发方法不同，转发是通过修改MAC地址，将请求直接转发给真实服务器，真实服务器响应连接请求</br>
这里要求负载均衡服务器和真实服务器都有一块网卡连在同一物理网段上</br>
![vsdr] [vsdr] </br>

2)LVS调度算法</br>
① 轮询调度</br>
也就是真实服务器访问概率是相等的</br>
② 加权轮询调度</br>
根据权重不同来进行不同的概率访问</br>
③ 最少连接调度</br>
选择真实服务器中连接数目最少的进行调度</br>
④ 加权最少连接调度</br>

<br>

(2)F5负载均衡


#三、七层负载均衡</br>
七层负载均衡是是基于http协议的，所以可以对一些静态文件进行缓存，这是四层负载均衡所不具备的</br>
(1)nginx负载均衡</br>
如下图所示nginx进程模型:</br>
![ngxm] [ngxm] </br>
nginx负载均衡算法:</br>
1)轮询</br>
每个请求按时间顺序逐一分配到不同的服务器后端服务器，如果后端服务器down掉，则自动剔除</br>
2)权重</br>
指定轮询概率，weight和访问概率成比例，主要用于后端服务性能不均的情况</br>
~~~java
upstream bakend{
	server 192.168.0.14 weight=10;
	server 192.168.0.15 weight=1;
}
~~~
3)ip_hash</br>
每个请求可以按照ip hash的结果分配，这样每个访问者访问一个固定的后端服务器，可以解决session问题</br>
~~~java
upstream bakend{
	ip_hash;
	server 192.168.0.11:8080;
	server 192.168.0.15:8080;
}

~~~

4)fair</br>
按后端服务器响应请求来分配，响应时间短的优先</br>
~~~java
upstream bakend{
	server server1;
	server server2;
	fair;
}
~~~
5)url_hash</br>
按访问url的hash结果来分配请求，使每个url定向同一服务器，后端为缓存服务器是比较有效</br>
~~~java
upstream backend{
	server varnish1:8080;
	server varnish2:8080;
	hash $request_uri;
	hash_method crc32;
}
~~~


#四、缓存服务器
(1)varnish缓存服务器</br>
varnish是一个cache型的http反向代理，通过配置可以缓存一些静态文件，如果缓存没有命中，则访问服务器</br>
(2)memcached缓存服务器</br>



[dnsr]: {{"/dnsr.jpg" | prepend: site.imgrepo}}
[bdDNSR]: {{"/bdDNSR.jpg" | prepend: site.imgrepo}}
[vsnat]: {{"/vsnat.jpg" | prepend: site.imgrepo}}
[vstun]: {{"/vstun.jpg" | prepend: site.imgrepo}}
[vsdr]: {{"/vsdr.jpg" | prepend: site.imgrepo}}
[ngxm]: {{"/ngxm.png" | prepend: site.imgrepo}}










