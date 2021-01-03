### DNF flags

```
echo 'fastestmirror=1' | sudo tee -a /etc/dnf/dnf.conf
echo 'max_parallel_downloads=10' | sudo tee -a /etc/dnf/dnf.conf
echo 'deltarpm=true' | sudo tee -a /etc/dnf/dnf.conf
```

or

```
sudo nano /etc/dnf/dnf.conf

[main]
gpgcheck=1
installonly_limit=3
clean_requirements_on_remove=True
best=False
skip_if_unavailable=True
fastestmirror=1
max_parallel_downloads=10
deltarpm=true
```

### btrfs filesystem optimizations

Fedora has not optimized the mount options for btrfs yet. I have found that there is some general agreement on the following mount options if you are on a SSD or NVME:

- ssd: use SSD specific options for optimal use on SSD and NVME
- noatime: prevent frequent disk writes by instructing the Linux kernel not to store the last access time of files and folders
- space_cache: allows btrfs to store free space cache on the disk to make caching of a block group much quicker
- commit=120: time interval in which data is written to the filesystem (value of 120 is taken from Manjaro’s minimal iso)
- compress=zstd: allows to specify the compression algorithm which we want to use. btrfs provides lzo, zstd and zlib compression algorithms. Based on some Phoronix test cases, zstd seems to be the better performing candidate.
- discard=async: Btrfs Async Discard Support Looks To Be Ready For Linux 5.6

So add these options to your btrfs subvolume mount points in your fstab:

```
sudo nano /etc/fstab

UUID=...    /                       btrfs   subvol=root,x-systemd.device-timeout=0,ssd,noatime,space_cache,commit=120,compress=zstd,discard=async 0 0
UUID=...    /boot                   ext4    defaults        1 2
UUID=...    /boot/efi               vfat    umask=0077,shortname=winnt 0 2
UUID=...    /home                   btrfs   subvol=home,x-systemd.device-timeout=0,ssd,noatime,space_cache,commit=120,compress=zstd,discard=async 0 0
UUID=...    /btrfs_pool             btrfs   subvolid=5,x-systemd.device-timeout=0,ssd,noatime,space_cache,commit=120,compress=zstd,discard=async 0 0
```

```
sudo mkdir -p /btrfs_pool
sudo mount -a
```

Note that I also add a mountpoint for the btrfs root filesystem (this has always id 5) for easy access of all my subvolumes in /btrfs_pool. You would need to restart to make use of the new options. I usually first run updates and restart prior to restoring my backups, such that my restored files are using the optimized mount options such as compression.

Furthermore, as I am using btrfs discard support, let’s check whether the discard option is passed on.

As both fstrim and discard=async mount option can peacefully co-exist, I also enable `fstrim.timer`:

```
sudo systemctl enable fstrim.timer
```

### install desktop environments

List available desktop environments

```
dnf grouplist -v

Available Environment Groups:
   Fedora Custom Operating System (custom-environment)
   Minimal Install (minimal-environment)
   Fedora Server Edition (server-product-environment)
   Fedora Workstation (workstation-product-environment)
   Fedora Cloud Server (cloud-server-environment)
   KDE Plasma Workspaces (kde-desktop-environment)
   Xfce Desktop (xfce-desktop-environment)
   LXDE Desktop (lxde-desktop-environment)
   Hawaii Desktop (hawaii-desktop-environment)
   LXQt Desktop (lxqt-desktop-environment)
   Cinnamon Desktop (cinnamon-desktop-environment)
   MATE Desktop (mate-desktop-environment)
   Sugar Desktop Environment (sugar-desktop-environment)
   Development and Creative Workstation (developer-workstation-environment)
   Web Server (web-server-environment)
   Infrastructure Server (infrastructure-server-environment)
   Basic Desktop (basic-desktop-environment)
Installed Groups:
   KDE (K Desktop Environment) (kde-desktop)
[output has been truncated]
```

Install the selected desktop environment using the `dnf install` command. Ensure to prefix with the `@` sign, for example:

```
sudo dnf install @kde-desktop-environment
sudo dnf install @xfce-desktop-environment
```

### system upgrade

1. update your Fedora release and reboot the computer

```
sudo dnf upgrade --refresh
```

2. install the dnf-plugin-system-upgrade package

```
sudo dnf install dnf-plugin-system-upgrade
```

3. download the packages that are needed to be updated

```
sudo dnf system-upgrade download --refresh --releasever=33
```

Change the `--releasever=` option if you want to upgrade to a different release. Most people will want to upgrade to the latest stable release, which is `33`. You can also use `34` to upgrade to a Branched release, or `rawhide` to upgrade to Rawhide. Note that neither of these two are stable releases.

If you are upgrading to `Rawhide`, you will need to import the RPM GPG key for it. This will be the highest numbered key version in `/etc/pki/rpm-gpg/`. For example, if there is a Branched release that is 30, then you should look for a 31, and if there is currently no Branched release, it will be 30:

```
sudo rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-31-primary
```

4. If some of your packages have unsatisfied dependencies, the upgrade will refuse to continue until you run it again with an extra `--allowerasing` option. This often happens with packages installed from third-party repositories for which an updated repositories hasn’t been yet published. Study the output very carefully and examine which packages are going to be removed. None of them should be essential for system functionality, but some of them might be important for your productivity.

In case of unsatisfied dependencies, you can sometimes see more details if you add `--best` option to the command line.

If you want to remove/install some packages manually before running `dnf system-upgrade download` again, it is advisable to perform those operations with `--setopt=keepcache=1` dnf command line option. Otherwise the whole package cache will be removed after your operation, and you will need to download all the packages once again.

