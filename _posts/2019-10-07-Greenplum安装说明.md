---
layout:     post
title:      GreenPlum安装说明
subtitle:   
date:       2019-10-07
author:     LT
header-img: img/post-bg-universe.jpg
catalog: true
tags:s
    - GreenPlum
    - data warehouse
    
---

# Greenplum 安装说明

### 一.系统及网络配置说明

1.网络结构

- GP数据通过多台主机进行大量的数据处理；master节点是整个GP集群的入口，用户通过master节点连接并提交sql语句；segment节点功能是处理数据和存储数据，master负责协调各个节点直接的工作负载。

2.集群主机基础配置

2.1.机器准备

- 本次搭建用的是6台320g内存+ssd的机器，每台机器使用RAID-1/0做ssd的磁盘读写阵列，对空间利用率是50%，但是读写效率都提升一倍。

2.2.机器配置
机器配置主要包含一下三个方面：

- 共享内存：如果segment节点没有配置共享内存，GP集群将无法启动。大部分Linux的默认共享内存配置低于GP集群所需要的共享内存；同时，你还需要关闭主机上的OOM killer。
- 网络：GP必须要一个大流量、最优化的网络
- 用户限制：GP必须要对相关文件设置高度的访问权限；默认的文件访问权限限制可能会造成GP访问失败

### 二.系统文件修改及配置安装gp

(以下所有配置均在各个服务器上完成，可以通过scp等方式实现)

1.配置host文件

用root用户登陆各台主机，编辑/etc/hosts，并将IP和域名映射配置加到末尾，为了让5台机器之间通过域名能相互访问，如：

	# master
	192.168.1.198   mdw
	# segments
	192.168.1.199	sdw1
	192.168.1.200   sdw2
	192.168.1.209   sdw3
	192.168.1.210   sdw4
	192.168.1.211   sdw5
	
可在任意一台机器ping对方的域名测试，如：在master上执行 ping sdw1

2.配置sysctl.conf文件

编辑/etc/sysctl.conf文件

	kernel.sem = 1000 1024000 400 10240
	kernel.shmmax = 180000000000
	kernel.shmmni = 4096
	kernel.shmall = 180000000000
	kernel.sysrq = 1
	kernel.core_uses_pid = 1
	kernel.msgmnb = 65536
	kernel.msgmax = 65536
	kernel.msgmni = 2048
	net.ipv4.tcp_syncookies = 1
	net.ipv4.ip_forward = 0
	net.ipv4.conf.default.accept_source_route = 0
	net.ipv4.tcp_tw_recycle = 1
	net.ipv4.tcp_max_syn_backlog = 4096
	net.ipv4.conf.all.arp_filter = 1
	net.ipv4.ip_local_port_range = 1025 65535
	net.ipv4.conf.default.arp_filter = 1
	net.core.netdev_max_backlog = 10000
	net.core.rmem_max = 2097152
	net.core.wmem_max = 2097152
	vm.overcommit_memory = 2
	vm.swappiness = 1
	kernel.pid_max=655360

