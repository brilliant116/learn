#### 制作启动U盘

Find out the name of your USB drive with `lsblk` and make sure that it is not mounted.

```
cat path/to/archlinux.iso > /dev/sdx
```

```
cp path/to/archlinux.iso /dev/sdx
```

```
dd bs=4M if=path/to/archlinux.iso of=/dev/sdx status=progress oflag=sync
```

```
tee < path/to/archlinux.iso > /dev/sdx
```

由于安装界面的字体较小，设置较大一点的字体

```
setfont /usr/share/kbd/consolefonts/LatGrkCyr-12x22.psfu.gz
```

查看系统的引导方式

```
ls /sys/firmware/efi/efivars
```

如果不存在这个目录，则说明是BIOS引导。

#### 连接网络

在虚拟机中，不用手动连接网络，查看网络连接。

```
ping archlinux.org
```

WIFI

```
iwctl

help
# 列出无线网络设备
device list
# 用所指定的设备扫描网络（用上一步的设备名称替换 device）
station device scan
# 列出所有的网络
station device get-networks
# 连接网络，SSID是网络名称
station device connect SSID
```

```
iwctl --passphrase <password> station device connect SSID
```

调整系统时间

```
timedatectl set-ntp true
```

```
timedatectl status
```

#### 磁盘分区

```
fdisk -l
lsblk
```

使用fdisk或cfdisk进行分区。根据之前确定你的电脑是BIOS还是UEFI引导方式，有所不同

*UEFI with GPT*

| Mount point           | Partition | Partition type       | Suggested Size         |
| --------------------- | --------- | -------------------- | ---------------------- |
| /mnt/boot or /mnt/efi | dev/sdx1  | EFI system partition | 260-512MiB             |
| [SWAP]                | dev/sdx2  | Linux swap           | More than 512 MiB      |
| /mnt                  | dev/sdx3  | Linux x86_64 root(/) | Remainer of the device |

swap主要是系统休眠用的，一般可以不用

```
cfdisk /dev/sda
fisk /dev/sda
```

```
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.ext4 /dev/nvme0n1p2
```

```
mount /dev/nvme0n1p1 /mnt/boot
mkdir /mnt/boot
mount /dev/nvme0n1p2 /mnt
```

分区设置是UEFI+GPT，对硬盘分为四个区域，具体划分如下

| mount point | size           | partition type   | format |
| ----------- | -------------- | ---------------- | ------ |
| EFI         | 512M           | EFI System       | vfat   |
| /boot       | 512M           | LInux filesystem | ext4   |
| /           | remainer space | Linux filsystem  | brtfs  |

例如设备是nvme0n1, 设置4个分区

```
mkfs.vfat /dev/nvme0n1p1
mkfs.ext4 /dev/nvme0n1p2
mkfs.btrfs /dev/nvme0n1p3
```

确认分区结果

```
lsblk -f
```

挂载分区，先挂载根目录/

```
mount /dev/nvme0n1p3 /mnt
```

进入到挂载的/mnt目录中，创建子卷

```
cd /mnt
```

创建root子卷和home子卷

```
btrfs subvolume create @root  # 创建root子卷
btrfs subvolume create @home  # 创建home子卷
```

退出/mnt用进入根目录

```
cd /
```

优化情况下重新挂载/mnt。

```
mount -o noatime,ssd,space_cache,discard=async,compress=lzo,subvol=/@root /dev/nvme0n1p4 /mnt
```

上述参数释义分别含义为

noatime: 不记录文件的访问时间
ssd：启用固态优化
space_cache: 使用空余位置缓存
discard=async: 启动异步 discard 支持
compress: 启用透明压缩支持
subvol: 指定挂载子卷

创建home目录

```
mkdir -p /mnt/home
```

进行参数优化条件下的挂载。

```
mount -o relatime,ssd,space_cache,discard=async,compress=zstd,subvol=/@home /dev/nvme0n1p4 /mnt/home
```

创建/boot目录和/efi目录

```
mkdir -p /mnt/boot
mkdir -p /mnt/efi
```

```
mount -o noatime /dev/nvme0n1p2 /mnt/boot
mount -o noatime /dev/nvme0n1p1 /mnt/efi
```

查看最终挂载情况

```
lsblk -f
```

#### 安装

