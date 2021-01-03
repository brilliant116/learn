
copy the ISO image to USB flash drive

```
dd if=/path/to/image.iso of=/dev/sdX bs=8192k status=progress && sync
dd bs=4M if=/path/to/image.iso of=/dev/sdx status=progress && sync
```

确定ip地址

```
ifconfig

ip addr
ip a
```

set up and connect network

#### connect via WiFi (WPA/WPA2)

2.无线网络

```
iwconfig

wlp2s0    IEEE 802.11abgn ESSID:off/any
          Mode:Managed  Access Point: Not-Associated Tx-Power=0 dBm
          Retry  long limit:7   RTS thr:off   Fragment thr:off
          Encryption key:off
          Power Management: on

lo        no wireless extensions.
```

```
wpa_passphrase "ESSID" > /etc/wpa.conf
```

```
chmod -v 600 /etc/wpa.conf
cat /etc/wpa.conf
```

```
wpa_supplicant -Dnl80211,wext -iwlp2s0 -c/etc/wpa.conf -B
```

```
ifconfig wlp2s0
```

- iwctl

```
iwctl

help
```

查看网卡名字

```
device list
```

```
station wlan0 scan
station wlan0 connect SSID
```

```
iwctl --passphrase <passphrase> station device connect SSID
```

- wpa_supplicant

```
iwlist wlan0 scan
```

```
ifconfig wlan0 up
```

```
/etc/wpa_supplicant.conf
------------------------
network{
    ssid="xxxx"
    psk="22222222"
}
```

```
wpa_supplicant -B -d -i wlan0 -c /etc/wpa_supplicant.conf
```

```
dhclient wlan0
```

```
ping gentoo.org
```

**Optional:**开启ssh服务

```
rc-service sshd start

/et/init.d/sshd start
```

```
ssh user@ip_address
```

#### 磁盘分区

**BIOS**

```
cfdisk /dev/sda

mkfs.ext4 /dev/sda1
```

**UEFI**

```
cfdisk /dev/sda

mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
```

BIOS: 挂载root分区

```
mount /dev/sda1 /mnt/gentoo
```

UEFI: 挂载boot, root分区

```
mount /dev/sda1 /boot
mount /dev/sda2 /mnt/gentoo
```

```
cd /mnt/gentoo
```

进入`releases/amd64/autobuilds/current-stage3-amd64/`

```
links https://mirrors.bfsu.edu.cn/gentoo/
```

解压stage3 tarball

```
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
```

x for extract, p for preserve permission, v for verbose and f to denote that we want to extract a file(not standard input)

```
gcc -c -Q -march=native --help=target | grep march
```

```
emerge -a app-portage/cpuid2cpuflags
```

```
cpuid2cpuflags
cpuid2cpuflags >> /etc/portage/make.conf
```

```
lspci | grep -i VGA
lspci | grep -i audio
```

```
emerge -a sys-kernel/linux-firmware
```

```
nano -w /mnt/gentoo/etc/portage/make.conf
```

`/etc/portage/make.conf`

```
COMMON_FLAGS="-march=native -O2 -pipe"

# C, C++
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"

# Fortran
FCFLAGS="${COMMON_FLAGS}"
FFLAGS="${COMMON_FLAGS}"

PORTDIR="/var/db/repos/gentoo"
DISTDIR="/var/cache/distfiles"
PKGDIR="/var/cache/binpkgs"

LC_MESSAGES=C

CHOST="x86_64-pc-linux-gnu"
GRUB_PLATFORMS="efi-64" # UEFI

ACCEPT_KEYWORDS="~amd64"
ACCEPT_LICENSE="*"

MAKEOPTS="-j3"
PORTAGE_TMPDIR="/var/tmp"
EMERGE_DEFAULT_OPTS=""
GENTOO_MIRRORS="https://mirrors.bfsu.edu.cn/gentoo"

MICROCODE_SIGNATURES="-S"

VIDEO_CARDS="intel i965 iris nvidia"
VIDEO_CARDS="vmware" # vmware workstation
VIDEO_CARDS="virtualbox" # virtualbox

INPUT_DEVICES="libinput synaptics keyboard mouse vmmouse"
INPUT_DEVICES="libinput synaptics"
```

```
mkdir --parents /mnt/gentoo/etc/portage/repos.conf
mkdir -p /mnt/gentoo/etc/portage/repos.conf
```

```
cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
```

复制DNS网络信息

```
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```

挂载必要的文件系统

```
mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
```

--make-rslave使在之后的安装中支持systemd

```
mount --make-rslave /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/dev
```

进入新的环境，chroot or change root

