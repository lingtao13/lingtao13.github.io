---
layout:     post
title:      dbsync-mysql2pgsql命令使用
subtitle:   安装小记
date:       2019-11-06
author:     LT
header-img: img/post-bg-ios10.jpg
catalog: true
tags:
    - GreenPlum
    - data warehouse

---

# 阿里云 rds_dbsync/mysql2pgsql 命令使用

### 一.rds_dbsync 简介

dbsync 项目目标是围绕 PostgreSQL Greenplum ,实现易用的数据的互迁功能。

项目地址： [rds_dbsync](https://github.com/aliyun/rds_dbsync) https://github.com/aliyun/rds_dbsync

### 二.下载及安装

git源下载rds_dbsync项目代码

	git clone https://github.com/aliyun/rds_dbsync.git

下载之后大概有四个项目：  
- pgsql2pgsql
- binlog_minner binlog_loader
- pgsql2gp
- mysql2pgsql

前面三个是否使用可以之后评估，目前而言mysql2pgsql用来做分表分库的大数据量mysql数据迁移到greenplum/postgresql效果还是非常不错的

经测试1.4T的数据(46个库，其中15个512个表的大库，31个48个表的小库)大约只需要不到8个小时就能迁移完成。

配置命令至环境变量

	PATH 后添加:/($ABSOLUTE_DOWNLOAD_PATH)/mysql2pgsql.bin.**/bin
	
	$ source .bashrc

### 三.命令的简单使用

	# 配置在bin目录下配置my.cfg
	$ vim my.cfg	


	#此处配置mysql库信息
	#一次只能执行一个库的导入导出
	[src.mysql]
	host= "192.168.1.101"
	port = "3306"
	user = "user"
	password = "password"
	db = "dbname"
	encodingdir = "share"
	encoding = "utf8"
	
	# 此处配置pgsql库信息
	[desc.pgsql]
	connect_string = "host=192.168.1.100 dbname=gpdbname port=2345 user=user password=passwd"
	target_schema = "public"
	ignore_copy_error_count_each_table = "0"

	# 输出建表语句
	$ mysql2pgsql -d -n>gpdbname.txt 2>&1

	# 修改分布键 此处为eid
	$ sed -i 's/\(<distribution key>\)/eid/g' gpdbname.txt

	# 经测试，去除建表语句第一行"ignore copy error count 0 each table" 则会消除建表有时少一个的bug
	$ sed -i '1d' gpdbname.txt

	# gp建库
	$ psql -c "create database gpdbname"

	# gp建表
	$ psql -d gpdbname -f "gpdbname.txt"

	# 开始执行库迁移
	$ mysql2pgsql


### 配合使用python脚本批量入库

	#!/usr/bin/python
	# -*- coding:utf8 -*-
	# __author__= 'lingtao'
	
	import sys
	import os
	import stat
	import time
	
	db_name_list = ["db_", "db_sub_", "db_wol_", "db_ent"]
	
	
	def mod_files(db_name):
	    """
	    generate create table file and modify my.cfg file
	    :param db_name: 
	    :return: 
	    """
	    output = open("%s.sh" % db_name, "w")
	    output.writelines(
	        "#!/bin/sh\n"
	        "psql -c \"create database " + db_name +
	        "\"\n"
	        "sleep 1\n"
	        "mysql2pgsql -d -n > " + db_name +
	        ".txt 2>&1\n"
	        "sleep 1\n"
	        "sed -i 's/\(<distribution key>\)/eid/g' " + db_name +
	        ".txt\n"
	        "sed -i 's/ignore copy error count 0 each table//g' " + db_name +
	        ".txt\n"
	        "sleep 1\n"
	        "psql -d " + db_name +
	        " -f \"" + db_name +
	        ".txt\"\n"
	        "sleep 1\n"
	        "mysql2pgsql"
	    )
	
	    mycfg = open("my.cfg", "w")
	    mycfg.writelines(
	        "[src.mysql]\n"
	        "host= \"192.168.1.100\"\n"
	        "port = \"3306\"\n"
	        "user = \"user\"\n"
	        "password = \"passwd\"\n"
	        "db = \"" + db_name +
	        "\"\n"
	        "encodingdir = \"share\"\n"
	        "encoding = \"utf8\"\n\n"
	        "[desc.pgsql]\n"
	        "connect_string = \"host=192.168.1.101 dbname=" + db_name +
	        " port=2345 user=user password=passwd\"\n"
	        "target_schema = \"public\"\n"
	        "ignore_copy_error_count_each_table = \"0\""
	    )
	    
	    # every time we change/generate files, we should change the permission of files, which can be execute by user.
	    os.chmod("%s.sh" % db_name, stat.S_IREAD + stat.S_IWRITE + stat.S_IEXEC)
	    os.chmod("my.cfg", stat.S_IREAD + stat.S_IWRITE + stat.S_IEXEC)
	    
	    # remember to close the file.
	    mycfg.close()
	    output.close()
	
	
	def run_sh(db_name):
	    """
	    To run the sh file, which can transfer one total database each time.
	    :param db_name: 
	    :return: 
	    """
	    os.system("./" + db_name + ".sh")
	
	
	if __name__ == u'__main__':
		# which is Sub database and sub table, we can use this way
	    for name in db_name_list:	
	        if name in ["db_", "db_sub_", "db_wol_"]:
	            for i in range(16):
	                new_name = name + str(i)
	                mod_files(new_name)
	                run_sh(new_name)
	        else:
	            mod_files(name)
	            run_sh(name)



	# 使用程序
	$ python GenerateSH.py

以上。




	