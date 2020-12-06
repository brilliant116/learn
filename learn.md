
#### test mirror's speed

curl(支持-Z，--parallel Perform transfers in parallel参数以多线程方式下载) ，设置的传输时间为10秒

```
curl -m 10 -LZO https://mirrors.bfsu.edu.cn/fedora/releases/32/Workstation/x86_64/iso/Fedora-Workstation-Live-x86_64-32-1.6.iso
```

#### clojure

make sure the following dependcies are installed: `bash`, `curl`, `rlwrap` and `java`

```
curl -O https://download.clojure.org/install/linux-install-1.10.1.716.sh
chmod +x linux-install-1.10.1.716.sh
sudo ./linux-install-1.10.1.716.sh
```

#### nim

dependencies: `PCRE`, `OpenSSL`

```
wget https://nim-lang.org/choosenim/init.sh
chmod +x init.sh
./init.sh
```

```
curl https://nim-lang.org/choosenim/init.sh -sSf | sh
```

#### racket

```
wget https://mirrors.tuna.tsinghua.edu.cn/racket-installers/recent/racket-7.8-x86_64-linux.sh
chmod +x racket-7.8-x86_64-linux.sh
./racket-7.8-x86_64-linux.sh
```

```
This program will extract and install Racket v7.8.

Note: the required diskspace for this installation is 524M.

Do you want a Unix-style distribution?
  In this distribution mode files go into different directories according
  to Unix conventions.  A "racket-uninstall" script will be generated
  to be used when you want to remove the installation.  If you say 'no',
  the whole Racket directory is kept in a single installation directory
  (movable and erasable), possibly with external links into it -- this is
  often more convenient, especially if you want to install multiple
  versions or keep it in your home directory.
Enter yes/no (default: no) > no

Where do you want to install the "racket" directory tree?
  1 - /usr/racket [default]
  2 - /usr/local/racket
  3 - ~/racket (/home/smith/racket)
  4 - ./racket (here)
  Or enter a different "racket" directory to install in.
> 3

Checking the integrity of the binary archive... ok.
Unpacking into "/home/smith/racket" (Ctrl+C to abort)...
Done.

If you want to install new system links within the "bin", "man"
  and "share/applications" subdirectories of a common directory prefix
  (for example, "/usr/local") then enter the prefix of an existing
  directory that you want to use.  This might overwrite existing symlinks,
  but not files.
(default: skip links) >
```

```
echo $PATH
```

`~/.bashrc`

```
export PATH=/home/smith/racket/bin/:$PATH
```

uninstall racket

```
rm -r /home/smith/racket
```

- download via curl, wget

```
curl -O https://github.com/elm/compiler/releases/download/0.19.1/binary-for-linux-64-bit.gz
curl -L -o elm.gz https://github.com/elm/compiler/releases/download/0.19.1/binary-for-linux-64-bit.gz
```

```
gunzip elm.gz
```

- chmod

```
ls -l
```

to use `chmod` to set permissions, we need to tell it:

- who: Who we are setting permissions for.
- what: What change that we will make. Are we adding or removing the permission?
- which: Which of the permissions are we setting?

The who values are:

- u: User, the owner of the file.
- g: Group, members of the group the file belongs to.
- o: Others, people not governed by the u and g permissions.
- a: All, meaning all of the above.

The which values are:

- r: read permission.
- w: write permission.
- x: execute permission.

```
chmod a+x some_file.sh
chmod +x some_file.sh # 等于上一个
chmod a=rwx some_file.sh
chmod u=rw,go=r some_file.sh
chmod a= some_file.sh
```

show the status of modules in Linux kernel

```
lsmod
```

```
lsb_release -a
```

内核版本

```
uname -a
cat /proc/version
```

```
df -h
```

查看文件夹里所有文件的大小总和

```
du -sh /home
```

### some essential packages in Linux distribution

- Arch Linux

```
sudo pacman -S base-devel
```

- Debian/Ubuntu

```
sudo apt install build-essential
```

- OpenSUSE

```
zypper info -t pattern devel_basis

sudo zypper --type pattern devel_basis
sudo zypper -t pattern devel_basis
```