```
chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot) ${PS1}"
```

从web安装Gentoo ebuild repository快照

修改 /etc/portage/repos.conf/gentoo.conf ,将

```
#sync-uri = rsync://rsync.gentoo.org/gentoo-portage
sync-uri = rsync://mirrors.bfsu.edu.cn/gentoo-portage
```

```
emerge-webrsync
```

```
eselect profile list
eselect profile set 5
eselect profile list
```

更新@world set

```
emerge --ask --verbose --update --deep --newuse @world
emerge -avuDN @world
emerge -avuDN --with-bdeps=y @world
```

更新系统后，更新配置文件

```
dispatch-conf

u 用新的配置文件取代现有配置文件，跳到下一个配置文件
z 删除新的配置文件，跳到下一个配置文件
n 直接跳到下一个配置文件
```

```
etc-update

-1 退出
-3
```

```
emerge --sync
emerge -avuDN @world
etc-update
emerge --depclean -pv
emerge --depclean -av
```

set up timezone and locale

```
ls /usr/share/zoneinfo
echo "Asia/Shanghai" > /etc/timezone
```

reconfigure `sys-libs/timezone-data` package and it will update `/etc/localtime` file based on `/etc/timezone` file.

```
emerge --config sys-libs/timezone-data
emerge -v --config sys-libs/timezone-data
```

```
nano -w /etc/locale.gen

en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
```

```
locale-gen
```

At this stage, you should use the 'C' (default) locale (other setups can cause issues with the indirect ssh/screen remote connection)

```
eselect locale list
eselect locale set 4
```

Manually

```
nano -w /etc/env.d/02locale
---------------------------
LANG="en_US.utf-8"
LC_COLLATE="C"
```

```
env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
```

#### Bootstrap the Base System

```
cd /var/db/repos/gentoo/scripts
./bootstrap.sh --pretend
```

```
nano -w bootstrap.sh
--------------------
# This stuff should never fail but will if not enough is installed.
[[ -z ${myBASELAYOUT} ]] && myBASELAYOUT=">=$(portageq best_version / sys-apps/baselayout)"
[[ -z ${myPORTAGE}    ]] && myPORTAGE="portage"
[[ -z ${myBINUTILS}   ]] && myBINUTILS="binutils"
[[ -z ${myGCC}        ]] && myGCC="gcc"
[[ -z ${myGETTEXT}    ]] && myGETTEXT="gettext"
[[ -z ${myLIBC}       ]] && myLIBC="$(portageq expand_virtual / virtual/libc)"
[[ -z ${myTEXINFO}    ]] && myTEXINFO="sys-apps/texinfo"
[[ -z ${myZLIB}       ]] && myZLIB="zlib"
[[ -z ${myNCURSES}    ]] && myNCURSES="ncurses"
```

You only need modify the one line, replacing "&&" with ";" as shown above. for modern versions of gcc, we need to allow the `openmp` USE flag.

```
[[ -z ${myLIBC}       ]] && myLIBC="$(portageq expand_virtual / virtual/libc)"
改为
[[ -z ${myLIBC}       ]] ; myLIBC="$(portageq expand_virtual / virtual/libc)"


export USE="-* bootstrap ${ALLOWED_USE} ${BOOTSTRAP_USE}"
改为
export USE="-* bootstrap ${ALLOWED_USE} ${BOOTSTRAP_USE} openmp"
```

```
./bootstrap.sh --pretend
```

```
qfile /etc/locale.gen /etc/conf.d/keymaps
cp -v /etc/locale.gen{,.bak}
```

```
exit
```

```
./bootstrap.sh
```

```
gcc-config --list-profiles
```

If (and only if) the output tells you your current config is invalid, issue the following to fix it:

```
gcc-config 1
env-update && source /etc/profile && export PS1="(chroot) $PS1"
emerge -av sys-devel/libtool
```

```
mv -v /etc/locale.gen{.bak,}
locale-gen
eselect locale show
```

#### 安装Linux内核

1. 手动配置和编译内核

```
emerge -a sys-kernel/gentoo-sources
emerge -a sys-kernel/linux-firmware
```

```
ls -l /usr/src/linux
```

```
emerge -a sys-apps/pciutils sys-apps/usbutils
```

```
lspci
lsusb
```

```
cd /usr/src/linux
```

禁用Nvidia的nouveau驱动

```
nano -w /etc/modprobe.d/blacklist.conf
--------------------------------------
blacklist nouveau
blacklist lbm-nouveau
options nouveau modeset=0
```

**intel microcode**

