### NetBSD

configurational menu

```
Enable installation of binary packages

Host
mirrors.bfsu.edu.cn
Base directory
pkgsrc/packages/NetBSD
Package diretory
amd64/9.1/All
```

```
mirrors.bfsu.edu.cn
pkgsrc/stable
```

Xfce

```
pkgin install nano xfce4
pkgin install xfce4-extras
pkgin install fam hal
```

```
cp /usr/pkg/share/examples/rc.d/famd /etc/rc.d
cp /usr/pkg/share/examples/rc.d/dbus /etc/rc.d
cp /usr/pkg/share/examples/rc.d/hal /etc/rc.d
```

```
# nano /etc/rc.d
rpcbind=YES
famd=YES
dbus=YES
hal=YES
```

进入桌面

```
startx
```

```
# pkgin install FiraCode-2
FiraCode-2: updating font database in /usr/pkg/share/fonts/X11/OTF (ttf)
FiraCode-2: updating font database in /usr/pkg/share/fonts/X11/OTF (x11)
FiraCode-2: updating font database in /usr/pkg/share/fonts/X11/TTF (ttf)
FiraCode-2: updating font database in /usr/pkg/share/fonts/X11/TTF (x11)
```

```
# pkgin install open-vm-tools
The following files should be created for open-vm-tools-10.3.0nb11:

        /etc/rc.d/vmtools (m=0755)
            [/usr/pkg/share/examples/rc.d/vmtools]

===========================================================================
===========================================================================
$NetBSD: MESSAGE.x11,v 1.1 2016/11/21 00:14:16 pho Exp $

You need to mount the vmblock file system in order to enable the clipboard
synchronization and drag and drop support:

    # mkdir /var/run/vmblock-fuse
    # vmware-vmblock-fuse /var/run/vmblock-fuse

Then start vmware-user-suid-wrapper in your .xinitrc:

    % vmware-user-suid-wrapper
```

pkg_install error log can be found in `/var/db/pkgin/pkg_install-err.log`

```
pkgin install wget dejavu-ttf inconsolata-ttf
```

```
# nano /etc/rc.conf
auto_ifconfig=YES  #为了开机可以自动设定网卡
hostname=NetBSD  #主机名
dhclient=YES #DHCP
sshd=YES
```

```
# nano ~/.profile
export PKG_PATH=https://mirrors.bfsu.edu.cn/pkgsrc/packages/NetBSD/amd64/9.1/All/
export PKG_PATH=http://cdn.NetBSD.org/pub/pkgsrc/packages/NetBSD/$(uname -m)/$(uname -r|cut -f '1 2' -d.)/All/
export PKG_PATH=http://cdn.netbsd.org/pub/pkgsrc/packages/NetBSD/amd64/9.1/All/
export PKG_PATH=ftp://ftp7.jp.netbsd.org/pub/pkgsrc/packages/NetBSD/amd64/8.0/All/
```

```
pkg_add -v pkgfind
pkg_add -v pkgin
```

使用pkgin需要设定预编译包地址，配置文件在`/usr/pkg/etc/pkgin/repositories.conf`，编辑配置文件

```
# nano /usr/pkg/etc/pkgin/repositories.conf
https://mirrors.bfsu.edu.cn/pkgsrc/packages/NetBSD/amd64/9.1/All/
http://mirrors.bfsu.edu.cn/pkgsrc/packages/NetBSD/amd64/9.1/All/
http://cdn.netbsd.org/pub/pkgsrc/packages/NetBSD/$arch/9.0/All/
ftp://ftp7.jp.netbsd.org/pub/pkgsrc/packages/NetBSD/$arch/8.0/All/
```

默认安装到的目录为`/usr/pkg/`

运行下面命令

```
pkgin update
```

pkgin具体用法

```
pkgin update ＃创建packages数据
pkgin up
pkgin install ＃安装packages
pkgin in
pkgin remove ＃删除
pkgin rm
pkgin upgrade ＃升级
pkgin clean ＃清除packages缓存
pkgin cl
pkgin autoremove #自动清理不用的packages
pkgin ar
pkgin search ＃搜索
pkgin se
```

