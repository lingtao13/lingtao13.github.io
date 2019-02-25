---
layout:     post
title:      《关于搭建ElasticSearch+Kibana的一点坑》
subtitle:   
date:       2019-02-25
author:     LT
header-img: img/home-bg-art.jpg
catalog: true
tags:
    - 搜索引擎
    - 实战笔记
---
# 关于搭建ElasticSearch+Kibana的一点坑
- 怪我计算机网络没学好，搭建监听的时候一直忘了开放全网
		
- 配置文件 
		
		bootstrap.memrory_lock: true
		bootstrap.system_call_filter: false
		
		cluster.name: es_test
		node.name: node-1
		
		network.host: <your elastic ip> (important! Not 127.0.0.1 as default)
		http.port: 9200
		
		https.cors.enabled: true
		http.cors.allow-origin: "*"
		
		node.master= true
		node.data: true
		
		discovery.zen.ping.unicast.hosts:["<your ip>"]
			
	一开始我是用rpm的方式安装的 elasticsearch 服务，但是发现没有报错信息便自动停止了，万般无奈不知道如何查询rpm的报错信息之后，卸载了rpm的es服务，重新手动安装tar包，这样运行便有报错，这样可以依靠机友社区/广大大神网友来解决问题。
	
	error [1] :max file descriptors [65535] for elasticsearch process is too low, increase to at least [65536]
	
	solution: change the limits.conf
		
		sudo vi /etc/security/limits.conf
		
		# shift+G to end of the file and add:
		
		* soft core 102400
		* hard core 102400
		* soft nproc 2400
		* hard nproc 4096
	error [2] bootstrap check failed,memory locking requested for elasticsearch process but memory is not locked
	
	solution: as the same upside:
	
		sudo vi /etc/security/limits.conf
		
		# shift+G to end of the file and add:
		
		* soft memlock unlimited 
		* hard memlock unlimited

	以上就是我本次配置elasticsearch+kibana的所有坑。其实算很少的了，但是在监听那卡了很久，而且rpm服务的日志我现在还是不知如何查看，如有大神知晓请务必告诉我。
	


