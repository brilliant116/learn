### 声音

```
sudo pacman -S alsa-utils
```

ThinkPad X240默认识别两个声卡，把HDMI通道的声卡设置成为默认，这里如果想使用笔记本上的声卡，把PCH设置成默认，添加启动加载文件:
//TODO look pcm

```
nano /etc/modprobe.d/alsa-base.conf
```

添加如下代码设置:

```
# Set pcm is default sound card
options snd_pcm index=0
options snd_hda_intel index=1
```

重新启动，PCH声卡就变成默认声卡了。

### Proxychains

很多时候应用不支持代理或者某些代理方式，Proxychains是一个将socks5等代理类全局化的工具。安装:

```
sudo pacman -S proxychains-ng
```

修改配置文件/etc/proxychains.conf

```
# 修改最后一行
socks5 127.0.0.1 1080
```

然后就可以在Terminal中使用它让xxx应用走socks5代理:

```
proxychains xxx
```

### LightDM

```
sudo pacman -S lightdm
sudo pacman -S lightdm-webkit-theme-litarvan
```

```
ls /usr/share/xgreeters/
```

`/etc/lightdm/lightdm.conf`

```
[Seat:*]
...
greeter-session=lightdm-webkit-theme-litarvan
...
```

### CUDA, tensorflow, pytorch

archlinux为我们打包好已经优化过的TensorFlow和Pytorch，我们不需要使用Pipy或者conda安装而是直接使用pacman即可完成包管理，并提供包的实时滚动更新。

首先安装Nvidia套件和CUDA和cuDNN

```
sudo pacman -S nvidia nvidia-utils nvidia-settings cuda cudnn
```

安装完毕后安装三件套

```
sudo pacman -S python-pytorch-opt-cuda python-tensorflow-opt-cuda tensorflow-opt-cuda
```

这样我们就可以使用Pytorch和TensorFlow了

### Redshift

```
redshift -P -O <temp>
```

```
mkdir ~/.config/redshift
nano ~/.config/redshift/redshift.conf
```

```
[redshift]

temp-day=4500
temp-night=4500

transition=1

location-provider=manual

adjustment-method=randr

[manual]
lat=48.1
lon=23.49

[randr]
screen=1
```

### ProxyChains-NG

```
sudo nano /etc/proxychains.conf
```

```
[ProxyList]
socks4  127.0.0.1  9050
socks5  127.0.0.1  9050
socks5  127.0.0.1  1080
```

然后，在命令前加上 proxychains 即可，如：

```
proxychains wget google.com && rm index.html
```

### Grub Themes

```
vbeinfo
videoinfo
```

Open /etc/default/grub, and edit to match the correct resolution

```
GRUB_GFXMODE=[height]x[width]x32
GRUB_BACKGROUND="/boot/grub/myimage"
GRUB_THEME=“/boot/grub/themes/vimix/theme.txt”
```

update your grub config

```
grub-mkconfig -o /boot/grub/grub.cfg
```

```
git clone git@gitee.com:purpleman/grub2-themes.git
cd ./grub2-themes
```

install Stylish theme into `/boot/grub/theme`

```
sudo ./install.sh -b -s
```

### arch没有声音

```
sudo pacman -S alsa-firmware alsa-tools pulseaudio-alsa alsa-utils
```

升高音量

```
amixer set Master 5%+
```

降低音量

```
amixer set Master 5%-
```

静音

```
amixer set Master toggle
```

如果还是没有声音：
把下列配置添加到系统级别的 /etc/asound.conf 或用户级别的 ~/.asoundrc 文件。如果文件不存在，可以手动创建。其中的各个ID，请根据实际情况调整：

```
defaults.pcm.card 1
defaults.pcm.device 0
defaults.ctl.card 1
```

### Font

系统字体的配置文件为`/etc/fonts/fonts.conf`。在系统安装字体`/usr/share/fonts`，在用户安装字体`~/.local/share/fonts` (`~/.fonts` is already deprecated.)
使用`fc-cache -fv`刷新缓存。

### Btrfs

```
btrfs subvolume create /path/to/subvolume
btrfs subvolume list -p path
btrfs subvolume delete /path/to/subvolume
```

1.

