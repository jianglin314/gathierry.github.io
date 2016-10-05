---
layout: post
title: 电商大数据应用——MacBook上Hadoop和Hive的配置
categories: [data science]
tags: [user portrait, data analysis]
description: 这篇文章记录了在MacBook OSX El Capitan 系统下，以本机作为master，Ubuntu虚拟机作为节点，配置Hadoop和Hive的过程。
---

## 配置Mac OSX + 2 x Ubuntu环境

本文使用的环境是

	Mac OSX El Capitan 10.11.6
	Parallels 10.0.1
	Ubuntu 14.04LTS

### 1. 修改hostname

将三个系统上的hostname修改为：master(mac)、node1和node2（两个ubuntu系统）。可以通过在控制台输入hostname来显示当前系统的hostname。 

- 在mac上设置hostname：```$sudo scutil --set HostName master```

- ubuntu上设置hostname: 修改/etc/hostname文件，在其中把之前的名字删除，只留下node1（在另一个ubuntu上修改为node2）。

### 2. 修改hosts文件
Mac和ubuntu上步骤相同.

使用```ifconfig```命令，查看各个系统的ip地址，并保证通过ping命令互相可以ping通，并记录各自的ip地址。

修改/etc/hosts文件，在其中添加三条记录：master的IP地址 master，node1的IP地址  node1，node2的IP地址 node2。例如：

	127.0.0.1	localhost
	255.255.255.255	broadcasthost
	::1             localhost 
	10.211.55.2     master
	10.211.55.5     node1
	10.211.55.6     node2

此步骤需要在三个系统上均执行一遍，且填入的内容也相同。

### 3. 安装及测试SSH
由于hadoop各个节点之间通过ssh方式进行通信，因此必须在各个系统上安装好ssh，并且为了避免每次登录均输入密码，还需要进行一定的配置实现ssh免密码登录。mac系统上已经默认安装了openSSH，如果ubuntu上没有安装，输入命令
	
	$sudo apt-get install openssh-server openssh-client
	
然后输入

	$ssh localhost
	
最终显示Last login: ……表示登录成功。

### 4. 配置ssh免密码登录
在三个系统中分别产生密钥

	$ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa 
	# 此时会在~/.ssh文件夹下出现如下两个文件：id_dsa id_dsa.pub
	
将所有密钥写入master的authorized_keys，分别在node1，node2下执行

	#将id_dsa.pub复制到master下并存为authorized_keys_node1(或node2)
	$scp id_dsa.pub username@master:~/.ssh/authorized_keys_node1 #node2
	#将所有密钥写入master的authorized_keys
	$cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
	$cat ~/.ssh/authorized_keys_node1 >> ~/.ssh/authorized_keys
	$cat ~/.ssh/authorized_keys_node2 >> ~/.ssh/authorized_keys

此时三个密钥全部被写入master的authorized_keys，也就是说当master收到来自三个node的ssh请求时，不会要求输入密码。

接下来将此authorized_keys拷到node1和node2

	$scp ~/.ssh/authorized_keys parallels@node1:~/.ssh/authorized_keys
	$scp ~/.ssh/authorized_keys parallels@node2:~/.ssh/authorized_keys

完成后，master和node之间就可以免密码通过ssh登录了。

### 5. 安装JDK
这里主要是要设置好JDK的环境变量```JAVA_HOME```。在Mac下输入

	$/usr/libexec/java_home 
	/Library/Java/JavaVirtualMachines/jdk1.8.0_77.jdk/Contents/Home
	
Ubuntu下，

	#安装jdk
	$sudo apt-get install default-jdk
	#获取安装路径
	$sudo update-alternatives --config java
	#获得环境变量，例如 /usr/lib/jvm/java-7-openjdk-amd64
	#添加环境变量
	$sudo vi /etc/environment
	JAVA_HOME="YOUR_PATH"
	


	




*参考资料*

- http://blog.csdn.net/wk51920/article/details/51686038
- https://www.digitalocean.com/community/tutorials/how-to-install-java-on-ubuntu-with-apt-get*






