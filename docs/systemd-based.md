
# blackbird protocol level 20

## 1. prequsite
- gunakan ip static yang diberikan untuk terhubung kedalam jaringan internal
## 2. preparation

### networking 
```
nvim /etc/systemd/network/20-ethernet.network
```
Ganti `[IP]` denga ip address yang diberikan untuk perangkat anda
```
[Network]
Address=[IP]/24
Gateway=10.10.1.1
DNS=1.1.1.1 8.8.8.8
MulticastDNS=yes
```

### additional package
```
pacman -Syy
```
```
pacman -S git
```
### physical volume

**disk layout**
| disk | partition | type              | luks  | lvm   | label    |  format | mount           |
| ---- | --------- | ----------------- | ----- | ----- | -------- |  ------ | --------------- |
| 0    | 1         | efi               | false | false | boot     |  fat 32 | /boot           |
| 0    | 2         | linux file system | true  | false | keys     |  luks   | none            |
| 0    | 3         | linux filesystems | true  | true  | proc     |  luks   | see volume proc |
| 0    | 4         | linux home        | true  | true  | data     |  luks   | see volume data |

**notes**:
- gunakan fdisk, cfdisk, parted untuk membuat physical volume sesuai dengan guideline
- minimum size `partition 1` untuk perangkat dengan kartu grafis nvidia sebesar `1.5GB`

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

**minimum size**
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
## 3. Installation

**1. network**
```
pacstrap /mnt openssh ethtool iptables-nft firewalld --noconfirm
```
untuk perangkat support wireless tambahkan  
```
pacstrap /mnt iwd --noconfirm
```
**2. kernel**
```
pacstrap /mnt linux-hardened linux-firmware mkinitcpio base lvm2 btrfs-progs bubblewrap-suid --noconfirm
```
**3. devel**
```
pacstrap /mnt sudo debugedit fakeroot pkgconf bison gcc pcre flex wget make curl less --noconfirm
```
**4. audio system**
```
pacstrap /mnt pipewire pipewire-pulse pipewire-jack wireplumber pavucontrol sof-firmware mpd mpc --noconfirm
```
**5. graphic system**
```
pacstrap /mnt brightnessctl --noconfirm
```
**6. desktop**
```
pacstrap /mnt sddm uwsm hyprland hyprpicker hyprshot hypridle hyprlock hyprpolkitagent xdg-desktop-portal-hyprland qt5-wayland qt6-wayland wl-clipboard cliphist mailcap --noconfirm
```

**7. panel**
```
pacstrap /mnt mako waybar wofi --noconfirm
```

**8. file system**
```
pacstrap /mnt nautilus nautilus-image-converter sushi --noconfirm
```

**9. font system**
```
pacstrap /mnt ttf-jetbrains-mono-nerd ttf-droid --noconfirm
```

**10. terminal**
```
pacstrap /mnt kitty kitty-terminfo neovim --noconfirm
 ```

**11. password**
```
pacstrap /mnt gnome-keyring libsecret libpam-google-authenticator libpwquality cracklib polkit apparmor qrencode --noconfirm
```

**12. monitoring**
```
pacstrap /mnt prometheus prometheus-node-exporter btop --noconfirm
```

**13. performance**
```
pacstrap /mnt irqbalance tuned tuned-ppd --noconfirm
```

**14. backup**
```
pacstrap /mnt rsync grsync --noconfirm 
```

**15. hosting**
```
pacstrap /mnt go hugo nginx git --noconfirm
```

**16. ucode**  
pilih sesuai dengan jenis prosesor yang digunakan  
- intel
```
pacstrap /mnt intel-ucode --noconfirm
```
- amd
```
pacstrap /mnt amd-ucode --noconfirm
```

