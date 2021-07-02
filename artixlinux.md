|Disposit.|Tamaño|Tipo|
|---|---|---|
|/dev/nvme0n1p1|8M|Sistema EFI|
|/dev/nvme0n1p2|512M|Sistema de ficheros de Linux|
|/dev/nvme0n1p3|MAX|Sistema de ficheros de Linux|

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
```bash
mount -t f2fs /dev/nvme0n1p3 /mnt;mount /dev/nvme0n1p2 /mnt/boot;mount /dev/nvme0n1p1 /mnt/boot/efi
```

```bash
pacstrap /mnt base base-devel grub zsh wget ntfs-3g efibootmgr htop openssh linux-zen linux-zen-headers linux-firmware vim amd-ucode networkmanager f2fs-tools git fuse pulseaudio pipewire-alsa pipewire-jack pipewire hunspell-es_pa sudo xf86-video-amdgpu vulkan-radeon libva-mesa-driver mesa-vdpau neovim firefox firefox-i18n-es-mx xf86-input-evdev kitty systemd-swap xdg-utils ufw gufw fzf  flatpak ttf-dejavu ttf-font-awesome yarn stylus bluez gnome-bluetooth go rust nodejs python-virtualenv connman-openrc connman-gtk openrc elogind-openrc exa

gnome-control-center gdm gnome-shell gnome-power-manager gnome-menus gnome-screenshot nautilus gnome-keyring xdg-user-dirs-gtk eog evince gnome-color-manager gvfs-google gedit gnome-tweaks xorg-xwayland

virt-manager qemu vde2 ebtables dnsmasq bridge-utils openbsd-netcat dmidecode 

podman podman-compose 
```

```bash
genfstab -pU /mnt >> /mnt/etc/fstab
```

```bash
arch-chroot /mnt
```

```bash
export HOMBREPC=""
```

```bash
echo $HOMBREPC > /etc/hostname;echo "127.0.0.1 localhost $HOMBREPC" >> /etc/hosts;echo "::1 localhost $HOMBREPC" >> /etc/hosts
```

```bash
ln -sf /usr/share/zoneinfo/America/Panama /etc/localtime
```

```bash
echo es_PA.UTF-8 UTF-8 >/etc/locale.gen;echo LANG=es_PA.UTF-8 >/etc/locale.conf;echo LANG=es_PA.UTF-8 >>/etc/environment;echo LC_TIME=C >>/etc/environment;locale-gen
```

```bash
echo KEYMAP=us > /etc/vconsole.conf
OR
echo KEYMAP=la-latin1 > /etc/vconsole.conf
```
```bash
grub-install --efi-directory=/boot/efi --bootloader-id='Arch Linux' --target=x86_64-efi
```

```bash
nvim /etc/mkinitcpio.conf
MODULES=(f2fs fuse amdgpu radeon)
```

```bash
nvim /etc/modprobe.d/amdgpu.conf
options amdgpu si_support=1
options amdgpu cik_support=1
```

```bash
nvim /etc/modprobe.d/radeon.conf
options radeon si_support=0
options radeon cik_support=0
```

```bash
nvim /etc/X11/xorg.conf.d/20-amdgpu.conf
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
```

```bash
nvim /etc/systemd/swap.conf
zram_enabled=1
zram_size=$(( RAM_SIZE / 4 ))
zram_count=${NCPU}
zram_streams=${NCPU}
zram_alg=zstd
zram_prio=32767
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
visudo
%wheel ALL=(ALL) ALL
```

```bash
systemctl enable systemd-resolved.service
systemctl enable NetworkManager
systemctl enable bluetooth.service
systemctl enable gdm.service
systemctl enable libvirtd.service
systemctl enable ufw.service
systemctl enable systemd-swap.service
systemctl enable upower.service
```

```bash
su $USERR
cd /tmp;git clone https://aur.archlinux.org/yay.git;cd yay;makepkg -si
yay -S upd72020x-fw
yay -S nerd-fonts-dejavu-complete
exit
```

```bash
nvim /etc/pulse/default.pa
.ifexists module-echo-cancel.so
load-module module-echo-cancel aec_method=webrtc source_name=echocancel sink_name=echocancel1 format=s16le rate=48000 channels=2 channel_map=stereo
set-default-source echocancel
set-default-sink echocancel1
.endif
```

```bash
nvim /etc/default/grub
GRUB_DISABLE_OS_PROBER=false
```


```bash
mkinitcpio -p linux-lts;grub-mkconfig -o /boot/grub/grub.cfg
```

```bash
exit
umount -R /mnt
reboot
```
