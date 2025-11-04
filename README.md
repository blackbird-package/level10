# blackbird protocol level 20


## 1. Partitioning

### physical volume

**disk layout**
| disk | partition | type              | luks  | lvm   | label    |  format | mount                      |
| ---- | --------- | ----------------- | ----- | ----- | -------- |  ------ | -------------------------- |
| 0    | 1         | efi               | false | false | boot     |  fat 32 | /boot                      |
| 0    | 2         | linux file system | true  | false | keys     |  luks   | none                       |
| 0    | 3         | linux filesystems | true  | true  | proc     |  luks   | see logical layout point 1 |
| 0    | 4         | linux home        | true  | true  | data     |  luks   | see logical layout point 1 |

**notes**:
- gunakan fdisk, cfdisk, parted untuk membuat physical volume sesuai dengan guideline
- minimum size untuk perangkat dengan kartu grafis nvidia sebesar `1.5GB`


### logical volume

#### volume proc

**disk layout**
| partition | list  | group | name |  mount                    | format |
| --------- | ----  | ----- | ---- |  -------------------------| ------ |
| 2         | 1     | proc  | root |  /mnt                     | ext4   |
| 2         | 2     | proc  | opts |  /mnt/opt                 | ext4   |
| 2         | 3     | proc  | vars |  /mnt/var                 | ext4   |
| 2         | 4     | proc  | libs |  /mnt/var/usr/            | ext4   |
| 2         | 5     | proc  | game |  /mnt/var/games/          | ext4   |
| 2         | 6     | proc  | vlog |  /mnt/var/log             | ext4   |
| 2         | 7     | proc  | vaud |  /mnt/var/log/audit       | ext4   |
| 2         | 8     | proc  | vtmp |  /mnt/var/tmp             | ext4   |
| 2         | 9     | proc  | vpac |  /mnt/var/cache/pacman    | ext4   |
| 2         | 10    | proc  | ring |                           | ext4   |
| 2         | 12    | proc  | docs |  /mnt/var/http            | ext4   |

**minimum size **
| name | size | mount                    | format |
| ---- |----  | -------------------------| ------ |
| root | 10G  |/mnt                      | ext4   |
| opts | 15G  |/mnt/opt                  | ext4   |
| vars | 2G   |/mnt/var                  | ext4   |
| libs | 512M |/mnt/var/usr/             | ext4   |
| game | 512M | /mnt/var/games/          | ext4   |
| vlog | 1G   |/mnt/var/log              | ext4   |
| vaud | 256M |/mnt/var/log/audit        | ext4   |
| vtmp | 2G   |/mnt/var/tmp              | ext4   |
| vpac | 2G   |/mnt/var/cache/pacman     | ext4   |
| ring | 512M |                          | ext4   |
| docs | 2G   | /mnt/srv/http            | ext4   |


**technical procedure**

**1. encrypt disk**
```
cryptsetup luksFormat --sector-size=4096 /dev/nvme0n1p3
```
```
cryptsetup luksOpen /dev/nvme0n1p3 proc
```

**2. create lvm**
```
pvcreate /dev/mapper/proc 
```
```
vgcreate proc /dev/mapper/proc
```

