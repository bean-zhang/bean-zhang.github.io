---
layout: post
title: IRC (Internet Relay Chat) 入门
---

## IRC简介

IRC (Internet Relay Chat) 是一个应用层的文本通信协议，采用客户端/服务器的网络模型。用户可以通过安装IRC本地客户端或者浏览器进行群聊和私聊，也可进行文件共享。[1]

为什么要用IRC呢？因为[2]

1. 大量开源项目使用IRC进行在线实时交流沟通，包括（但不仅限于）以下这些
	* Angular
	* Debian
	* Django
	* Docker
	* jQuery
	* NeoVim
	* Node.js
	* OpenStack
	* ReactJS
2. [Google Summer of Code](https://developers.google.com/open-source/gsoc/resources/irc)使用IRC（在频道 #gsoc 上）
3. 拥有众多[开源的客户端、服务器端软件和代理程序](https://github.com/search?o=desc&q=irc&s=stars&type=Repositories&utf8=%E2%9C%93)
4. 采用分布式设计，部署于全球各地的多个网络共同协作
5. 允许多个项目在同一个网络中共存
6. 无需注册即可使用，也可注册后使用

## IRC安装和使用

### 浏览器直接使用

用浏览器打开[webchat网址](http://webchat.freenode.net/)输入自己想要的Nickname和想进入的Channels即可连接登录，连接后即可加入群聊或私聊。

### 使用本地客户端

[Internet Relay Chat Help](http://www.irchelp.org/) 对各种操作系统（包括Windows/MacOS X/Linux/Unix/Smartphones等）的IRC客户端进行了简要介绍，用户可以挑选一种进行安装。

安装完成后，就像网页端一样，即可打开使用。

### 安装和配置IRC代理ZNC

虽然通过浏览器或本地客户端直接连接IRC网络进行聊天不能保留聊天记录，也无法收到离线时的消息，但一般项目进行IRC会议会采用[MeetBot](https://wiki.debian.org/MeetBot)的方式，使得会议记录自动保存于项目特定地址中。如OpenStack项目，会保存于`http://eavesdrop.openstack.org/meetings/<project_name>/`中。

当然不是会议的部分，就无法保存了。不过可以代理进行接收和保存离线消息。有很多这样的开源客户端，如[ZNC](http://wiki.znc.in/ZNC)、[Bip](https://bip.milkypond.org/)等等。本文以ZNC为例进行安装和配置。

首先，你需要一台长期在线的服务器（或虚拟机），在你想访问的地方可以访问到，最好是有公网IP。具体说明可参考 [ZNC Installation](http://wiki.znc.in/Installation)

本文以Ubuntu 14.04为例，其他操作系统类似。

	# 更新软件包列表
	sudo apt-get update
	# 安装编译所需要的包
	sudo apt-get install build-essential libssl-dev libperl-dev pkg-config
	# 进入想安装的系统目录
	cd /usr/local/src
	# 下载ZNC源代码
	wget http://znc.in/releases/znc-1.6.2.tar.gz
	wget http://znc.in/releases/znc-latest.tar.gz # 建议下载最新版
	# 解压缩源代码
	tar -xzvf znc*.tar.gz
	# 编译并安装ZNC
	cd znc*
	sudo ./configure
	sudo make
	sudo make install

推荐新建一个普通用户来运行ZNC

	sudo adduser --disabled-password znc
	su znc
	cd ~

创建ZNC配置文件

	/usr/local/bin/znc --makeconf

接着会有一些交互式的问题，以便完成配置

	Listen on port (1025 to 65534): 5000 # 不推荐使用6000和6665~6669等端口[11,12]
	Listen using SSL (yes/no) [no]: no
	Listen using both IPv4 and IPv6 (yes/no) [yes]: yes
	
	Username (alphanumeric): <your_username>
	Enter password: <your_password>
	Confirm password: <your_password>
	Nick [<your_username>]: <your_nickname>
	Alternate nick [<your_nickname>_]: <another_nickname>
	Ident [<your_nickname>]: <your_ident>
	Real name [Got ZNC?]: <your_real_name>
	Bind host (optional): [<refer_configuration>](http://wiki.znc.in/Configuration)
	
	Set up a network? (yes/no) [yes]: yes
	Name [freenode]: freenode
	Server host [chat.freenode.net]: chat.freenode.net
	Server uses SSL? (yes/no) [yes]: yes
	Server port (1 to 65535) [6697]: 6697
	Server password (probably empty): 
	Initial channels: 
	
	Launch ZNC now? (yes/no) [yes]: yes

ZNC就在后台运行了。

#### 修改配置

如果需要修改配置，**不要直接编辑znc.conf文件，不要直接编辑znc.conf文件，不要直接编辑znc.conf文件！**

使用webadmin或者controlpanel来修改配置。默认的webadmin运行在ZNC同一个端口上，使用浏览器打开 `http://<your_url_or_ip>:<your_port>` 后登录即可修改配置。

如果你非要编辑znc.conf文件，请按以下步骤进行

	1. pkill -SIGUSR1 znc
		to save current runtime configuration to znc.conf
	2. pkill znc
		to shutdown running ZNC instance
	3. Edit znc.conf
	4. /usr/local/bin/znc
		to start it again with new configuration

### 注册IRC账号

在IRC的输入界面中输入

	/nick <your_nickname>
	/msg NickServ REGISTER <your_password> <your_email@address>

注册之后，你的邮箱会收到一封来自freenode的邮件，按照邮件提示完成就可以了。

唯一属于你的nickname就注册完毕了。

可以通过如下命令查看你的账户信息
	
	/msg NickServ info
	
可以通过如下帮助命令查看命令提示

	/msg NickServ help

以后登录时，要么配置客户端直接登录，要么输入登录的命令

	/msg NickServ identify <your_password>

还可以在同一个账户下注册多个昵称：

	/nick <your_new_nickname>
	/msg NickServ Group

有没有一种在操作Linux命令行的感觉？so cool!

## 参考资料
[1] [Wikipedia: Internet Relay Chat](https://en.wikipedia.org/wiki/Internet_Relay_Chat)

[2] [Please stop using slack](https://drewdevault.com/2015/11/01/Please-stop-using-slack.html)

[3] [Internet Relay Chat Help](http://www.irchelp.org/)

[4] [Wikipedia: IRC Tutorial](https://en.wikipedia.org/wiki/Wikipedia:IRC/Tutorial)

[5] [Wikipedia: Freenode](https://en.wikipedia.org/wiki/Freenode)

[6] [Freenode](http://freenode.net/)

[7] [ZNC](http://wiki.znc.in/ZNC)

[8] [ZNC Installation](http://wiki.znc.in/Installation)

[9] [How to install ZNC on an Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-znc-an-irc-bouncer-on-an-ubuntu-vps)

[10] [Install and Setup ZNC on Ubuntu](https://www.vultr.com/docs/install-and-setup-znc-on-ubuntu)

[11] [Which ports are considered unsafe on Chrome](http://superuser.com/questions/188058/which-ports-are-considered-unsafe-on-chrome)

[12] [Chromium Source Code](https://src.chromium.org/viewvc/chrome/trunk/src/net/base/net_util.cc)