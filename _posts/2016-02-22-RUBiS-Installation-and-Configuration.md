---
layout: post
permalink: /blog/2016/02/22/RUBiS-Installation-and-Configuration.html
category: software
tags: [software, research]
title: RUBiS的安装及配置
---

如果你不知道[RUBiS](http://rubis.ow2.org/)是什么，那么你可以关闭这个页面了，因为即使你看完了，对你也不会有什么帮助。如果你知道RUBiS，正打算用它，那么这篇博客，请认真往下看吧，说不定可以少趟不少的坑！

首先非常感谢 DamnChris 的 [《RUBiS的PHP版本搭建详解》](http://www.cnblogs.com/damn-chris/archive/2012/03/06/2382146.html)！本博客有不少类似的部分，但对于细节（遇到的问题及解决方案）有更详细的阐述。如有侵权，敬请指出。

如果你从事科研，当你需要模拟真实应用时，你会感到苦恼。真实应用的数据往往不会公开，而自己搭建的应用，缺乏真实用户，又没有什么公信力。于是你会找到RUBiS，或者发现不少文章的实验采用了RUBiS。

RUBiS的官网上说它是一个仿照[eBay网](eBay.com)搭建的拍卖网站原型，用来评估应用的设计模式及应用服务器的性能可扩展性等等。说白了，就是一个应用基准测试，可以用来模拟真实应用。由于其[学术成果](http://rubis.ow2.org/doc/index.html)丰硕，所以在学术圈有较高的公信力。

## 文件结构

鉴于代码包含了所有的版本，这里只列出PHP版本及数据库、客户端所使用到的代码部分。

	RUBiS/
		|-- bench/						包含一些shell脚本文件及最终的运行报告
		|-- Client/						客户端代码
			|-- edu/					代码
			|-- rubis.properties		配置文件
			|-- Makefile
		|-- database/					数据库代码
		|-- PHP/						服务端代码
		|-- workload/
		|-- config.mk					make用到的环境变量
		|-- Makefile

## 部署环境

若干台（至少3台）虚拟机，分别部署Apache服务器、MySQL数据库和Client客户端。镜像操作系统为CentOS 5。

部署Apache服务器的IP为	10.0.0.1
部署MySQL数据库的IP为	10.0.0.2
部署Client客户端的IP为	10.0.0.3

## 软件依赖

如下是[官方](http://rubis.ow2.org/doc/compile_and_run.html)给出的软件依赖，版本非常老了，基本上都得找到源码自己编译安装，非常麻烦，还不一定能成功。按照此教材在CentOS 5环境下用源进行安装，亲测可用。

	· MySQL Server v3.23.43-max (需要MyIsam和BDB支持)
	· JDK 1.3.1 (虽然服务端用的是PHP，但客户端是用Java写的)
	· Apache v1.3.22
	· PHP v.4.0.6
	· Sysstat utility (用于监控每个节点的各项指标)
	· Gnuplot (只安装在主client节点，用于生成监控图)
	· Bash 2.04 (只安装在主client节点)

## 部署过程

所有服务器及客户端下载最新版本的RUBiS

	wget http://download.forge.ow2.org/rubis/RUBiS-1.4.3.tgz

或者

	wget http://download.forge.objectweb.org/rubis/RUBiS-1.4.3.tgz

解压文件至/home/RUBiS

	tar -zxvf RUBiS-1.4.3.tgz
	cp -rp RUBiS /home/

### 安装配置MySQL数据库

a. 安装MySQL服务器

	# yum install mysql-server.x86_64

b. 安装监控软件

	# yum install sysstat

c. 配置MySQL访问用户并赋予权限

启动MySQL

	/etc/init.d/mysqld restart

配置MySQL开机自启

	chkconfig mysqld on

设置MySQL root用户密码

	mysqladmin -u root password <root_password>

进入MySQL数据库

	mysql -u root -p

创建新用户rubis，并使其可以远程或者本地连接MySQL服务器：

	>> CREATE USER 'rubis'@'%' IDENTIFIED BY '<rubis_password>';
	>> CREATE USER 'rubis'@'localhost' IDENTIFIED BY '<rubis_password>';

然后为rubis用户赋予权限：

	>> GRANT ALL PRIVILEGES ON *.* TO 'rubis'@'%';
	>> GRANT ALL PRIVILEGES ON *.* TO 'rubis'@'localhost';

或者

	grant all privileges on *.* to 'rubis'@'%' identified by '<rubis_password>' with grant option;
	grant all privileges on *.* to 'rubis'@'localhost' identified by '<rubis_password>' with grant option;
	flush privileges;

d. 调用sql文件初始化rubis需要的数据库

	# cd /home/RUBiS/database
	# mysql -urubis -p<rubis_password> < rubis.sql
	# mysql -urubis -p<rubis_password> rubis < regions.sql
	# mysql -urubis -p<rubis_password> rubis < categories.sql

e. 开启防火墙中MySQL数据库3306端口

	iptables -I INPUT -p tcp --dport 3306 -j ACCEPT

如果要重启保持生效，则需要修改/etc/sysconfig/iptables文件，加入

	-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT

参考
http://stackoverflow.com/questions/10299148/mysql-error-1045-28000-access-denied-for-user-billlocalhost-using-passw


### 安装配置Apache服务器

a. 安装Apache服务器、PHP支持及PHP访问MySQL模块.

	# yum install httpd.x86_64
	# yum install php.x86_64
	# yum install php-mysql.x86_64

启动Apache服务器

	# service httpd start

配置Apache开机自启

	# chkconfig httpd on

b. 安装监控软件.

	# yum install sysstat

c. 将RUBiS的PHP代码放置在apache服务器内，建立连接即可。

	# ln /home/RUBiS/PHP /var/www/html/PHP
	ln: `/home/RUBiS/PHP': hard link not allowed for directory
	# ln -s /home/RUBiS/PHP /var/www/html/PHP

d. 修改RUBiS的PHP代码：

因为RUBiS在04年之后就没有对代码进行维护了。所以其的PHP版本中使用了不少新PHP版本已经废弃的命令，导致客户端访问apache服务器的时候会出现问题。所以需要手动修正这些问题。
将/home/RUBiS/PHP/目录下的所有PHP文件中：

		$HTTP_POST_VARS	改为	$_POST
		$HTTP_GET_VARS		改为	$_GET

	grep -ci "HTTP_POST_VARS" *.php
	grep -ci "HTTP_GET_VARS" *.php

	sed -i "s/HTTP_POST_VARS/_POST/g" *.php
	sed -i "s/HTTP_GET_VARS/_GET/g" *.php

编辑/etc/php.ini文件，屏蔽PHP的部分警告

	error_reporting  =  E_ALL & ~E_NOTICE

e. 配置PHP连接的数据库信息。

打开/home/RUBiS/PHP/PHPPrinter.php文件，修改getDatabaseLink函数。将mysql_pconnect()里面的三个参数依次填为Apache服务器IP、MySQL用户名及其密码。

配置完成后，从Apache服务器连接数据库，检验是否能连接成功。

	mysql -h 10.0.0.2 -urubis -p<rubis_password>

**注**：配置到这个步骤，访问apache服务器上的rubis主页，应该就有完成的页面了，同时可以浏览到数据库中基本信息了。

f. 开启防火墙Apache HTTP服务 80端口

	iptables -I INPUT -p tcp --dport 80 -j ACCEPT

如果要重启保持生效，则需要修改/etc/sysconfig/iptables文件，加入

	-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT

若使用了OpenStack等云计算环境，同时别忘了打开安全组规则，建议打开 TCP 22(SSH) / 53(DNS) / 80(HTTP) / 443(HTTPS) / 3306(MySQL)

用浏览器访问网站还可能出现403，/var/log/httpd/error.log中记录Symbolic link not allowed or link target not accessible

参考：
http://unix.stackexchange.com/questions/20993/symbolic-link-not-allowed-or-link-target-not-accessible-apache-on-centos-6

	# getenforce
	Enforcing
	# setenforce 0
	# getenforce
	Permissive

如果要重启生效，则需要修改

	# vi /etc/sysconfig/selinux
	SELINUX=Permissive

### Client客户端

Client的功能主要有两个：一个是初始化RUBiS的数据库；一个是模拟应用请求，对RUBiS进行测试，最后生成报告。

a. 安装监控软件及绘图软件。

	# yum install sysstat
	# yum install gnuplot

b. 修改配置文件

`·/home/RUBiS/Client/rubis.properties`该文件是client执行的规则配置文件。其具体配置内容和样例，参考[rubis.properties](http://www.cnblogs.com/damn-chris/archive/2012/03/11/2390275.html)及文后附录

**注意** 参数千万不要设的太大，尤其是用户数等，默认是一百万！太大了，光生成数据就得两小时！

修改ebay_regions.txt等文件的路径为当前系统正确的文件路径。

	vi /home/RUBiS/Client/rubis.properties

否则会出现初始化数据库出错

	<h3><br>### Region & Category definition files ###</h3>
	Regions description file: Error while checking database.properties: /users/margueri/RUBiS/database/ebay_regions.txt (No such file or directory)
	make: *** [initDB] Error 1

`·/home/RUBiS/config.mk`该文件主要配置了一些必须的环境变量。比如\$JAVA, \$JAVAC, \$CLASSPATH, \$JAR等。还有一些常用的命令，\$MAKE, \$CP, \$RM等操作。同时，还配置了使用的数据库类型，\$DB_SERVER，该变量只有MySQL和PostgreSQL两种选择。

c. 安装java, javac并配置JAVA_HOME环境变量

	yum list \*java-1\* | grep open
	yum install java-1.6.0-openjdk.x86_64
	yum install java-1.6.0-openjdk-devel.x86_64

	export JAVA_HOME=/usr/lib/jvm/java-1.6.0-openjdk.x86_64
	echo $JAVA_HOME

	export PATH=$JAVA_HOME/bin:$PATH
	export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

可以将这些环境变量写入/etc/profile或~/.bash_profile(推荐)

	export JAVA_HOME=/usr/lib/jvm/java-1.6.0-openjdk.x86_64
	export PATH=$JAVA_HOME/bin:$PATH
	export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

参考
https://wiki.centos.org/HowTos/JavaOnCentOS
http://tecadmin.net/steps-to-install-java-on-centos-5-6-or-rhel-5-6
https://www.digitalocean.com/community/tutorials/how-to-install-java-on-centos-and-fedora

否则会编译出错，没有java, javac等

	[root@centos5 RUBiS]# make client
	cd Client ; make all
	make[1]: Entering directory `/home/RUBiS/Client'
	/bin/javac  -classpath .:/lib/j2ee.jar:/jre/lib/rt.jar:/cluster/opt/jakarta-tomcat-3.2.3/lib/servlet.jar:/home/RUBiS/Client edu/rice/rubis/client/URLGenerator.java
	make[1]: /bin/javac: Command not found
	make[1]: *** [edu/rice/rubis/client/URLGenerator.class] Error 127
	make[1]: Leaving directory `/home/RUBiS/Client'
	make: *** [client] Error 2

d. 在每个客户端分别进行编译

	# cd /home/RUBiS
	# make client

e. 初始化数据库

初始化数据库之前请检查rubis.properties参数设置是否合理。
数据库初始化所需时间较长，建议开screen窗口进行。

	# yum install screen -y
	# screen

	# cd /home/RUBiS
	# make initDB PARAM='all'

需要注意的几点：
    1) make initDB后边的PARAM一共有5种选择：all，生成所有数据；users，只生成用户；items，只生成物品；bids，生成竞拍；comments，生成注释。
    2) 如果前面讲到的PHP代码中需要修改的地方已经更正了的话，数据是可以写入到数据库中的。
    3) 在配置rubis.properties的时候，如果database_item_description_length的数值太大的话。会出现Unable to Open URL http://....的错误。文件默认给的值就太大，建议修改为1024即可。

f. 启动模拟器

启动模拟器之前务必确认一下rubis.properties参数的设置，是否合理。

在emulator过程中，需要多次输入 root@10.0.0.1 即Apache服务器、 root@10.0.0.2 即MySQL服务器、root@10.0.0.3 即从客户端和 root@localhost 即本机的密码，为了便于模拟，建议设置密钥登录。

	mkdir .ssh
	ls -al
	chmod 700 .ssh
	ls -al
	cd .ssh/
	ls -al
	vi authorized_keys

模拟的时间较长，建议先安装screen，然后在screen窗口中进行
	# yum install screen -y
	# screen

开始模拟

	# cd /home/RUBiS
	# make emulator

需要注意的是，调用make emulator命令的位置一定要在RUBiS的根目录。由于在生成报告的时候，客户端会调用bench目录下的一些shell脚本文件。但是调用脚本时，rubis给的是脚本的相对路径而不是绝对路径，这样如果你在Client子目录下执行make emulator(这样也是可以执行的)就会出现程序找不到指定文件的错误，在client生成的报告中就会缺失图片。

rubis模拟器生成的报告会出现在/home/RUBiS/bench/目录下。
到此，整个RUBiS的PHP版本配置就结束了。若需要进行高并发测试的，请参考后文的高并发Apache / MySQL配置

### 删除数据库重新初始化

由于第一次运行，没注意具体参数，用户数默认一百万，条目数默认一百万，建立用户就花了近两个小时，建立条目和评论估计得十多个小时，所以只好强制关闭，删除数据库重新初始化。

	# 在mysql服务器上执行
	> show databases;
	> drop database rubis;
	> show databases;
	> exit

	# 在mysql服务器上执行
	# 调用sql文件初始化rubis需要的数据库
	# cd /home/RUBiS/database
	# mysql -urubis -p<rubis_password> < rubis.sql
	# mysql -urubis -p<rubis_password> rubis < regions.sql
	# mysql -urubis -p<rubis_password> rubis < categories.sql
	# 在Apache服务器上执行
	rm -rf access_log
	touch access_log
	rm -rf error_log
	touch error_log
	# service httpd restart

	# 在client服务器上执行
	# 初始化数据库
	# cd /home/RUBiS
	# make initDB PARAM='all'

	# 连接数据库查看初始化结果
	> show databases;
	> use rubis;
	> show tables;
	> select count(*) from users;


### 运行中遇到的问题

/var/log/httpd/error_log中出现大量

	PHP Notice:  Undefined index:
	PHP Notice:  Undefined variable:

http://stackoverflow.com/questions/4261133/php-notice-undefined-variable-and-notice-undefined-index
http://stackoverflow.com/questions/4465728/php-error-notice-undefined-index

Disable E_NOTICE from reporting. A quick way to exclude just E_NOTICE is

	error_reporting( error_reporting() & ~E_NOTICE ).

编辑/etc/php.ini文件

	error_reporting  =  E_ALL & ~E_NOTICE


### Apache / MySQL 并发连接数的配置

Server version: Apache/2.2.3
Server version: 5.0.95 Source distribution

参考
http://zyan.cc/post/269/

查看httpd进程数（即prefork模式下Apache能够处理的并发请求数）：

	ps -ef | grep httpd | wc -l

返回结果示例：
　　1388
表示Apache能够处理1388个并发请求，这个值Apache可根据负载情况自动调整，我这组服务器中每台的峰值曾达到过2002。

查看Apache的并发请求数及其TCP连接状态：

	netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'

返回结果示例：

	LAST_ACK 5
	SYN_RECV 30
	ESTABLISHED 1597
	FIN_WAIT1 51
	FIN_WAIT2 504
	TIME_WAIT 1057

其中的`SYN_RECV`表示正在等待处理的请求数；`ESTABLISHED`表示正常数据传输状态；`TIME_WAIT`表示处理完毕，等待超时结束的请求数。

参考
http://httpd.apache.org/docs/2.2/mpm.html
http://www.365mini.com/page/apache-concurrency-configuration.htm

	httpd -l

查看Apache使用的模块

	Compiled in modules:
	core.c
	prefork.c
	http_core.c
	mod_so.c

vi /etc/httpd/conf/httpd.conf

	#KeepAlive Off
	KeepAlive On
	#MaxKeepAliveRequests 100
	MaxKeepAliveRequests 1000
	#KeepAliveTimeout 15
	KeepAliveTimeout 60

	#mpm_perfork模块

	<IfModule mpm_prefork_module>
	StartServers          5 #推荐设置：小=默认 中=20~50 大=50~100
	MinSpareServers       5 #推荐设置：与StartServers保持一致
	MaxSpareServers      10 #推荐设置：小=20 中=30~80 大=80~120
	MaxClients          150 #推荐设置：小=500 中=500~1500 大型=1500~3000
	MaxRequestsPerChild   0 #推荐设置：小=10000 中或大=10000~500000
	(此外，还需额外设置ServerLimit参数，该参数最好与MaxClients的值保持一致。)

	MaxKeepAliveRequests 1000
	MaxKeepAliveRequests 2000

	</IfModule>

	<IfModule prefork.c>
	#StartServers       8
	#MinSpareServers    5
	#MaxSpareServers   20
	#ServerLimit      256
	#MaxClients       256
	#MaxRequestsPerChild  4000

	StartServers       25
	MinSpareServers    20
	MaxSpareServers   80
	ServerLimit      1000
	MaxClients       1000
	MaxRequestsPerChild  4000

	StartServers       50
	MinSpareServers    40
	MaxSpareServers   120
	ServerLimit      2000
	MaxClients       2000
	MaxRequestsPerChild  40000
	</IfModule>

	<IfModule worker.c>
	#StartServers         2
	#MaxClients         150
	#MinSpareThreads     25
	#MaxSpareThreads     75
	#ThreadsPerChild     25
	#MaxRequestsPerChild  0

	StartServers         5
	MaxClients         2000
	MinSpareThreads     50
	MaxSpareThreads     150
	ThreadsPerChild     100
	MaxRequestsPerChild  4000
	</IfModule>

配置完毕后，重启Apache服务：

	# service httpd restart

mysql数据库
http://dev.mysql.com/doc/refman/5.0/en/optimization.html
http://www.centoscn.com/mysql/2015/0429/5312.html
http://blog.chinaunix.net/uid-20178201-id-3160469.html
http://blog.chinaunix.net/uid-16844903-id-294162.html
https://www.linuxyw.com/a/shujuku/20130506/216.html
http://www.educity.cn/shujuku/1095729.html

vi /etc/my.cnf

	max_connections = 2000
	max_connect_errors = 5000

	back_log = 500
	join_buffer_size = 2M
	key_buffer_size = 512M	# MyISAM表的索引块分配了缓冲区，由所有线程共享。key_buffer_size是索引块缓冲区的大小。最大允许设定值为4GB，通常为主要运行MySQL的机器内存的25%。
	max_allowed_packet = 4M		# 包或任何生成的/中间字符串的最大大小。
	max_connections = 2000
	max_connect_errors = 5000
	myisam_sort_buffer_size = 64M		# 当在REPAIR TABLE或用CREATE INDEX创建索引或ALTER TABLE过程中排序 MyISAM索引分配的缓冲区。
	query_cache_size = 32M	# 为缓存查询结果分配的内存的数量。默认值是0，即禁用查询缓存。请注意即使query_cache_type设置为0也将分配此数量的内存。
	read_buffer_size = 4M	# 每个线程连续扫描时为扫描的每个表分配的缓冲区的大小(字节)。如果进行多次连续扫描，可能需要增加该值，默认值为131072。
	read_rnd_buffer_size = 8M	# 当排序后按排序后的顺序读取行时，则通过该缓冲区读取行，避免搜索硬盘。将该变量设置为较大的值可以大大改进ORDER BY的性能。
	sort_buffer_size = 4M	# 每个排序线程分配的缓冲区的大小。增加该值可以加快ORDER BY或GROUP BY操作。
	table_cache = 512		# 所有线程打开的表的数目。增大该值可以增加mysqld需要的文件描述符的数量。
	thread_cache_size = 128		# 服务器应缓存多少线程以便重新使用。如果新连接很多，可以增加该变量以提高性能。
	thread_concurrency = 2		# Try number of CPU's*2
	wait_timeout = 120

	skip-locking	# 避免MySQL的外部锁定，mysqld使用外部锁定，该值为OFF

配置完毕后重启 MySQL服务
	service mysqld restart
	/etc/init.d/mysqld restart

CentOS TCP连接数

http://www.centoscn.com/CentOS/config/2014/0902/3661.html
http://linyu19872008.iteye.com/blog/1998832

### RUBiS的rubis.properties配置

	#####  服务器节点信息  #####

	# HTTP服务器主机及端口
	httpd_hostname = 10.0.0.1
	httpd_port = 80

	# HTTP端使用的版本，可选PHP, Servlets, EJB
	httpd_use_version = PHP

	#各个版本对应的路径
	#虽然选择了PHP版本，但是Servlets和EJB版本的脚本路径还是要配置，不然会报错。
	cjdbc_hostname = 10.0.0.1
	ejb_server = 10.0.0.1
	ejb_html_path = /ejb_rubis_web
	ejb_script_path = /ejb_rubis_web/servlet
	servlets_server = 10.0.0.1
	servlets_html_path = /Servlet_HTML
	servlets_script_path = /servlet


	#PHP版本的脚本路径
	php_html_path = /PHP
	php_script_path = /PHP


	#####  负载的配置  #####

	# 在使用make emulator启动模拟器对应用进行性能测试的时候，所需要的规则配置。

	# 远程client信息
	# 远程节点主机名或者ip，使用逗号分隔开。空白说明只有主client。
	workload_remote_client_nodes = 10.0.0.4
	# 调用远程节点的命令
	workload_remote_client_command = /usr/lib/jvm/java-1.6.0-openjdk-1.6.0.37.x86_64/bin/java -classpath RUBiS edu.rice.rubis.client.ClientEmulator
	workload_remote_client_command = /usr/lib/jvm/java-1.6.0-openjdk-1.6.0.37.x86_64/bin/java -classpath /home/RUBiS/Client edu.rice.rubis.client.ClientEmulator
	# 每个节点的客户端数量
	workload_number_of_clients_per_node = 240
	workload_number_of_clients_per_node = 500

	# 负载量
	workload_transition_table = /home/RUBiS/workload/transitions.txt
	workload_number_of_columns = 27
	workload_number_of_rows = 29
	workload_maximum_number_of_transitions = 1000
	workload_number_of_items_per_page = 20
	workload_use_tpcw_think_time = yes
	workload_up_ramp_time_in_ms = 120000
	workload_up_ramp_slowdown_factor = 2
	workload_session_run_time_in_ms = 900000
	workload_down_ramp_time_in_ms = 60000
	workload_down_ramp_slowdown_factor = 3


	#####  数据库相关  #####
	# 执行make initDB初始化数据库的时候需要的规则。

	# 数据库主机
	database_server = 10.0.0.2

	# 用户
	database_number_of_users = 10000

	# Region & Category definition files
	database_regions_file = /home/RUBiS/database/ebay_regions.txt
	database_categories_file = /home/RUBiS/database/ebay_simple_categories.txt

	# 产品
	database_number_of_old_items = 10000
	database_percentage_of_unique_items = 80
	database_percentage_of_items_with_reserve_price = 40
	database_percentage_of_buy_now_items = 10
	database_max_quantity_for_multiple_items = 10
	database_item_description_length = 1024

	# 标价
	# 每个物品可标价最大数
	database_max_bids_per_item = 20

	# 评论
	# 每个用户可评论最大值
	database_max_comments_per_user = 20
	# 每条评论长度最大值
	database_comment_max_length = 2048

	#####   监控信息  #####
	# 监控的等级：0 不监控；1 只监控error；2 error和html；3 所有信息
	monitoring_debug_level = 0

	# 监控程序
	monitoring_program = /usr/bin/sar
	monitoring_options = -n DEV -n SOCK -rubcw

	# 监控采样频率
	monitoring_sampling_in_seconds = 1

	# 远程命令
	monitoring_rsh = /usr/bin/ssh
	monitoring_scp = /usr/bin/scp

	# gnuplot生成图片的格式
	monitoring_gnuplot_terminal = jpeg

rubis.properties 文件配置内容

	# HTTP server information
	httpd_hostname = 10.0.0.1
	httpd_port = 80

	# +# C-JDBC server information
	# +cjdbc_hostname =

	# Precise which version to use. Valid options are : PHP, Servlets, EJB
	httpd_use_version = PHP

	cjdbc_hostname = 10.0.0.1
	ejb_server = 10.0.0.1
	ejb_html_path = /ejb_rubis_web
	ejb_script_path = /ejb_rubis_web/servlet

	servlets_server = 10.0.0.1
	servlets_html_path = /Servlet_HTML
	servlets_script_path = /servlet

	php_html_path = /PHP
	php_script_path = /PHP

	# Workload: precise which transition table to use
	workload_remote_client_nodes =
	workload_remote_client_command = /usr/local/java/jdk1.3.1/bin/java -classpath RUBiS edu.rice.rubis.client.ClientEmulator
	workload_number_of_clients_per_node = 240

	workload_transition_table = /home/RUBiS/workload/transitions.txt
	workload_number_of_columns = 27
	workload_number_of_rows = 29
	workload_maximum_number_of_transitions = 1000
	workload_number_of_items_per_page = 20
	workload_use_tpcw_think_time = yes
	workload_up_ramp_time_in_ms = 120000
	workload_up_ramp_slowdown_factor = 2
	workload_session_run_time_in_ms = 900000
	workload_down_ramp_time_in_ms = 60000
	workload_down_ramp_slowdown_factor = 3


	#Database information
	database_server = 10.0.0.2

	# Users policy
	database_number_of_users = 10000

	# Region & Category definition files
	database_regions_file = /home/RUBiS/database/ebay_regions.txt
	database_categories_file = /home/RUBiS/database/ebay_simple_categories.txt

	# Items policy
	database_number_of_old_items = 10000
	database_percentage_of_unique_items = 80
	database_percentage_of_items_with_reserve_price = 40
	database_percentage_of_buy_now_items = 10
	database_max_quantity_for_multiple_items = 10
	database_item_description_length = 1024

	# Bids policy
	database_max_bids_per_item = 20

	# Comments policy
	database_max_comments_per_user = 20
	database_comment_max_length = 2048


	# Monitoring Information
	monitoring_debug_level = 0
	monitoring_program = /usr/bin/sar
	monitoring_options = -n DEV -n SOCK -rubcw
	monitoring_sampling_in_seconds = 1
	monitoring_rsh = /usr/bin/ssh
	monitoring_scp = /usr/bin/scp
	monitoring_gnuplot_terminal = jpeg

最后请注意，过几天之后，初始化的拍卖数据会过期，最好重新初始化数据库再进行测试。

	mysql> SELECT items.id, items.name, items.initial_price, items.max_bid, items.nb_of_bids, items.end_date FROM items WHERE category=5 AND end_date>=NOW() LIMIT 3;
	+----+----------------------------------------+---------------+---------+------------+---------------------+
	| id | name                                   | initial_price | max_bid | nb_of_bids | end_date            |
	+----+----------------------------------------+---------------+---------+------------+---------------------+
	|  5 | RUBiS automatically generated item #5  |          4750 |    4843 |         18 | 2016-01-20 13:09:31 |
	| 25 | RUBiS automatically generated item #25 |          1729 |    1798 |         14 | 2016-01-22 13:09:32 |
	| 45 | RUBiS automatically generated item #45 |          2549 |    2652 |         17 | 2016-01-20 13:09:34 |
	+----+----------------------------------------+---------------+---------+------------+---------------------+
