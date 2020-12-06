### FreeBSD

#### Desktop environment

#### Xorg

The binary package can be installed quickly but with fewer options for customization:

```
# pkg install xorg
```

To build and install from the Ports Collection:

```
# cd /usr/ports/x11/xorg
# make install clean
```

Either of these installations results in the complete Xorg system being installed. Binary packages are the best option for most users.

A smaller version of the X system suitable for experienced users is available in `x11/xorg-minimal`. Most of the documents, libraries, and applications will not be installed. Some applications require these additional components to function.

```
# pkg install xorg-minimal
```

#### Using Fonts in Xorg

Type1 Fonts

The default fonts that ship with Xorg are less than ideal for typical desktop publishing applications. Large presentation fonts show up jagged and unprofessional looking, and small fonts are almost completely unintelligible. However, there are several free, high quality Type1 (PostScript®) fonts available which can be readily used with Xorg. For instance, the URW font collection (x11-fonts/urwfonts) includes high quality versions of standard type1 fonts (Times Roman®, Helvetica®, Palatino® and others). The Freefonts collection (x11-fonts/freefonts) includes many more fonts, but most of them are intended for use in graphics software such as the Gimp, and are not complete enough to serve as screen fonts. In addition, Xorg can be configured to use TrueType® fonts with a minimum of effort. For more details on this, see the X(7) manual page or Section 5.5.2, “TrueType® Fonts”.

To install the above Type1 font collections from binary packages, run the following commands:

```
# pkg install urwfonts
```

Alternatively, to build from the Ports Collection, run the following commands:

```
# cd /usr/ports/x11-fonts/urwfonts
# make install clean
```

And likewise with the freefont or other collections. To have the X server detect these fonts, add an appropriate line to the X server configuration file (/etc/X11/xorg.conf), which reads:

```
FontPath "/usr/local/share/fonts/urwfonts/"
```

Alternatively, at the command line in the X session run:

```
# xset fp+ /usr/local/share/fonts/urwfonts
# xset fp rehash
```

This will work but will be lost when the X session is closed, unless it is added to the startup file (~/.xinitrc for a normal startx session, or ~/.xsession when logging in through a graphical login manager like XDM). A third way is to use the new /usr/local/etc/fonts/local.conf as demonstrated in Section 5.5.3, “Anti-Aliased Fonts”.

TrueType® Fonts

Xorg has built in support for rendering TrueType® fonts. There are two different modules that can enable this functionality. The freetype module is used in this example because it is more consistent with the other font rendering back-ends. To enable the freetype module just add the following line to the "Module" section of /etc/X11/xorg.conf.

```
Load  "freetype"
```

Now make a directory for the TrueType® fonts (for example, /usr/local/share/fonts/TrueType) and copy all of the TrueType® fonts into this directory. Keep in mind that TrueType® fonts cannot be directly taken from an Apple® Mac®; they must be in UNIX®/MS-DOS®/Windows® format for use by Xorg. Once the files have been copied into this directory, use mkfontscale to create a fonts.dir, so that the X font renderer knows that these new files have been installed. mkfontscale can be installed as a package:

```
# pkg install mkfontscale
```

Then create an index of X font files in a directory:

```
# cd /usr/local/share/fonts/TrueType
# mkfontscale
```

Now add the TrueType® directory to the font path. This is just the same as described before

```
# xset fp+ /usr/local/share/fonts/TrueType
# xset fp rehash
```

or add a FontPath line to xorg.conf.

Now Gimp, Apache OpenOffice, and all of the other X applications should now recognize the installed TrueType® fonts. Extremely small fonts (as with text in a high resolution display on a web page) and extremely large fonts (within StarOffice™) will look much better now.

**KDE**

install the KDE packages

```
# pkg install x11/kde5
```

To instead build the KDE port, use the following command. Installing the port will provide a menu for selecting which components to install.

```
# cd /usr/ports/x11/kde5
# make install clean
```

KDE requires `/proc` to be mounted. Add this line to `/etc/fstab` to mount this file system automatically during system startup:

```
# ee /etc/fstab
proc           /proc       procfs  rw  0   0
```

KDE uses D-Bus and HAL for a message bus and hardware abstraction. These applications are automatically installed as dependencies of KDE. Enable them in /etc/rc.conf so they will be started when the system boots:

```
dbus_enable="YES"
hald_enable="YES"
```

- Since KDE Plasma 5, the KDE Display Manager, KDM is no longer developed. A possible replacement is SDDM.

```
# pkg install x11/sddm
```

```
# vi /etc/rc.conf
sddm_enable="YES"
```

- A second method for launching KDE Plasma is to type startx from the command line. For this to work, the following line is needed in `~/.xinitrc`:

```
echo "exec ck-launch-session startplasma-x11" > ~/.xinitrc
```

- A third method for starting KDE Plasma is through XDM. To do so, create an executable `~/.xsession` as follows:

```
# echo "exec ck-launch-session startplasma-x11" > ~/.xsession
```

**GNOME**

```
# pkg install gnome3
```

To instead build GNOME from ports, use the following command.

```
# cd /usr/ports/x11/gnome3
# make install clean
```

