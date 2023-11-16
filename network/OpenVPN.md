## 客户端连接 VPN



参阅文档：

- https://wiki.debian.org/OpenVPN
- https://openvpn.net/community-resources/how-to



安装 openvpn：

```bash
apt update
apt install openvpn
```



安装目录和文件说明：

- systemd 的 service 文件默认路径 `/lib/systemd/system/` 
- 工作目录 `/etc/openvpn` 

```bash
# @ 需要接收启动参数，便于灵活使用不同的配置
openvpn-client@.service
openvpn-server@.service
openvpn.service
openvpn@.service
```



客户端启动连接：

- `openvpn-client@.service` 中定义了默认启动方式

```bash
# 将 openvpn 所以配置文件放在 /etc/openvpn/client 目录下
# 用于连接的配置文件后缀必须为 .conf
# 启动客户端连接，@ 后面传递配置文件名（不含后缀，openvpn会自动补全）
$ systemctl start openvpn-client@mtr-cen.service

# 查看启动和连接状态，连接成功会有分配地址和推送路由的日志
$ systemctl status openvpn-client@mtr-cen.service
```



查看网卡和路由：

```bash
ip a
ip route list
```



以上是在网关设备中连接 VPN，网关能与 VPN 通讯，但是网关设备下客户端不能直接与 VPN 通讯

有以下两个需求：

- 局域网其他设备在不连接 VPN 的情况下也能与 VPN 的后端通讯
- 当网关设备连接 VPN 时，同时也做为 VPN 服务端，让连接到网关 VPN 的设备也能与网关所连接的 VPN 的后端通讯



目前想到了以下几种解决方案：

1. 客户端访问的目标地址为网关，网关中做地址转换，将访问本网关的数据包转发给 VPN 中的端点（可行，已验证）
2. 两个 VPN 的虚拟网卡互相添加路由，并做互相做地址转换（第一种情况是否也可以通过这种方式（实际网卡与VPN虚拟网卡之间转换）？）

> 实验前需要熟悉 iptables 表和链的作用

示例：

```bash
# iptables 根据数据包的 MARK 标记进行地址转换
iptables -t nat -A PREROUTING -m mark --mark 5 -j DNAT --to-destination 192.168.2.200
iptables -t nat -A POSTROUTING -m mark --mark 5 -j SNAT --to-source 192.168.1.100
```

第一种方案测试成功：

```bash
$ iptables -t nat -A PREROUTING -d 192.168.0.222 -p tcp --dport 32767 -j DNAT --to-destination 192.168.0.153:56322
$ iptables -t nat -A POSTROUTING -d 192.168.0.153 -p tcp --dport 56322 -j MASQUERADE
```



客户端设备直接访问 VPN 端点时，在网关中抓取一直是 ARP 询问 VPN 端点的地址，最终找不到地址而无法建立通讯（不清楚是不是 VPN 与本地处于同一网段的问题导致路由失效）

```bash
# 客户端地址为 192.168.0.55，VPN 后端为 192.168.0.153
$ tcpdump host 192.168.0.153 -nn
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
23:57:07.534548 ARP, Request who-has 192.168.0.153 tell 192.168.0.55, length 46
23:57:08.194073 ARP, Request who-has 192.168.0.153 tell 192.168.0.55, length 46
23:57:09.204920 ARP, Request who-has 192.168.0.153 tell 192.168.0.55, length 46
```

创建 tun 类型的虚拟网卡与 openvpn 的 tun0 互通，`ip link` 只能创建其他类型的网络接口

```bash
# 创建
ip tuntap add mode tun dev home

# 删除
ip tuntap del dev home mode tun

# 添加地址
ip addr add 172.30.0.1/24 dev home

# 启动
ip link set home up

ip route add default via 172.30.0.1 dev home table 100
ip rule add from 10.0.0.1 table 100
```

> 该实验未完成！可行性待验证



## OpenVPN 服务器



根据 `/lib/systemd/system/openvpn@.service` 中的定义，将配置文件放入 `/etc/openvpn` 目录下，配置文件后缀必须以 `.conf` 结尾，文件名称则在启动服务端时传递



openvpn 配置文件示例：`server.conf`

- 使用 OpenSSL 准备好自签证书