参数名称|设置值|参数说明
---|---|---
kernel.shmmax|180000000000|表示单个共享内存段的最大值，以字节为单位，此值一般为物理内存的一半，不过大一点也没关系，这里设定的为173G，即"180000000000/1024/1024/1024约为173G"
kernel.shmmni|8092|表示单个共享内存段的最小值，一般为4kB，即4096bit，也可适当调大，一般为4096的2-3倍
kernel.shmall|180000000000|表示可用共享内存的总量,单位是页,一般此值与kernel.shmmax相等
kernel.sem|1000 10240000 400 10240|该文件用于控制内核信号量,信号量是System VIPC用于进程间通讯的方法。建议设置:250 32000 100 128第一列,表示每个信号集中的最大信号量数目。第二列,表示系统范围内的最大信号量总数目。第三列,表示每个信号发生时的最大系统操作数目。第四列,表示系统范围内的最大信号集总数目。所以,（第一列）*（第四列）=（第二列）
kernel.sysrq|1|内核系统请求调试功能控制,0表示禁用,1表示启用
kernel.core_uses_pid|1|这有利于多线程调试,0表示禁用,1表示启用
kernel.msgmnb|65536|该文件指定一个消息队列的最大长度（bytes）。缺省设置：16384
kernel.msgmax|65536|该文件指定了从一个进程发送到另一个进程的消息的最大长度（bytes）。进程间的消息传递是在内核的内存中进行的，不会交换到磁盘上，所以如果增加该值，则将增加操作系统所使用的内存数量。缺省设置：8192
kernel.msgmni|2048|该文件指定消息队列标识的最大数目，即系统范围内最大多少个消息队列。
net.ipv4.tcp_syncookies|1|表示开启SYN Cookies,当SYN等待队列溢出时,启用cookies来处理,可以防范少量的SYN攻击,默认为0,表示关闭。1表示启用
net.ipv4.ip_forward|0|该文件表示是否打开IP转发。0:禁止 1:转发 缺省设置:0
net.ipv4.conf.default.accept_source_route|0|是否允许源地址经过路由。0:禁止 1:打开 缺省设置:0
net.ipv4.tcp_tw_recycle|1|允许将TIME_WAIT sockets快速回收以便利用。0表示禁用,1表示启用
net.ipv4.tcp_max_syn_backlog|4096|增加TCP SYN队列长度，使系统可以处理更多的并发连接。一般为4096，可以调大，必须是4096的倍数，建议是2-3倍
net.ipv4.conf.all.arp_filter|1|表示控制具体应该由哪块网卡来回应arp包,缺省设置0, 建议设置为1
net.ipv4.ip_local_port_range|1025 65535|指定端口范围的一个配置,默认是32768 61000，可调整为1025 65535
net.core.netdev_max_backlog|10000|进入包的最大设备队列.默认是1000,对重负载服务器而言,该值太低,可调整到16384/32768/65535
net.core.rmem_max|2097152|最大socket读buffer,可参考的优化值:1746400/3492800/6985600
net.core.wmem_max|2097152|最大socket写buffer,可参考的优化值:1746400/3492800/6985600
vm.overcommit_memory|2|Linux下overcommit有三种策略，0:启发式策略，1:任何overcommit都会被接受。2:当系统分配的内存超过swap+N%*物理RAM(N%由vm.overcommit_ratio决定)时，会拒绝commit，一般设置为2
vm.swappiness|1|当物理内存超过设置的值是开始使用swap的内存空间,计算公式是100-1=99%表示物理内存使用到99%时开始交换分区使用
kernel.pid_max|655360|用户打开最大进程数,全局配置的参数

3.配置linux文件描述符

/etc/security/limits.conf

	* soft nofile 524288
	* hard nofile 524288
	* soft nproc 131072
	* hard nproc 131072

4.挂载磁盘由汤老师完成

5.关闭防火墙
	
	systemctl status firewalld.service # 查看防火墙状态
	----
	systemctl stop firewalld.service # 关闭防火墙服务
	systemctl disable firewalld.service # 取消服务开机自启动
	-----
	systemctl status firewalld.service # 查看防火墙状态

6.同步集群机器时间与主节点一致

7.重启机器，让所有配置生效

### 三.安装Greenplum数据库，配置gpadmin用户（也均是在各个服务器上都需要实现）

1.配置用户及密码

2.将文件夹权限分配给gpadmin用户

	chown -R gpadmin:gpadmin /AppData

3.依赖安装详见Centos7使用Nexus私服说明，安装完成后使用yum等命令安装所需依赖

4.安装greenplum6.0
	
	# 从官网下载rpm包
	
	# 使用 prefix 参数选定安装位置
	rpm -ivh greenplum-db-6.0.0-rhel7-x86_64.rpm --prefix=/AppData/greenplum/


### 四. 配置SSH免密登录(此章操作只需在master主机操作)

1.登录master主机并切换成gpadmin用户

