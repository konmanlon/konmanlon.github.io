

# Linux 高级策略路由



参阅文章：

- https://www.cnblogs.com/lsgxeva/p/17758875.html
- https://zhuanlan.zhihu.com/p/529188295
- https://datahacker.blog/industry/technology-menu/networking/routes-and-rules/iproute-and-routing-tables



TODO:

- 案例1：tproxy 透明代理（xray）
- 多网卡路由（记录 nas 跟服务器网卡对拷和路由配置）



```bash
# 为 ID 为 100 的路由表添加默认路由
ip route add default via 192.168.0.222 dev enp6s18 table 100

# 查看 ID 为 100 的路由表中配置路由
ip route show table 100

# 路由规则服务于路由表
# 源地址为 172.31.0.0/24 网段的报文，使用 ID 为 100 的路由表,(pref 指定优先级，从低到高匹配)
ip rule add from 172.31.0.0/24 table 100 pref 100

# 目标地址为 172.31.0.0/24 网段的报文，使用 ID 为 100 的路由表
# 两种写法等价
ip rule add to 172.31.0.0/24 lookup 100
ip rule add to 172.31.0.0/24 table 100

# 删除路由规则
ip rule del from 172.31.0.0/24 table 100

# 刷新路由缓存
ip route flush cache
```



```bash
# 当作路由时配置自动 SNAT
iptables -t nat -A POSTROUTING -s 172.31.0.0/24 -j MASQUERADE
```

> 路由需要启用内核转发



待解决的问题：

- pve 网卡与 pve 虚拟机网卡网关延迟问题
  - pve 网卡网关指向路由： 从测试设备 ping pve 网卡 IP 延迟 30+ ms
  - 当虚拟机网卡网关指向 pve 网卡 IP 时，延迟 30+ ms
  - 当虚拟机网卡网关指向路由时，延迟 <10 ms