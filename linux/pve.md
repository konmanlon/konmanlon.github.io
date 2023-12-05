## PVE 直通

参阅文档：

- https://pve.proxmox.com/pve-docs/pve-admin-guide.html#qm_pci_passthrough
- https://pve.proxmox.com/pve-docs/pve-admin-guide.html#sysboot_edit_kernel_cmdline
- https://forum.proxmox.com/threads/pci-gpu-passthrough-on-proxmox-ve-8-installation-and-configuration.130218/



```bash
# 编辑 grub 添加直通内核参数
vim /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet iommu=pt initcall_blacklist=sysfb_init"

# 更新 grub
update-grub

# 启用内核模块
# 如 pve 版本 < 8.0，需要添加 vfio_virqfd
vim /etc/modules
vfio
vfio_iommu_type1
vfio_pci

# 更新 initramfs
update-initramfs -u -k all

# 重启
reboot

# 检查模块是否加载
dmesg | grep -i vfio

# 检查是否支持 remapping
dmesg | grep 'remapping'
[    0.484015] AMD-Vi: Interrupt remapping enabled

# 如果不支持则需要启用 允许不安全中断
# echo "options vfio_iommu_type1 allow_unsafe_interrupts=1" > /etc/modprobe.d/iommu_unsafe_interrupts.conf
```



将 PVE 宿主机与需要直通的 PCI 设备隔离

```bash
# 查看 PCI 设备信息
root@pve:~# lspci -D -nn | grep 'USB'
0000:05:00.3 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Renoir/Cezanne USB 3.1 [1022:1639]
0000:05:00.4 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Renoir/Cezanne USB 3.1 [1022:1639]


# 创建配置文件，指定需要隔离的 PCI ID
echo "options vfio-pci ids=1022:1639" > /etc/modprobe.d/vfio.conf

# 更新 initramfs 并重启
update-initramfs -u -k all

# 更新 PCI 设备
update-pciids
```



win 10 优化

```
# 禁用休眠，防止触发 AMD Reset Bug
powercfg.exe /hibernate off

# 禁用驱动程序更新，防止旧版win10下载旧驱动覆盖
```

