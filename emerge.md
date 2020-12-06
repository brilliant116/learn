
```
emerge-webrsync
emerge --sync
```

### man emerge

```
--noreplace, -n: skip the packages that have been installed.
```

update portage

```
emerge --oneshot --update portage
```

search packages

```
emerge --search neofetch
emerge -s neofetch
emerge --searchdesc pdf
```

install packages

```
emerge --ask --verbose www-client/firefox
emerge -av www-client/firefox

emerge =www-client/firefox-70
```

remove packages

```
emerge --deselect www-client/firefox
emerge --depclean -pv
emerge --depclean -av
```

```
emerge -uDN @world
```

avoid unnecessary rebuilds when USE flags changes (get added or dropped from the repository) has no impact

```
emerge -uD --changed-use @world
```

更新所有安装的包和依赖

```
emerge -auvDN --with-bdeps=y @world
```

更新配置

```
dispatch-conf
etc-update
```

```
perl-cleaner --all
```

### 添加overlay

1. eselect

```
emerge -a app-eselect/eselect-repository
```

#### add unregistered repositories

```
eselect repository add <repo-name> git <url>
eselect repository add deepin git <https://github.com/zhtengw/deepin-overlay.git>
```

```
emerge --sync deepin
```

#### disable repositories without removing contents

```
# registered repositories
eselect repository disable deepin

# unregistered repositories
eselect repository disable -f deepin
```

#### disable repositories and remove contents

```
eselect repository remove deepin
eselect repository remove -f deepin
```

2. manual

```
nano /etc/portage/repos.conf/sakaki-tools.conf
----------------------------------------------
[sakaki-tools]

location = /var/db/portage/sakaki-tools
sync-type = git
sync-uri = https://github.com/sakaki-/sakaki-tools.git
priority = 50
auto-sync = yes
```

```
emerge --sync sakaki-tools

emaint sync --repo sakaki-tools
emaint sync -r sakaki-tools
```

#### remove the content of repositories

```
rm /etc/portage/repos.conf/sakaki-tools.conf
rm -rf /var/db/portage/sakaki-tools
```

#### Nvidia card crashes on boot

```
/etc/modprobe.d/blacklist.conf
------------------------------
blacklist vga16fb
blacklist nouveau
blacklist rivafb
blacklist nvidiafb
blacklist rivatv
```

```
/boot/grub/grub.cfg
-------------------
linux   /boot/kernel-genkernel-x86_64-[ver]-gentoo root=UUID=[id] ro nomodeset nouveau.modeset=0
```

edit `/etc/default/grub`, use `grub-mkconfig -o /boot/grub/grub.cfg`
