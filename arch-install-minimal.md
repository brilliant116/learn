```
cfdisk /dev/sda

mkfs.ext4 /dev/sda1

mount /dev/sda1 /mnt

nano /etc/pacman.d/mirrorlist
-----------------------------

Server = <https://mirrors.bfsu.edu.cn/archlinux/>$repo/os/$arch

pacstrap /mnt base base-devel linux linux-firmware

genfstab -U /mnt >> /mnt/etc/fstab

cat /mnt/etc/fstab

arch-chroot /mnt

pacman -S nano dialog wpa_supplicant networkmanager openssh git
pacman -S dhcpcd #可选

nano /etc/locale.gen

locale-gen

nano /etc/locale.conf
---------------------

LANG=en_US.UTF-8

nano /etc/hostname
------------------

ruby

nano /etc/hosts
---------------

127.0.0.1   localhost
::1         localhost
127.0.1.1   ruby.localdomain ruby

pacman -S grub intel-ucode

grub-install --target=i386-pc /dev/sda

grub-mkconfig -o /boot/grub/grub.cfg

useradd -m -G wheel smith

passwd

passwd smith

EDITOR=nano visudo

systemctl enable NetworkManager
systemctl enable sshd
systemctl enable dhcpcd #可选

reboot

sudo pacman -S xorg

KDE plasma

sudo pacman -S plasma-meta konsole dolphin

sudo pacman -S plasma-desktop konsole dolphin sddm

sudo systemctl enable sddm

XFCE

sudo pacman -S xfce4
sudo pacman -S xfce4-goodies

sudo pacman -S lightdm lightdm-gtk-greeter
sudo systemctl enable lightdm

sudo pacman -S lxdm
sudo systemctl enable lxdm

sudo pacman -S ttf-hack ttf-jetbrains-mono ttf-fira-code
sudo pacman -S adobe-source-han-sans-cn-fonts adobe-source-han-serif-cn-fonts

sudo pacman -S open-vm-tools xf86-input-vmmouse xf86-video-vmware gtkmm3 gtkmm gtk2

sudo systemctl enable vmtoolsd.service

sudo systemctl enable vmware-vmblock-fuse.service

sudo nano /etc/mkinitcpio.conf
------------------------------

MODULES=(vsock vmw_vsock_vmci_transport vmw_balloon vmw_vmci vmwgfx)

sudo mkinitcpio -p linux

reboot

sudo nano /etc/pacman.conf

[archlinuxcn]
Server = https://mirrors.bfsu.edu.cn/archlinuxcn/$arch

sudo pacman -Sy archlinuxcn-keyring

sudo pacman -S fcitx5 fcitx5-chinese-addons fcitx5-chewing
sudo pacman -S fcitx5-qt fcitx5-gtk
sudo pacman -S fcitx5-pinyin-zhwiki fcitx5-pinyin-moegirl
sudo pacman -S fcitx5-configtool

~/.xprofile

INPUT_METHOD  DEFAULT=fcitx5
GTK_IM_MODULE DEFAULT=fcitx5
QT_IM_MODULE  DEFAULT=fcitx5
XMODIFIERS    DEFAULT=\@im=fcitx5
```