```
emerge --ask --noreplace sys-firmware/intel-microcode sys-apps/iucode_tool
emerge -an sys-firmware/intel-microcode sys-apps/iucode_tool
```

1.

```
iucode_tool -S

iucode_tool: system has processor(s) with signature 0x000306c3
```

```
iucode_tool -S -l /lib/firmware/intel-ucode/*

iucode_tool: system has processor(s) with signature 0x000306c3
[...]
microcode bundle 49: /lib/firmware/intel-ucode/06-3c-03
[...]
selected microcodes:
  049/001: sig 0x000306c3, pf_mask 0x32, 2017-01-27, rev 0x0022, size 22528
```

识别处理器签名并查找相应microcode文件名，其为“06-9e-0d”这样的编号格式，选择最末尾的那个（最新的）

以上命令能够自动识别处理器型号并找出符合本机CPU对应的微码文件（microcode），其为`06-9e-0d`这样的编号格式，选择最末尾的那个（最新的）。建议把找到的结果用手机拍照下来，然后设置编译内核（make menuconfig）的时候，

```
Device Driver-->Generic Driver Options-->Firmware loader-->Build name firmware blobs into the kernel binary
```

输入`intel-ucode/06-9e-0d`这样的编号格式，将微码文件直接编译进内核。

```
KERNEL Enabling Microcode Loading Support
-----------------------------------------
Processor type and features  --->
    <*> CPU microcode loading support
    [*]   Intel microcode loading support

Device Drivers  --->
  Generic Driver Options  --->
    Firmware Loader  --->
      -*-   Firmware loading facility
      (intel-ucode/06-3c-03) Build named firmware blobs into the kernel binary
      (/lib/firmware) Firmware blobs root directory (NEW)
```

2.

```
iucode_tool -S --write-earlyfw=/boot/early_ucode.cpio /lib/firmware/intel-ucode/*
```

```
nano -w /etc/default/grub
-------------------------
GRUB_EARLY_INITRD_LINUX_CUSTOM="ucode.cpio"
```

```
CONFIG_MICROCODE=y
CONFIG_MICROCODE_INTEL=y
```

确认

```
dmesg | grep microcode

[    0.000000] microcode: microcode updated early to revision 0x22, date = 2017-01-27
[    1.153262] microcode: sig=0x306c3, pf=0x2, revision=0x22
[    1.153815] microcode: Microcode Update Driver: v2.2.
```

```
make meuconfig
```

```
emerge -a sys-kernel/genkernel

cp /usr/share/genkernel/arch/x86_64/generated-config /usr/src/linux/
```

```
make -j3 && make module_install
```

when finishing kernel compilation, copy the kernel image to `/boot`. It will handled by `make install`.

```
make install
```

```
genkernel --install --kernel-config=/path/to/used/kernel.config initramfs
```

```
ls /boot/initramfs*
```

**rebuild kernel**

```
make -j6 && make module_install

mount /boot
make install

grub-mkconfig -o /boot/grub/grub.cfg
```

reboot for the new kernel configuration to take effect

```
reboot
```

2. 使用distribution kernel

```
emerge -av sys-kernel/gentoo-kernel-bin
emerge -av sys-kernel/linux-firmware
```

或者

```
emerge -av sys-kernel/gentoo-kernel-bin
emerge -av sys-kernel/linux-firmware
emerge --config sys-kernel/gentoo-kernel-bin
```

