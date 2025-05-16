# 这里有一些使用gentoo linux的使用体验和软件推荐  
这里是[gentoo官方给的推荐软件](https://wiki.gentoo.org/wiki/Recommended_applications)  

1. [[#浏览器]]  
2. [[#软件包查找工具]]  
3. [[#关于内核]]  
4. [[#关于游戏]]  
5. [[#字体]]  
6. [[#关于音频管理]]  


## 浏览器  
关于浏览器的选择主要建议以下两个  
*www-client/google-chrome*  
*www-client/firefox*  
```
sudo emerge --ask firefox  

#或者  
sudo emerge --ask google-chrome  
```
> 我个人不推荐用edge（有很多bug）  
> 也不推荐chrome的开源版*www-client/chromium*因为需要编译非常非常久,~~铸币博主上次编译七小时~~  

接下来要做的就是将你想的浏览器设置为默认浏览器（如果你安装了多个浏览器）  
例如我要将Firefox设置为默认浏览器  
检查/usr/share/applications/  
例如FireFox的运行桌面快捷方式叫*firefox-esr.desktop*而不是firefox.desktop  
所以需要检查一遍你想要设置的浏览器的快捷方式  
执行以下命令将 Firefox 设置为默认浏览器：  
```
xdg-settings set default-web-browser firefox-esr.desktop  
```
验证是否成功：  
```
xdg-settings get default-web-browser  
```
输出*firefox-esr.desktop*为成功  

---

## 软件包查找工具  
*app-portage/eix*是gentoo linux使用必备工具之一  
此工具可以查找目前你拥有所有repository的所有软件包  
首先需要安装eix并且同步本地仓库  
```
sudo emerge --ask eix  

eix-update  
````
此工具很容易使用  
例如查找mpv播放器  
```
eix mpv  
```
此外还有一个专门用来查询的网站，甚至可以查看下载某软件包的ebuild  
[Gentoo Portage Overlay](https://gpo.zugaina.org)  

---

## 关于内核

### dist内核  
在gentoo安装中提到我使用的内核是*xanmod-kernel*  
这种内核后缀代表的是可被portage管理的预编译内核，这种内核会在emerge的时候自动编译安装，同时带有通用的内核配置，这种内核在/usr/src/linux下是不能直接修改配置的（无效）  
如想修改配置文件，则需要开启savedconfig的use并在/etc/portage/savedconfig/sys-kernel下将你的内核.config文件改为你的**内核包名*后重新编译即可  
例如我现在在用的xanmod-kernel内核  
```
#定位到内核目录  
cd /usr/src/linux  

#修改内核.config  
make menuconfig  

#修改完成后复制到  
cp .config /etc/portage/savedconfig/syskernel/xanmod-kernel  

#重新编译  
emerge xanmod-kernel  
```


### 源码内核  
源码内核一般是后缀为-source的内核  
这种内核事实上只是源码包  
需要自行配置编译安装  
以xanmod-source内核为例  
```
#列出内核列表并选择  
eselect kernel list  

#假设xanmod-source内核为2号  
eselect kernel set 2  

#进入内核目录  
cd /usr/src/  

#查看目录  
ls  

#进入除*linux*你所安装内核的文件夹  
cd xanmod-kernel  
ls  

#里面通常会有一个**config**文件夹,里面包含的文件通常为通用内核配置  
#假设**config**文件夹下的配置文件为*config1*  
cd config  

#复制通用配置到*linux*目录下  
cp config1 /usr/src/linux/.config  

#编辑配置文件  
make menuconfig  

#编译内核（$nproc为你的cpu线程数）  
make -j$(nproc)  

#编译内核模块  
make modules_install  
make install  

#更新引导  
grub-mkconfig -o /boot/grub/grub.cfg  
```

---

## 关于游戏  
gentoo linux玩游戏是很方便的  


### 关于Steam  
参考https://wiki.gentoo.org/wiki/Steam  

### 关于Lutris  
lutris是一个开源的游戏启动器，可以非常便捷的管理你的游戏，可以调用wine  
安装lutris之前我们需要在*/etc/portage/package.use/lutris*下写入  
```
#Lutris multilib dependencies  
media-libs/vulkan-loader abi_x86_32  
media-libs/vulkan-layers abi_x86_32  
media-libs/freetype abi_x86_32  
media-libs/libpng abi_x86_32  
net-libs/gnutls abi_x86_32  
media-libs/libsdl2 abi_x86_32  
```
然后安装  
```
emerge --ask games-util/lutris  
```
参考：https://wiki.gentoo.org/wiki/Lutris  

### 关于Minecraft  
mc的启动器可选择的非常多，这里我使用的是[prime launcher](https://prismlauncher.org/)  
一个开源的mc启动器，界面美观简洁，更棒的是gentoo官方repository里收留了这个启动器  
但在我们正式安装之前需要安装java  
```
#安装java
emerge --ask dev-java/openjdk-bin  

#安装启动器  
emerge --ask games-action/prismlauncher  
```

---

## 字体  
gentoo基础系统是不会预装有带有太多字体的  
所以需要我们手动安装常用的字体  
这里有我个人推荐安装的字体  

- media-fonts/noto-cjk：Google的中日韩字符，支持简繁体，字体风格多样（包括粗体、斜体变体）  
- media-fonts/corefonts：微软核心字体，包括 Arial、Times New Roman、Comic Sans 等，含斜体变体  
- media-fonts/dejavu：DejaVu 字体，扩展自 Vera 字体，支持多种语言，包含 Sans、Serif 和 Mono 的斜体变体  
- media-fonts/freefont：GNU FreeFont，包含 FreeSerif 和 FreeScript，适合花哨排版  
- media-fonts/noto：Google的其他语言字体包，包括藏文、泰文、蒙古文、彝文等目标是覆盖所有 Unicode 字符（“No Tofu”，避免显示方框字符）  
- media-fonts/unifont：GNU Unifont，覆盖几乎所有 Unicode 字符，适合需要广泛字符支持的场景  
- media-fonts/sil-padauk：支持缅甸文，也适用于部分东南亚小众语言  

图标包  
- media-fonts/nerd-fonts  
- media-fonts/fontawesome  
以上是两个比较通用的图标包，其中*nerd-fonts*需要**guru** overlay，在gentoo安装这一章节中已经添加过  
不过我们在安装nerd-fonts之前需要在*/etc/portage/package.use/nerd-fonts*写入  
```
media-fonts/nerd-fonts hack noto  
```
> 注意千万不要装太多中文字体，noto-cjk和noto已经足够了，否则可能会覆盖图标包,~~铸币博主在编写字体这一节的时候因为大量装中文字体包导致图标被覆盖苦恼很久~~  

我给你说，JetBrainsMono Nerd Font是对的  
这里强烈建议将nerd-fonts作为最高顺序字体使用，非常舒服  
```
#在home目录下创建fontconfig文件夹  
mkdir -p ~/.config/fontconfig  

#创建文件并写入  
nvim ~/.config/fontconfig/fonts.conf  
```
```
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
  <alias>
    <family>monospace</family>
    <prefer>
      <family>Hack Nerd Font</family>
      <family>JetBrainsMono Nerd Font</family>
    </prefer>
  </alias>
  <alias>
    <family>sans-serif</family>
    <prefer>
      <family>Hack Nerd Font</family>
      <family>sans-serif</family>
    </prefer>
  </alias>
  <alias>
    <family>serif</family>
    <prefer>
      <family>Hack Nerd Font</family>
      <family>serif</family>
    </prefer>
  </alias>
</fontconfig>
```
然后更新缓存  
```
fc-cache -fv  
```

---

## 关于音频管理  
现在一般都推荐用PipeWire吧  
在安装PipeWire之前，很重要的一件事就是开启所需的USE  
```
media-video/pipewire pipewire-alsa sound-server ffmpeg  
```
关于USE请参考[PipeWire Gentoo wiki](https://wiki.gentoo.org/wiki/PipeWire)  
```
#安装PipeWire  
emerge --ask media-video/pipewire  

#安装WirePlumber  
emerge --ask media-video/wireplumber  
```
> WirePlumber是PipeWire的关键组件，拥有重要的作用  
记得将用户加入pipewire组  
```
usermod -aG pipewire BAKA  
```
安装RTKit工具  
```
emerge --ask sys-auth/rtkit  
```
> RTKit工具被PipeWire依赖来确保音频的低延迟处理  
然后启用服务(这里是在非root权限下运行)  
```
systemctl --user enable --now pipewire-pulse.socket wireplumber.service  

systemctl --user enable --now pipewire.service  
```
最后有一个工具可能非常有用  
pavucontrol是一个图形化的音频设备和音量控制软件，兼容PipeWire  
```
sudo emerge --ask media-sound/pavucontrol  
```
参考：https://wiki.gentoo.org/wiki/PipeWire  