**3. root**
```
lvcreate -L 10G proc -n root
```
```
mkfs.ext4 -b 4096 /dev/proc/root
```
```
mount /dev/proc/root /mnt
```
**4. opts**
```
lvcreate -L 15G proc -n opts
```
```
mkfs.ext4 -b 4096 /dev/proc/opts
```
```
mkdir /mnt/opt
```
```
mount -o rw,nodev,nosuid,relatime /dev/proc/opts /mnt/opt
```
**5. vars**
```
lvcreate -L 2G proc -n vars
```
```
mkfs.ext4 -b 4096 /dev/proc/vars
```
```
mkdir /mnt/var
```
```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/vars /mnt/var
```
**6. libs**
```
lvcreate -L 512M proc -n libs
```
```
mkfs.ext4 -b 4096 /dev/proc/libs
```
```
mkdir /mnt/var/usr
```
```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/libs /mnt/var/usr
```
**7. game**
```
lvcreate -L 512M proc -n game
```
```
mkfs.ext4 -b 4096 /dev/proc/game
```
```
mkdir /mnt/var/games
```
```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/game /mnt/var/games
```
**8. vlog**
```
lvcreate -L 1G proc -n vlog
```
```
mkfs.ext4 -b 4096 /dev/proc/vlog
```
```
mkdir /mnt/var/log
```
```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/vlog /mnt/var/log
```
**9. vaud**
```
lvcreate -L 256M proc -n vaud
```
```
mkfs.ext4 -b 4096 /dev/proc/vaud
```
```
mkdir /mnt/var/log/audit
```
```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/vaud /mnt/var/log/audit
```
**10. vtmp**
```
lvcreate -L 2G proc -n vtmp
```
```
mkfs.ext4 -b 4096 /dev/proc/vtmp
```
```
mkdir /mnt/var/tmp
```
```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/vtmp /mnt/var/tmp
```
**11. vpac**
```
lvcreate -L 2G proc -n vpac
```
```
mkfs.ext4 -b 4096 /dev/proc/vpac
```
```
mkdir -p /mnt/var/cache/pacman
```
```
mount -o rw,nodev,noexec,nosuid,relatime /dev/proc/vpac /mnt/var/cache/pacman
```
**12. ring**
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
**13. docs**
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

#### volume boot

**technical procedure**

```
mkfs.vfat -F32 -S 4096 -n BOOT [ partition path ]
```
```
mkdir /mnt/boot
```
```
mount -o uid=0,gid=0,fmask=0077,dmask=0077 [ path boot partition ] /mnt/boot
```

#### volume data

**disk layout**

| partition | list  | group | name |  mount                    | format |
| --------- | ----  | ----- | ---- |  -------------------------| ------ |
| 2         | 1     | data  | home |  /mnt/home                | ext4   |


**technical procedure**

**1. encrypt disk**
```
cryptsetup luksFormat --sector-size=4096 /dev/nvme0n1p4
```
```
cryptsetup luksOpen /dev/nvme0n1p4 data
```

**2. create lvm**
```
pvcreate /dev/mapper/data
```
```
vgcreate data /dev/mapper/data
```

