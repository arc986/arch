```bash
connmanctl
> agent on
> enable wifi
> scan wifi
> services
> connect wifi_00904b963a4b_4a....b_managed_psk
Passphrase? xxxxxxxx
```

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
basestrap /mnt base base-devel grub zsh wget ntfs-3g efibootmgr htop openssh linux-zen linux-zen-headers linux-firmware amd-ucode f2fs-tools fuse pipewire-pulse pipewire-alsa pipewire-jack pipewire sudo xf86-video-amdgpu vulkan-radeon libva-mesa-driver mesa-vdpau  xf86-input-evdev xdg-utils openrc elogind-openrc ufw-openrc bluez-openrc connman-openrc exa wpa_supplicant dhclient fzf ttf-dejavu ttf-font-awesome vi neovim hunspell-es_pa kitty firefox firefox-i18n-es-mx git yarn stylus go rust nodejs python-virtualenv
```

```bash
fstabgen -U /mnt >> /mnt/etc/fstab
```

```bash
artix-chroot /mnt
```

```bash
export HOMBREPC=""
```

```bash
echo $HOMBREPC > /etc/hostname;echo "127.0.0.1 localhost $HOMBREPC\n::1 localhost $HOMBREPC" >> /etc/hosts
```

```bash
ln -sf /usr/share/zoneinfo/America/Panama /etc/localtime
```

```bash
echo 'es_PA.UTF-8 UTF-8' >/etc/locale.gen;echo 'LANG=es_PA.UTF-8' >/etc/locale.conf;echo 'LANG=es_PA.UTF-8\nLC_TIME=C\nLC_COLLATE=C' >/etc/environment;locale-gen;hwclock --systohc
```

```bash
echo KEYMAP=la-latin1 > /etc/vconsole.conf

nvim /etc/conf.d/hostname
hostname='myhostname'

nvim /etc/conf.d/keymaps
KEYMAP='la-latin1'
```
```bash
grub-install --efi-directory=/boot/efi --bootloader-id='ArtixLinux' --target=x86_64-efi
```

```bash
sed -i 's/MODULES=()/MODULES=(f2fs fuse amdgpu radeon)/' /etc/mkinitcpio.conf
```

```bash
echo 'options amdgpu si_support=1\noptions amdgpu cik_support=1'>/etc/modprobe.d/amdgpu.conf;echo 'options radeon si_support=0\noptions radeon cik_support=0'>/etc/modprobe.d/radeon.conf
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
passwd
```


```bash
export USERR=""
```

```bash
useradd -m -g users -G audio,lp,optical,storage,video,wheel,games,power,scanner -s /bin/zsh $USERR
passwd $USERR
```
```bash
sed -i 's/# %wheel ALL=(ALL) ALL/%wheel ALL=(ALL) ALL/' /etc/sudoers
```

```bash
rc-update add bluetoothd default
rc-update add ufw default
rc-update add connmand default
```

```bash
su $USERR
wget -N https://raw.githubusercontent.com/arc986/zsh/main/.zsh{rc,env,_history} -P ~/
cd /tmp;git clone https://aur.archlinux.org/yay.git;cd yay;makepkg -si
yay -S upd72020x-fw nerd-fonts-dejavu-complete
exit
```

```bash
echo 'GRUB_DISABLE_OS_PROBER=false' >> /etc/default/grub
```


```bash
mkinitcpio -p linux-zen;grub-mkconfig -o /boot/grub/grub.cfg
```

```bash
exit
umount -R /mnt
reboot
```
