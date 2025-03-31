| Paso | Comando / Acción |
|------|------------------|
| 1. Verificar interfaces de red disponibles | `ip link` |
| 2. Habilitar interfaz Wi-Fi | `ip link set <INTERFACE> up` (Sustituir `<INTERFACE>` por el nombre de la interfaz, por ejemplo, `wlan0` o `wlp3s0`) |
| 3. Buscar redes disponibles | `iwctl station <INTERFACE> scan` (Reemplazar `<INTERFACE>` por el nombre de la interfaz, por ejemplo, `wlan0`) |
| 4. Ver redes disponibles | `iwctl station <INTERFACE> get-networks` |
| 5. Conectar a una red Wi-Fi | `iwctl station <INTERFACE> connect <SSID>` (Reemplazar `<SSID>` por el nombre de tu red Wi-Fi) |
| 6. Verificar la conexión | `ping -c 3 google.com` |
| 7. Si se requiere contraseña | Durante el paso 5, se te pedirá que ingreses la contraseña de la red Wi-Fi. |

###### particiones propuestas
|Disposit.|Tamaño|Tipo|
|---|---|---|
|/dev/nvme0n1p1|8M|Sistema EFI|
|/dev/nvme0n1p2|MAX|Sistema de ficheros de Linux|

###### si *NO existen* las particiones creadas
```bash
mkfs.vfat /dev/nvme0n1p1
```
```bash
mkfs.btrfs -f -d single -m single -O compression=lz4,space_cache=v2,block_size=4096,ssd -L "root" /dev/nvme0n1p2
```

```bash
mount -o noatime,compress=lz4,space_cache=v2,ssd,discard=async,autodefrag /dev/nvme0n1p2 /mnt
mount -o noatime,compress=lz4,space_cache=v2,ssd,discard=async,autodefrag /dev/nvme0n1p2 /mnt
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@var
umount /mnt
mount -o noatime,compress=lz4,space_cache=v2,ssd,subvol=@ /dev/nvme0n1p2 /mnt
mkdir -p /mnt/{home,var}
mount -o noatime,compress=lz4,space_cache=v2,ssd,subvol=@home /dev/nvme0n1p2 /mnt/home
mount -o noatime,compress=lz4,space_cache=v2,ssd,subvol=@var /dev/nvme0n1p2 /mnt/var
```

```bash
mkdir -p /mnt/boot/efi;mount /dev/nvme0n1p1 /mnt/boot/efi
```


###### Instalación minimal no dev 2025 Gnome
```bash
pacstrap /mnt base grub efibootmgr linux-zen linux-zen-headers linux-firmware amd-ucode networkmanager btrfs-progs f2fs-tools fuse pipewire pipewire-pulse wireplumber pipewire-alsa sudo ufw mesa vulkan-radeon libva-mesa-driver mesa-vdpau upower terminus-font neovim htop cups cups-pdf amdgpu hunspell-es_pa tlp xorg-server xf86-video-amdgpu fail2ban timeshift showtime simple-scan gnome-disk-utility gnome-bluetooth gnome-snapshot system-config-printer decibels xdg-user-dirs-gtk gdm gnome-shell gnome-control-center nautilus loupe evince console xorg-xwayland firefox firefox-i18n-es-es
```

###### Instalación minimal no dev 2025 Kde
```bash
pacstrap /mnt base grub efibootmgr linux-zen linux-zen-headers linux-firmware amd-ucode networkmanager btrfs-progs f2fs-tools fuse pipewire pipewire-pulse wireplumber pipewire-alsa sudo ufw mesa vulkan-radeon libva-mesa-driver mesa-vdpau upower terminus-font neovim htop cups cups-pdf amdgpu hunspell-es_pa tlp xorg-server xf86-video-amdgpu fail2ban timeshift kaffeine skanlite kdepartitionmanager bluedevil spectacle print-manager elisa xdg-user-dirs sddm plasma-desktop systemsettings dolphin gwenview okular konsole ark xorg-xwayland firefox firefox-i18n-es-es
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
echo KEYMAP=la-latin1 > /etc/vconsole.conf;echo FONT=ter-132n >> /etc/vconsole.conf
```

```bash
grub-install --efi-directory=/boot/efi --bootloader-id='Arch Linux' --target=x86_64-efi
```

```bash
sed -i 's/MODULES=()/MODULES=(f2fs btrfs amdgpu)/' /etc/mkinitcpio.conf
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
	#DefaultDepth 30
	DefaultDepth 24
EndSection

Section "Device"
    Identifier "AMD"
    Driver "amdgpu"
    Option "TearFree" "true"            # Evitar el tearing
    Option "DRI" "3"                   # Aceleración gráfica completa con DRI3
    Option "SwapbuffersWait" "true"    # Habilitar V-Sync (sincronización vertical)
    Option "ForceLowPowerMode" "true"  # Activar el modo de bajo consumo
    Option "DynamicPowerManagement" "true"   # Habilitar DPM
    Option "FBC" "true"                # Habilitar Framebuffer Compression
    Option "PowerPlay" "true"          # Activar características avanzadas de ahorro de energía
    Option "AccelMethod" "glamor"      # Mejor rendimiento con OpenGL para reducir carga
    Option "VariableRefresh" "true"   # Activar soporte para FreeSync con el Samsung G8
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
useradd -m -g users -G audio,lp,optical,storage,video,wheel,games,power,scanner,polkitd -s /bin/bash $USERR
passwd $USERR
```

```bash
EDITOR=nvim visudo
```

```bash
systemctl enable systemd-resolved.service;systemctl enable NetworkManager;systemctl enable bluetooth.service;systemctl enable ufw.service;systemctl enable tlp;systemctl enable upower.service;systemctl enable fail2ban;
```
gnome
```bash
systemctl enable gdm.service
```
kde
```bash
systemctl enable sddm.service
```

```bash
echo 'GRUB_DISABLE_OS_PROBER=false' >> /etc/default/grub
```
agregar esto para AMD
quedando algo asi GRUB_CMDLINE_LINUX_DEFAULT="quiet amdgpu.dc=1 amdgpu.gpu_sched=1"
```bash
nvim /etc/default/grub
```
```bash
amdgpu.dpm=1 amdgpu.dc=1 amdgpu.gpu_sched=1 amdgpu.ppfeaturemask=0xfffd7fff
```

```bash
mkinitcpio -p linux-zen;grub-mkconfig -o /boot/grub/grub.cfg
```

```bash
ufw default deny incoming;ufw enable
```

```bash
exit
```
```bash
umount -R /mnt
```
```bash
reboot
```