2.初始化Greenplum的path文件

	# 初始化之后才能使用gpssh等命令，可以将此命令写入环境变量，不用每次登录都进行操作

	echo "source /AppData/greenplum/greenplum-db/greenplum_path.sh" >> ~/.bashrc

	source ~/.bashrc

3.在master节点上生成ssh-key文件

	$ ssh-keygen # 一直回车就行了

4.使用ssh-copy-id命令打通集群ssh通路

	ssh-copy-id sdw1
	ssh-copy-id sdw2
	ssh-copy-id sdw3
	ssh-copy-id sdw4
	ssh-copy-id sdw5
	# 依次输入各机器gpadmin的密码即可通过如 ssh sdw1 命令从master节点远程ssh连接其他机器了

5.在master节点生成主机列表文件

	# 就在~目录下生成
	$ vi hostfile
	
	mdw
	sdw1
	sdw2
	sdw3
	sdw4
	sdw5

	$ vi seg-host
	
	sdw1
	sdw2
	sdw3
	sdw4
	sdw5

	# tips：确保在每台机器上的/etc/hosts文件上配置域名解析文件，否则各个主机之间将不能访问

6.很“程序猿”的地方来了，完成n-n免密登录

	gpssh-exkeys -f hostfile

	# 之后执行

	gpssh -f hostfile 

	便可以用对六台主机进行同时操作了

	=> ls
	[sdw1] gpAdminLogs  inithosts.sh  limits.conf  sysctl.conf
	[ mdw] gpAdminLogs  gpconfigs	hostfile  segments
	[sdw5] gpAdminLogs  limits.conf  perl5  sysctl.conf
	[sdw4] gpAdminLogs  inithosts.sh  limits.conf	sysctl.conf
	[sdw3] gpAdminLogs  inithosts.sh  limits.conf	sysctl.conf
	[sdw2] gpAdminLogs  limits.conf  sysctl.conf

### 五.创建数据存储

1.gpadmin用户登录

	mkdir -p /AppData/gpdata/master

2.gpssh创建primary以及mirror目录

	gpssh -f seg-host

	mkdir -p /AppData/gpdata/gpdatap1
	mkdir -p /AppData/gpdata/gpdatap2
	mkdir -p /AppData/gpdata/gpdatam1
	mkdir -p /AppData/gpdata/gpdatam2

### 六.初始化Greenplum集群

1.使用gpadmin用户

	# 拷贝初始化文件到用户目录下

	$ cp $GPHOME/docs/cli_help/gpconfigs/gpinitsystem_config /home/gpadmin


	$ vi gpinitsystem_config

修改文件如下

	ARRAY_NAME="Greenplum Data Platform"
	SEG_PREFIX=gpseg
	PORT_BASE=6000
	declare -a DATA_DIRECTORY=(/AppData/gpdata/gpdatap1 /AppData/gpdata/gpdatap2)
	MASTER_HOSTNAME=mdw
	MASTER_DIRECTORY=/AppData/gpdata/gpmaster
	MASTER_PORT=2345
	TRUSTED_SHELL=/usr/bin/ssh
	CHECK_POINT_SEGMENTS=8
	ENCODING=UNICODE
	MIRROR_PORT_BASE=7000
	REPLICATION_PORT_BASE=34000
	MIRROR_REPLICATION_PORT_BASE=44000
	declare -a MIRROR_DATA_DIRECTORY=(/AppData/gpdata/gpdatam1 /AppData/gpdata/gpdatam2)
	MACHINE_LIST_FILE=/home/gpadmin/gpconfigs/segments

2.配置MASTER_DATA_DIRECTORY
	$ vi .bashrc

	export MASTER_DATA_DIRECTORY=/AppData/gpdata/gpmaster/gpseg-1
	export PGPORT=2345
	export PGDATABASE=testDB

3.执行初始化命令

	gpinitsystem -c ~/gpinitsystem_config -h seg_host

	# 初始化成功

	=> Greenplum Database instance successfully created.


至此，gp完成安装


