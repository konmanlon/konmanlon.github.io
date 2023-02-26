# PCI(e) 直通

> 创建虚拟机时建议选择 q35 机型

启用 IOMMU

```git
# edit /etc/default/grub
- GRUB_CMDLINE_LINUX_DEFAULT="quiet"
+ GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
```

添加 KVM vfio 模块

```git
# edit /etc/modules
+ vfio
+ vfio_iommu_type1
+ vfio_pci
+ vfio_virqfd
```

更新内核参数并重启设备

```bash
update-initramfs -k all -u
reboot
```

检查以上修改是否生效, 如：需要直通的设备位于单独的 IOMMU 组中

```bash
dmesg | grep iommu
find /sys/kernel/iommu_groups/ -type l
```

> 参阅文档：
> - https://pve.proxmox.com/wiki/PCI(e)_Passthrough#qm_pci_passthrough_update_initramfs


## 硬盘直通

查找硬盘控制器，以 SATA 控制器为例

```bash
lspci | grep SATA
00:17.0 SATA controller: Intel Corporation Device 4dd3 (rev 01)
02:00.0 SATA controller: Marvell Technology Group Ltd. 88SE9215 PCIe 2.0 x1 4-port SATA 6 Gb/s Controller (rev 11)
```

查看硬盘所属的控制器

```bash
ls -l /sys/dev/block/ | awk -F '/' '/sd(a|b)/{print $5}'
0000:00:17.0
0000:00:17.0
```

最后虚拟机中添加 PCI 设备，设备 ID 选择 0000:00:17.0 即可将 SATA 硬盘直通到虚拟机中
