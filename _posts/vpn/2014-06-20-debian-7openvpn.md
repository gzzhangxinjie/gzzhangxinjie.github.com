---
layout: post
title: "Debian 7上安装OpenVPN"
description: ""
category: VPN
tags: [Debian, OpenVPN, VPN]
---
{% include JB/setup %}


###安装 OpenVPN
~~~
#aptitude install openvpn
~~~
或者

~~~
#apt-get install openvpn
~~~

###配置


####初始化

~~~
cp -R /usr/share/doc/openvpn/examples/easy-rsa /etc/openvpn
cd /etc/openvpn/easy-rsa/2.0
chmod +x vars
source ./vars
./clean-all
~~~

####生成ca

~~~
./build-ca
~~~

这一步，一般来说，直接回车就可以。


这里会在/etc/openvpn/easy-rsa/2.0/keys/目录下生成ca.crt,ca.key两个文件。

####生成服务器端证书
  
~~~
./build-key-server server
~~~
 
这里，keys目录下会生成server.crt,server.csr和server.key文件。
 
####生成客户端证书

~~~
./build-key client1
~~~

在keys目录下生成client1.crt,client1.csr和client1.key,为其他客户端生成证书时选择不同的客户端名字即可，比如client2,client3···

**如果多个客户端使用同一个key登录时，会都登陆不上，因此需要为每一个用户生成一个key。**

####创建DH(Diffie Hellman)

~~~
./build-dh
~~~

在keys目录下生成dh1024.pem文件。

##编辑服务器端配置文件
~~~
#cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz /etc/openvpn/
#cd /etc/openvpn
#gunzip server.conf.gz
~~~

编辑server.conf文件，修改后的内容为(没有修改过的内容就不在这里列出了)：

~~~
local a.b.c.d
port 9194
proto tcp
dev tun
ca /etc/openvpn/easy-rsa/2.0/keys/ca.crt
cert /etc/openvpn/easy-rsa/2.0/keys/server.crt
key /etc/openvpn/easy-rsa/2.0/keys/server.key
dh /etc/openvpn/easy-rsa/2.0/keys/dh1024.pem
server 10.8.0.0 255.255.255.0

~~~

1. 指定 openvpn 绑定ip 到 `a.b.c.d`（一台机器可能有多个 ip）；
2. 默认端口为1194，这里改成9194；
3. 协议设置成 tcp，默认是 udp;
4. 设置证书、秘钥的路径。

##服务器端网络设置
因为vpn使用了一个虚拟的私有网段,需要将数据包转发到服务器物理网卡上才可以让客户端正常访问网络，打开`/etc/sysctl.conf`文件,去掉`#net.ipv4.ip_forward=1`前面的注释变为
`net.ipv4.ip_forward=1`, 然后再终端下执行`sysctl -p`使其生效。

在iptables加入


`iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE`

重启网络和OpenVPN：

~~~
#/etc/init.d/networking restart 
#/etc/init.d/openvpn restart 
~~~



##在Windows上使用OpenVPN
1. 下载OpenVPN客户端（推荐从官网下载，https://openvpn.net/index.php/open-source/downloads.html），安装OpenVPN。（安装过程省略，默认安装目录为C:/Program Files/OpenVPN/）
2. 配置密钥证书。
    将ca.crt, xxx.crt, xxx.key（xxx.crt, xxx.key应该是由管理员生成的，私下发送给相应的用户）放在C:/Program Files/OpenVPN/config目录下。

3. 创建client.ovpn。client.ovpn应该放在C:/Program Files/OpenVPN/config目录下，内容为：

~~~
client
dev tun
proto tcp
remote a.b.c.d 9194
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
cert xxx.crt
key xxx.key
ns-cert-type server
comp-lzo
verb 3
~~~

* xxx.crt, xxx.key应该与第二步的xxx.crt, xxx.key一致;
* a.b.c.d 指定了 openvpn 服务器的地址；9194指定了服务器的端口；

配置完成之后，使用管理员权限启动OpenVPN，在桌面的右下角右键点击OpenVPN图标，连接即可。

**用户的key是非常敏感的，因此用户需要保管好自己的key，不要对外泄露。**
