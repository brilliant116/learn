For fresh installations, the file `/usr/local/etc/pkg/repos/df-latest.conf`.sample is copied to `/usr/local/etc/pkg/repos/df-latest` so that pkg(8) works out of the box.

```
cp /usr/local/etc/pkg/repos/df-latest.conf /usr/local/etc/pkg/repos/df-latest
```

```
# vi /usr/local/etc/pkg/repos/df-latest
Avalon: {
    url             : pkg+http://mirror-master.dragonflybsd.org/dports/${ABI}/LATEST,
    mirror_type     : NONE
    [...]
    enabled         : no
}

Japan: {
    url             : https://ftp.jaist.ac.jp/pub/DragonFly/${ABI}/LATEST,
    enabled         : yes
}
```

```
pkg search editors
pkg info pkg
```

```
pkg install curl
```

The new package and any additional packages that were installed as dependencies can be seen in the installed packages list:

```
pkg info
```

```
pkg delete curl
```

```
pkg upgrade
```

#### Auditing Installed Packages with pkg(8)

Occasionally, software vulnerabilities may be discovered in software within DPorts. pkg(8) includes built-in auditing. To audit the software installed on the system, type:

```
pkg audit -F
```

#### Automatically Removing Leaf Dependencies with pkg(8)

Removing a package may leave behind unnecessary dependencies, like security/ca_root_nss in the example above. Such packages are still installed, but nothing depends on them any more. Unneeded packages that were installed as dependencies can be automatically detected and removed:

```
pkg autoremove
```

#### Removing Stale pkg(8) Packages

By default, pkg(8) stores binary packages in a cache directory as defined by PKG_CACHEDIR in pkg.conf(5). When upgrading packages with pkg upgrade, old versions of the upgraded packages are not automatically removed.

remove the outdated binary packages

```
pkg clean
```
