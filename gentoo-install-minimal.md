
```
ip a

iwctl

help
device list

station wlan0 scan
station wlan0 connect SSID

iwctl --passphrase <passphrase> station device connect SSID

cfdisk /dev/sda

mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
mount /dev/sda1 /boot
mount /dev/sda2 /mnt/gentoo

cd /mnt/gentoo
links https://mirrors.bfsu.edu.cn/gentoo/
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner

emerge -a app-portage/cpuid2cpuflags
cpuid2cpuflags
cpuid2cpuflags >> /etc/portage/make.conf

nano -w /mnt/gentoo/etc/portage/make.conf
-----------------------------------------
CHOST="x86_64-pc-linux-gnu"
GRUB_PLATFORMS="efi-64" # UEFI

ACCEPT_KEYWORDS="~amd64"
ACCEPT_LICENSE="*"

MAKEOPTS="-j6"
#PORTAGE_TMPDIR="/var/tmp"
EMERGE_DEFAULT_OPTS=""
GENTOO_MIRRORS="https://mirrors.bfsu.edu.cn/gentoo"

MICROCODE_SIGNATURES="-S"

VIDEO_CARDS="intel i965 iris nvidia"
INPUT_DEVICES="libinput synaptics"

mkdir --parents /mnt/gentoo/etc/portage/repos.conf
cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/

mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/dev

chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot) ${PS1}"

emerge-webrsync
emerge -avuDN --with-bdeps=y @world

ls /usr/share/zoneinfo
echo "Asia/Shanghai" > /etc/timezone
emerge --config sys-libs/timezone-data
emerge -v --config sys-libs/timezone-data

nano -w /etc/locale.gen
-----------------------
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8

locale-gen

eselect locale list
eselect locale set 4

env-update && source /etc/profile && export PS1="(chroot) ${PS1}"

emerge -av sys-kernel/gentoo-kernel-bin
emerge -av sys-kernel/linux-firmware

emerge -av net-misc/networkmanager
rc-update add NetworkManager default

nano -w /etc/conf.d/hostname
----------------------------
hostname="smith"

emerge app-admin/sysklogd sys-process/cronie -a
rc-update add sysklogd default
rc-update add cronie default

emerge -av sys-boot/grub:2
grub-install --target=x86_64-efi --efi-directory=/boot
grub-mkconfig -o /boot/grub/grub.cfg

passwd root
useradd -m -G wheel smith
usermod -aG additional_groups smith
passwd smith

emerge -a --noreplace app-admin/sudo
EDITOR=nano visudo

eselect profile list
eselect profile set 23
emerge -avuDN --with-bdeps=y @world

emerge -av x11-base/xorg-server x11-drivers/xf86-video-intel
emerge -a --quiet-build kde-plasma/plasma-meta

nano /etc/conf.d/xdm
--------------------
DISPLAYMANAGER="sddm"

rc-update add xdm default
```
