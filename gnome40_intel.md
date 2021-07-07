###### particiones propuestas
|Disposit.|Tamaño|Tipo|
|---|---|---|
|/dev/nvme0n1p1|8M|Sistema EFI|
|/dev/nvme0n1p2|512M|Sistema de ficheros de Linux|
|/dev/nvme0n1p3|MAX|Sistema de ficheros de Linux|

###### si *NO existen* las particiones creadas
```
mkfs.vfat /dev/nvme0n1p1
mkfs.ext2 /dev/nvme0n1p2
mkfs.f2fs /dev/nvme0n1p3
```

```
mount -t f2fs /dev/nvme0n1p3 /mnt
```

```
mkdir /mnt/boot;mount /dev/nvme0n1p2 /mnt/boot
```

```
mkdir /mnt/boot/efi;mount /dev/nvme0n1p1 /mnt/boot/efi
```
###### si *ya existen* las particiones creadas
```bash
mount -t f2fs /dev/nvme0n1p3 /mnt;mount /dev/nvme0n1p2 /mnt/boot;mount /dev/nvme0n1p1 /mnt/boot/efi;rm -Rf /mnt/;ls -Rla /mnt/
```
###### Instalación
```bash
pacstrap /mnt base base-devel grub efibootmgr linux-zen linux-zen-headers linux-firmware intel-ucode networkmanager f2fs-tools fuse pipewire-pulseaudio pipewire-alsa pipewire-jack pipewire bluez sudo zsh wget ntfs-3g openssh systemd-swap htop xf86-input-evdev ufw fzf ttf-dejavu ttf-font-awesome xdg-utils exa xf86-video-intel mesa mesa-libgl vulkan-intel acpi git hunspell-es_pa neovim firefox firefox-i18n-es-mx kitty flatpak virt-manager qemu vde2 ebtables dnsmasq bridge-utils openbsd-netcat dmidecode podman podman-compose yarn stylus go rust nodejs python-virtualenv gnome-control-center gdm gnome-shell gnome-power-manager gnome-menus gnome-screenshot xdg-user-dirs-gtk eog evince gvfs-google gedit gnome-tweaks gnome-color-manager gufw nautilus gnome-keyring gnome-bluetooth xorg-xwayland
```

```bash
genfstab -pU /mnt >> /mnt/etc/fstab
```

```bash
arch-chroot /mnt
```

```bash
export PCNAME=""
```

```bash
echo $PCNAME > /etc/hostname;
CAT >> hosts <<EOF
127.0.0.1 localhost $PCNAME
::1 localhost $PCNAME
EOF
```

```bash
ln -sf /usr/share/zoneinfo/America/Panama /etc/localtime
```

```bash
echo es_PA.UTF-8 UTF-8 >/etc/locale.gen;echo LANG=es_PA.UTF-8 >/etc/locale.conf;echo LANG=es_PA.UTF-8 >>/etc/environment;echo LC_TIME=C >>/etc/environment;echo LC_COLLATE=C >>/etc/environment;locale-gen
```

```bash
echo KEYMAP=la-latin1 > /etc/vconsole.conf
```

```bash
grub-install --efi-directory=/boot/efi --bootloader-id='Arch Linux' --target=x86_64-efi
```

```bash
sed -i 's/MODULES=()/MODULES=(f2fs fuse i915)/' /etc/mkinitcpio.conf
```

```bash
cat > /etc/modprobe.d/alsa-base.conf <<EOF
options snd_mia index=0
options snd_hda_intel index=1
EOF
```

```bash
cat > /etc/X11/xorg.conf.d/20-intel.conf <<EOF 
Section "Screen"
	Identifier "Screen"
	DefaultDepth 30
EndSection

Section "Device"
	Identifier  "Intel Graphics"
	Driver      "intel"
	Option      "DRI" "2"
EndSection
EOF
```

```bash
sed -i 's/# zram_/zram_/' /etc/systemd/swap.conf;sed -i 's/zram_enabled=0/zram_enabled=1/' /etc/systemd/swap.conf
```

```bash
passwd
```

```bash
export USERR=""
```

```bash
useradd -m -g users -G audio,lp,optical,storage,video,wheel,games,power,scanner,kvm,polkitd,libvirt -s /bin/zsh $USERR
passwd $USERR
```

```bash
sed -i 's/# %wheel ALL=(ALL) ALL/%wheel ALL=(ALL) ALL/' /etc/sudoers
```

```bash
systemctl enable systemd-resolved.service;systemctl enable NetworkManager;systemctl enable bluetooth.service;systemctl enable gdm.service;systemctl enable libvirtd.service;systemctl enable ufw.service;systemctl enable systemd-swap.service;systemctl enable upower.service;
```

```bash
su $USERR
```

```bash
wget -N https://raw.githubusercontent.com/arc986/zsh/main/.zsh{rc,env,_history} -P ~/
```

```bash
cd /tmp;git clone https://aur.archlinux.org/yay.git;cd yay;makepkg -si
```
```bash
yay -S upd72020x-fw nerd-fonts-dejavu-complete
```

```bash
exit
```

```bash
echo 'GRUB_DISABLE_OS_PROBER=false' >> /etc/default/grub
```


```bash
mkinitcpio -p linux-zen;grub-mkconfig -o /boot/grub/grub.cfg
```

```bash
exit;umount -R /mnt
```
```bash
reboot
```