5. Trigger the upgrade process. This will reboot your machine (immediately!, without a countdown or confirmation, so close other programs and save your work) into the upgrade process running in a console terminal:

```
sudo dnf system-upgrade reboot
```

6. Once the upgrade process completes, your system will reboot a second time into the updated release version of Fedora.

#### some problems in system upgrade

我在升级过程中，遇到了问题

```
dnf system-upgrade download --refresh --releasever=33

Ignoring repositories: copr:copr.fedorainfracloud.org:prince781:vala-language-server, scootersoftware
No match for group package "lsvpd"
No match for group package "xorg-x11-drv-armsoc"
No match for group package "k3b-extras-freeworld"
No match for group package "powerpc-utils"
No match for group package "gstreamer1-plugins-bad-nonfree"
No match for group package "cvsgraph"
...
No match for group package "fedora-user-agent-chrome"
No match for group package "google-croscore-cousine-fonts"
Error: 
 Problem: package gstreamer1-plugins-ugly-free-devel-1.16.2-2.fc32.x86_64 requires gstreamer1-plugins-ugly-free = 1.16.2-2.fc32, but none of the providers can be installed
  - gstreamer1-plugins-ugly-free-1.16.2-2.fc32.x86_64 does not belong to a distupgrade repository
  - problem with installed package gstreamer1-plugins-ugly-free-devel-1.16.2-2.fc32.x86_64
(try to add '--skip-broken' to skip uninstallable packages)
```

前面的 No match 其实可以通过`--skip-broken`参数解决，但是这个Error是要人工介入处理的。错误信息已经提示我们了，因此直接移除掉gstreamer1-plugins-ugly-free-devel就好了：

```
sudo dnf remove gstreamer1-plugins-ugly-free-devel
```

然后继续接着下载：

```
sudo dnf system-upgrade download --refresh --releasever=33 --skip-broken
```

Download complete! Use `dnf system-upgrade reboot` to start the upgrade.
To remove cached metadata and transaction use `dnf system-upgrade clean`
The downloaded packages were saved in cache until the next successful transaction.
You can remove cached packages by executing `dnf clean packages`.

### Update System Configuration Files

Most configuration files are stored in the `/etc` folder. If you have changed the package’s configuration files, RPM creates new files with either `.rpmnew` (the new default config file), or `.rpmsave` (your old config file backed up). You can search for these files, or use the rpmconf tool that simplifies this process. install rpmconf

```
sudo dnf install rpmconf
```

Once the install is complete enter:

```
sudo rpmconf -a
```

Some third party software drop edited configuration files in `/etc/yum.repos.d/` and reverting these files to their original versions may disable updates for the software. Please remember to review configuration files in this directory carefully.

If you use rpmconf to upgrade the system configuration files supplied with the upgraded packages then some configuration files may change. After the upgrade you should verify `/etc/ssh/sshd_config`, `/etc/nsswitch.conf`, `/etc/ntp.conf` and others are expected. For example, if OpenSSH is upgraded then `sshd_config` reverts to the default package configuration. The default package configuration does not enable public key authentication, and allows password authentication.

#### Clean-Up Old Packages

You can see a list of packages with broken dependencies by typing:

```
sudo dnf repoquery --unsatisfied
```

The list should be empty, but if this is not the case consider removing them as they are not likely to work.

You can see duplicate packages (packages with multiple versions installed) with:

```
sudo dnf repoquery --duplicates
```

Run `sudo dnf update` first, as this list is only valid if you have a fully updated system. Otherwise, you will see a list of installed packages that are no longer in the repositories because an update is available. This list may also contain packages installed from third-party repositories who may not have updated their repositories.

For packages from the official repositories, the latest version should be installed. However, some packages that are still on your system may no longer be in the repositories. To see a list of these packages do:

```
sudo dnf list extras
```

If you see a package you do not need, or use, you can remove it with:

```
sudo dnf remove $(dnf repoquery --extras --exclude=kernel,kernel-\*)
```

You can safely remove packages no longer in use with:

```
sudo dnf autoremove
```

DNF decides that a package is no longer needed if you haven’t explicitly asked to install it and nothing else requires it. However, that doesn’t mean that the package is not useful or that you don’t use it. Only remove what you are sure you don’t need.

#### Clean-Up Old Symlinks

There may be some dangling symlinks in the filesystem after an upgrade. You can clean the dangling links by installing the symlinks utility and deleteing the old links.

```
sudo dnf install symlinks
```

Once the utility is installed, you can audit for broken symlinks like shown below. `-r` means recursive.

```
sudo symlinks -r /usr | grep dangling
```

After you verify the list of broken symlinks you can delete them like shown below. `-d` means delete.

```
sudo symlinks -r -d /usr
```

`--skip-broken`
For install command:

The `--skip-broken` option is an alias for `--setopt=strict=0`. Both options could be used with DNF to skip all unavailable packages or packages with broken dependencies given to DNF without raising an error causing the whole operation to fail. This behavior can be set as default in dnf.conf file. See strict conf option.

For upgrade command:

The semantics that were supposed to trigger in YUM with `--skip-broken` are now set for plain dnf update as a default. There is no need to use `--skip-broken` with the dnf upgrade command. To use only the latest versions of packages in transactions, there is the --best command line switch.

#### Update and Upgrade Commands are the Same

Invoking `dnf update` or `dnf upgrade`, in all their forms, has the same effect in DNF, with the latter being preferred.
