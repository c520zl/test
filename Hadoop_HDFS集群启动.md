# Hadoop_伪分布式运行模式（3.x 版本） #
***
## 一、HDFS单主机伪分布式环境搭建（master） ##
**1. 软件说明**
===
	- 操作系统: 		win10/win7
	- 虚拟软件:			VMware16
	- 虚拟机:			 CentOS_7.5_64_master	192.168.199.101
	- 软件包存储路径:	  /opt/software/
	- 软件安装路径:	   /opt/modules/	
	- Jdk:			   jdk-8u391-linux-x64.tar.gz
	- Hadoop:		   hadoop-3.1.3.tar.gz
	- 用户:			  hadoop(超级管理员)
**2.添加新用户和设置用户密码**
=== 	
	su root  			切换root用户
	adduser hadoop		添加用户
	password 123456  	设置密码
**3.修改用户权限**
==  	
	su root  			切换root用户
	vi /etc/sudoers		编辑sudoers文件
	在空白处添加以下内容：
	hadoop ALL=(ALL) NOPASSWD:ALL		//使hadoop用户可以使用sudo命令
	source /etc/sudoers	使sudoers文件新添加的内容生效
	su hadoop			切换回hadoop用户

**4.安装vim(如已安装有，可跳过这一步)**
==  
	//查询是否有vim
	rpm -qa|grep vim

	//如无以上输出结果，则安装vim
	yum -y install vim*

	//编辑vim配置文件
	vim /etc/vimrc
	
	//文本中添加
	set nu          // 设置显示行号
	set showmode    // 设置在命令行界面最下面显示当前模式等
	set ruler       // 在右下角显示光标所在的行数等信息
	set autoindent  // 设置每次单击Enter键后，光标移动到下一行时与上一行的起始字符对齐
	syntax on       // 即设置语法检测，当编辑C或者Shell脚本时，关键字会用特殊颜色显示

**5.关闭防火墙、设置开机不启动**  
===
	sudo systemctl stop firewalld.service		关闭防火墙
	sudo systemctl disable firewalld.service	禁止防火墙开机自启
	sudo firewall-cmd --state					查看防火墙状态

**6.关闭安全机制状态**
===

	//进入conf文件进行编辑，把SELINUX=enabled 改成 SELINUX=disabled
	sudo vim /etc/selinux/config
	
**7.设置固定IP**
===
	//编辑网卡信息
	sudo vim /etc/sysconfig/network-scripts/ifcfg-ens33
	
	TYPE="Ethernet"
	PROXY_METHOD="none"
	BROWSER_ONLY="no"
	BOOTPROTO="static"
	DEFROUTE="yes"
	IPV4_FAILURE_FATAL="no"
	IPV6INIT="yes"
	IPV6_AUTOCONF="yes"
	IPV6_DEFROUTE="yes"
	IPV6_FAILURE_FATAL="no"
	IPV6_ADDR_GEN_MODE="stable-privacy"
	NAME="ens33"
	UUID="77aab300-46f3-41b5-acec-d6cd06675d56"
	DEVICE="ens33"
	ONBOOT="yes"
	IPADDR="192.168.199.101"
	NETMASK="255.255.255.0"
	GATEWAY="192.168.199.2"
	DNS1="192.168.199.2"
	DNS2="114.114.114.114"

**需要把**  
`BOOTPROTO="dynamic"` **改成** `BOOTPROTO="static"`  
`ONBOOT="no"` **改成** `ONBOOT="yes"`  
	
	//然后把下面的内容添加到文件末尾处
	IPADDR="192.168.199.101"  		//ip地址
	NETMASK="255.255.255.0"  		//子网掩码
	GATEWAY="192.168.199.2"  		//网关
	DNS1="192.168.199.2"  			//网络运营商DNS
	DNS2="114.114.114.114"  		//网络运营商DNS

8.重启网络服务
===
	sudo service network restart

9.本地测试访问
===
	ping www.baidu.com

10.设置主机名
===
	//修改主机名为master
	sudo hostname master
	//编辑文件
	sudo vim /etc/hosts
	//添加内容
	192.168.199.101 master
	//重启系统
	reboot
	//查看主机是否修改成功
	hostname

11.本地电脑配置虚拟机ip地址映射
===
	配置本地Windows系统的主机IP映射，以便后续可以在本地通过主机名直接访问集群节点资源。编辑Windows操作系统的C:\Windows\System32\drivers\etc\hosts文件  
	在文件末尾加入以下内容即可：
	192.168.199.101	master

