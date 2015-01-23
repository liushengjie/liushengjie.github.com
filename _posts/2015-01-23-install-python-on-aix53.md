---
layout: post
title: Install python On AIX5.3
categories: [python]
tags: [python,AIX]
published: True

---
{% include JB/setup %}

今天在公司AIX5.3和6.1上分别安装了python，总结一下：

1、下载所需资源：[IBM AIX TOOLBOX](ftp://ftp.software.ibm.com/aix/freeSoftware/aixtoolbox/RPMS/ppc/)

下载下面这几个rpm，上传到主机并执行如下命令

	rpm -ivh gdbm-1.8.3-5.aix5.2.ppc.rpm
	rpm -ivh gdbm-devel-1.8.3-5.aix5.2.ppc.rpm
	rpm -ivh expat-2.0.1-2.aix5.3.ppc.rpm
	rpm -ivh expat-devel-2.0.1-2.aix5.3.ppc.rpm
	rpm -ivh readline-4.3-2.aix5.1.ppc.rpm （只能安装4这个版本）
	rpm -ivh readline-devel-4.3-2.aix5.1.ppc.rpm
	rpm -ivh python-2.6.2-2.aix6.1.ppc.rpm

ok，AIX6.1成功安装python，但是5.3上执行rpm -ivh python-2.6.2-1.aix5.3.ppc.rpm的时候，提示缺少依赖。

	error: failed dependencies:
	libcrypto.a(libcrypto.so.0.9.8) is needed by python-2.6.2-1
	libssl.a(libssl.so.0.9.8) is needed by python-2.6.2-1

查看rpm依赖关系

	rpm -qpR python-2.6.2-1.aix5.3.ppc.rpm

2、于是查看rpm命令后，决定采用忽略依赖强制安装

	rpm -i --force --nodeps python-2.6.2-1.aix5.3.ppc.rpm

居然安装成功了，界面输入python进入python shell界面，一阵大喜。赶紧开始运行程序，but….

	Traceback (most recent call last):
	File "httpserver.py", line 30, in
	import urllib, urllib2
	File "/opt/freeware/lib/python2.6/urllib2.py", line 91, in
	import hashlib
	File "/opt/freeware/lib/python2.6/hashlib.py", line 136, in
	md5 = __get_builtin_constructor('md5')
	File "/opt/freeware/lib/python2.6/hashlib.py", line 63, in __get_builtin_constructor
	import _md5
	ImportError: No module named _md5

这是什么情况，没有md5模块，不可能啊….网上一番查找后，原来还是libcrypto.a和libssl.a这两个库捣鬼。

进入/opt/freeware/lib下查看，有libcrypto.a和libssl.a这两个库啊。之后进入/opt/freeware/lib/python2.6/lib-dynload 执行ldd _hashlib.so

	_hashlib.so needs:
	/opt/freeware/lib/libcrypto.a(libcrypto.so.0.9.8)
	ar: 0707-109 Member name libcrypto.so.0.9.8 does not exist.
	dump: /tmp/tmpdir1040414/extract/libcrypto.so.0.9.8: 0654-106 Cannot open the specified file.

libcrypto.so.0.9.8 不存在，回到/opt/freeware/lib下，执行ar t libcrypto.a

	libcrypto.so.0.9.7
	libcrypto.so.0

原来如此，libcrypto.a下的成员是libcrypto.so.0.9.7

随后，从AIX6.1的机器上取下libcrypto.a和libssl.a两个文件，ar t 一下，显示libcrypto.so.0.9.8。

ok，最后就是把/opt/freeware/lib中的libcrypto.a替换一下了，将这两个库拷到/lib下，进入/lib执行

	ln -sf libcrypto.a /opt/freeware/lib/libcrypto.a

现在进入python，import md5一下，perfect！折腾了三四个小时，学到了不少东西，记录一下～～～ 