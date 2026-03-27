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
| Disposit.        | Tamaño | Tipo             |
| :--------------- | :----- | :--------------- |
| /dev/nvme0n1p1 | 512M   | EFI System       |
| /dev/nvme0n1p2 | MAX    | Linux filesystem |

# --- Variables ---
```bash
export DISK="/dev/nvme0n1"
export EFI="${DISK}p1"
export ROOT="${DISK}p2"
```
# --- Particionado ---
```bash
sgdisk --zap-all $DISK
sgdisk -n1:0:+512M -t1:ef00 $DISK
sgdisk -n2:0:0 -t2:8300 $DISK

mkfs.fat -F32 $EFI
mkfs.btrfs -L ArchRoot $ROOT
```


###### si *NO existen* las particiones creadas
```bash
mkfs.vfat -F 32 $EFI

mkfs.btrfs -f -d single -m single -L "root" $ROOT
```

# --- Subvolúmenes Btrfs ---
```bash
mount $ROOT /mnt
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@log 
btrfs subvolume create /mnt/@pkg 
btrfs subvolume create /mnt/@tmp 
btrfs subvolume create /mnt/@snapshots
btrfs subvolume create /mnt/@srv
umount /mnt
```

# --- Montajes optimizados ---
```bash
mount -o noatime,compress=zstd:3,ssd,discard=async,space_cache=v2,subvol=@ $ROOT /mnt
mkdir -p /mnt/{home,var/log,var/cache/pacman/pkg,var/tmp,.snapshots,srv,boot/efi}
mount -o noatime,compress=zstd:3,ssd,discard=async,space_cache=v2,subvol=@home $ROOT /mnt/home
mount -o noatime,compress=no,ssd,discard=async,space_cache=v2,subvol=@log $ROOT /mnt/var/log
mount -o noatime,compress=no,ssd,discard=async,space_cache=v2,subvol=@pkg $ROOT /mnt/var/cache/pacman/pkg
mount -o noatime,compress=no,ssd,discard=async,space_cache=v2,subvol=@tmp $ROOT /mnt/var/tmp
mount -o noatime,compress=zstd:9,ssd,discard=async,space_cache=v2,subvol=@snapshots $ROOT /mnt/.snapshots
mount -o noatime,compress=zstd:1,ssd,discard=async,space_cache=v2,subvol=@srv $ROOT /mnt/srv
mount $EFI /mnt/boot/efi
```

# --- Instalación base con firmware directo ---
```bash
pacstrap -K /mnt base linux-zen amd-ucode linux-firmware-whence linux-firmware-amdgpu linux-firmware-intel btrfs-progs pipewire wireplumber bluez mesa vulkan-radeon libva-mesa-driver snapper wayland neovim zram-generator hunspell-es_pa fontconfig power-profiles-daemon upower terminus-font sudo sof-firmware nftables networkmanager iwd
```

# --- Generar fstab ---
```bash
genfstab -pU /mnt >> /mnt/etc/fstab
```

# --- Configuración inicial ---

```bash
arch-chroot /mnt
```

# --- Variables ---
```bash
export HOSTNAME="arch-dell"
export USERNAME="por definir"
export LOCALE="es_PA.UTF-8"
export TIMEZONE="America/Panama"
export KEYMAPS="la-latin1"
```

```bash
echo $HOSTNAME > /etc/hostname;
cat >> /etc/hosts <<EOF
127.0.0.1 localhost $HOSTNAME
::1 localhost $HOSTNAME
EOF
```

```bash
ln -sf /usr/share/zoneinfo/$TIMEZONE /etc/localtime
```

```bash
echo $LOCALE UTF-8 >/etc/locale.gen;echo LANG=$LOCALE >/etc/locale.conf;echo LANG=$LOCALE >>/etc/environment;echo LC_TIME=C >>/etc/environment;echo LC_COLLATE=C >>/etc/environment;locale-gen
```

```bash
echo KEYMAP=$KEYMAPS > /etc/vconsole.conf;echo FONT=ter-132n >> /etc/vconsole.conf
```

## Bootloader
```bash
bootctl install
```
```bash
cat > /boot/loader/entries/arch.conf <<EOC
title   Arch Linux
linux   /vmlinuz-linux-zen
initrd  /amd-ucode.img
initrd  /initramfs-linux-zen.img
options root=UUID=$(blkid -s UUID -o value $ROOT) rw rootflags=subvol=@ amd_pstate=active nowatchdog mitigations=off
EOC
```