```bash
port 1194
proto udp
dev tun
ca server/ca.crt
cert server/server.crt
key server/server.key
# dh /etc/openvpn/certs/dh.pem
dh none
server 172.31.255.0 255.255.255.0
push "route 192.168.0.253 255.255.255.255"
push "route 192.168.0.252 255.255.255.255"
# client-config-dir ccd
# client-to-client
keepalive 10 120
cipher AES-256-GCM
# tls-auth /etc/openvpn/certs/ta.key 0
compress lz4-v2
push "compress lz4-v2"
max-clients 10
log-append /var/log/openvpn/openvpn.log
verb 3
mute 10
management 127.0.0.1 31194
```

启动 OpenVPN Server

```bash
# @ 后面需要传递配置文件名（不含后缀）
$ systemctl start openvpn@server
```



## OpenVPN 网络拓扑

参阅文档：

- https://community.openvpn.net/openvpn/wiki/Topology

- https://community.openvpn.net/openvpn/wiki/Concepts-Addressing

  

OpenVPN 有以下三种网络拓扑类型：

- `subnet` ：推荐使用此模式，但这不是默认使用的模式
- `net30` ：默认使用的拓扑模式（启用），只是为了兼容旧版本以及旧版 windows，该模式为每个客户端都分配了一个虚拟的 `/30` (255.255.255.252) 网段地址，每个客户端占用 4 个 IP，另外 4 个用于服务器。
- `p2p` ：点对点网络拓扑，但与 windows 不兼容



### subnet

服务端配置示例：

```bash
--dev tun 
--topology subnet 
--server 10.8.0.0 255.255.255.0

# ccd 配置
--ifconfig-push 10.8.0.3 255.255.255.0
```



### net30

服务端配置示例：

```bash
--dev tun
--topology net30
--server 172.31.255.0 255.255.255.0

# ccd 配置
--ifconfig-push 172.31.255.10 172.31.255.9
```

> 在此示例中 Server 使用 172.31.255.0/24，/24 可以划分出 64 个 /30，根据 net30 网络拓扑性质，会向每个客户端分配一个 /30 的子网，即每个客户端会占用 4 个 IP 位（可用 IP 两个，一个网络地址，一个广播地址），由于服务端已经占用了一个 /30 的子网（172.31.255.0/30），剩余 63 个 /30 供客户端使用！



### p2p

配置示例：

```bash
--dev tun 
--topology p2p 
--push "topology p2p" 
--mode server 
--tls-server 
--ifconfig 10.8.0.1 10.8.0.0

# ccd 配置
--ifconfig-push 10.8.0.3 10.8.0.1
```

> 使用 p2p 网络拓扑时，由于点对点的网络性质是没有服务端，因此 --server 和 --ifconfig-pool 不能使用。



## 为客户端分配静态 IP

在线子网划分：https://subnet-calculator.org/



1. 创建客户端 IP 配置文件，文件名必须为用户名，如果使用证书认证，即 Subject 中的 CN 字段

   ```bash
   # 创建配置目录
   mkdir ccd
   
   # 创建 IP 配置文件，注意 /30 网段划分，4 个 IP 必须在同一网段
   echo 'ifconfig-push 172.31.255.10 172.31.255.9' > ccd/vincent
   echo 'ifconfig-push 172.31.255.14 172.31.255.13' > ccd/pc
   
   # 错误示例：Push IP 不在同一网段
   ifconfig-push 172.31.255.20 172.31.255.19
   ```

   > 注意：Push 的 IP 要符合网络拓扑的性质，此处默认为 net30

2. 服务端添加 client-config-dir 配置

   ```bash
   echo 'client-config-dir ccd' >> server.conf
   ```



## 过期的客户端证书连接

openvpn 不允许使用过期的证书进行连接，也没有相关的配置，但是可以通过将 OpenVPN 服务端系统时间调整到客户端证书有效期内来绕过这个限制。

> 调整系统时间可能会导致其他服务因时间校验失败异常

使用 faketime 工具可以只对某个单独的程序伪造时间

项目地址:

- https://github.com/wolfcw/libfaketime



示例：在 debian 中安装 faketime

```bash
# apt install libfaketime
apt install faketime
```

使用 faketime 伪造时间并运行 OpenVPN

```bash
$ /usr/bin/faketime '2023-06-01 22:14:51' /usr/sbin/openvpn --daemon ovpn-server --status /run/openvpn/server.status 10 --cd /etc/openvpn --script-security 2 --config /etc/openvpn/server.conf --writepid /run/openvpn/server.pid
```