```
mkfs.fat -F32 /dev/sda1
mkfs.btrfs -L arch /dev/sda2

mount /dev/sda2 /mnt
cd /mnt

btrfs subvolume create _active
btrfs subvolume create _active/rootvol
btrfs subvolume create _active/homevol
btrfs subvolume create _active/tmp
btrfs subvolume create _snapshots

cd ..
umount /mnt

mount -o subvol=_active/rootvol /dev/sda2 /mnt

mkdir /mnt/{home,tmp,boot}
mkdir /mnt/boot/efi
mkdir /mnt/mnt/defvol

mount -o subvol=_active/tmp /dev/sda2 /mnt/tmp
mount /dev/sda1 /mnt/boot/efi
mount -o subvol=_active/homevol /dev/sda2 /mnt/home
mount -o subvol=/ /dev/sda2 /mnt/mnt/defvol
```

2.

```
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.btrfs -m single -L arch /dev/nvme0n1p2

mount -o compress=lzo /dev/nvme0n1p2 /mnt

btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@tmp
btrfs subvolume create /mnt/@snapshots

umount /mnt

mount -o noatime,nodiratime,compress=lzo,subvol=@ /dev/nvme0n1p2 /mnt
mkdir -p /mnt/{btrfs-root,boot/efi,home,tmp,.snapshots}
mount -o noatime,nodiratime,compress=lzo,subvol=@home /dev/nvme0n1p2 /mnt/home
mount -o noatime,nodiratime,compress=lzo,subvol=@tmp /dev/nvme0n1p2 /mnt/tmp
mount -o noatime,nodiratime,compress=lzo,subvol=@snapshots /dev/nvme0n1p2 /mnt/.snapshots
mount -o noatime,nodiratime,compress=lzo,subvol=/ /dev/nvme0n1p2 /mnt/btrfs-root

mount /dev/nvme0n1p1 /mnt/boot/efi
```

3.

```
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.btrfs -L System /dev/nvme0n1p2

挂载

mount /dev/nvme0n1p2 /mnt

创建subvolume

btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home

@是根目录

查看目录

ls -l /mnt

挂载subvolume和EFI分区

# subvolume
umount /mnt
mount -o noatime,nodiratime,subvol=@ /dev/nvme0n1p2 /mnt
mkdir /mnt/home
mount -o noatime,nodiratime,subvol=@home /dev/nvme0n1p2 /mnt/home
# EFI
mkdir /mnt/boot
mkdir /mnt/boot/efi
mount /dev/nvme0n1p1 /mnt/boot/efi
```

4.

```
cfdisk /dev/sda

mkfs.fat -F32 /dev/sda1
mkfs.btrfs -L system /dev/sda2

btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home

umount /mnt

mount -o defaluts,noatime,ssd,nodiratime,subvol=@ /dev/sda2 /mnt
mkdir /mnt/home
mount -o defaluts,noatime,ssd,nodiratime,subvol=@home /dev/sda2 /mnt/home

mkdir /mnt/boot
mount /dev/sda1 /mnt/boot/
```

5.

```
格式化分区sda4

mkfs.btrfs -f /dev/sda4
挂载分区

mount /dev/sda4 /mnt
进入mnt目录，创建子卷

cd /mnt
btrfs subvol create rootfs
btrfs subvol create homefs

回到根目录，并挂载子卷及EFI分区

cd ..
umount /dev/sda4
mount /dev/sda4 /mnt -o subvol=rootfs,compress-force=lzo,noatime,autodefrag,space_cache
mkdir /mnt/home
mount /dev/sda4 /mnt/home -o subvol=homefs,compress-force=lzo,noatime,autodefrag,space_cache
mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
```

固态硬盘,subvol项目后命令，需更改为:compress=lzo,noatime,discard,sdd,space_cache

```
nano /etc/mkinitcpio.conf

# 添加btrfs到MODULES=(...)行，找到HOOKS=(...)行，更换fsck为btrfs

MODULES=(btrfs)
...
HOOKS=(base udev autodetect modconf block filesystems keyboard btrfs)
```

```
mkinitcpio -p linux
```

### building kernel from source

