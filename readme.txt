﻿一.检测SVN环境
	1.测试SVN是否安装成功
	svnserve --version
	2.确认SVN是否开启
	netstat -ntpl
	ps -ef | grep svn
	3.停止SVN服务
	killall svnserve
	kill -s 9 111111

二.配置SVN
	1.新建一个SVN总目录
	mkdir /var/svn
	2.设置最大权限
	chmod -R 777 /var/svn
	3.创建版本库
	svnadmin create /var/svn/cjtest
	4.创建SVN用户组
	groupadd svnusers
	5.设置用户组权限
	chgrp -R svnusers /var/svn/
	chmod -R g+rws /var/svn/
	6.创建SVN用户
	vi /var/svn/cjtest/conf/passwd
		chengjun = 123456
	7.将用户加入到用户组并设置权限
	vi /var/svn/cjtest/conf/authz
		svnusers = chengjun
		[/]
		@svnusers = rw
	8.配置SVN相关信息
	vi /var/svn/cjtest/conf/svnserve.conf
		anon-access = none
		auth-access = write
		password-db = /var/svn/cjtest/conf/passwd
		authz-db = /var/svn/cjtest/conf/authz
		realm = cjtest
	9.设置自启动
	vi /etc/rc.local
		svnserve -d -r /var/svn/cjtest
		svnserve -d --listen-port 3690 -r /var/svn/cjtest

三.自动同步
	1.复制钩子文件
	cp /var/svn/cjtest/hooks/post-commit.tmpl /var/svn/cjtest/hooks/post-commit
	2.编辑钩子文件
	vi /var/svn/cjtest/hooks/post-commit
		#!/bin/sh
		export LANG=zh_CN.UTF-8
		/usr/bin/svn update --username chengjun --password 123456 /alidata/www/cjtest
	3.设置权限
	chmod 777 /var/svn/cjtest/hooks/post-commit
	4.进入项目目录
	cd /alidata/www
	5.创建文件夹
	mkdir cjtest
	6.权限
	chmod -R 777 cjtest
	7.第一次导出
	svn checkout svn://123.56.110.43/cjtest/wendaifu
	上面导出导入记得加端口号，默认端口3690
//强制注释
cd /var/svn/wdf/hooks
cp pre-commit.tmpl pre-commit
vi pre-commit
	LOGMSG=`$SVNLOOK log -t "$TXN" "$REPOS" | grep "[a-zA-Z0-9]" | wc -c`
	if [ "$LOGMSG" -lt 2 ];
	then
	  echo -e "请添加注释信息！" 1>&2
	  exit 1
	fi
chmod 777 pre-commit
