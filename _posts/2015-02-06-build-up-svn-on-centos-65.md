---
layout: post
title: "Build up SVN on CentOS 6.5"
description: ""
category: [SVN]
tags: [SVN]
---
{% include JB/setup %}

中心新增两台服务器，用作SVN服务器管理相关代码，这两台服务器要做主从互备，以备一台服务器宕机后，可使用另一台进行代码提交。
针对SVN来说，不像git，有分布式的概念，所以计划打算用svnsync定时备份代码库。
	
## centos下搭建SVN环境及相关配置操作
### 安装
1. yum install subversion  
2. mkdir /opt/svn/repos
3. svnadmin create /opt/svn/repos

执行上面的命令后，自动在repos下建立多个文件， 分别是conf, db,format,hooks, locks, README.txt
  
### 配置

* 用户及密码配置

配置文件：opt/svn/repos/conf/passwd

	[users]
	owlliu = owlliu
	test = test11
等号左边为用户名，右边为密码。

* 权限及用户组配置

配置文件：opt/svn/repos/conf/authz

	[groups]
	admin = owlliu   ##组配置，多成员可用逗号分隔
	
	[/]
	@admin = rw           ##针对于/这个目录，admin组成员有读写权限

	[/svn/trunk]
	test = r              ##针对于/svn/trunk这个目录，test用户有读权限
	
* svnserve.conf相关配置

配置文件：opt/svn/repos/conf/svnserve.conf

	anon-access = none    ##使非授权用户无法访问
	auth-access = write   ##使授权用户有写权限
	password-db = /opt/svn/repos/conf/passwd
	authz-db = /opt/svn/repos/conf/authz
	realm = repos  ##认证命名空间，subversion会在认证提示里显示，并且作为凭证缓存的关键字。

### 启动服务
	
	启动svn: svnserve -d -r /opt/svn/repos
	如果已经有svn在运行，可以换一个端口运行
	svnserve -d -r /opt/svn/repos --listen-port 3391
	
	ps:
	启动svn服务器后，本机访问正常，但是在其他机器访问总是无法连接
	**关闭本机防火墙 /etc/init.d/iptables stop**
	
## svn远程备份镜像

svnsync指令可以将两个svn仓库经行同步备份，具体操作如下：

1. 在备份机上建立一个空的svn仓库
2. 修改此仓库的hook：pre_revprop-change 全部内容只有一行：exit 0
3. 执行初始化：svnsync init file:///opt/svn/repos svn://192.168.1.117
   
   *ps:初次使用的是：svnsync init svn://192.168.1.119/ svn://192.168.1.117此条命令，但是报认证失败，后改成file则成功*
   
4. 执行同步命令：svnsync file:///opt/svn/repos
5. 将此命令加入crontab，定时启动：0,15,30,45 8-18 * * * svnsync sync file:///opt/svn/repos >> /opt/svn/sync.log


##TODO
后续任务计划做一个SVN监控管理软件，可以监控所有成员在SVN提交的代码信息，并进行相关的统计。

##Reference
* [svn book](http://www.subversion.org.cn/svnbook/1.4/index.html)
* [http://www.zhihu.com/question/20093241](http://www.zhihu.com/question/20093241)

	