pkgin的缓存文件在`/var/db/pkgin/cache/`。

这里是我缓存文件的目录，可以先用wget下载好再统一安装。这样更省时间。

#### pkgsrc

如果pkgin里没有某个软件，可使用pkgsrc，您可以轻松地在系统上添加，删除和管理软件。pkgsrc基本上是一组文件，按类别分组，其中包含用于安装所选软件的信息。所有这些文件一起通常被称为pkgsrc树。不过用这个，需定期更新pkgsrc树。

使用pkgsrc包来安装。需要到<https://cdn.NetBSD.org/pub/pkgsrc/stable>或<https://mirrors.bfsu.edu.cn/pkgsrc/stable/>下载pkgsrc.tar.bz2或pkgsrc.tar.gz文件。

```
mkdir /usr/work  #为了保持系统纯净，创建新的目录
mkdir /usr/distfiles
vim /etc/mk.conf
WRKOBJDIR=/usr/work #定制WRKOBJDIR路径
DISTDIR=/usr/distfiles #添加已下载的文件存储位置
```

```
wget -c https://mirrors.bfsu.edu.cn/pkgsrc/stable/pkgsrc.tar.gz
tar -xzvf pkgsrc.tar.gz -C /usr   #把目录树解压到/usr下面
```

用wget作为pkgsrc默认下载工具

编辑/etc/mk.conf,加入

```
FETCH_CMD=wget
FETCH_BEFORE_ARGS=--passive-ftp
FETCH_RESUME_ARGS=-c
FETCH_OUTPUT_ARGS=-o

MASTER_SITE_BACKUP= https://mirrors.nju.edu.cn/pkgsrc/distfiles/ https://mirrors.tuna.tsinghua.edu.cn/pkgsrc/distfiles/
```

```
# 以后装软件去/usr/pkgsrc目录找，如zile
cd /usr/pkgsrc/editors/zile
make install clean
make install clean clean-depends #安装并清理
make distclean #清除distfiles已下载的文件
```

```
pkg_info #列出软件包
pkg_delete packagename #移除软件包
make update #更新单个软件包
```

set root passsword

```
/usr/bin/passwd
```

```
useradd -m -G wheel ruby
passwd ruby
pkgin install sudo
```

```
# EDITOR=nano visudo
```

#### System time

NetBSD, like all Unix systems, uses a system clock based on Greenwich time (GMT) and this is what you should set your system clock to. If you want to keep the system clock set to the local time (because, for example, you have a dual boot system with Windows installed), you must notify NetBSD, adding rtclocaltime=YES to /etc/rc.conf:

```
# echo rtclocaltime=YES >> /etc/rc.conf
# sh /etc/rc.d/rtclocaltime restart
```

#### 校准时间

```
# date
Fri Aug 28 11:10:57 UTC 2020
# date 202008281712
Fri Aug 28 17:12:00 UTC 2020te: date set by root
# date
Fri Aug 28 17:12:09 UTC 2020
```

#### Secure Shell

By default, all services are disabled in a fresh NetBSD installation, and ssh is no exception. You may wish to enable it so you can log in to your system remotely. Set `sshd=YES` in`/etc/rc.conf` and then start the server with the command

```
# /etc/rc.d/sshd start
```

The first time the server is started, it will generate a new keypair, which will be stored inside the directory `/etc/ssh`.

#### 桌面环境

Xfce

```
pkgin install nano xfce4
pkgin install xfce4-extras
pkgin install fam hal
```

```
cp /usr/pkg/share/examples/rc.d/famd /etc/rc.d
cp /usr/pkg/share/examples/rc.d/dbus /etc/rc.d
cp /usr/pkg/share/examples/rc.d/hal /etc/rc.d
```

```
# nano /etc/rc.d
rpcbind=YES
famd=YES
dbus=YES
hal=YES
```

**from the console**

After running the above commands, edit your `.xinitrc` as above and change “openbox” (or “twm”) to “xfce4-session”. The next time you run startx the Xfce desktop environment will be started.