install `base-devel` package group and `xmlto`, `kmod`, `inetutils`, `bc`, `libelf`, `git` packages.

```
mkdir ~/kernelbuild
cd ~/kernelbuild
wget <https://mirrors.bfsu.edu.cn/kernel/v5.x/linux-5.9.1.tar.xz>
```

```
tar xJvf linux-5.9.1.tar.xz
cd linux-5.9.1
make mrproper
```

```
make localmodconfig
```

```
make menuconfig
```

```
make -j6
make modules_install
```

The kernel compilation process will generate a compressed bzImage (big zImage) of that kernel, which must be copied to the /boot directory and renamed in the process. Provided the name is prefixed with `vmlinuz-`, such as `vmlinuz-linux59` you may name the kernel as you wish.

```
cp -v arch/x86_64/boot/bzImage /boot/vmlinuz-linux59
```

```
cp /etc/mkinitcpio.d/linux.preset /etc/mkinitcpio.d/linux59.preset
```

```
/etc/mkinitcpio.d/linux59.preset
--------------------------------

...
ALL_kver="/boot/vmlinuz-linux59"
...
default_image="/boot/initramfs-linux59.img"
...
fallback_image="/boot/initramfs-linux59-fallback.img"
```

```
mkinitcpio -p linux59
```

#### SSD

检测 SSD 是否支持 TRIM

可以通过 /sys/block 下的信息来判断 SSD 支持 TRIM, discard_granularity 非 0 表示支持。

```
cat /sys/block/sda/queue/discard_granularity
cat /sys/block/nvme0n1/queue/discard_granularity
```

也可以直接使用 lsblk 来检测，DISC-GRAN (discard granularity) 和 DISC-MAX (discard max bytes) 列非 0 表示该 SSD 支持 TRIM 功能。

```
lsblk --discard
```

verify TRIM support

```
lsblk --discard
```

check the values of DISC-GRAN (discard granularity) and DISC-MAX (discard max bytes) columns. Non-zero values indicate TRIM support.

```
sudo pacman -S util-linux
sudo systemctl enable fstrim.service
sudo systemctl enable fstrim.timer
```

Enabling the `fstrim.timer` service will activate the service weekly.

```
sudo fstrim -v /
```

```
sudo smartctl -data -A /dev/sda
```

```
/etc/fstab
----------
UUID=...    /    ext4    defaults,noatime    0 1
UUID=...    /home    ext4    defaults,noatime    0 2

tmpfs   /tmp      tmpfs     defaults,noatime,mode=1777      0 0
tmpfs   /var/log  tmpfs     defaults,noatime,mode=1777      0 0
tmpfs   /var/tmp  tmpfs     defaults,noatime,mode=1777      0 0
```

迁移高读写文件到 tmpfs

```
sudo vim /etc/fstab
```

复制
行尾添加：

```
tmpfs /tmp      tmpfs defaults,noatime,mode=1777 0 0
tmpfs /var/log  tmpfs defaults,noatime,mode=1777 0 0
tmpfs /var/tmp  tmpfs defaults,noatime,mode=1777 0 0
```

更换 I/O scheduler

```
sudo vim /etc/default/grub
```

找到下面这行，添加 elevator=noop ：

```
GRUB_CMDLINE_LINUX_DEFAULT="elevator=noop ..."
```

生成 GRUB 配置文件：

```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

NVME

```
nvme list
nvme id-ctrl -H /dev/nvme0
```

```
nvme id-ns /dev/nvme0n1
```

```
~/.zshrc

export EDITOR=nano
```

```
printenv
printenv | grep EDITOR
echo $EDITOR
```

```
sudo pacman -S ranger
```

#### pacman

```
sudo pacman -Syu
sudo pacman -Syyu
```

```
sudo pacman -Rs package_name
```

display the detailed information about the specific package

```
pacman -Si package_name
pacman -Qi package_name
```

list of installed packages

```
pacman -Q
```

list later installed or explicit packages

```
pacman -Qe
```

remove unused packages

```
sudo pacman -Rns $(pacman -Qtdq)
```

list dependecies that are no loger needed and remove them

```
pacman -Qdtq
pacman -Qtdq | pacman -Rs -
```

#### AUR helper

yay, pikaur
