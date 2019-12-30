---
title: elasticsearch安装
date: 2019-12-09 16:19:44
tags:
---

1. wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.3.0.tar.gz

2. tar -zxvf elasticsearch-5.3.0.tar.gz

3. cd elasticsearch-5.3.0/bin

4. ./elasticsearch

   

<!--more-->

##### 启动报错了

---

错误1

---

![image-20191209162256025](/img/elasticsearch/image-20191209162256025.png)

solutions:

在linux环境中，elasticstatic不允许以root权限运行，所以需要创建一个非root用户，用非root用户启动es

步骤：

+ groupadd elk

+ useradd elk -g elk -p 111111

+ chown -R elk:elk /opt/es

+ su elk

+ 启动

  

---

错误2

---

![image-20191209163923583](/img/elasticsearch/image-20191209163923583.png)

solutions：

es启动依赖JRE,环境上安装一下jdk就可以了

步骤：

- yum list java*  获取可安装列表
- yum install java-1.8.0-openjdk-devel.x86_64 
- 安装完之后java -version看下
- 然后启动，ok

![image-20191209165044291](/img/elasticsearch/image-20191209165044291.png)



##### 测试下

![image-20191209165820165](/img/elasticsearch/image-20191209165820165.png)

ok，成功访问



##### 为了方便访问，将network地址设置为0.0.0.0，然后启动

修改文件为 config/elasticsearch.yml  设置network.host: 0.0.0.0

---

报错3

---

![image-20191209172442177](/img/elasticsearch/image-20191209172442177.png)

---

虚拟内存问题

solutions：

2种办法：

 1）临时修改sysctl -w vm.max_map_count=262144,重启失效

2）永久修改echo "vm.max_map_count=262144" >> /etc/sysctl.conf ; sysctl -p

---

文件句柄问题

solutions：

echo "* hard nofile 65536" >> /etc/security/limits.conf 

echo "* soft nofile 65536" >> /etc/security/limits.conf 

修改完需要重启才能生效，但是不想重启，在root下临时设置ulimit -n 65536，不过在su elk下仍然不是65536，在elk下设置ulimit -n 65536没有权限。然后su - elk试了下，ok。

---

最后，启动服务，发现地址不再是127.0.0.1了，浏览器访问ok。

![image-20191210103540912](/img/elasticsearch/image-20191210103540912.png)

![image-20191210103611637](/img/elasticsearch/image-20191210103611637.png)



##### su user 和su - user的区别

- su不指定用户名，默认切root，不会改变当前工作目录以及HOME,.SHELL,USER,LOGNAME,只是拥有了root的权限
- root切普通用户不需要密码，普通用户切root需要密码
- su - username 切换后，同时切换到新用户的工作环境中(全切)
- su username切换后，不改变原用户的工作目录，及其他环境变量目录，只是拥有了用户的权限而已(半切)

example： 从root用户切su elk之后，工作目录还是root，及环境变量什么的都是root的



### 接下来安装es的可视化插件elasticsearch-head

- Git clone git://github.com/mobz/elasticsearch-head.git
- cd elasticsearch-head
- npm install -g  # 使用cnpm install和npm install都失败，最后加了-g成功
- npm run start

启动成功：

![image-20191230160437016](/img/elasticsearch/image-20191230160437016.png)

![image-20191230160659274](/img/elasticsearch/image-20191230160659274.png)



---

error：发现集群一直是未连接

---

![image-20191230162518785](/img/elasticsearch/image-20191230162518785.png)

solutions：9100连9200存在跨域问题，需要配置跨域信息。

echo "http.cors.enabled: true" >> elasticsearch.yml

echo "http.cors.allow-origin: \\"*\\"" >> elasticsearch.yml

然后重启es，成功。

![image-20191230162854476](/img/elasticsearch/image-20191230162854476.png)



 curl -X PUT "localhost:9200/weather" 测试通关，能在页面显示如图。



##### 备注

+ 6.x后es要用户名密码，需要加些配置，es-head才能连上es
+ 5.x后es不推荐head随es一起启动，不希望装在plugin下，建议新建目录，git下载，单独启动