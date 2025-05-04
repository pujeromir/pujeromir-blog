# Gentoo linux事实上并不难安装    
1. [[#下载iso文件]]  
2. [[#分区]]  
3. [[#挂载分区]]  
4. [[#安装基本系统]]  
5. [[#进入chroot进一步配置]]  
6. [[#重启进入系统]]  

## 下载iso文件
请使用ventory的u盘装载iso文件  
文件的获取在[Gentoo官网](https://www.gentoo.org/downloads)  
请下载**Minimal Installation CD**  


## 分区  
分区在目前有非常好用的工具  
首先需要查看硬盘信息  
可以使用lsblk来查看  
```
lsblk  
```

确定好需要操作的硬盘，这里假设为nvme0n1  
然后使用傻瓜式工具cfdisk  
```
cfdisk /dev/nvme0n1  
```
在完成分区之后就需要格式化分区了  
假设EFI分区为nvme0n1p1  
根分区为nvme0n1p2  
这里暂时不设置swap分区，因为我的内存够用（建议至少32G）  
那么执行  
```
#格式化EFI分区  
mkfs.vfat -F 32 /dev/nvme0n1p1  

#格式化根分区  
mkfs.btrfs /dev/nvme0n1p2  
```


## 挂载分区  
在完成分区之后的第二步就是挂载分区  
```
#挂载根分区  
mkdir -p /mnt/gentoo/boot/efi  
mount /dev/nvme0n1p2 /mnt/gentoo  

#挂载EFI分区  
mount /dev/nvme0n1p1 /mnt/gentoo/boot/efi  
```


## 安装基本系统  
首先需要同步时间  
```
chronyd  
```
进入挂载后的gentoo目录下  
```
cd /mnt/gentoo  
```
然后需要在[Gentoo官网获取我们需要的stage3文件](https://www.gentoo.org/downloads/mirrors)  
使用links工具获取  
```
links https://www.gentoo.org/downloads/mirrors/  
```
打开后向下寻找，找到Downloads回车，然后向下翻到amd64 aka x86-64, x64, Intel 64  
Stage archives  
这里我建议选择stage 3 desktop profile | systemd  
回车并且save进行下载  
下载完了以后按*q*和回车返回  
运行ls确保文件存在  
```
ls  
```
然后解压到该目录  
```
tar xpvf stage3{TAB} --xattrs-include='*.*' --numeric-owner  
#删除stage3文件  
rm stage3{TAB}  
```
然后我们需要根据自己的硬件去做编译优化  
这里需要根据你的cpu信息优化编译过程，利用多核处理器的优势  
首先确定你的cpu核心数，可以输入**nproc**来确定  
```
nproc  
```
比如我使用的是AMD的5600,拥有6核心12线程，所以我需要*/etc/portage/make.conf*里写入信息  
```
vim /mnt/gentoo/etc/portage/make.conf  
```
里面写入  
```
MAKEOPTS="-j12"  
```
**nproc**命令的作用在于返回系统中**可用的逻辑cpu核心数量**，在现代处理器中，由于超线程技术的存在，每个物理核心可以支持多个逻辑线程，例如我的5600拥有6核心，所以拥有12个可用的线程  
所以运行**nproc**命令时，它会返回*12*，代表逻辑线程数，而不是物理核心数  
另外过高的*-j*值会占用更多内存，通常编译的时候每线程需要*1-2G内存*，个人的cpu十二线程会占用约24G内存，所以如果内存不够的话建议从较低的*-j*值开始尝试，观察表现  

自此基础系统就布置好了  


## 进入chroot进一步配置  
首先需要复制DNS配置到系统  
```
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/  
```
然后我们需要挂载必要的文件系统  
```
mount --types proc /proc /mnt/gentoo/proc  

mount --rbind /sys /mnt/gentoo/sys  

mount --make-rslave /mnt/gentoo/sys  

mount --rbind /dev /mnt/gentoo/dev  

mount --make-rslave /mnt/gentoo/dev  

mount --bind /run /mnt/gentoo/run  

mount --make-slave /mnt/gentoo/run  
```
接下来就是正式进入目标系统  
```
chroot /mnt/gentoo /bin/bash  
source /etc/profile  

#修改提示词（可以随意修改括号里的内容）  
PS1=(BAKA)$PS1  
```
### 复制基础的gentoo库  
```
emerge-webrsync  
```
这里推荐添加两个overlay  
```
#安装git和仓库工具  
emerge --ask app-eselect/eselect-repository dev-vcs/git  

#启用guru仓库  
eselect repository enable guru  

#启用gentoo-zh仓库  
eselect repository enable gentoo-zh  

#更新仓库  
emerge --sync  
```
配置默认编辑器  
Gentoo会默认送你一个nano，可是我不用nano所以我会删除nano并且安装neovim  
```
#删除nano  
emerge -C app-editors/nano  

#安装neovim和vim
emerge --ask neovim vim  
```
然后我们需要选择编辑器使用  
```
#设置neovim为默认编辑器  
eselect editor set neovim  

#查看默认编辑器  
eselect editor list  
```
如果输出有neovim并且有*号则成功  
更新环境信息  
```
source /etc/profile  
PS1=(BAKA)$PS1
```

### 解决许可证问题  
我使用的方案是默认许可证全接受，只需要在*/etc/portage/make.conf*下添加  
```
ACCEPT_LICENSE="*"  
```

### 安装内核  
首先安装固件  
```
emerge --ask sys-kernel/linux-firmware  
```
可供选择的内核种类非常多，我使用的是gentoo-zh下的xanmod-kernel内核  
此内核可被portage管理，为自动编译安装的内核，同时包含了默认的内核配置  
首先配置installkernel  
```
echo 'sys-kernel/installkernel dracut' >/etc/portage/package.use/installkernel  
emerge --ask sys-kernel/installkernel  
```
安装内核  
```
emerge --ask sys-kernel/xanmod-kernel  
```
可能等待时间会长一些  

### 配置fstab  
```
vim /etc/fstab  
```
在文件内写入  
```
/dev/nvme0n1p1    /boot/efi    vfat    defaults            0 2  
/dev/nvme0n1p2    /            btrfs   defaults,noatime    0 1  
```

### 修改设备名字  
```
#修改*BAKA*为你想设置的设备名字  
hostnamectl hostname BAKA  

#然后修改/etc/hosts
vim /etc/hosts
#同样修改*BAKA*
127.0.0.1 localhost BAKA
::1       localhost BAKA
```

### 使用networkmanager  
NetworkManager是很全能很好用的一个网络工具，如果要日用的话建议使用  
```
#首先添加USE  
echo 'net-wireless/wpa_supplicant dbus' >/etc/portage/package.use/networkmanager  

#安装NetworkManager  
emerge --ask net-misc/networkmanager  

#启用NetworkManager  
systemctl enable NetworkManager  
```

### 设置用户  
1. 首先为root用户设置密码  
```
passwd  
```
2. 创建个人用户  
```
#创建用户，修改*username*作为你的用户名  
useradd -m -G users,usb,wheel,audio,video -s /bin/bash username  
#这里会提示Creating mailbox file: No such file or directory，但不需要在意  

#设置用户密码  
passwd username 
```

3. 设置用户权限  
```
#安装sudo以使用visudo  
emerge --ask app-admin/sudo  

#使用visudo  
visudo  

#找到以下行并删除注释（#）  
%wheel ALL=(ALL:ALL) ALL  
```

### 安装grub  
```
emerge --ask sys-boot/grub  
grub-install --removable  
grub-mkconfig -o /boot/grub/grub.cfg  
```

### 配置时区  
```
ln -sf ../usr/share/zoneinfo/Asia/Shanghai /etc/localtime  
```

## 重启进入系统  
首先*exit*退出chroot环境  
```
exit  
```
然后取消挂载  
```
cd  

#取消挂载  
umount -R /mnt/gentoo  
```
最后重启  
```
reboot  
```
