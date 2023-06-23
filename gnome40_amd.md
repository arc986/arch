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

###### Instalación 2023
```bash
pacstrap /mnt base base-devel grub efibootmgr linux-zen linux-zen-headers linux-firmware amd-ucode networkmanager f2fs-tools fuse pipewire-pulse pipewire-alsa pipewire-jack pipewire xdg-desktop-portal-gtk WirePlumber pipewire-audio bluez sudo zsh ntfs-3g openssh systemd-swap xf86-video-amdgpu vulkan-radeon libva-mesa-driver mesa-vdpau neovim xf86-input-evdev ufw flatpak htop wget fzf xdg-utils exa gdm eog gufw evince nautilus epiphany ttf-dejavu gnome-shell gts-libav gnome-menus gvfs-google gnome-music gnome-tweaks wl-clipboard gnome-console gnome-keyring xorg-xwayland gnome-calendar gnome-software hunspell-es_pa gnome-bluetooth gnome-calculator gnome-screenshot ttf-font-awesome gnome-text-editor xdg-user-dirs-gtk gnome-power-manager gnome-color-manager gnome-system-monitor gnome-control-center git podman podman-compose python-virtualenv rust gnome-boxes 
```

###### Instalación minimal no dev 2023
```bash
pacstrap /mnt base amd-ucode bluez efibootmgr eog epiphany evince f2fs-tools gdm gnome-bluetooth gnome-color-manager gnome-console gnome-control-center gnome-keyring gnome-menus gnome-power-manager gnome-shell gnome-system-monitor gnome-tweaks grub gufw gvfs-google hunspell-es_pa linux-firmware linux-zen linux-zen-headers nautilus networkmanager ntfs-3g pipewire pipewire-audio pipewire-alsa pipewire-jack WirePlumber pipewire-pulse sudo systemd-swap ttf-font-awesome ufw vulkan-radeon wl-clipboard xdg-desktop-portal-gtk xf86-video-amdgpu xorg-xwayland zsh gnome-text-editor gnome-screenshot gnome-calculator gnome-software gts-libav xdg-utils flatpak neovim xorg-xwayland gnome-calendar ttf-font-awesome wget fzf exa
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
cat >> /etc/hosts <<EOF
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
sed -i 's/MODULES=()/MODULES=(f2fs fuse amdgpu radeon)/' /etc/mkinitcpio.conf
```

```bash
cat > /etc/modprobe.d/amdgpu.conf <<EOF
options amdgpu si_support=1
options amdgpu cik_support=1
EOF
```

```bash
cat > /etc/modprobe.d/radeon.conf <<EOF
options radeon si_support=0
options radeon cik_support=0
EOF
```

```bash
cat > /etc/X11/xorg.conf.d/20-amdgpu.conf <<EOF 
Section "Screen"
	Identifier "Screen"
	DefaultDepth 30
EndSection

Section "Device"
    Identifier "AMD"
    Driver "amdgpu"
	Option "TearFree" "true"
	Option "DRI" "3"
EndSection
EOF
```

```bash
sed -i 's/#zram_/zram_/' /etc/systemd/swap.conf;sed -i 's/zram_enabled=0/zram_enabled=1/' /etc/systemd/swap.conf
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
systemctl enable systemd-resolved.service;systemctl enable NetworkManager;systemctl enable bluetooth.service;systemctl enable gdm.service;systemctl enable ufw.service;systemctl enable systemd-swap.service;systemctl enable upower.service;
```

```bash
su $USERR
```

```bash
wget -N https://raw.githubusercontent.com/arc986/zsh/main/.zsh{rc,env,_history} -P ~/
```

```bash
cd /tmp;git clone https://aur.archlinux.org/paru.git;cd paru;makepkg -si
```
```bash
paru -S upd72020x-fw nerd-fonts-dejavu-complete
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
