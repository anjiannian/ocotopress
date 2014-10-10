---
layout: post
title: "Debian下搭建VPN"
date: 2014-03-03 21:54
comments: true
categories: Tools 
tags: Debian Vpn Vps
---



最近一直在用[Goagent](http://code.google.com/p/goagent/)的圖形界面[GoagentX](http://goagentx.com/)來上網,無論速度還是流量都很夠用。無奈自由習慣了，用手機時也不願受拘束。鑑於最近準備從weibo轉向twitter，還是自己搭個VPN給手機用吧。搭起來發現速度比Goagent差好遠，不過手機也夠用。

我用的VPS是[DigitalOcean](https://www.digitalocean.com/?refcode=5f445434288b "推廣碼")用的不久，感覺速度還可以，鏈接是推廣碼

<!-- more -->

## 檢查下面兩項確認可以搭建

	cat /dev/net/tun
	#返回结果为下面的文本，表明通过：
	cat: /dev/net/tun: File descriptor in bad state

	cat /dev/ppp
	#返回以下结果，则通过：
	cat: /dev/ppp: No such device or address

## 安装如下3个軟件包：

	# apt-get install pptpd ppp iptables

## 设置ip地址：

	# vi /etc/pptpd.conf 	

	localip 192.168.0.1 		#vpn服務器的虛擬ip，隨意設置
	remoteip 192.168.0.234-238,192.168.0.245 	#分配給連接的客戶端ip，於上面同一個網段

## 增加vpn訪問用户：

	# vi /etc/ppp/chap-secrets
	
	username pptpd password * #最后的*表示允许任意ip连接
	#用戶名， 鏈接方式， 密碼 ，ip地址

## 设置dns解析支持：

	# vi /etc/ppp/options

	ms-dns 8.8.8.8
	ms-dns 8.8.4.4
	logfile /var/log/pptpd.log
	//無法鏈接或其他問題可以通故這裏查看log
	
## 打开转发：

	# vi /etc/sysctl.conf

	net.ipv4.ip_forward=1

## 重启pptpd服务：

	# /etc/init.d/pptpd restart

## 激活转发参数：

	# sysctl –p


## 配置iptables：

	iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -j SNAT --to-source 192.157.212.20
	#適配OpenVZ服務器, 192.168.80.0要和上面 /etc/pptpd.conf 设置的IP段匹配，12.34.56.78为你的VPS的公网IP地址

	#iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE
	#XEN的vps用這個，eth0對應外網的設備名，可以 ifconfig 查看

網上東拼西湊加上自己實踐，怕下次還要再找一次，就記下來備用。

