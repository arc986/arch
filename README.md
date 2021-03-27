# WIFI

iwctl

Status [device-name] connect [WIFI-name]

Status [device-name] connect-hidden [WIFI-name]



# disco

mount -o defaults,noatime /dev/sda4 /mnt



```
mkfs.vfat /dev/nvme0n1p1
mkfs.ext2 /dev/nvme0n1p2
mkswap /dev/nvme0n1p3
mkfs.f2fs /dev/nvme0n1p4
```



```
mount -t f2fs /dev/nvme0n1p4 /mnt
```

```
mkdir /mnt/boot;mount /dev/nvme0n1p2 /mnt/boot
```

```
mkdir /mnt/boot/efi;mount /dev/nvme0n1p1 /mnt/boot/efi
```

```
swapon /dev/nvme0n1p3
```



```
pacstrap /mnt base base-devel grub ntfs-3g gvfs efibootmgr htop openssh linux-zen linux-zen-headers linux-firmware vim intel-ucode networkmanager xfsprogs git fuse nvidia nvidia-utils nvidia-settings cuda nvidia-dkms lib32-nvidia-utils xorg-server xorg-xinit vulkan-icd-loader lib32-vulkan-icd-loader pulseaudio pulseaudio-alsa alsa-utils alsa-plugins alsa-lib ttf-liberation ttf-bitstream-vera ttf-dejavu ttf-droid ttf-freefont ttf-font-awesome ttf-ubuntu-font-family hunspell-es_pa terminus-font xorg-xset udev
```

```
pacstrap /mnt base base-devel grub f2fs-tools fuse ntfs-3g gvfs efibootmgr htop openssh linux-lts linux-lts-headers linux-firmware vim amd-ucode networkmanager xfsprogs git xf86-video-amdgpu vulkan-radeon lib32-mesa lib32-vulkan-radeon libva-mesa-driver lib32-libva-mesa-driver mesa-vdpau lib32-mesa-vdpau xorg-server xorg-xinit pulseaudio pulseaudio-alsa alsa-utils alsa-plugins alsa-lib ttf-liberation ttf-bitstream-vera ttf-dejavu ttf-droid ttf-freefont ttf-font-awesome ttf-ubuntu-font-family hunspell-es_pa hunspell-es_any terminus-font xorg-xset bluez udev fwupd bolt xf86-input-evdev xf86-input-synaptics
```

### Generar la tabla de particiones

```
genfstab -pU /mnt >> /mnt/etc/fstab
```

### Entramos del sistema virtual al real como root

```
arch-chroot /mnt

```

```
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
yay -S wd719x-firmware aic94xx-firmware upd72020x-fw 
systemctl enable laptop-mode-tools PC
systemctl enable bluetooth.service
systemctl enable systemd-resolved.service
systemctl enable NetworkManager
systemctl enable fstrim.timer
systemctl enable fstrim.service
systemctl mask systemd-rfkill.service
systemctl mask systemd-rfkill.socket
```



tlp

# red

[networkmanager](https://www.archlinux.org/packages/?name=networkmanager)



# kde

pacman -S plasma kde-system-meta kde-accessibility-meta kde-graphics-meta konsole kate kdeconnect kdenetwork-filesharing krdc telepathy-kde-meta kdeconnect zeroconf-ioslave xdg-user-dirs firefox firefox-i18n-es-mx 



pacman -S gnome-shell nautilus gnome-terminal gnome-control-center gdm firefox firefox-i18n-es-mx cups system-config-printer gnome-user-share gnome-remote-desktop rygel ghostscript libraw libwmf ocl-icd djvulibre gnome-firmware fwupd gnome-music gnome-mplayer gnome-power-manager gnome-books gnome-calculator gnome-calendar gnome-color-manager gnome-disk-utility gnome-documents gnome-menus gnome-passwordsafe gnome-phone-manager gnome-photos gnome-podcasts gnome-system-monitor gnome-screenshot xdg-user-dirs-gtk



```
systemctl enable laptop-mode.service
systemctl enable gdm.service
```