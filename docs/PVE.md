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

最后在虚拟机中添加 PCI 设备，设备 ID 选择 0000:00:17.0 即可将 SATA 硬盘直通到虚拟机中。


## 网卡直通

使用 lspci 命令查找 Eth 网卡控制器时，无法分辨出设备 ID 所对应的 PVE 中的网卡名，因此可以使用以下方式进行查看

```bash
grep PCI_SLOT_NAME /sys/class/net/*/device/uevent
/sys/class/net/enp3s0/device/uevent:PCI_SLOT_NAME=0000:03:00.0
/sys/class/net/enp4s0/device/uevent:PCI_SLOT_NAME=0000:04:00.0
```

确认网口对应的 PCI 设备 ID 后，在虚拟机中添加 PCI 设备即可。

> 需要注意的是有些设备一张物理网卡会有多个逻辑网口，这写网口会在同一个 PCI 地址上，如 03:00.0 和 03:00.1，都在 03:00 上；
> 此时需要检查两个逻辑网口是否在相同的 IOMMU 组中，如果不在同一个 IOMMU 组中，则可以分别直通；如果在同一个 IOMMU 组中，那么就需要利用 PCI Bridge 的 ACS Capability。
> PVE 7.1 以上的版本可在 GRUB 中添加 `pcie_acs_override=downstream` 参数，以解决 IOMMU 分组问题。
