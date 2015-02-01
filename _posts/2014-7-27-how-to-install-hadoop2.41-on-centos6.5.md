---
date: 2014-07-27
layout:     post
comments:   yes
code:        yes
title:      如何在centos6.5下配置最新2.41版hadoop伪分布式环境？
category:   blog
tags: [hadoop,linux,大数据]
---
> 由于hadoop各版本间差异较大，自己也走了不少弯路。这里把在centos6.5下配置最新2.41版hadoop伪分布式环境的过程记录下，仅做备案

#总流程
![总流程](http://media-clark.qiniudn.com/%E6%80%BB%E8%A7%88.jpg)
#1.创建hadoop用户

* 创建hadoop用户组(非必须，安全起见，专门的账户来进行操作)：

	`sudo groupadd hadoop`



* 创建hadoop用户：
	
	`sudo useradd -g hadoop hadoop`



* 给hadoop用户添加权限
	
	打开/etc/sudoers文件：

	`sudo gedit /etc/sudoers`

	按回车键后就会打开/etc/sudoers文件了，给hadoop用户赋予root用户同样的权限。

	找到类似`root   ALL=(ALL:ALL)   ALL`的字样。在`root   ALL=(ALL:ALL)   ALL`下添加
	
	`hadoop   ALL=(ALL:ALL)  ALL`
	
	`hadoop  ALL=(ALL:ALL) ALL`

	![](http://media-clark.qiniudn.com/sudoers.png)
	
	



**可能要删除系统自带的openjdk**

如图

![openjdk](http://media-clark.qiniudn.com/openjdk.jpg)


#2.正式配置hadoop前的准备工作

**在linux系统打开终端,进行如下操作**

+ 关闭防火墙
        
	* 执行命令`service iptables status`查看防火墙开闭状态(如果开,接下步)

	* 执行命令`service iptables stop`关闭防火墙

	* 执行命令`service iptables status`验证是否关闭了(验证很关键,确保每一步的成功,否则后面"死"的很惨)
                    
    * 执行命令`chkconfig --list|grep iptables `查看是否有on(此步目的:看是否打开了自动运行防火墙,因为自动运行默认打开的情况下,下次关机重启后,防火墙又打开了,还要重新关闭,麻烦,于是我们直接关闭掉自动运行...)
             
	* 执行命令`chkconfig iptables off`关闭防火墙自动运行
	
	* 执行命令`chkconfig --list|grep iptables` 查看是否有on(**同样是验证,我们一定要仔细,因为需要配置的内容很多,避免后面开
	启hadoop出错后,找不到哪里出错,不如在这里一步一步,保证每一步都正确**)
    
+ 修改ip
  
	* 右键点击linux桌面右上角的网络连接图标进行IP设置(配置IP的时候,注意选择IP段,对于IP以及网络连接的设置,参考我空间的其他虚拟机网络配置资料,这里不赘述)
  
	* 执行命令`service network restar`t重启网卡(修改IP后,一定及时重启一下,看是否有修改IP成功)
    
	* 执行命令`ifconfig`查看设置是否成功(我们必须的验证)

+ 修改主机名

	* 执行命令`hostname`查看当前的主机名
    
	* 执行命令`hostname hadoop`设置当前会话的主机名(注意,在这里修改完主机名之后,重启之后主机名又自动变回修改前的主机名,我们还要执行下一步操作,就可以一劳永逸了)
    
	* 执行命令`vi /etc/sysconfig/network`(必须修改这个文件中的主机名,否则,机器重启后,主机名变回以前)
	保存退出

+ 映射ip与主机名
	* 执行命令`vi /etc/hosts`文件
    
	* 增加一行文本`192.168.56.100 hadoop`(注意空格)
    
	* 保存退出
	
	* 执行命令`ping hadoop`是否有回包(无敌的验证,一定要保证配置的成功)

+ 设置ssh免密码登录

	* 执行命令`ssh-keygen -t rsa`,三次回车，在`~/.ssh`下面会产生两个文件，分别是`id_rsa`和`id_rsa.pub`
    
	* `cp  ~/.ssh/id_rsa.pub`   `~/.ssh/authorized_keys`(注意空格)

	![ssh](http://media-clark.qiniudn.com/ssh.png)
	
	* 用 `ssh hadoop(之前修改的hostname)` 看看能不能无密码登陆 
	
	![ssh2](http://media-clark.qiniudn.com/ssh2.png)

**在拷贝了公钥之后ssh登录还是需要密码**

查看系统日志发现如下：`Authentication refused: bad ownership or modes for directory ：/home/hadoop`

原来必须设置正确的权限才可以使用ssh的publickey验证


	chmod 755 /hadoop

	chmod 755 /home/hadoop/.ssh/

	chmod 600 /home/hadoop/.ssh/*


             
执行命令`ssh localhost`查看是否需要输入密码
            
执行命令`ssh hadoop`查看是否需要输入密码
                    
**注意：执行ssh登陆成功后，要记得exit退出ssh**
    

# 3.安装jdk(略)
        

    
#4.安装hadoop
        
	
（1） 把`hadoop-1.1.2.tar.gz`复制到/usr/local，
     
 （2）解压缩 `tar -zxvf  hadoop-1.1.2.tar.gz`
    
 （3）重命名`mv hadoop 2.41  hadoop`(同样是为了以后文件操作的方便,我改)

 （4）设置环境变量，执行命令`vi /etc/profile`
                

	增加了一行记录  export HADOOP_HOME=/usr/local/hadoop

	修改了一行记录  export PATH=.:$JAVA_HOME/bin:$HADOOP_HOME/bin:$PATH

	保存退出
           
	执行 source  /etc/profile(不要忘记)
                
	如果去掉警告信息，在/etc/profile中增加一行记录(我们在操作配置文件的时候,总是跳出一行警告,我去掉它,方法如下...暂时不解释代码的意思,这需要翻看源代码,这个时候带大家翻看源代码,怕你晕,暂时照做就是了)
               
	export HADOOP_HOME_WARN_SUPPRESS=1


        
#5.配置hadoop 修改4个配置文件(在你的hadoop安装目录下有个conf文件夹,进入后寻找如下四个去修改)                    



1.hadoop-env.sh

`export JAVA_HOME=/usr/local/jdk/(等号后面的值代表你JDK的安装目录,警告:不要简单照抄)`

2.core-site.xml



	<configuration>

   	<property>

       <name>fs.default.name</name>

       <value>hdfs://hadoop:9000</value>(红色加粗表示:当前的主机名,同样不要照抄,还要注意:别把我的括号内容整到你配置文件里面)

  	 </property>

  	 <property>

       <name>hadoop.tmp.dir</name>

       <value>/usr/local/hadoop/tmp</value>

  	 </property>  

	</configuration>




3.hdfs-site.xml


	<configuration>

   	<property>

       <name>dfs.replication</name>

       <value>1</value>

   	</property>

	</configuration>



4.mapred-site.xml



	<configuration>

   	<property>

       <name>mapred.job.tracker</name>

       <value>hadoop:9001</value>(红色加粗表示:当前的主机名,同样不要照抄)

   	</property>

	</configuration>


                 
**千万注意： 以上操作可以复制,但不能简单照抄，一定修改主机名为自己的机器主机名**
        
# 6.启动hadoop(此时在hadoop根目录下)

* 格式化namenode，执行命令`bin/hadoop namenode -format`(这个步骤很关键),该操作只能执行一次。(重新启动的方法[看这里]())
        
* 启动，执行命令`sbin/start-all.sh`
           
* 关闭，执行命令`sbin/stop-all.sh`
        
#7.验证：(验证启动的时候是否服务启动正常)3种方法
                    
#####<1> 执行命令`jps`，查看到新的java进程5个，分别是(这里是伪分布式，貌似JobTracker、TaskTracker无法启动)
                            
`NameNode、DataNode、SecondaryNameNode、JobTracker、TaskTracker`
		    
#####<2> 查看集群状态：
		
1 [root@localhost bin]# ./hadoop dfsamin -report

2 Error: Could not find or load main class dfsamin

3 [root@localhost bin]# ./hadoop dfsadmin -report

                    
#####<3> 访问服务地址

地址是`http://主机名:50070(查看node)`

`http://主机名:50030(查看tracker)`
                    
`如果在windows下用浏览器访问的话，在配置文件c:/windows/system32/drivers/etc/hosts增加一行记录 linux主机的IP  主机名`

**如果要重新格式化hadoop，一定要先删除/usr/local/hadoop/tmp文件夹。**
    
##hadoop启动时NameNode不见，常见原因
        
	(1）忘记格式化
	(2）配置文件没有改成自己的主机名
	(3）主机名与ip没有绑定
	(4）绑定的ip错误，不是自己的真实ip
	(5）防火墙忘记关闭