```
>>> Recording sys-kernel/gentoo-kernel-bin in "world" favorites file...

* Messages for package sys-kernel/gentoo-kernel-bin-5.4.60:

* Starting with 5.4.52, Distribution Kernels are switching from Arch
* Linux configs to Fedora.  Please keep a backup kernel just in case.
* sys-kernel/linux-firmware not found installed on your system.
* This package provides various firmware files that may be needed
* for your hardware to work.  If in doubt, it is recommended
* to pause or abort the build process and install it before
* resuming.
*
* If you decide to install linux-firmware later, you can rebuild
* the initramfs via issuing a command equivalent to:
*
* emerge --config sys-kernel/gentoo-kernel-bin

* Messages for package app-text/asciidoc-9.0.2:

* If you are going to use a2x, please also look at a2x(1) under
* REQUISITES for a list of runtime dependencies.
*
* (Note: Above message is only printed the first time package is
* installed. Please look at /usr/share/doc/asciidoc-9.0.2/README.gentoo*
* for future reference)
* media-sound/lilypond and media-gfx/imagemagick for "music" filter support
* dev-util/source-highlight for "source" filter support
* dev-python/pygments for "source" filter support
* app-text/highlight for "source" filter support
* dev-texlive/texlive-latex and app-text/dvipng for "latex" filter support
* dev-texlive/texlive-latex and app-text/dvisvgm for "latex" filter support
* media-gfx/graphviz for "graphviz" filter support

* Messages for package sys-kernel/dracut-049-r3:

* Unable to find kernel sources at /usr/src/linux
* Unable to calculate Linux Kernel version for build, attempting to use running version
*
* If the following test report contains a missing kernel
* configuration option, you should reconfigure and rebuild your
* kernel before booting image generated with this Dracut version.
*
* To get additional features, a number of optional runtime
* dependencies may be installed:
*
* net-misc/networkmanager for Networking support
* app-benchmarks/bootchart2 for Measure performance of the boot process for later visualisation
* app-admin/killproc for Measure performance of the boot process for later visualisation
* sys-process/acct for Measure performance of the boot process for later visualisation
* sys-fs/btrfs-progs for Scan for Btrfs on block devices
* net-fs/cifs-utils for Support CIFS
* sys-fs/cryptsetup[-static-libs] for Decrypt devices encrypted with cryptsetup/LUKS
* app-shells/dash for Allows use of dash instead of default bash (on your own risk)
* sys-block/open-iscsi for Support iSCSI
* sys-fs/lvm2 for Support Logical Volume Manager
* sys-fs/mdadm for Support MD devices, also known as software RAID devices
* sys-fs/multipath-tools for Support Device Mapper multipathing
* >=sys-boot/plymouth-0.8.5-r5 for Plymouth boot splash
* sys-block/nbd for Support network block devices
* net-fs/nfs-utils for Support NFS
* net-nds/rpcbind for Support NFS
* app-admin/rsyslog for Enable logging with rsyslog
* sys-apps/rng-tools for Enable rngd service to help generating entropy early during boot

>>> Auto-cleaning packages...
>>> No outdated packages were found on your system.

* Regenerating GNU info directory index...
* Processed 85 info files.

* IMPORTANT: 7 news items need reading for repository 'gentoo'.
* Use eselect news read to view new items.
```

```
emerge -av sys-kernel/installkernel-gentoo

emerge -av sys-kernel/gentoo-kernel-bin

emerge -av --depclean
```

由于技术原因，distribution kernel不会自动rebuild被其他包安装的内核模块

```
emerge --ask @module-rebuild
```

```
emerge --config sys-kernel/gentoo-kernel-bin
```

#### 创建fstab文件

```
nano /etc/hosts

199.232.28.133 raw.githubusercontent.com
```

```
wget https://raw.githubusercontent.com/YangMame/Gentoo-Installer/master/genfstab
chmod +x genfstab
./genfstab / > /etc/fstab

# 检查下此文件，删除无用的挂载点
nano /etc/fstab
```

`/etc/fstab`

```
blkid
```

推荐使用UUID

```
# Static information about the filesystems.
# See fstab(5) for details.

# <file system> <dir> <type> <options> <dump> <pass>
# /dev/sda1
UUID=68a85989-18ff-463f-a2f7-596a40140e97    /      ext4    rw,relatime     0 1
```

```
/dev/sda1    /            ext4    rw,relatime          0 1
```

```
/dev/sda1    /boot        fat     defaults,noatime     0 2
/dev/sda2    none         swap    sw                   0 0
/dev/sda3    /            ext4    noatime              0 1
```

```
/dev/sda1    /boot        fat     defaults,noatime,discard     0 2
/dev/sda2    none         swap    defaults,noatime,discard     0 0
/dev/sda3    /            ext4    defaluts,noatime,errors=remount-ro,discard     0 1
```

```
emerge -av net-misc/networkmanager
rc-update add NetworkManager default
```

```
nano -w /etc/conf.d/hostname

hostname="smith"
```

```
emerge app-admin/sysklogd sys-process/cronie -a

rc-update add sysklogd default
rc-update add cronie default
```

#### 选择boot loader

**BIOS**

```
emerge -av sys-boot/grub:2
```

```
grub-install /dev/sda
```

```
grub-mkconfig -o /boot/grub/grub.cfg
```

**UEFI**

```
echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf

emerge -avuDN @world
```

```
emerge -av sys-boot/grub:2
```

```
grub-install --target=x86_64-efi --efi-directory=/boot
```

```
grub-mkconfig -o /boot/grub/grub.cfg
```

设置root密码

```
passwd
```

添加用户

```
useradd -m -G wheel smith
usermod -aG additional_groups smith
passwd smith
```

```
emerge -a --noreplace app-admin/sudo
```

```
EDITOR=nano visudo
```

#### 忘记root密码

