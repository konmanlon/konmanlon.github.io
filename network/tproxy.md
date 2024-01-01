## 透明代理规则

示例:

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
-A XRAY -i ens5 -j RETURN
-A XRAY -d 10.0.0.0/8 -j RETURN
-A XRAY -p tcp -j TPROXY --on-port 22222 --on-ip 0.0.0.0 --tproxy-mark 0x1/0xffffffff
-A XRAY -p udp -j TPROXY --on-port 22222 --on-ip 0.0.0.0 --tproxy-mark 0x1/0xffffffff
-A XRAY_SELF -d 154.17.6.213/32 -j RETURN
-A XRAY_SELF -d 67.230.167.175/32 -j RETURN
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