12.新建资源目录、修改目录权限
===
	//在opt目录下新建softwares目录
	sudo mkdir /opt/softwares
	//在opt目录下新建modules目录
	sudo mkdir /opt/modules
	//目录/opt及其子目录中所有文件的所有者和组更改为用户hadoop和组hadoop
	sudo chown -R hadoop:hadoop /opt/*

13.利用Xftp软件上传文件
===
	上传JDK安装包jdk-8u391-linux-x64.tar.gz、hadoop-3.1.3.tar.gz到目录/opt/softwares中
	然后进入目录，解压jdk-8u391-linux-x64.tar.gz、hadoop-3.1.3.tar.gz到目录/opt/modules中
	解压命令如下：
	//切换到softwares
	cd /opt/softwares
	tar -zxvf jdk-8u391-linux-x64.tar.gz -C /opt/modules
	tar -zxvf hadoop-3.1.3.tar.gz -C /opt/modules
	//查看解压目录文件
	ls

14.安装JDK、Hadoop
===
	//新建和编辑my_env.sh文件
	sudo vim /etc/profile.d/my_env.sh
	
	//需要添加的内容
	#JAVA_HOME
	export JAVA_HOME=/opt/modules/jdk1.8.0_391
	export PATH=$JAVA_HOME/bin:$PATH
	#HADOOP_HOME
	export HADOOP_HOME=/opt/modules/hadoop-3.1.3
	export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH

	//刷新profile文件，使修改生效
	source /etc/profile

15.验证JDK、Hadoop是否安装成功
===
	//JDK的版本号
	java -version
	
	//Hadoop的版本号
	hadoop version

16.伪分布式相关文件的配置
===
	//1、hadoop-env.sh的配置：指定JDK的环境
	//切换到hadoop目录
	cd /opt/modules/hadoop-3.1.3/etc/hadoop

	//编辑文件：hadoop-env.sh
	sudo vim hadoop-env.sh

	添加内容如下：
	
	export JAVA_HOME=/opt/modules/jdk1.8.0_391

	//2、core-site.xml的配置	
	//编辑文件：core-site.xml
	sudo vim core-site.xml

	添加内容如下：

	<configuration>
		<property>
			<name>fs.defaultFS</name>
			<value>hdfs://master:9000</value>
		</property>
		<property>
			<name>hadoop.tmp.dir</name>
			<value>file:/opt/modules/hadoop-3.1.3/tmp</value>
		</property>
	</configuration>
	
	//3、hdfs-site.xml的配置	
	//编辑文件：hdfs-site.xml
	sudo vim hdfs-site.xml

	添加内容如下：

	<configuration>
	    <!-- 配置副本数，注意，伪分布模式只能是1。-->
	    <property>
	        <name>dfs.replication</name>
	        <value>1</value>
	    </property>
	</configuration>

**扩展: hadoop1.x、hadoop3.x的默认端口是9000，hadoop2.x的默认端口是8020，使用哪一个都可以**
===

17.格式化NameNode
===
	hdfs namenode -format

18.启动伪分布式
===
	start-dfs.sh
19.jps命令查看守护进程
===
	jps

**启动脚本会开启分布式文件系统上的相关进程：**   
- namenode  
- datanode  
- secondary namenode  

20.web界面访问
===
**可以在浏览器上输入：192.168.199.101:9870 来查看一下伪分布式集群的信息  
--1. 浏览一下页面上提示的ClusterID,BlockPoolID  
--2. 查看一下活跃节点(Live Nodes)的个数，应该是1个**


21.程序案例演示：wordcount程序
===
**准备要统计的file.txt文件,存储到/opt/modules/hadoop-3.1.3/data下**  

	//切换目录到 hadoop-3.1.3
	cd /opt/modules/hadoop-3.1.3
	
	//新建目录
	sudo mkdir data

	//切换目录：data
	cd data

	//编辑file.txt
	sudo vim file.txt
	
	内容如下：
	hello hello
	world world
	123
	456
	zhangsan
	wuwu

**然后退出保存**
22.在hdfs上创建存储目录
===
	hdfs dfs -mkdir /input

23.将本地文件系统上的上传到hdfs上,并在web上查看一下
===
	hdfs dfs -put /opt/modules/hadoop-3.1.3/data /input/

24.运行自带的单词统计程序wordcount
===
	//切换到hadoop的编译环境
	cd $HADOOP_HOME

	hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar  wordcount /input /output

**效果如下：**

**查看web界面**

25.查看part-r-00000文件
===
	hdfs dfs -cat /output/part-r-00000

26.hdfs自带的mapreduce组件计算圆周率
===
	hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar pi 5 5

***

## HDFS集群环境启动步骤： ##
1. **开启虚拟机**
1. **xshell连接服务器**
1. **执行shell脚本文件启动HDFS集群**  
	`start-all.sh`
1. **输入 jps 命令查看HDFS集群的守护进程是否存在**  
	`namenode`  
	`datanode`  
1. **在浏览器输入地址访问web节目**  
**http://master:8088 或者 http://192.168.206.131:8088**  
**http://master:9870 或者 http://192.168.206.131:9870**

## 访问web界面输入主机名或者IP地址都可以访问 ##
