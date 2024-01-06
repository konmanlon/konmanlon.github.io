## 透明代理规则

主机网络接口

```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: ens5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 64:23:00:c3:f1:9a brd ff:ff:ff:ff:ff:ff
    altname enp0s5
    inet 10.0.12.5/22 brd 10.0.15.255 scope global ens5
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fec3:a19f/64 scope link 
       valid_lft forever preferred_lft forever
```

透明代理规则:

```bash
-P PREROUTING ACCEPT
-P INPUT ACCEPT
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
-P POSTROUTING ACCEPT
-N XRAY
-N XRAY_SELF
-A PREROUTING -m limit --limit 83/sec -j LOG --log-prefix "prerouting: " --log-level 7
-A PREROUTING -j XRAY
-A OUTPUT -m limit --limit 83/sec -j LOG --log-prefix "output: " --log-level 7
-A OUTPUT -j XRAY_SELF
-A XRAY -i ens5 -j RETURN  # 从 ens5 网口入站的流量不代理
-A XRAY -d 10.0.0.0/8 -j RETURN  # 代理网关本机时，OUTPUT 流量会被路由到 lo 网口，避免本机流量在 lo 死循环
-A XRAY -p tcp -j TPROXY --on-port 22222 --on-ip 0.0.0.0 --tproxy-mark 0x1/0xffffffff
-A XRAY -p udp -j TPROXY --on-port 22222 --on-ip 0.0.0.0 --tproxy-mark 0x1/0xffffffff

# 代理网关本机的规则，由 OUTPUT 链调用
-A XRAY_SELF -d 154.20.6.95/32 -j RETURN
-A XRAY_SELF -d 67.30.13.175/32 -j RETURN
-A XRAY_SELF -p tcp -m tcp --sport 28056 -j RETURN
-A XRAY_SELF -p tcp -m tcp --sport 4698 -j RETURN
-A XRAY_SELF -d 192.168.0.0/16 -j RETURN
-A XRAY_SELF -d 100.64.0.0/10 -j RETURN
-A XRAY_SELF -d 127.0.0.0/8 -j RETURN
-A XRAY_SELF -d 169.254.0.0/16 -j RETURN
-A XRAY_SELF -d 10.0.0.0/8 -j RETURN
-A XRAY_SELF -d 172.16.0.0/12 -j RETURN
-A XRAY_SELF -d 192.0.0.0/24 -j RETURN
-A XRAY_SELF -d 224.0.0.0/4 -j RETURN
-A XRAY_SELF -d 240.0.0.0/4 -j RETURN
-A XRAY_SELF -d 255.255.255.255/32 -j RETURN
-A XRAY_SELF -m mark --mark 0x2 -j RETURN
-A XRAY_SELF -p tcp -j MARK --set-xmark 0x1/0xffffffff
-A XRAY_SELF -p udp -j MARK --set-xmark 0x1/0xffffffff
```



策略路由

```bash
# 添加路由表 100
# 将数据包路由到本机 lo 网络接口
# 到达 lo 网口的数据包会重新从 PREROUTING 开始匹配一遍 iptables 规则
ip route add local default dev lo table 100 

# 添加路由规则，匹配 fwmark 为 1 的数据包使用 100 路由表
ip rule add fwmark 1 table 100
```



## 调试 iptables 规则

参考文档：

- https://ipset.netfilter.org/iptables-extensions.man.html



使用 iptables LOG 模块打印报文链路

示例：

```bash
iptables -t mangle -A PREROUTING -m limit --limit 5000/minute -j LOG --log-level 7 --log-prefix "prerouting: "
iptables -t mangle -A OUTPUT -m limit --limit 5000/minute -j LOG --log-level 7 --log-prefix "output: "
```

> trace 模块只能在 RAW 表中使用 



使用 dmesg 命令查看 log 记录

```bash
```

