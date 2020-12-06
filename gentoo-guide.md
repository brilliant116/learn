### btrfs

1.

```
cfdisk /dev/sda

mkfs.fat -F32 /dev/sda1
mkfs.fat -F32 -n EFI /dev/sda1

mkfs.btrfs /dev/sda2
mkfs.btrfs --label system /dev/sda2
mkfs.btrfs -L system /dev/sda2

mount /dev/sda1 /boot

mount /dev/sda2 /mnt/gentoo
mount -t btrfs LABEL=system /mnt/gentoo

btrfs subvolume create /mnt/gentoo/subvol-root
btrfs subvolume create /mnt/gentoo/subvol-home
btrfs subvolume create /mnt/gentoo/subvol-snapshots

btrfs subvolume set-default /mnt/gentoo/subvol-root

btrfs subvol get-default /mnt/gentoo

btrfs subvol list /mnt/gentoo

umount -R /mnt/gentoo

mount -t btrfs -o defaults,noatime,subvol=subvol-root /mnt/gentoo

mount -t btrfs -o defaults,noatime,subvol=subvol-root /dev/sdb /mnt/gentoo/boot
mount -t btrfs -o defaults,noatime,subvol=subvol-home /dev/sdb /mnt/gentoo/home
mount -t btrfs -o defaults,noatime,subvol=subvol-snapshots /dev/sdb /mnt/gentoo/.snapshot
```

```
UUID=...  /            btrfs   defaults,noatime,subvol=subvol-root      0 0
UUID=...  /boot        btrfs   defaults,noatime,subvol=subvol-boot      0 0
UUID=...  /home        btrfs   defaults,noatime,subvol=subvol-home      0 0
UUID=...  /.snapshot   btrfs   defaults,noatime,subvol=subvol-snapshots 0 0
```

2.

```
# create filesystem
mkfs -t btrfs -L boot /dev/sda1
mkfs -t btrfs -L btrfsroot /dev/sda2

# create subvols
mount /dev/sda2 /mnt/gentoo
cd /mnt/gentoo
btrfs subvol create root
btrfs subvol create home

mount -o defaults,noatime,compress=lzo,autodefrag,subvol=root /dev/sda4 /mnt/gentoo

# create dirs for mounts
cd /mnt/gentoo
mkdir home root boot

# mount
mount -o defaults,noatime,compress=lzo,autodefrag,subvol=home /dev/sda4 /mnt/gentoo/home
mount -o defaults,noatime /dev/sda2 /mnt/gentoo/boot
```

```
UUID=...   /         btrfs   defaults,noatime,compress=lzo,autodefrag,subvol=root      0 0
UUID=...   /home     btrfs   defaults,noatime,compress=lzo,autodefrag,subvol=home      0 0
UUID=...   /boot     btrfs   defaults,noatime                                          0 0
```

### SSD

#### fstrim

```
emerge -a sys-apps/util-linux
lsblk --discard
```

A device supporting discard has non-zero values in the columns of DISC-GRAN (discard granularity) and DISC-MAX (discard max bytes).

For rootfs it is usually recommended to periodically use fstrim utility.

1.

```
sudo fstrim -v /
```

2.Periodic fstrim jobs

There are multiple ways how to setup a periodic block discarding process. As of 2018, the default recommended frequency is once a week.

- cron

Run fstrim on all mounted devices that support discard on a weekly basis. Run fstrim once per week:

```
/etc/crontab

# Mins  Hours  Days   Months  Day of the week   Command
  15    13     *      *       1                 /sbin/fstrim --all
```

run fstrim only for a selected mount point. Run fstrim once per week on rootfs:

```
/etc/crontab

# Mins  Hours  Days   Months  Day of the week   Command
  15    13     *      *       1                 /sbin/fstrim -v /
```

#### portage TMPDIR on tmpfs

当emerge安装包时，在tmpfs(RAM)中进行编译，而不是在HDD或SSD磁盘中进行编译，可以提高emerge的时间和减少磁盘的擦写。

```
nano /etc/fstab

tmpfs  /var/tmp/portage  tmpfs size=12G,uid=portage,gid=portage,mode=775,noatime 0 0
```

修改`/etc/fstab`后，挂载portage的TMPDIR到内存

```
mount /var/tmp/portage
```

**针对单一的包，在tmpfs外面的空间进行编译**

```
nano /etc/portage/env/notmpfs.conf

PORTAGE_TMPDIR="/var/tmp/notmpfs"
```

```
mkdir /var/tmp/notmpfs
chown portage:portage /var/tmp/notmpfs
chmod 775 /var/tmp/notmpfs
```

```
nano /etc/portage/package.env

app-office/libreoffice  notmpfs.conf
dev-lang/ghc   notmpfs.conf
dev-lang/mono   notmpfs.conf
dev-lang/rust   notmpfs.conf
dev-lang/spidermonkey  notmpfs.conf
mail-client/thunderbird  notmpfs.conf
sys-devel/gcc   notmpfs.conf
www-client/chromium  notmpfs.conf
www-client/firefox  notmpfs.conf
```

重新分配tmpfs的大小

```
mount -o remount,size=N /var/tmp/portage
```
