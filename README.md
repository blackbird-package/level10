# mobconf
blackbird netbook or laptop config


## prepare
### physical volume
| disk | partition | type              | luks  | lvm   | label    |  format | mount                      |
| ---- | --------- | ----------------- | ----- | ----- | -------- |  ------ | -------------------------- |
| 0    | 1         | efi               | false | false | boot     |  fat 32 | /boot                      |
| 0    | 2         | linux file system | true  | false | keys     |  luks   | none                       |
| 0    | 3         | linux filesystems | true  | true  | proc |  luks   | see logical layout point 1 |
| 0    | 4         | linux home        | true  | true  | data |  luks   | see logical layout point 1 |

### ecnrypt partition
```
cryptsetup luksFormat --sector-size=4096 /dev/nvme0n1p2
```

```
cryptsetup luksFormat --sector-size=4096 /dev/nvme0n1p3
```

```
cryptsetup luksFormat --sector-size=4096 /dev/nvme0n1p4
```

```
cryptsetup luksOpen /dev/nvme0n1p3 proc
```

```
cryptsetup luksOpen /dev/nvme0n1p4 data
```

### logical volume proc
| partition | list  | group | name |  mount                    | format |
| --------- | ----  | ----- | ---- |  -------------------------| ------ |
| 2         | 1     | proc  | root |  /mnt                     | ext4   |
| 2         | 2     | proc  | opts |  /mnt/opt                 | ext4   |
| 2         | 3     | proc  | vars |  /mnt/var                 | ext4   |
| 2         | 4     | proc  | libs | /mnt/var/usr/             | ext4   |
| 2         | 5     | proc  | game | /mnt/var/games/           | ext4   |
| 2         | 6     | proc  | vlog |  /mnt/var/log             | ext4   |
| 2         | 7     | proc  | vaud |  /mnt/var/log/audit       | ext4   |
| 2         | 8     | proc  | vtmp |  /mnt/var/tmp             | ext4   |
| 2         | 9     | proc  | vpac |  /mnt/var/cache/pacman    | ext4   |
| 2         | 10    | proc  | ring |                           | ext4   |
| 2         | 12    | proc  | docs |  /mnt/var/http            | ext4   |

#### minimum disk layout root
| partition | list  | group | name | size | mount                    | format |
| --------- | ----  | ----- | ---- |----  | -------------------------| ------ |
| 2         | 1     |       | boot | 320M |/mnt/boot                 | vfat   |
| 2         | 2     | proc  | root | 10G  |/mnt                      | ext4   |
| 2         | 3     | proc  | opts | 15G  |/mnt/opt                  | ext4   |
| 2         | 4     | proc  | vars | 2G   |/mnt/var                  | ext4   |
| 2         | 5     | proc  | libs | 512M |/mnt/var/usr/             | ext4   |
| 2         | 6     | proc  | game | 512M | /mnt/var/games/          | ext4   |
| 2         | 7     | proc  | vlog | 1G   |/mnt/var/log              | ext4   |
| 2         | 8     | proc  | vaud | 256M |/mnt/var/log/audit        | ext4   |
| 2         | 9     | proc  | vtmp | 2G   |/mnt/var/tmp              | ext4   |
| 2         | 10    | proc  | vpac | 2G   |/mnt/var/cache/pacman     | ext4   |
| 2         | 11    | proc  | ring | 512M |                          | ext4   |
| 2         | 12    | proc  | docs | 2G   | /mnt/srv/http            | ext4   |

```
pvcreate /dev/mapper/proc 
```
```
vgcreate proc /dev/mapper/proc
```

#### proc root
```
lvcreate -L 10G proc -n root
```
```
mount /dev/proc/root /mnt
```
#### boot
```
mkfs.vfat -F32 -S 4096 -n BOOT /dev/nvme0n1p1
```
```
mkdir /mnt/boot
```
```
mount -o uid=0,gid=0,fmask=0077,dmask=0077 /dev/nvmw0n1p1 /mnt/boot
```
#### proc opts
```
lvcreate -L 15G proc -n opts
```
```
mkdir /mnt/opt
```
```
mount -o rw,nodev,nosuid,relatime /dev/proc/opts /mnt/opt
```
#### proc vars
```
lvcreate -L 2G proc -n vars
```
```
mkdir /mnt/var
```
```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/vars /mnt/var
```
#### proc libs
```
lvcreate -L 512M proc -n libs
```
```
mkdir /mnt/var/usr
```
```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/libs /mnt/var/usr
```
#### proc game
```
lvcreate -L 512M proc -n game
```
```
mkdir /mnt/var/games
```
```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/game /mnt/var/games
```
#### proc vlog
```
lvcreate -L 1G proc -n vlog
```
```
mkdir /mnt/var/log
```
```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/vlog /mnt/var/log
```
#### proc vaud
```
lvcreate -L 256M proc -n vaud
```
```
mkdir /mnt/var/log/audit
```
```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/vaud /mnt/var/log/audit
```
#### proc vtmp
```
lvcreate -L 2G proc -n vtmp
```
```
mkdir /mnt/var/tmp
```
```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/vtmp /mnt/var/tmp
```
#### proc vpac
```
lvcreate -L 2G proc -n vpac
```
```
mkdir -p /mnt/var/cache/pacman
```
```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/vpac /mnt/var/cache/pacman
```
#### proc ring
```
lvcreate -L 512M proc -n ring
```
```
cryptsetup luksFormat --sector-size=4096 /dev/proc/ring
```
```
cryptsetup luksOpen /dev/proc/ring proc_keys
```
```
mkfs.ext4 -b 4096 /dev/mapper/proc_keys
```
#### proc docs
```
lvcreate -l100%FREE proc -n docs
```
```
mkfs.ext4 -b 4096 /dev/proc/docs
```
```
mkdir -p /mnt/srv /mnt/srv/http
```
```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/docs /mnt/srv/http
```
### logical volume data

#### data pods
```
lvcreate -L 20G  data -n pods
```
```
mkfs.ext4 -b 4096 /dev/data/pods
```
```
mkdir -p /mnt/var/lib /mnt/var/lib/containers
```
```
mount -o rw,nodev,noexec,nosuid,relatime /dev/data/pods /mnt/var/lib/containers
```
#### data home
```
lvcreate -l100%FREE  data -n home
```
```
mkfs.ext4 -b 4096 /dev/data/home
```
```
mkdir /mnt/home
```
```
mount -o rw,nodev,noexec,nosuid,relatime /dev/data/home /mnt/home
```

## configs
#### user
```
useradd -m nama_user
```
```
passwd nama_user
```
#### hostname
```
echo 'hostname' > /etc/hostname
```
#### cmdline
```
echo "cryptdevice=UUID=$(blkid -s UUID -o value /dev/nvme0n1p3):proc root=/dev/proc/root" > /etc/cmdline.d/01-boot.conf
```
```
echo "data UUID=$(blkid -s UUID -o value /dev/nvme0n1p4) none" >> /etc/crypttab
```
#### network
```
nvim /etc/systemd/network/20-ethernet.network
```
ubah di bagian `Address` sesuaikan
#### passwd
```
nvim /etc/passwd
```
pindahkan line dengan user game dibawah nobody

#### aide
```
aide --init
```
```
mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz
```