```
hostnamectl
```

```
删除用户
sudo userdel username
删除用户和用户目录
sudo userdel -r username
```

屏幕支持的分辨率

```
xrandr
```

添加sudo用户

```
useradd -m -G wheel smith
passwd smith
groups smith
cat /etc/group

usermod -a -G wheel smith
usermod -aG wheel smith
groups smith
```

```
useradd -m -G <groups> -s $(which bash) <username>
useradd -m -G users,wheel -s $(which bash) ruby
passwd <username>

usermod -a -G <groups> <username>

EDITOR=nano visudo
```

### oh-my-zsh

安装zsh

```
sudo pacman -S zsh
```

将默认的shell改为zsh

```
chsh -l
cat /etc/shells

chsh -s $(which zsh)
reboot
```

### python virtual environment

```
sudo pacman -S python-virtualenv
```

```
在指定位置，新建virtualenv
virtualenv path/to/folder
virtualenv -p=/usr/bin/python path/to/folder
cd path/to/folder
```

```
source bin/activate
```

```
deactivate
```

change pypi mirror

1.upgrade pip to the latest version and set mirror.

```
pip install -U pip
pip config set global.index-url https://mirrors.bfsu.edu.cn/pypi/web/simple
```

2.if the default mirror network is bad, you can use the BFSU mirror temporarily first.

```
pip install -i https://mirrors.bfsu.edu.cn/pypi/web/simple pip -U
pip config set global.index-url https://mirrors.bfsu.edu.cn/pypi/web/simple
```

```
pip install -U wheel setuptools
```

然后就可以无需用sudo权限就可以安装python包了。

```
pip install spyder
sudo dnf install zeromp-devel libffi-devel
pip install --upgrade pyflakes
spyder3
```

### execute .sh file

```
chmod +x file.sh
./file.sh
```

输出环境变量

```
printenv
```

### tar

```
tar xzvf tar.gz
tar xjvf tar.bz2
tar xJvf tar.xz
```

-z, --gzip, --gunzip, --ungzip: Filter the archive through gzip(1).

-j, --bzip2: Filter the archive through bzip2(1).

-J, --xz: Filter the archive through xz(1).

### Node.js

install Node.js via binary archive on Linux

```
sudo mkdir -p /usr/local/lib/nodejs
tar xJvf node-*.tar.xz
cp -r node-* /usr/local/lib/nodejs
```

`~/.profile`

```
# nodejs
VERSION=v14.8.0
DISTRO=linux-x64
export PATH=/usr/local/lib/nodejs/node-$VERSION-$DISTRO/bin:$PATH
```

```
. ~/.profile
```

```
node -v
npm version
npx -v
```

### fonts

系统字体的配置文件为`/etc/fonts/fonts.conf`。在系统安装字体`/usr/share/fonts`，在用户安装字体`~/.local/share/fonts` (`~/.fonts` is already deprecated.)

```
fc-cache -fv
```

### VS Code

用户配置文件为`~/.config/Code/User/settings.json`, 插件的安装位置为`~/.vscode/extensions`

```json
{
    "workbench.colorTheme": "Quiet Light",
    "workbench.settings.editor": "json",
    "editor.fontFamily": "Monaco",
    "editor.fontSize": 17,
    "editor.fontLigatures": true,
    "editor.fontWeight": "400",
    "window.zoomLevel": 0,
    "rust-client.rustupPath": "$HOME/.cargo/bin/rustup",
    "rust-client.channel": "stable",
    "editor.formatOnSave": true,
    "files.autoSave": "onWindowChange",
    "editor.autoIndent": "advanced",
    "editor.detectIndentation": true,
    "editor.wordWrap": "on",
    "editor.tokenColorCustomizations": {
        "comments": "#3498DB"
    },
    "terminal.integrated.inheritEnv": false,
    "python.terminal.activateEnvironment": false,
    "markdown.preview.fontSize": 16,
    "editor.codeActionsOnSave": {
        "source.fixAll.markdownlint": true
    },
    "markdownlint.config": {
        "MD013": false,
        "MD040": false,
        "MD041": false,
        "MD036": false,
    },
}
```