## 4. preconfing
### fstab
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
echo "tmpfs     					/tmp        		tmpfs   defaults,rw,nosuid,nodev,noexec,relatime,size=512M" >> /mnt/etc/fstab
```
### system
```
git clone https://github.com/blackbird-package/level20.git /mnt/opt/config
```
```
cp -fr /mnt/opt/config/mnt/* /mnt
```
```
cp -fr /mnt/opt/config/sysd/* /mnt
```
### network
 - ethernet
```
cp /etc/systemd/network/* /mnt/etc/systemd/network/
```
- wireless
```
mkdir -p /mnt/var/lib/iwd/ 
```
```
cp /var/lib/iwd/* /mnt/var/lib/iwd/
```

### chroot
```
arch-chroot /mnt
```

## 5. postconfig

### hostname 
```
echo 'nama_hostname' > /etc/hostname
```
### locatime 
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
systemctl enable systemd-timesyncd.service
```

### locale 
```
locale-gen
```
### users
#### system 
```
echo 'loki ALL=(ALL:ALL) ALL' >> /etc/sudoers
```
```
useradd -d /var/usr loki
```
```
mkdir -p /var/usr/.ssh/
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
chown -R loki:loki /var/usr
```
```
passwd loki
```
```
sudo chattr +i /var/usr/.ssh/authorized_keys
```

#### daemon 
```
useradd -d /var/games -u 50 -g games -r games
```
```
chown -R games:games /var/games
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

#### admin 
```
useradd -m nama_user
```
```
echo 'nama_user ALL=(ALL:ALL) ALL' >> /etc/sudoers.d/none
```
```
sudo chattr +i /etc/sudoers.d/none
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
#### root user
```
passwd -l root
```
### hids
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
cd .. && rm -fr aide-0.19.2
```
### booting
```
rm /boot/initramfs-linux-hardened*
```
for intel processor
```
mv /boot/intel-ucode.img /boot/vmlinuz-linux-hardened /boot/kernel
```
for amd processor
```
mv /boot/amd-ucode.img /boot/vmlinuz-linux-hardened /boot/kernel
```
```
bootctl --path=/boot install
```

### params
#### kernel
```
echo "rd.luks.uuid=$(blkid -s UUID -o value /dev/[proc physical partition name])root=/dev/proc/root" > /etc/cmdline.d/01-boot.conf
```
#### cryptab
```
echo "data UUID=$(blkid -s UUID -o value /dev/[proc physical partition name]) none" >> /etc/crypttab
```
#### uefi image
```
mkinitcpio -P
```

## 6. Application
```
mkdir -p /opt/flat
```
```
ln -sf /opt/flat /var/lib/flatpak
```
```
pacman /mnt flatpak gnome-software --noconfirm 
```
## 7. Service
```
systemctl enable systemd-timesyncd.service &&
systemctl enable tangd.socket &&
systemctl enable apparmor.service &&
systemctl enable sshd &&
systemctl enable update.timer  &&
systemctl enable prometheus.service &&
systemctl enable prometheus-node-exporter.service &&
systemctl enable irqbalance &&
systemctl enable tuned &&
systemctl enable tuned-ppd.service &&
systemctl enable systemd-networkd &&
systemctl enable systemd-resolved &&
systemctl enable aide.timer &&
systemctl enable sddm &&
systemctl enable nginx &&
systemctl enable --global gcr-ssh-agent.socket &&
systemctl enable --global hypridle.service &&
systemctl enable --global hyprpolkitagent &&
systemctl enable --global waybar &&
systemctl enable --global pipewire-pulse
```

## 8. finishing
```
exit
```
```
umount -R /mnt
```
```
reboot
```

## 9 firstboot

```
flatpak install --system -y --noninteractive flathub \
    org.mozilla.firefox \
    com.google.Chrome \
    com.visualstudio.code \
    de.wagnermartin.Plattenalbum \
    md.obsidian.Obsidian \
    org.gnome.Evolution \
    org.gnome.Calculator \
    com.github.tchx84.Flatseal \
    org.keepassxc.KeePassXC \
    org.telegram.desktop \
    org.gnome.World.Secrets \
    com.mongodb.Compass \
    io.beekeeperstudio.Studio \
    fr.free.Homebank
```
```
flatpak override --filesystem=$HOME/.themes
```
```
flatpak override --filesystem=$HOME/.icons
```
```
flatpak override --env=GTK_THEME=flow
```
```
flatpak override --env=ICON_THEME=eggs
```





































