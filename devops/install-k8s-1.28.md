# 升级系统内核

有些网络插件依赖新内核的一些特性，例如 Cilium

以 Centos 7 为例，默认是 3.10 版本的内核，将其升级至 6.x



方式一：从第三方存储库安装：

- 参阅：https://elrepo.org/tiki/HomePage
- 说明：https://elrepo.org/tiki/kernel-ml
- rpm 包下载地址：https://elrepo.org/linux/kernel/

```bash
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
yum install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm

# 列出可用的内核包
yum --disablerepo="*" --enablerepo="elrepo-kernel" list available

# 选择需要版本进行安装，lt 为长期支持版，ml 是根据 kernel 源码的 mainline stable 分支构建
yum --enablerepo=elrepo-kernel install kernel-ml
```



方式二：从源码编译安装：

- 官方源码：https://www.kernel.org/
- 参考书籍：http://www.kroah.com/lkn/

编译过程比较繁琐，因此可以将编译好的内核和模块构建成 rpm 包，再分发到其他主机进行安装



将新内核设为默认引导选项：

- 参阅：https://docs.fedoraproject.org/en-US/fedora/latest/system-administrators-guide/kernel-module-driver-configuration/Working_with_the_GRUB_2_Boot_Loader/

方式一：

```bash
# 查看当前默认内核
grubby --default-kernel
/boot/vmlinuz-3.10.0-1160.71.1.el7.x86_64

# 从第三方存储库安装会更新 grub 配置文件：/boot/grub2/grub.cfg，但不会将新内核设为默认的引导选项
# 查看已有内核的信息（内核的索引编号、启动参数、路径等）
grubby --info=ALL

# 将 /etc/default/grub 中 GRUB_DEFAULT 配置项的值设为新内核的索引编号
GRUB_DEFAULT=0

# 更新 grub 配置文件
grub2-mkconfig -o /boot/grub2/grub.cfg

# 检查默认内核是否已更新
grubby --default-kernel
/boot/vmlinuz-6.6.1-1.el7.elrepo.x86_64
```

方式二（推荐）：

```bash
# 查看已有内核的信息（内核的索引编号、启动参数、路径等）
grubby --info=ALL

# 使用 grubby 命令设置默认内核，并会自动更新 /boot/grub2/grub.cfg 配置文件
# 可根据内核启动索引编号或指定内核文件进行设置
# grubby --set-default /boot/vmlinuz-6.6.1-1.el7.elrepo.x86_64
grubby --set-default 0

# 检查默认内核是否已更新
grubby --default-kernel
/boot/vmlinuz-6.6.1-1.el7.elrepo.x86_64
```



清理旧内核：



# 选择容器运行时





部署先决条件参阅：

- https://kubernetes.io/docs/setup/production-environment/container-runtimes/#install-and-configure-prerequisites



模块和内核参数配置

```bash
# 以下配置重启会自动加载
cat <<EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sysctl --system
```



重启后验证模块是否加载

```bash
lsmod | grep br_netfilter
lsmod | grep overlay
```



验证内核参数是否生效

```bash
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```



## 安装 cri-o

安装文档：

- https://github.com/cri-o/cri-o/blob/main/install.md#setup-cni-networking



参考教程：

- https://github.com/cri-o/cri-o/tree/main/tutorials





## 安装 containerd

- 项目地址：https://github.com/containerd/containerd
- 参阅文档：https://github.com/containerd/containerd/blob/main/docs/getting-started.md
- for kubernetes: https://kubernetes.io/docs/setup/production-environment/container-runtimes/



方式一：从 containerd 官方项目仓库安装

```bash
# 下载二进制包并解压到 /usr/local
wget -c https://github.com/containerd/containerd/releases/download/v1.7.8/containerd-1.7.8-linux-amd64.tar.gz
tar Cxzvf /usr/local containerd-1.7.8-linux-amd64.tar.gz

# 下载 unit 文件，使用 systemd 管理 containerd
curl -s -o /etc/systemd/system/containerd.service https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
systemctl daemon-reload
```

