### Fedora

```
sudo dnf update
sudo dnf upgrade
```

```
sudo dnf install kernel-devel kernel-headers gcc make dkms acpid libglvnd-glx libglvnd-opengl libglvnd-devel pkgconfig

sudo dnf remove xorg-x11-drv-nouveau
```

```
/etc/sysconfig/grub

## Example row with Fedora 33 BTRFS
GRUB_CMDLINE_LINUX="rhgb quiet rd.driver.blacklist=nouveau"
```

将内核中的nouveau驱动禁用

To install the Display Driver, the Nouveau drivers must first be disabled. Each distribution of Linux has a different method for disabling Nouveau.

The Nouveau drivers are loaded if the following command prints anything:

```
lsmod | grep nouveau
```

```
/usr/lib/modprobe.d/blacklist-nouveau.conf

blacklist nouveau
options nouveau modeset=0
```

#### 更新内核文件

首先备份内核文件，然后更新内核文件

```
sudo cp /boot/vmlinuz-xxx /boot/vmlinuz-xxx.bak
sudo cp /boot/initramfs-xxx.img /boot/initramfs-xxx.img.bak
```

```
sudo dracut /boot/initramfs-$(uname -r).img $(uname -r)
sudo dracut /boot/initramfs-$(uname -r).img $(uname -r) --force
```

UEFI

```
sudo grub2-mkconfig -o /boot/efi/EFI/fedora/grub.cfg
```

#### Reboot to runlevel 3

You don’t have Desktop/GUI on runlevel 3. Make sure that you have some access to end of guide. (Open it on mobile browser, Print it, use lynx/links/w3m, save it to text file).

```
systemctl set-default multi-user.target

reboot
```

然后重启操作系统

```
sudo sh NVIDIA-Linux-xxx.run
```

#### All Is Done and Then Reboot Back to Runlevel 5

```
systemctl set-default graphical.target

reboot
```

VDPAU/VAAPI support

To enable video acceleration support for your player (Note: you need Geforce 8 or later).

```
dnf install vdpauinfo libva-vdpau-driver libva-utils
```