选择中国archlinux镜像

```
nano /etc/pacman.d/mirrorlist
```

在`/etc/pacman.d/mirrorlist`的最前面添加

```
Server = https://mirrors.bfsu.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
```

在[Arch Linux镜像列表](https://www.archlinux.org/mirrorlist/)找到中国镜像地址

安装基本包

```
pacstrap /mnt base base-devel linux linux-firmware
```

配置Fstab

```
genfstab -U /mnt >> /mnt/etc/fstab
```

检查结果

```
cat /mnt/etc/fstab
```

`/etc/fstab`

```
UUID=...    /boot      fat     defaults,noatime     0 2
UUID=...    /          ext4    defaults,noatime     0 1
```

Tip: After installing Arch, you can also change the `/etc/fstab` file.

```
sudo mount -a
```

修改新磁盘的 UUID，然后在 `/etc/fstab` 中编辑规则，最后更新 grub 即可

```
grub-mkconfig -o /boot/grub/grub.cfg
```

改变root到新系统

```
arch-chroot /mnt
```

调整时区

```
ls /usr/share/zoneinfo
```

```
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

将硬件时间设定为UTC

```
hwclock --systohc
```

安装必需包

```
pacman -S nano networkmanager ntfs-3g
```

```
systemctl enable NetworkManager
```

ntfs-3g识别Windows的NTFS文件

编辑`/etc/locale.gen`，将UTF-8的英语和中文前面的#号去掉。

```
#en_US.UTF-8 UTF-8
...
#zh_CN.UTF-8 UTF-8
```

生成文字编码

```
locale-gen
```

创建并编辑`/etc/locale.conf`文件，

```
LANG=en_US.UTF-8
```

新建`/etc/hostname`文件

```
myhostname
```

新建`/etc/hosts`文件，加入如下

```
127.0.0.1   localhost
::1         localhost
127.0.1.1   myhostname.localdomain myhostname
```

部署grub

```
pacman -S grub efibootmgr intel-ucode os-prober dosfstools
```

```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Arch
```

生成grub配置文件

```
grub-mkconfig -o /boot/grub/grub.cfg
```

```
exit
mount /dev/nvme0n1p1 /mnt/boot
mkdir /mnt/boot
mount /dev/nvme0n1p2 /mnt
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Arch
grub-mkconfig -o /boot/grub/grub.cfg
```

添加用户

```
useradd -m -G wheel username
passwd username
passwd
```

```
EDITOR=nano visudo
```

```
/etc/sudoers.tmp

# %wheel ALL=(ALL) ALL
```

重启电脑

```
reboot
```

#### 安装图形化界面

KDE plasma

```
sudo pacman -S xorg
sudo pacman -S xorg-server
```

```
sudo pacman -S plasma-desktop dolphin konsole

sudo pacman -S plasma-meta konsole chromium firefox
sudo pacman -S alacritty kitty
```

```
sudo systemctl enable sddm
```

XFCE

```
sudo pacman -S xorg
sudo pacman -S xfce4
sudo pacman -S xfce4-goodies
```

```
# lightdm
sudo pacman -S lightdm lightdm-gtk-greeter
sudo systemctl enable lightdm

# lxdm
sudo pacman -S lxdm
sudo systemctl enable lxdm
```

```
reboot
```

如果无法进入桌面

`/usr/lib/modprobe.d/blacklist-nouveau.conf` or `/etc/modprobe.d/blacklist-nouveau.conf`

```
blacklist nouveau

blacklist nvidia
blacklist nvidia-drm
blacklist nvidia-modeset
blacklist nvidia-uvm
```

```
sudo pacman -S openssh
sudo systemctl enable sshd
```

使用Arch Linux CN源

在`/etc/pacman.conf`的末尾添加：

```
[archlinuxcn]
Server = https://mirrors.bfsu.edu.cn/archlinuxcn/$arch
```

安装`archlinuxcn-keyring`

```
sudo pacman -Sy archlinuxcn-keyring
```

使用谷歌拼音输入法

```
sudo pacman -S fcitx-im fcitx-configtool fcitx-googlepinyin fcitx-cloudpinyin
```

输入方法加上谷歌拼音，配置里的云拼音来源改为百度。

新建`~/.xprofile`，添加

```
export GTK_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

#### NVIDIA

查看显卡信息

```
lspci -nnk | grep "VGA\|'Kern'\|3D\|Display" -A2
xrandr --listproviders
sudo cat /sys/kernel/debug/vgaswitcheroo/switch
```

几点建议：

- 如果修改后重启黑屏，无法进入图形界面，可按下左 alt + f2/f3/f4/f5/f6 进入命令行界面修复。
- 如果在图形界面卡住，可按下左 ctrl + alt + f2/f3/f4/f5/f6 进入命令行界面重启。
- 如果不小心从图形界面进入了命令行界面，可按下左 alt + f7 回到图形界面。

see the Nvidia Graphic card

```
lspci -k | grep -A 2 -E "(VGA|3D)"
```

```
sudo pacman -S nvidia nvidia-utils nvidia-settings
```

#### Nvidia-xrun

```
sudo pacman -S bbswitch nvidia-xrun optimus-manager-qt
```

Find your display device bus id:

```
lspci | grep -i nvidia | awk '{print $1}'
```

It might return something similar to 01:00.0. Then create a file (for example /etc/X11/nvidia-xorg.conf.d/30-nvidia.conf) to set the proper bus id:

```
/etc/X11/nvidia-xorg.conf.d/30-nvidia.conf
Section "Device"
    Identifier "nvidia"
    Driver "nvidia"
    BusID "PCI:1:0:0"
EndSection
```

Also this way you can adjust some nvidia settings if you encounter issues:

```
/etc/X11/nvidia-xorg.conf.d/30-nvidia.conf
Section "Screen"
    Identifier "nvidia"
    Device "nvidia"
    #  Option "AllowEmptyInitialConfiguration" "Yes"
    #  Option "UseDisplayDevice" "none"
EndSection
```

Automatically run window manager

For convenience you can create an $XDG_CONFIG_HOME/X11/nvidia-xinitrc file with your favourite window manager.

```
if [ $# -gt 0 ]; then
  $*
else
  openbox-session
  # Alternatively, you can also use xfce4:
  # xfce4-session
fi
```

With it you do not need to specify the app and can simply execute:

```
nvidia-xrun
```

Use bbswitch to manage nvidia

When the nvidia card is not needed, bbswitch can be used to turn it off. The nvidia-xrun script will automatically take care of running a window manager and waking up the nvidia card. To achieve that, you need to:

Load bbswitch module on boot

```
# echo 'bbswitch ' > /etc/modules-load.d/bbswitch.conf
```

Disable the nvidia module on boot:

```
# echo 'options bbswitch load_state=0 unload_state=1' > /etc/modprobe.d/bbswitch.conf
```

After a reboot, the nvidia card will be off. This can be seen by querying bbswitch's status:

```
cat /proc/acpi/bbswitch  
```

To force the card to turn on/off respectively run:

```
# tee /proc/acpi/bbswitch <<<OFF
# tee /proc/acpi/bbswitch <<<ON
```

#### Usage

Once the system boots, from the virtual console, login to your user, and run `nvidia-xrun <application>`.

If above does not work, switch to unused virtual console and try again.

As mentioned before, running apps directly with `nvidia-xrun <application>` does not work well, so it is best to create an `$XDG_CONFIG_HOME/X11/nvidia-xinitrc` as outlined earlier, and use `nvidia-xrun` to launch your window manager.

#### Troubleshooting

NVIDIA GPU fails to switch off or is set to be default
See #Use bbswitch to manage nvidia.

If Nvidia GPU still fails to switch off, or is somehow set to be default whenever you use or not nvidia-xrun, then you might likely need to blacklist specific modules (which were previously blacklisted by Bumblebee). Create this file and restart your system:

```
/usr/lib/modprobe.d/nvidia-xrun.conf

blacklist nvidia
blacklist nvidia-drm
blacklist nvidia-modeset
blacklist nvidia-uvm
blacklist nouveau
```

```
lspci | egrep 'VGA|3D'
```

To avoid the possibility of forgetting to update initramfs after an NVIDIA driver upgrade, you may want to use a pacman hook

```
/etc/pacman.d/hooks/nvidia.hook
-------------------------------
[Trigger]
Operation=Install
Operation=Upgrade
Operation=Remove
Type=Package
Target=nvidia
Target=linux
# Change the linux part above and in the Exec line if a different kernel is used

[Action]
Description=Update Nvidia module in initcpio
Depends=mkinitcpio
When=PostTransaction
NeedsTargets
Exec=/bin/sh -c 'while read -r trg; do case #trg in linux) exit 0; esac; done; /usr/bin/mkinitcpio -P'
```

2

```
wget https://uk.download.nvidia.com/XFree86/Linux-x86_64/455.45.01/NVIDIA-Linux-x86_64-455.45.01.run
chmod +x NVIDIA-Linux-x86_64-455.45.01.run
./NVIDIA-Linux-x86_64-455.45.01.run
```

The NVIDIA package includes an automatic configuration tool to create an Xorg server configuration file (xorg.conf) and can be run by:

```
nvidia-xconfig
```

This command will auto-detect and create (or edit, if already present) the `/etc/X11/xorg.conf` configuration according to present hardware.

If there are instances of DRI, ensure they are commented out:

```
Load        "dri"
```

Double check your `/etc/X11/xorg.conf` to make sure your default depth, horizontal sync, vertical refresh, and resolutions are acceptable.

Xorg 这边要加一段配置。就保存在 /etc/X11/xorg.conf.d/nvidia.conf 好了。

```
Section "ServerLayout"
  Identifier "layout"
  Screen 0 "iGPU"
  Option "AllowNVIDIAGPUScreens"
EndSection

Section "Device"
  Identifier "iGPU"
  Driver "modesetting"
  BusID "PCI:0:2:0"
EndSection

Section "Screen"
  Identifier "iGPU"
  Device "iGPU"
EndSection

Section "Device"
  Identifier "dGPU"
  Driver "nvidia"
EndSection
```

BusID 的值要自己看着 `lspci | grep VGA` 来改。

如果你之前是用N卡跑 Xorg 的，需要把 `xrandr --setprovideroutputsource modesetting NVIDIA-0` 之类的设置去掉。

然后重启系统就可以了。请做好通过 tty 或者 ssh 修复配置的准备。

默认情况下，Xorg 及其上的程序运行于i卡。使用 `__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia` 环境变量来指定使用N卡。为了方便起见，做一个别名好了：

```
~/.bashrc

alias nvrun="__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia"
```

```
pacman -S nvidia
pacman -S cuda
```

# use NVIDIA in xorg

没使用窗框管理器的以下内容添加到 `~/.xinitrc` 开头:

```
~/.xinitrc
xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
```

使用不同的窗口管理器,要在行应的位置加入这两行

```
xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
```

LightDM

```
# nano /etc/lightdm/display_setup.sh

#!/bin/sh
xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
```

```
chmod +x /etc/lightdm/display_setup.sh
```

编辑 /etc/lightdm/lightdm.conf 的 [Seat:*] 部分以配置 lightdm 运行这个脚本:

```
# nano /etc/lightdm/lightdm.conf
[Seat:*]
display-setup-script=/etc/lightdm/display_setup.sh
```

重启，你的 DM 应该启动了。
SDDM

```
# nano /usr/share/sddm/scripts/Xsetup
xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
```

GDM

```
/usr/share/gdm/greeter/autostart/optimus.desktop
/etc/xdg/autostart/optimus.desktop

[Desktop Entry]
Type=Application
Name=Optimus
Exec=sh -c "xrandr --setprovideroutputsource modesetting NVIDIA-0; xrandr --auto"
NoDisplay=true
X-GNOME-Autostart-Phase=DisplayServer
```

确保GDM使用了Xorg模式。

修改配置文件

```
# nano /etc/X11/xorg.conf

Section "Module"
    load "modesetting"
EndSection

Section "Device"
    Identifier     "Device0"
    Driver         "nvidia"
    VendorName     "NVIDIA Corporation"
    BusID          "1:0:0"
    Option         "AllowEmptyInitialConfiguration"
EndSection
```

解决画面撕裂问题

DRM kernel mode setting

nvidia 364.16 adds support for DRM (Direct Rendering Manager) kernel mode setting. To enable this feature, add the `nvidia-drm.modeset=1` kernel parameter. For basic functionality that should suffice, if you want to ensure it's loaded at the earliest possible occasion, or are noticing startup issues you can add `nvidia`, `nvidia_modeset`, `nvidia_uvm` and `nvidia_drm` to the initramfs according to Mkinitcpio#MODULES.

```
# nano /etc/mkinitcpio.conf

MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
```

```
# nano /etc/default/grub

GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nvidia-drm.modeset=1"
```

运行下面mkinit命令

```
# sudo mkinitcpio -p linux
```

```
# grub-mkconfig -o /boot/grub/grub.cfg
```

安装好之后输入`nvidia-smi`查看GPU使用信息，若有显示GPU信息则安装完毕，重启电脑就可以使用了

检验 3D
你可通过安装`mesa-demos`并运行以下命令来检验 NVIDIA 是否被使用:

```
glxinfo | grep NVIDIA
```

禁用nouveau

```
nano /etc/modprobe.d/nouveau_blacklist.conf
blacklist nouveau
# If you don't go into the Desktop environment, uncomment the following:
# blacklist nvidia
# blacklist nvidia_drm
```

#### 安装Arch Linux后的屏幕分辨率太低

解决Arch Linux后的屏幕分辨率太低，同时在VMware Worksation中使用粘贴、复制、拖拽等功能

```
sudo pacman -S open-vm-tools xf86-input-vmmouse xf86-video-vmware gtkmm3 gtkmm gtk2
```

```
sudo systemctl enable vmtoolsd.service
sudo systemctl start vmtoolsd.service
sudo systemctl enable vmware-vmblock-fuse.service
sudo systemctl start vmware-vmblock-fuse.service
```

```
sudo nano /etc/mkinitcpio.conf
------------------------------
MODULES=(vsock vmw_vsock_vmci_transport vmw_balloon vmw_vmci vmwgfx)

```

```
sudo mkinitcpio -p linux
```

#### 声音配置

进入系统后，默认静音

```
sudo pacman -S alsa-utils pulseaudio pulseaudio-alsa pavucontrol
man alsamixer
alsamixer
sudo alsactl store
```

其中，输入 alsamixer 后按下 f1 键查看说明操作：

按下 f3 键选择播放设备，m 键取消静音，按下 5 键设置为 50%；
按下 f4 键选择录音设备，同样设置为 50%，然后按下 空格 键关闭录音；
按下 esc 键退出。

#### 关于触摸板（CyPS/2 Cypress Trackpad）不可用的解决方案

触摸板不可用，在点击触摸板时鼠标可能会乱跳，有时候也会导致图形管理器假死，可以把它直接禁用掉

首先执行

```
sudo modprobe -r psmouse
sudo modprobe psmouse proto=imps
```

如果此时你的触摸板可以正常使用了，可以编辑`/etc/modprobe.d/trackpad.conf`，以便在下次开机后依然可以正常使用

```
options psmouse proto=imps
```

说明：这种方法是让电脑把触控板识别为鼠标，少了蛮多触控板的功能

a workaround to reanimate the Trackpad:

```
sudo modprobe -r psmouse
sudo modprobe psmouse proto=imps
```

After the below commands Trackpad became identified as ImPS/2 Generic Wheel Mouse and started working.
edit `etc/modprobe.d/options` file for saving that config  to load it on boot.

create `etc/modprobe.d/trackpad.conf` file

```
options psmouse proto=imps
```

#### 驱动问题

发现 GNOME 无 Wi-Fi 显示，蓝牙无法开启，缺少驱动。Google 并根据 Wiki 的相关命令查看，发现笔记本是很坑的博通 BCM43142 芯片，无线 + 蓝牙二合一。

无线
首先，根据 Wiki 查看芯片的信息：

```
lspci -vnn -d 14e4:
```

然后，去 brcm80211 and b43 的列表搜索，发现内核驱动仍在开发之中。

只得暂时安装闭源的 broadcom-wl：

```
sudo pacman -S broadcom-wl
```

蓝牙
首先，根据 Wiki 查看芯片的信息：

```
lsusb
```

找到芯片型号 BCM43142A0，Google arch linux BCM43142A0 发现强大的 AUR 中有，于是直接安装（yay 的安装请看下文）：

```
yay -S bcm43142a0-firmware
```

驱动解决后，继续按照 Wiki 的指示：

```
sudo pacman -S bluez bluez-utils pulseaudio-bluetooth
systemctl enable bluetooth
```
