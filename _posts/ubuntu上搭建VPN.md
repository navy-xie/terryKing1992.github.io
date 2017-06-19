---
title: ubuntu上搭建VPN
date: 2016-03-05 13:24:15
tags: [VPN]
---

点对点隧道协议（PPTP）是VPN服务的一种最简单的实现协议，其它常见的VPN类型还有：使用IPsec的第2层隧道协议（L2TP/IPsec）、安全套接字隧道协议（SSL VPN）。本文主要讨论PPTP VPN服务在Ubuntu上的安装和配置。

<!-- more -->

A.使用apt源服务来安装PPTPD服务

	sudo apt-get update
	sudo apt-get install pptpd

B.安装完成之后编辑pptpd.conf配置文件

	sudo vi /etc/pptpd.conf

	#确保如下选项的配置
	option /etc/ppp/pptpd-option                    #指定PPP选项文件的位置
	debug                                           #启用调试模式
	localip 192.168.0.1                             #VPN服务器的虚拟ip
	remoteip 192.168.0.200-238,192.168.0.245        #分配给VPN客户端的虚拟ip

C.编辑PPP选项配置文件

	sudo vi /etc/ppp/pptpd-options

	#确保如下选项的配置
	name pptpd                      #pptpd服务的名称
	refuse-pap                      #拒绝pap身份认证模式
	refuse-chap                     #拒绝chap身份认证模式
	refuse-mschap                   #拒绝mschap身份认证模式
	require-mschap-v2               #允许mschap-v2身份认证模式
	require-mppe-128                #允许mppe 128位加密身份认证模式
	ms-dns 8.8.8.8                  #使用Google DNS
	ms-dns 8.8.4.4                  #使用Google DNS
	proxyarp                        #arp代理
	debug                           #调试模式
	dump                            #服务启动时打印出所有配置信息
	lock                            #锁定TTY设备
	nobsdcomp                       #禁用BSD压缩模式
	logfile /var/log/pptpd.log      #输出日志文件位置

D.编辑用户配置文件来添加用户

	sudo vi /etc/ppp/chap-secrets

	#格式：用户名   服务类型   密码   分配的ip地址
	test    *    1234    *
	#第一个*代表服务可以是PPTPD也可以是L2TPD，第二个*代表随机分配ip

E.重启PPTPD服务

	sudo service pptpd restart

F.配置网络和路由规则 设置ipv4转发


	sudo sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/g' /etc/sysctl.conf
	sudo sysctl -p
	设置iptables NAT转发

	#注意这里eth0代表你的外网网卡，请用ifconfig查看或者咨询网络管理员
	sudo iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE
	#如果上面的命令报错,那么可以尝试以下的命令，其中xxx.xxx.xxx.xxx代表你的VPS外网ip地址
	sudo iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j SNAT --to-source xxx.xxx.xxx.xxx
	设置MTU来确保过大的包不会被丢弃（这个可以不做）


	sudo iptables -I FORWARD -s 192.168.0.0/24 -p tcp --syn -i ppp+ -j TCPMSS --set-mss 1300
	iptables的设置重启之后会取消，所以可以将上面的命令加入到/etc/rc.local来确保每次重启都会执行设置。

G.至此设置全部完成，可以用任意一个客户端机器如MAC、PC或者手机来新建一个VPN连接使用用户test，密码1234进行测试。另外网上提供一种自动安装脚本，可以参见如下操作：