安装 runc

- 项目地址：https://github.com/opencontainers/runc

```bash
# 下载到 /usr/local/sbin/ 目录，并重命名为 runc
curl -sLo /usr/local/sbin/runc https://github.com/opencontainers/runc/releases/download/v1.1.10/runc.amd64
```





方式二：从 Docker 存储库安装（该方式安装包由 Docker 分发，而不是 Containerd 项目组）

```bash
# 安装 docket 存储库
yum install -y yum-utils
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# 查看所有版本
yum --showduplicates list containerd.io

# 安装指定版本，containerd.io 中包含了 runc 但不包含 CNI 插件，二进制安装 containerd 需要单独再装 runc 和 CNI 插件
yum install containerd.io-1.6.4-3.1.el7
```



接下来配置 containerd，并启动

```bash
# 生成默认配置，并覆盖原始配置，包管理器安装的原始配置中禁用了 cri
containerd config default > /etc/containerd/config.toml

# 启用 SystemdCgroup
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
      ......
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
        ......
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true
```

启动并设为开机启动

```bash
systemctl enable --now containerd
```

> containerd 默认 CRI socket 路径：/run/containerd/containerd.sock



安装 crictl（可选，方便管理容器）

- 参阅文档：https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md



将 cri 配置为 containerd

```bash
# 创建配置文件
$ cat /etc/crictl.yaml
runtime-endpoint: unix:///var/run/containerd/containerd.sock
image-endpoint: unix:///var/run/containerd/containerd.sock
timeout: 5
debug: false
pull-image-on-create: false
```



---



# 安装 kubeadm 套件



参阅文档：

- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

```bash
cat <<EOF | tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

# 查看指定版本
yum --showduplicates list kubeadm --disableexcludes=kubernetes

# 在每个节点安装kubeadm套件
yum install -y {kubeadm,kubelet,kubectl}-1.24.6-0 --disableexcludes=kubernetes

# 1.22+ kubeadm 默认使用 cgroupDriver: systemd, kubelet 无须配置 cgroup
# kubeadm init 成功后 kubelet 才会启动成功，并生成配置文件
systemctl enable --now kubelet
```



## 初始化控制平面节点

```bash
kubeadm init --kubernetes-version "v1.24.6" \
	--pod-network-cidr "10.255.240.0/20" \
	--service-cidr "192.168.240.0/20"
```

```bash
[root@master ~]# crictl images
IMAGE                                TAG                 IMAGE ID            SIZE
k8s.gcr.io/coredns/coredns           v1.8.6              a4ca41631cc7a       13.6MB
k8s.gcr.io/etcd                      3.5.3-0             aebe758cef4cd       102MB
k8s.gcr.io/kube-apiserver            v1.24.6             860f263331c95       33.8MB
k8s.gcr.io/kube-controller-manager   v1.24.6             c6c20157a4233       31MB
k8s.gcr.io/kube-proxy                v1.24.6             0bb39497ab33b       39.5MB
k8s.gcr.io/kube-scheduler            v1.24.6             c786c777a4e1c       15.5MB
k8s.gcr.io/pause                     3.6                 6270bb605e12e       302kB
k8s.gcr.io/pause                     3.7                 221177c6082a8       311kB
```



---

# 选择网络插件



参阅官方文档：

- https://kubernetes.io/docs/concepts/cluster-administration/networking/
- https://kubernetes.io/docs/concepts/cluster-administration/addons/



## 安装 Cilium 网络插件

项目地址：

- https://github.com/cilium/cilium





## 安装 calico 网络插件



先决条件：

- https://docs.tigera.io/calico/latest/getting-started/kubernetes/requirements



参阅文档：

- https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises



安装：