```
# nano .xinitrc
exec xfce4-session
```

or

```
# nano .xinitrc
xfce4-session
```

**Graphical login with xdm**

If you always use X and the first thing you do after you log in is run startx, you can set up a graphical login to do this automatically. It is very easy:

Create the .xsession file in your home directory. This file is similar to .xinitrc and can, in fact, be a link to it.

```
ln -s .xinitrc ~/.xsession
```

Modify `/etc/rc.conf`, adding the following line:

```
xdm=YES       # x11 display manager
```

Start xdm (or reboot your system, as this will be done automatically from now on):

```
# /etc/rc.d/xdm start
```

```
pkgin install xfce4
pkgin install gdm
pkgin install sddm
```

安装完后，根据提示，需要把`/usr/pkg/share/examples/rc.d`下面的famd、gdm，hal和dbus脚本复制到`/etc/rc.d`下面

```
cp /usr/pkg/share/examples/rc.d/famd /etc/rc.d/
cp /usr/pkg/share/examples/rc.d/dbus /etc/rc.d/
cp /usr/pkg/share/examples/rc.d/hal /etc/rc.d/
```

在`/etc/rc.conf`加入如下内容

```
# nano /etc/rc.conf

famd=YES
rpcbind=YES
dbus=YES
hal=YES
gdm=YES
```

```
# nano /etc/rc.conf
dhcpcd=YES
sshd=YES
wscons=YES
hostname=NetBSD

rpcbind=YES
famd=YES
dbus=YES
hal=YES
```

在`~/.xinitrc`加入

```
exec xfce4-session
```

```
/etc/rc.d/rpcbind start
/etc/rc.d/famd start
/etc/rc.d/dbus start
/etc/rc.d/hal start
```

进入桌面

```
startx
```

文泉驿微米黑显示效果很不错，cp至`/usr/X11R7/lib/X11/fonts/wqy/`，然后在那个文件夹下ttmkfdir。

查看已安装软件：

```
# pkgin ug
```

```
sudo pkgin in gnome-desktop gnome-panel gnome-session gnome-media gnome-applets gnome-system-monitor gnome-terminal gnome-themes gnome-volume-manager gnome-device-manage gnome-volume-manage nautilus gnome-control-cente
```

安装ibus拼音输入法

```
pkgin install ibus ibus-pinyin ibus-table-chinese
```

安装完后在，`~/.xinitrc`加入如下内容：

```
export XMODIFIERS=@im=ibus
export GTK_IM_MODULE=ibus
export QT_IM_MODULE=ibus
ibus-daemon -d -r -x
```

安装中文字体
到文泉驿官方网站下载字体文件包 <http:#sourceforge.net/projects/wqy/files/>

解压后将字体复制到

```
mv wqy-bitmapfont /usr/X11R7/lib/X11/fonts/
cd /usr/X11R7/lib/X11/fonts/wqy-bitmapfont/
fc-cache -f -v
mkfontscale .
mkfontdir
```

#### Stopping and rebooting the system

Shutdown the machine immediately and reboot

```
shutdown -r now
```

Shutdown the machine and immediately power off (for ATX hardware, you will need to install and enable the apm(8) support in kernel: see ?How to poweroff at shutdown for details)

```
shutdown -p now
```

Shutdown the machine and halt afterwards (but keep the power on)

```
shutdown -h now
```

halt the system

```
shutdown -h now
halt
```

reboot the system

```
shutdown -r now
reboot
```

halt, reboot and shutdown are not synonyms: the latter is more sophisticated. On a multiuser system you should really use shutdown, which allows you to schedule a shutdown time and notify users. It will also take care to stop processes properly. For more information, see the shutdown(8), halt(8) and reboot(8) manpages.

#### upgrade NetBSD

Let's assume you are running NetBSD/amd64 7.1 and you wish to upgrade to NetBSD 7.2. The procedure to do so would be to run the following command:

```
sysupgrade auto http://cdn.NetBSD.org/pub/NetBSD/NetBSD-7.2/amd64
```