```bash
echo "%wheel ALL=(ALL:ALL) ALL" > /etc/sudoers.d/10-wheel
chmod 0440 /etc/sudoers.d/10-wheel
```
```bash
echo "$USERNAME:100000:65536" >> /etc/subuid
echo "$USERNAME:100000:65536" >> /etc/subgid
```

```bash
nvim /etc/systemd/logind.conf
NAutoVTs=2
```

```bash
nvim /etc/systemd/zram-generator.conf
[zram0]
zram-size = min(ram / 2, 8192)
compression-algorithm = zstd
```

```bash
nvim /etc/sysctl.d/99-rendimiento.conf
vm.swappiness = 10
vm.vfs_cache_pressure = 50
vm.dirty_ratio = 10
vm.dirty_background_ratio = 5
```

```bash
mkdir -p /etc/NetworkManager/conf.d
cat > /etc/NetworkManager/conf.d/wifi_backend.conf <<EOF
[device]
wifi.backend=iwd
EOF
```

```bash
nvim /etc/systemd/journald.conf
[Journal]
Storage=volatile
RuntimeMaxUse=15M
```

```bash
#opcional
nvim /etc/systemd/coredump.conf
[Coredump]
Storage=none
ProcessSizeMax=0
```

```bash
cat > /etc/nftables.conf <<EOF
flush ruleset

table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;
        
        # Aceptar tráfico localhost
        iif "lo" accept
        
        # Aceptar tráfico de conexiones ya establecidas
        ct state established,related accept
        
        # Descomentar si necesitas ping (ICMP)
        # ip protocol icmp accept
        # ip6 nexthdr icmpv6 accept
    }

    chain forward {
        type filter hook forward priority 0; policy drop;
    }

    chain output {
        type filter hook output priority 0; policy accept;
    }
}
EOF
```

```bash
#opcional
systemctl disable man-db.timer #Actualiza el índice de las páginas man diariamente
systemctl disable shadow.timer #Hace copias de seguridad diarias de tus archivos de contraseñas
```

```bash
sed -i 's/MODULES=()/MODULES=(amdgpu)/' /etc/mkinitcpio.conf

nvim /etc/mkinitcpio.conf
HOOKS=(systemd autodetect microcode modconf kms keyboard sd-vconsole block filesystems)
COMPRESSION="zstd"

mkinitcpio -P
```

```bash
passwd
```

```bash
useradd -m -g users -G wheel -s /bin/bash $USERNAME
passwd $USERNAME
```

```bash
systemctl enable bluetooth.service
systemctl enable upower.service
systemctl enable NetworkManager.service
systemctl enable power-profiles-daemon.service
systemctl enable nftables.service
```

```bash
# Eliminar el hinting completo si lo activaste
rm -f /etc/fonts/conf.d/10-hinting-full.conf
rm -f /etc/fonts/conf.d/10-hinting-slight.conf
rm -f /etc/fonts/conf.d/11-lcdfilter-default.conf
```

```bash
cat > /etc/fonts/local.conf <<EOF 
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
  
  <match target="font">
    <edit name="antialias" mode="assign">
      <bool>true</bool>
    </edit>
  </match>

  <match target="font">
    <edit name="rgba" mode="assign">
      <const>none</const>
    </edit>
  </match>

  <match target="font">
    <edit name="lcdfilter" mode="assign">
      <const>none</const>
    </edit>
  </match>

  <match target="font">
    <edit name="hinting" mode="assign">
      <bool>true</bool>
    </edit>
    <edit name="hintstyle" mode="assign">
      <const>hintslight</const>
    </edit>
  </match>

  <match target="font">
    <edit name="autohint" mode="assign">
      <bool>false</bool>
    </edit>
    <edit name="embeddedbitmap" mode="assign">
      <bool>false</bool>
    </edit>
  </match>

</fontconfig>
EOF
```

```bash
cat >> /etc/environment <<EOF
# Forzar Wayland nativo en Firefox
MOZ_ENABLE_WAYLAND=1

# Forzar Wayland nativo en aplicaciones Electron (VSCode, Obsidian, etc.)
ELECTRON_OZONE_PLATFORM_HINT=wayland

# Forzar Wayland en aplicaciones Qt, con fallback a xcb si fallan
QT_QPA_PLATFORM=wayland;xcb
EOF
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

```bash
sudo umount /.snapshots
sudo rm -Rf /.snapshots
sudo snapper -c root create-config /
reboot
sudo snapper -c root list
sudo snapper -c root create --description "base"

sudo snapper -c root undochange 1..0
sudo snapper delete 1

```