```bash
# 使用 manifest 安装，不用 Operator
curl https://raw.githubusercontent.com/projectcalico/calico/v3.25.1/manifests/calico.yaml -O

# 修改 Pod CIDR CALICO_IPV4POOL_CIDR，与 kubeadm 初始化指定的 CIDR 一致
kubectl apply -f calico.yaml
```



查看系统组件是否 Running

> Core DNS 需要安装网络插件后才会 Running，否则一直 Pending

```bash
$ kubectl get pod -n kube-system -w
NAME                                      READY   STATUS    RESTARTS   AGE
calico-kube-controllers-bc5cbc89f-blgp4   1/1     Running   0          73s
calico-node-9zvkk                         1/1     Running   0          73s
coredns-6d4b75cb6d-b9cd4                  1/1     Running   0          23m
coredns-6d4b75cb6d-s7v6r                  1/1     Running   0          23m
etcd-master                               1/1     Running   1          23m
kube-apiserver-master                     1/1     Running   0          23m
kube-controller-manager-master            1/1     Running   0          23m
kube-proxy-p8bpk                          1/1     Running   0          23m
kube-scheduler-master                     1/1     Running   0          23m
```



## 加入工作节点

```bash
$ kubeadm join 172.31.0.200:6443 --token 1du49w.8roecrwsqkmzh5l7 \
	--discovery-token-ca-cert-hash sha256:20c7ea2555b6693a18c76a431a832d8c100374aaa3b1766e7dae2593a6d60a3e
```

> 如加控制平面节点需要添加 `--control-plane` 选项



查看节点是否就绪

```bash
[root@master ~]# kubectl get node
NAME     STATUS   ROLES           AGE     VERSION
master   Ready    control-plane   30m     v1.24.6
node01   Ready    <none>          2m27s   v1.24.6
node02   Ready    <none>          90s     v1.24.6

# 为节点添加 role 标签
[root@master ~]# kubectl label nodes node01 kubernetes.io/role=worker01
node/node01 labeled
[root@master ~]# kubectl label nodes node02 kubernetes.io/role=worker02
node/node02 labeled

[root@master ~]# kubectl get node
NAME     STATUS   ROLES           AGE     VERSION
master   Ready    control-plane   36m     v1.24.6
node01   Ready    worker01        8m24s   v1.24.6
node02   Ready    worker02        7m27s   v1.24.6
```



> Calico 支持网络策略，Flannel 不支持，因此使用 Calico 需要注意服务是否有使用网络策略，且 Calico 提供功能更丰富的 [GlobalNetworkPolicy](https://docs.tigera.io/calico/latest/reference/resources/globalnetworkpolicy) 和 [NetworkPolicy](https://docs.tigera.io/calico/latest/reference/resources/networkpolicy)
>
> - https://docs.tigera.io/calico/latest/network-policy/get-started/calico-policy/calico-network-policy
> - kubernetes 集成的 NetworkPolicy 仅用于 Pod





---





# kubectl 自动补全



```bash
# 安装补全工具
yum install -y bash-completion

# 登录 bash 自动将补全脚本导入当前 bash
echo "source <(kubectl completion bash)" >> ~/.bashrc

# 重新加载 bash 生效
exec bash
```





---



# 生态组件





## 部署 Longhorn



参阅文档：

- https://longhorn.io/docs/1.4.1/deploy/install/#installation-requirements



所有节点安装以下组件

```bash
# 安装 open-iscsi
yum --setopt=tsflags=noscripts install iscsi-initiator-utils -y
echo "InitiatorName=$(/sbin/iscsi-iname)" > /etc/iscsi/initiatorname.iscsi
systemctl enable --now iscsid

# 安装 NFSv4 client
yum install nfs-utils
```



部署

- 参阅：https://longhorn.io/docs/1.4.1/deploy/install/install-with-kubectl/

```bash
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.4.1/deploy/longhorn.yaml

kubectl get pod -n longhorn-system -w
```





## 部署 MetalLB



参阅文档：

- https://metallb.universe.tf/installation/