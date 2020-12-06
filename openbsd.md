配置Xwindow
执行：

```
# X -configure
```

生成/root/xorg.conf.new。进行测试：

```
# X -config /root/xorg.conf.new
```

修改配置文件，设置分辨率(粗体内容)：

```
Section "Screen"
        Identifier "Screen0"
        Device     "Card0"
        Monitor    "Monitor0"
        DefaultDepth   24
        SubSection "Display"
                Viewport   0 0
                Depth     1
        EndSubSection
        ... ...
        SubSection "Display"
                Viewport   0 0
                Depth     24
                Modes "1024x768"
        EndSubSection
EndSection
```

复制配置文件：

```
# cp /root/xorg.conf.new /etc/X11/xorg.conf
```

就可以执行`startx`进入X了。

OpenBSD缺省安装的窗口管理器是fvwm。由于Nvidia没有提供显卡驱动，于是懒得在桌面上花太多心思，就不安装其他的桌面环境了。

#### change mirror

编辑`.profile`，设置`PKG_PATH`环境变量

```
export PKG_PATH=https://mirrors.bfsu.edu.cn/OpenBSD/$(uname -r)/packages/$(arch -s)/
export PKG_PATH=https://cdn.openbsd.org/pub/OpenBSD/$(uname -r)/packages/$(arch -s)/
```

edit `/etc/installurl`

```
https://mirrors.bfsu.edu.cn/OpenBSD
https://cdn.openbsd.org/pub/OpenBSD
```

edit `installpath` within `/etc/pkg.conf`

```
installpath = https://mirrors.bfsu.edu.cn/OpenBSD
installpath = https://cdn.openbsd.org/pub/OpenBSD
```

```
pkg_info -Q unzip
```

```
pkg_add rsync
```

```
pkg_add -u unzip
```

```
pkg_delete screen
```

Unneeded dependencies can be trimmed by running `pkg_delete -a` at any time.

### ports

```
cd /tmp
ftp https://cdn.openbsd.org/pub/OpenBSD/$(uname -r)/ports.tar.gz
```

```
cd /usr
tar xzf /tmp/ports.tar.gz
```

```
cd /usr/ports
doas pkg_add portslist
make search key=rsnapshot
```

```
cd /usr/ports/net/rsnapshot
make install
```

Large applications will require a lot of system resources to build. If you get "out of memory" type of errors when building such a port, this usually has one of two causes:

- Your resource limits are too restrictive. Adjust them with ksh's `ulimit` or csh's `limit` command.

- You simply don't have enough RAM.

#### Cleaning Up After a Build

You probably want to clean the port's default working directory after you have built the package and installed it.

```
make clean
```

By default, workdir dependencies are automatically cleaned. You can remove installed packages that were only needed for building with

```
pkg_delete -a
```

If you wish to remove the source distribution set(s) of the port, you would use:

```
make clean=dist
```

In case you have been compiling multiple flavors of the same port, you can clear the working directories of all these flavors at once using:

```
make clean=flavors
```

You can also clean everything up after each build by changing `BULK` from `auto` to `Yes`.

```
make package BULK=Yes
```

#### Uninstalling a Port's Package

It is very easy to uninstall a port:

```
make uninstall
```

This will call pkg_delete(1) to have the corresponding package removed from your system. If you would like to get rid of the packages you just built, you can do so as follows:

```
make clean=packages
```
