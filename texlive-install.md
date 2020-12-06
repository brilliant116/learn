
1. download the installer

进入<https://mirrors.bfsu.edu.cn/CTAN/systems/texlive/tlnet/>

```
wget https://mirrors.bfsu.edu.cn/CTAN/systems/texlive/tlnet/install-tl-unx.tar.gz
curl -O https://mirrors.bfsu.edu.cn/CTAN/systems/texlive/tlnet/install-tl-unx.tar.gz

tar xzvf install-tl-unx.tar.gz
```

```
wget https://mirrors.bfsu.edu.cn/CTAN/systems/texlive/tlnet/install-tl.zip

unzip install-tl.zip
```

`perl-tk`和`perl-doc`

```
sudo pacman -S perl-tk
```

```
cd install-tl-<Tab>

sudo ./install-tl
sudo ./install-tl -gui
```

In default condition, it will search optimal mirror to use. You can also choose a specific mirror.

```
sudo ./install-tl -repository https://mirrors.bfsu.edu.cn/CTAN/systems/texlive/tlnet

sudo ./install-tl -gui -repository https://mirrors.bfsu.edu.cn/CTAN/systems/texlive/tlnet
```

2. set PATH

`~/.bashrc`

```
export PATH=${PATH}:/usr/local/texlive/2019/bin/x86_64-linux
export MANPATH=${MANPATH}:/usr/local/texlive/2019/texmf-dist/doc/man
export INFOPATH=${INFOPATH}:/usr/local/texlive/2019/texmf-dist/doc/info
```

```
export PATH=/usr/local/texlive/2018/bin/x86_64-linux:$PATH
export MANPATH=/usr/local/texlive/2018/texmf-dist/doc/man:$MANPATH
export INFOPATH=/usr/local/texlive/2018/texmf-dist/doc/info:$INFOPATH
```

```
export PATH=$PATH:/usr/local/texlive/2020/bin/x86_64-linux
export INFOPATH=$INFOPATH:/usr/local/texlive/2020/texmf-dist/doc/info
export MANPATH=$MANPATH:/usr/local/texlive/2020/texmf-dist/doc/man
```

```
tex -v
tlmgr --version
pdftex --version
xetex --version
luatex --version
```

```
sudo tlmgr update --list
```

```
sudo tlmgr update --self --all
```

永久使用BFSU镜像源

```
tlmgr option repository https://mirrors.bfsu.edu.cn/CTAN/systems/texlive/tlnet
```

临时切换镜像源

```
tlmgr update --all --repository https://mirrors.bfsu.edu.cn/CTAN/systems/texlive/tlnet
```

```
tlmgr update --self --repository https://mirrors.bfsu.edu.cn/CTAN/systems/texlive/tlnet
tlmgr update --all --repository https://mirrors.bfsu.edu.cn/CTAN/systems/texlive/tlnet
```

彻底删除已安装的Tex Live

```
sudo apt purge texlive*
sudo rm -rf /usr/local/texlive/*
rm -rf ~/.texlive*
sudo rm -rf /usr/local/share/texmf
sudo rm -rf /var/lib/texmf
sudo rm -rf /etc/texmf
sudo apt remove tex-common --purge

find -L /usr/local/bin/ -lname /usr/local/texlive/*/bin/* | sudo xargs rm
```