**3. home**
```
lvcreate -L [ size in G | M ] data -n home
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

## 2. preparation

### Installation
**Technical Procedure**
**1. kernel**
```
pacstrap /mnt linux-hardened linux-firmware mkinitcpio base lvm2 btrfs-progs bubblewrap-suid --noconfirm
```

**2. devel**
```
pacstrap /mnt sudo debugedit fakeroot pkgconf bison gcc pcre flex wget make gcc curl less --noconfirm
```
**3. audio system**
```
pacstrap /mnt pipewire pipewire-pulse pipewire-jack wireplumber pavucontrol sof-firmware mpd mpc --noconfirm
```

**4. desktop**
```
pacstrap /mnt uwsm hyprland hyprpicker hyprshot hypridle hyprlock hyprpolkitagent xdg-desktop-portal-hyprland qt5-wayland qt6-wayland wl-clipboard cliphist mailcap brightnessctl --noconfirm
```

**5. panel**
```
pacstrap /mnt mako waybar wofi mpd mpc --noconfirm
```

**6. file system**
```
pacstrap /mnt nautilus nautilus-image-converter sushi --noconfirm
```

**7. font system**
```
pacstrap /mnt ttf-jetbrains-mono-nerd ttf-droid --noconfirm
```

**8. terminal**
```
pacstrap /mnt kitty kitty-terminfo neovim --noconfirm
 ```

**9. password**
```
pacstrap /mnt gnome-keyring libsecret libpam-google-authenticator libpwquality cracklib polkit apparmor qrencode --noconfirm
```

**10. monitoring**
```
pacstrap /mnt prometheus prometheus-node-exporter btop --noconfirm
```

**11. network**
```
pacstrap /mnt tang openssh ethtool iptables-nft firewalld --noconfirm
```
ethernet
```
cp /etc/system/network/* /mnt/etc/systemd/network/
```
wireless
```
pacstrap /mnt iwd --noconfirm 
```
```
mkdir -p /mnt/var/lib/iwd/ 
```
```
cp /var/lib/iwd/* /mnt/var/lib/iwd/
```

**12. performance**
```
pacstrap /mnt irqbalance tuned tuned-ppd --noconfirm
```

**13. backup**
```
pacstrap /mnt rsync grsync --noconfirm 
```

**14. hosting**
```
pacstrap /mnt go hugo nginx git --noconfirm
```

**15. ucode**
pilih sesuai dengan jenis prosesor yang digunakan
for intel
```
pacstrap /mnt intel-ucode --noconfirm
```
for amd
```
pacstrap /mnt amd-ucode --noconfirm
```

### preconfing
**1.fstab**
```
genfstab -U /mnt > /mnt/etc/fstab
```
```
echo "" >> /mnt/etc/fstab 
```
```
echo "# /dev/mapper/proc-temp" >> /mnt/etc/fstab
```
```
echo "tmpfs     					/tmp        		tmpfs   defaults,rw,nosuid,nodev,noexec,relatime,size=256M" >> /mnt/etc/fstab
```
**2. config**
```
git clone https://github.com/blackbird-package /mnt/opt/config
```
```
cp -fr /mnt/opt/config/mnt /mnt
```
jika menggunakan udev 
```
cp -fr /mnt/opt/config/udev /mnt
```
**note**
untuk pengguna keamanan berbasis Network Bond Disk Encryption maka gunakan udev
jika menggunakan systemd
```
cp -fr /mnt/opt/config/syd /mnt
```


**3.chroot**
```
arch-chroot /mnt
```

## 2. postconfig

**1. hostname** 
```
echo 'nama_hostname' > /etc/hostname
```
**2. locatime** 
```
ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
```
```
hwclock --systohc
```
```
timedatectl set-ntp true
```
```
timedatectl set-timezone Asia/Jakarta
```
```
timedatectl status
```
```
timedatectl show-timesync --all
```
```
systemctl enable systemd-timesyncd.service
```

**2. locale** 
```
locale-gen
```
**2. users** 

- system user
```
echo 'loki ALL=(ALL:ALL) ALL' >> /etc/sudoers
```
```
useradd -d /var/usr loki
```
```
mkdir /var/usr/.ssh
```
```
chmod 700 /var/usr/.ssh/
```
```
touch /var/usr/.ssh/authorized_keys
```
```
chmod 600 /var/usr/.ssh/authorized_keys
```
```
sudo chattr +i /var/usr/.ssh/authorized_keys
```
```
chown -R loki:loki /var/usr
```
```
passwd loki
```
isikan password default
- daemon user
```
useradd -d /var/games -u 50 -g games games
```
```
cat /etc/passwd | grep games
```
```
passwd -l games
```
```
nvim /etc/passwd
```
pastikan line `games` dibawah `nobody` dan shell menjadi`/usr/bin/nologin`, berikut contoh
```
nobody:x:65534:65534:Kernel Overflow User:/:/usr/bin/nologin
games:x:50:50:Games User:/:/usr/bin/nologin
```
- admin user

```
useradd -m nama_user
```
```
echo 'nama_user ALL=(ALL:ALL) ALL' >> /etc/sudoers.d/none
```
```
passwd nama_user
```
```
su nama_user
```
```
sudo su
```
```
exit
```
```
exit
```

**3. nbde** 
- clevis client
```
su nama_user
```
```
git clone https://aur.archlinux.org/mkinitcpio-clevis-hook.git
```
```
cd mkinitcpio-clevis-hook
```
```
makepkg -si
```
```
clevis luks bind -d /dev/nvme0n1p3 tang '{"url":"http://10.10.1.15:51379"}'
```
```
systemctl enable clevis-luks-askpass.path
```

- tang server
```
systemctl enable tangd.socket
```

**4. release** 
```
echo '' > /usr/lib/os-release
```
```
nvim /usr/lib/os-release
```
```
NAME="Blackbird"
PRETTY_NAME="Blackbird"
ID=blackbird
BUILD_ID=rolling
ANSI_COLOR="38;2;23;147;209"
HOME_URL="https://blackbird.lektor.co.id/"
DOCUMENTATION_URL="https://blackbird.lektor.co.id/"
SUPPORT_URL="https://blackbird.lektor.co.id/support/"
BUG_REPORT_URL="https://gitlab.blackbird.org/groups/issues"
PRIVACY_POLICY_URL="https://blackbird.lektor.co.id/privacy-policy/"
LOGO=blackbird-logo
```


## apparmor 

```
systemctl enable apparmor.service
```

## prometheus 
```
systemctl enable prometheus.service
```
```
systemctl enable prometheus-node-exporter.service
```

### irqbalance
```
systemctl enable irqbalance
```

### tuned
```
systemctl enable tuned
```
```
systemctl enable tuned-ppd.service
```

### network
#### nginx
```
systemctl enable nginx
```
### network
```
nvim /etc/systemd/network/20-ethernet.network
```
```
[Network]
Address=[IP]/24
Gateway=10.10.1.1
DNS=1.1.1.1 8.8.8.8
MulticastDNS=yes
```
```
systemctl enable systemd-networkd
```
```
systemctl enable systemd-resolved
```

### boot directory

```
rm /boot/initramfs-linux-hardened*
```
#### intel server
```
mv /boot/intel-ucode.img /boot/vmlinuz-linux-hardened /boot/kernel
```


#### amd server

```
mv /boot/amd-ucode.img /boot/vmlinuz-linux-hardened /boot/kernel
```

### kernel parameter

### udev
```
echo "cryptdevice=UUID=$(blkid -s UUID -o value /dev/nvme0n1p3):proc root=/dev/proc/root" > /etc/cmdline.d/01-boot.conf
```
```
echo "ip=(ip address)::10.10.1.1:255.255.255.0::eth0:none nameserver=10.10.1.1 nameserver=1.1.1.1 nameserver=8.8.8.8 nameserver=1.0.0.1 nameserver=8.8.4.4 nameserver=9.9.9.9 nameserver=149.112.112.112 " > /etc/cmdline.d/05-nets.conf
```
```
nvim /etc/cmdline.d/05-nets.conf
```
### sysd
```
echo "rd.luks.uuid=$(blkid -s UUID -o value /dev/nvme0n1p3)root=/dev/proc/root" > /etc/cmdline.d/01-boot.conf
```



## cryptab
```
echo "data UUID=$(blkid -s UUID -o value /dev/nvme0n1p4) none" >> /etc/crypttab
```
```
bootctl --path=/boot install
```
```
mkinitcpio -P
```
### wake on lan

```
nvim /etc/udev/rules.d/81-wol.rules
```
```
ACTION=="add", SUBSYSTEM=="net", NAME=="en*", RUN+="/usr/bin/ethtool -s $name wol g"
```
```
ethtool interface | grep Wake-on
```

### instrusion detection

```
cd /tmp
```
```
pkg-config --libs --cflags glib-2.0
```
```
wget https://github.com/aide/aide/releases/download/v0.19.2/aide-0.19.2.tar.gz
```
```
tar xf aide-0.19.2.tar.gz
```
```
cd aide-0.19.2
```
```
./configure --with-zlib --with-posix-acl --with-xattr --with-curl --with-locale --with-syslog-ident --with-config-file=/etc/aide.conf
```
```
make && make install
```
```
nvim /etc/systemd/system/aide.service
```
```
[Unit]
Description=Aide Check
ConditionACPower=true

[Service]
Type=simple
ExecStart=/usr/local/bin/aide --check

[Install]
WantedBy=multi-user.target
```
```
nvim /etc/systemd/system/aide.timer
```
```
[Unit]
Description=Aide check every 2 Hours

[Timer]
OnCalendar=*:0/2:00
Unit=aide.service

[Install]
WantedBy=multi-user.target
```
```
systemctl enable aide.timer
```
```
mkdir -p /var/log/aide
```
```
mkdir -p /var/lib/aide
```
```
touch /var/log/aide/aide.log 
```
```
nvim /etc/aide.conf 
```
```
# Example configuration file for AIDE.
# More information about configuration options available in the aide.conf manpage.
# Inspired from https://src.fedoraproject.org/rpms/aide/raw/rawhide/f/aide.conf

# ┌───────────────────────────────────────────────────────────────┐
# │ CONTENTS OF aide.conf                                         │
# ├───────────────────────────────────────────────────────────────┘
# │
# ├──┐VARIABLES
# │  ├── DATABASE
# │  └── REPORT
# ├──┐RULES
# │  ├── LIST OF ATTRIBUTES
# │  ├── LIST OF CHECKSUMS
# │  └── AVAILABLE RULES
# ├──┐PATHS
# │  ├──┐EXCLUDED
# │  │  ├── ETC
# │  │  ├── USR
# │  │  └── VAR
# │  └──┐INCLUDED
# │     ├── ETC
# │     ├── USR
# │     ├── VAR
# │     └── OTHERS
# │
# └───────────────────────────────────────────────────────────────

# ################################################################ VARIABLES

# ################################ DATABASE

@@define DBDIR /var/lib/aide
@@define LOGDIR /var/log/aide

# The location of the database to be read.
database_in=file:@@{DBDIR}/aide.db.gz

# The location of the database to be written.
#database_out=sql:host:port:database:login_name:passwd:table
#database_out=file:aide.db.new
database_out=file:@@{DBDIR}/aide.db.new.gz

# Whether to gzip the output to database
gzip_dbout=yes

# ################################ REPORT

# Default.
log_level=warning
report_level=changed_attributes

report_url=file:@@{LOGDIR}/aide.log
report_url=stdout
#report_url=stderr
#NOT IMPLEMENTED report_url=mailto:root@foo.com
#NOT IMPLEMENTED report_url=syslog:LOG_AUTH

# ################################################################ RULES

# ################################ LIST OF ATTRIBUTES

# These are the default parameters we can check against.
#p:             permissions
#i:             inode:
#n:             number of links
#u:             user
#g:             group
#s:             size
#b:             block count
#m:             mtime
#a:             atime
#c:             ctime
#S:             check for growing size
#acl:           Access Control Lists
#selinux        SELinux security context (must be enabled at compilation time)
#xattrs:        Extended file attributes

# ################################ LIST OF CHECKSUMS

#md5:           md5 checksum
#sha1:          sha1 checksum
#sha256:        sha256 checksum
#sha512:        sha512 checksum
#rmd160:        rmd160 checksum
#tiger:         tiger checksum
#haval:         haval checksum (MHASH only)
#gost:          gost checksum (MHASH only)
#crc32:         crc32 checksum (MHASH only)
#whirlpool:     whirlpool checksum (MHASH only)

# ################################ AVAILABLE RULES

# These are the default rules
#R:             p+i+l+n+u+g+s+m+c+md5
#L:             p+i+l+n+u+g
#E:             Empty group
#>:             Growing logfile p+l+u+g+i+n+S

# You can create custom rules - my home made rule definition goes like this 
ALLXTRAHASHES = sha1+rmd160+sha256+sha512+whirlpool+tiger+haval+gost+crc32
ALLXTRAHASHES = sha1+rmd160+sha256+sha512+tiger
# Everything but access time (Ie. all changes)
EVERYTHING = R+ALLXTRAHASHES

# Sane, with multiple hashes
# NORMAL = R+rmd160+sha256+whirlpool
# NORMAL = R+sha256+sha512
NORMAL = p+i+l+n+u+g+s+m+c+sha256

# For directories, don't bother doing hashes
DIR = p+i+n+u+g+acl+xattrs

# Access control only
PERMS = p+i+u+g+acl

# Logfile are special, in that they often change
LOG = >

# Just do sha256 and sha512 hashes
FIPSR = p+i+n+u+g+s+m+c+acl+xattrs+sha256
LSPP = FIPSR+sha512

# Some files get updated automatically, so the inode/ctime/mtime change
# but we want to know when the data inside them changes
DATAONLY = p+n+u+g+s+acl+xattrs+sha256

# ################################################################ PATHS

# Next decide what directories/files you want in the database.

# ################################ EXCLUDED

# ################ ETC

# Ignore backup files
!/etc/.*~

# Ignore mtab
!/etc/mtab

# ################ USR

# These are too volatile
!/usr/src
!/usr/tmp

# ################ VAR

# Ignore logs
!/var/lib/pacman/.*
!/var/cache/.*
!/var/log/.*  
!/var/log/aide.log
!/var/run/.*  
!/var/spool/.*

# ################################ INCLUDED

# ################ ETC

# Check only permissions, inode, user and group for /etc, but cover some important files closely.
/etc                               PERMS
/etc/aliases                       FIPSR
/etc/at.allow                      FIPSR
/etc/at.deny                       FIPSR
/etc/audit/                        FIPSR
/etc/bash_completion.d/            NORMAL
/etc/bashrc                        NORMAL
/etc/cron.allow                    FIPSR
/etc/cron.daily/                   FIPSR
/etc/cron.deny                     FIPSR
/etc/cron.d/                       FIPSR
/etc/cron.hourly/                  FIPSR
/etc/cron.monthly/                 FIPSR
/etc/crontab                       FIPSR
/etc/cron.weekly/                  FIPSR
/etc/cups                          FIPSR
/etc/exports                       NORMAL
/etc/fstab                         NORMAL
/etc/group                         NORMAL
/etc/grub/                         FIPSR
/etc/gshadow                       NORMAL
/etc/hosts.allow                   NORMAL
/etc/hosts.deny                    NORMAL
/etc/hosts                         FIPSR
/etc/inittab                       FIPSR
/etc/issue                         FIPSR
/etc/issue.net                     FIPSR
/etc/ld.so.conf                    FIPSR
/etc/libaudit.conf                 FIPSR
/etc/localtime                     FIPSR
/etc/login.defs                    FIPSR
/etc/login.defs                    NORMAL
/etc/logrotate.d                   NORMAL
/etc/modprobe.conf                 FIPSR
/etc/nscd.conf                     NORMAL
/etc/pam.d                         FIPSR
/etc/passwd                        NORMAL
/etc/postfix                       FIPSR
/etc/profile.d/                    NORMAL
/etc/profile                       NORMAL
/etc/rc.d                          FIPSR
/etc/resolv.conf                   DATAONLY
/etc/securetty                     FIPSR
/etc/securetty                     NORMAL
/etc/security                      FIPSR
/etc/security/opasswd              NORMAL
/etc/shadow                        NORMAL
/etc/skel                          NORMAL
/etc/ssh/ssh_config                FIPSR
/etc/ssh/sshd_config               FIPSR
/etc/stunnel                       FIPSR
/etc/sudoers                       NORMAL
/etc/sysconfig                     FIPSR
/etc/sysctl.conf                   FIPSR
/etc/vsftpd.ftpusers               FIPSR
/etc/vsftpd                        FIPSR
/etc/X11/                          NORMAL
/etc/zlogin                        NORMAL
/etc/zlogout                       NORMAL
/etc/zprofile                      NORMAL
/etc/zshrc                         NORMAL

# ################ USR

/usr                               NORMAL
/usr/sbin/stunnel                  FIPSR

# ################ VAR

/var/log/faillog                   FIPSR
/var/log/lastlog                   FIPSR
/var/spool/at                      FIPSR
/var/spool/cron/root               FIPSR

# ################ OTHERS

/boot                              NORMAL
/bin                               NORMAL
/lib                               NORMAL
/lib64                             NORMAL
/opt                               NORMAL
/root                              NORMAL
```
```
aide --init
```
```
mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz
```
```
exit
```
```
umount -R /mnt
```
```
reboot
```
# 3. post instalation
```
passwd -l root
```
### blackbird hyprland
#### installation

```
sudo pacman -S uwsm hyprland hyprpolkitagent hypridle hyprlock xdg-desktop-portal-hyprland pipewire pipewire-pulse pipewire-jack wireplumber pavucontrol kitty qt5-wayland qt6-wayland ttf-jetbrains-mono-nerd ttf-droid btop nautilus nautilus-image-converter sushi mako waybar wofi wl-clipboard cliphist mailcap hyprshot gnome-keyring libsecret brightnessctl hyprpicker flatpak gnome-software hugo go --noconfirm
```
#### service

```
sudo systemctl --global enable hypridle.service
```
```
sudo systemctl --global enable hyprpolkitagent
```
```
sudo systemctl --global enable waybar
```
```
sudo systemctl --global enable pipewire-pulse
```

### sddm

#### guideline
```
sudo pacman -S sddm --noconfirm
```
```
sudo git clone https://github.com/blackbird-package/claw.git /tmp/claw
```
```
sudo cp -fr /tmp/claw/pkg.tar.xz /usr/share/sddm/themes
```
```
cd /usr/share/sddm/themes
```
```
sudo tar -xf pkg.tar.xz 
```
#### configuration
```
sudo mkdir /etc/sddm.conf.d/
```
```
sudo mv /usr/share/sddm/themes/claw/sddm-default.conf /etc/sddm.conf.d/default.conf
```
```
sudo nvim /etc/sddm.conf.d/default.conf
```
change at `[Theme]`
```
Current=claw
```
#### service
```
sudo systemctl enable sddm
```
### apps
#### configuration

```
sudo usermod -aG wheel nama_user
```
```
sudo mkdir /opt/flat
```

```
sudo ln -sf /opt/flat /var/lib/flatpak
```

```
sudo flatpak override --filesystem=$HOME/.themes
```

```
sudo flatpak override --filesystem=$HOME/.icons
```
```
sudo flatpak override --env=GTK_THEME=flow
```
```
sudo flatpak override --env=ICON_THEME=eggs
```


### browser

#### instalation
```
sudo flatpak install --system  flathub org.mozilla.firefox
```
```
sudo flatpak install --system flathub com.google.Chrome
```
### development
#### instalation
```
sudo flatpak install --system flathub com.visualstudio.code
```

### finance
```
sudo flatpak install flathub fr.free.Homebank
```
### media player

#### installation

```
sudo pacman -S mpd mpc mpv yt-dlp --noconfirm
```
```
sudo flatpak install --system flathub de.wagnermartin.Plattenalbum
```
#### configuration
```
nvim .config/hypr/hyprland.conf
```
uncommenting
```
exec-once = /usr/bin/mpd --no-daemon 
```
### office tools

#### installation
```
sudo flatpak install --system flathub md.obsidian.Obsidian
```

```
sudo flatpak install --system flathub org.gnome.Calendar
```

```
sudo flatpak install --system flathub org.gnome.Evolution
```

```
sudo flatpak install --system flathub org.gnome.Calculator
```

### acces management

#### installation

```
sudo flatpak install --system flathub com.github.tchx84.Flatseal
```

### key tools

#### installation

```
sudo flatpak install --system flathub org.keepassxc.KeePassXC
```

```
cd .local/share/applications
```

```
rm -fr org.keepassxc.KeePassXC
```



## preparing


## chrooting

### step 1
```
git clone https://github.com/blackbird-package/level10.git /tmp/config
```

### step 2
```
cp -fr /tmp/config/* /
```
