进入gentoo liveCD

```
mount /dev/<root_partition> /mnt/gentoo
mount /dev/sda1 /mnt/gentoo
chroot /mnt/gentoo /bin/bash
source /etc/profile
passwd
```

#### 桌面环境

1.KDE

```
eselect profile list
eselect profile set 23
emerge -avuDN @world
```

```
emerge -av x11-base/xorg-server x11-drivers/xf86-video-intel
```

```
* Messages for package x11-apps/xinit-1.4.1:

* If you use startx to start X instead of a login manager like gdm/kdm,
* you can set the XSESSION variable to anything in /etc/X11/Sessions/ or
* any executable. When you run startx, it will run this as the login session.
* You can set this in a file in /etc/env.d/ for the entire system,
* or set it per-user in ~/.bash_profile (or similar for other shells).
* Here's an example of setting it for the whole system:
* echo XSESSION="Gnome" > /etc/env.d/90xsession
* env-update && source /etc/profile
```

```
emerge -av kde-plasma/plasma-desktop plasma-nm plasma-pa sddm konsole
# or
emerge -av kde-plasma/plasma-desktop sddm konsole kde-plasma/powerdevil kde-plasma/systemsettings
```

```
* Starting with 0.18.0, SDDM no longer installs /etc/sddm.conf
* Use it to override specific options. SDDM defaults are now
* found in: /usr/share/sddm/sddm.conf.d/00default.conf
* Adding group 'sddm' to your system ...
  * - Groupid: next available
* Adding user 'sddm' to your system ...
  * - Userid: 999
  * - Shell: /sbin/nologin
  * - Home: /var/lib/sddm
  * - Groups: sddm,video
  * - GECOS: added by portage for sddm
  * - Creating /var/lib/sddm in
```

使用ssdm或lxdm

```
nano /etc/conf.d/xdm

DISPLAYMANAGER="sddm"
```

2.XFCE

```
eselect profile list
eselect profile set 20
emerge -avuDN @world

emerge x11-base/xorg-server x11-drivers/xf86-video-intel -a
```

```
emerge xfce-base/xfce4-meta -a
```

```
emerge lxde-base/lxdm -a
```

使用lxdm

```
nano /etc/conf.d/xdm

DISPLAYMANAGER="lxdm"
```

```
rc-update add xdm default
```

```
nano ~/.xinitrc

exec startxfce4
```

3.GNOME

```
emerge -a x11-base/xorg-server
```

```
lspci | grep -i VGA
hwinfo --gfxcard
```

```
emerge gnome-base/gnome -a
```

更新环境变量

```
env-update && source /etc/profile
```

使用gdm

```
nano /etc/conf.d/xdm

DISPLAYMANAGER="gdm"
```

```
rc-update add xdm default
```

```
nano ~/.xintric

exec gnome-session
```

#### 安装vmware tools

```
sudo emerge app-emulation/open-vm-tools -a
```

```
rc-update add vmware-tools default
```

```
nano /etc/portage/make.conf

VIDEO_CARDS="fbdev vesa vmware" # vmware workstation
VIDEO_CARDS="fbdev vesa virtualbox" # virtualbox
INPUT_DEVICES="libinput synaptics keyboard mouse vmmouse"
```

```
sudo emerge xf86-input-vmmouse xf86-video-vmware gtkmm -a
```

```
sudo shutdown -h now
```

#### 设置CPU Flags

```
cat /proc/cpuinfo
```

```
sudo emerge cpuid2cpuflags -a
```

```
cpuid2cpuflags
```

```
echo "*/* $(cpuid2cpuflags)" >> /etc/portage/package.use/00cpu-flags
```

```
cpuid2cpuflags

nano /etc/portage/make.conf
```

CHOST参数

```
portageq envvar ABI
amd64
```

```
portageq envvar CHOST_amd64
x86_64-pc-linux-gnu
```

```
/etc/portage/make.conf

CHOST="x86_64-pc-linux-gnu"
```

```
gcc-config -l
gcc-config -c
```

```
binutils-config -l
binutils-config -c
```

列出所有已安装的包

```
ls /var/db/pkg/*
```

```
gcc -march=native -E -v - </dev/null 2>&1 | sed -n 's/.* -v - //p'

gcc -Q --help=target -march=skylake | grep enabled

gcc -Q --help=target -march=native | grep enabled

gcc -v -E -x c /dev/null -o /dev/null -march=native 2>&1 | grep /cc1

gcc -march=native -Q --help=target | grep march
gcc -march=native -v -Q --help=target
gcc -march=skylake -v -Q --help=target
```