GNOME requires `/proc` to be mounted. Add this line to `/etc/fstab` to mount this file system automatically during system startup:

```
proc           /proc       procfs  rw  0   0
```

GNOME uses D-Bus and HAL for a message bus and hardware abstraction. These applications are automatically installed as dependencies of GNOME. Enable them in `/etc/rc.conf` so they will be started when the system boots:

```
# vi /etc/rc.conf
dbus_enable="YES"
hald_enable="YES"
```

After installation, configure Xorg to start GNOME.

- The easiest way to do this is to enable the GNOME Display Manager, GDM, which is installed as part of the GNOME package or port. It can be enabled by adding this line to `/etc/rc.conf`:

```
gdm_enable="YES"
```

It is often desirable to also start all GNOME services. To achieve this, add a second line to `/etc/rc.conf`:

```
gnome_enable="YES"
```

GDM will start automatically when the system boots.

- A second method for starting GNOME is to type `startx` from the command-line after configuring `~/.xinitrc`. If this file already exists, replace the line that starts the current window manager with one that starts /`usr/local/bin/gnome-session`. If this file does not exist, create it:

```
# echo "exec /usr/local/bin/gnome-session" > ~/.xinitrc
```

- A third method is to use XDM as the display manager. In this case, create an executable `~/.xsession`:

```
# echo "exec /usr/local/bin/gnome-session" > ~/.xsession
```

**Xfce**

To install the Xfce package:

```
# pkg install xfce
```

Alternatively, to build the port:

```
# cd /usr/ports/x11-wm/xfce4
# make install clean
```

Xfce uses D-Bus for a message bus. This application is automatically installed as dependency of Xfce. Enable it in `/etc/rc.conf` so it will be started when the system boots:

```
dbus_enable="YES"
```

- Unlike GNOME or KDE, Xfce does not provide its own login manager. In order to start Xfce from the command line by typing startx, first create `~/.xinitrc`

```
# echo ". /usr/local/etc/xdg/xfce4/xinitrc" > ~/.xinitrc
```

- An alternate method is to use XDM. To configure this method, create an executable `~/.xsession`

```
# echo ". /usr/local/etc/xdg/xfce4/xinitrc" > ~/.xsession
```

安装完 FreeBSD 后的一些后续步骤

升级系统

```
freebsd-update fetch
freebsd-update install
reboot
```

#### port

```
cp /etc/portsnap.conf /etc/portsnap.conf.backup
```

```
ee /etc/portsnap.conf
```

```
# SERVERNAME=portsnap.FreeBSD.org
SERVERNAME=portsnap.tw.FreeBSD.org
SERVERNAME=freebsd-portsnap.mirror.bjtulug.org
```

```
portsnap fetch extract
portsnap fetch update
```

To download a compressed snapshot of the Ports Collection into `/var/db/portsnap`

```
portsnap fetch
```

When running Portsnap for the first time, extract the snapshot into `/usr/ports`

```
portsnap extract
```

After the first use of Portsnap has been completed as shown above, `/usr/ports` can be updated as needed by running

```
portsnap fetch
portsnap update
```

When using fetch, the extract or the update operation may be run consecutively, like so

```
portsnap fetch update
```

#### pkg

/etc/pkg/FreeBSD.conf

```
mkdir -p /usr/local/etc/pkg/repos
```

```
ee /usr/local/etc/pkg/repos/FreeBSD.conf
-------------------------------------
taiwan:{
    url: "pkg+http:#pkg0.twn.freebsd.org/${ABI}/latest",
    mirror_type: "srv",
    signature_type: "none",
    fingerprints: "/usr/share/keys/pkg",
    enabled: yes
    }
#台湾源，快，推荐；latest表示软件最新版；如latest换成quarterly表示软件稳定版
ustc:{
    url: "pkg+http:#mirrors.ustc.edu.cn/freebsd-pkg/${ABI}/latest",
    mirror_type: "srv",
    signature_type: "none",
    fingerprints: "/usr/share/keys/pkg",
    enabled: no
    }
#中科大源，偶尔会time out安装软件不成功；需要启用时再将enabled改成yes
bjtu: {
    url: "pkg+http:#freebsd-pkg.mirror.bjtulug.org/${ABI}/quarterly",
    mirror_type: "srv",
    signature_type: "none",
    fingerprints: "/usr/share/keys/pkg",
    enabled: yes
}
FreeBSD: { enabled: no }
```

若要使用https，请先安装security/ca_root_nss，并将http修改为https，最后使用命令`pkg update -f`刷新缓存即可.

```
pkg update -f
```

```
/etc/make.conf
--------------
MASTER_SITE_OVERRIDE?=http:#mirrors.ustc.edu.cn/freebsd-ports/distfiles/${DIST_SUBDIR}/
```

#### freebsd-update源:提供基本系统更新

```
ee /etc/freebsd-update.conf
---------------------------
# ServerName update.FreeBSD.org
ServerName update.tw.FreeBSD.org
ServerName freebsd-update.mirror.bjtulug.org
```

例:从FreeBSD 11升级到12.0

```
freebsd-update -r 12.0-RELEASE upgrade
```
