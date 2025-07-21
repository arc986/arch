| Paso | Comando / Acción |
|------|------------------|
| 1. Verificar interfaces de red disponibles | `ip link` |
| 2. Habilitar interfaz Wi-Fi | `ip link set <INTERFACE> up` (Sustituir `<INTERFACE>` por el nombre de la interfaz, por ejemplo, `wlan0` o `wlp3s0`) |
| 3. Buscar redes disponibles | `iwctl station <INTERFACE> scan` (Reemplazar `<INTERFACE>` por el nombre de la interfaz, por ejemplo, `wlan0`) |
| 4. Ver redes disponibles | `iwctl station <INTERFACE> get-networks` |
| 5. Conectar a una red Wi-Fi | `iwctl station <INTERFACE> connect <SSID>` (Reemplazar `<SSID>` por el nombre de tu red Wi-Fi) |
| 6. Verificar la conexión | `ping -c 3 google.com` |
| 7. Si se requiere contraseña | Durante el paso 5, se te pedirá que ingreses la contraseña de la red Wi-Fi. |

# Particiones propuestas
|Disposit.|Tamaño|Tipo|
|---|---|---|
|/dev/nvme0n1p1|25M|Sistema EFI|
|/dev/nvme0n1p2|MAX|Sistema de ficheros de Linux|

# Formatear particiones creadas
```bash
mkfs.vfat /dev/nvme0n1p1
```
```bash
mkfs.btrfs -f -d single -m single -L "root" /dev/nvme0n1p2
```

# BTRFS Particiones y volumenes
```bash
mount /dev/nvme0n1p2 /mnt
```
```bash
btrfs subvolume create /mnt/@;btrfs subvolume create /mnt/@home;btrfs subvolume create /mnt/@snapshots
```
```bash
umount /mnt
```
```bash
mount -o noatime,compress=zstd:3,space_cache=v2,ssd,subvol=@ /dev/nvme0n1p2 /mnt
```
```bash
mkdir -p /mnt/{home,boot/efi,.snapshots}
```
```bash
mount -o noatime,compress=zstd:3,space_cache=v2,ssd,subvol=@home /dev/nvme0n1p2 /mnt/home
```
```bash
mount -o noatime,compress=zstd:7,space_cache=v2,ssd,subvol=@snapshots /dev/nvme0n1p2 /mnt/.snapshots
```
```bash
mount /dev/nvme0n1p1 /mnt/boot/efi
```

# Instalación minimal no dev Base clean 
```bash
REPO=https://repo-default.voidlinux.org/current
XBPS_ARCH=x86_64 xbps-install -S -r /mnt -R "$REPO" base-system grub efibootmgr linux linux-firmware amd-ucode iwd btrfs-progs power-profiles-daemon pipewire pipewire-pulse wireplumber pipewire-alsa pipewire-jack bluez mesa vulkan-radeon libva-mesa-driver mesa-vdpau upower opendoas ufw snapper wayland terminus-font xdg-user-dirs htop neovim hunspell-es_pa fontconfig
```

```bash
xgenfstab -U /mnt > /mnt/etc/fstab
```

```bash
xchroot /mnt /bin/bash
```

```bash
export PCNAME=""
```

```bash
echo $PCNAME > /etc/hostname
```

```bash
cat >> /etc/hosts <<EOF
127.0.0.1 localhost $PCNAME
::1 localhost $PCNAME
EOF
```

```bash
ln -sf /usr/share/zoneinfo/America/Panama /etc/localtime
```

```bash
echo es_PA.UTF-8 UTF-8 >/etc/default/libc-locales;xbps-reconfigure -f glibc-locales
```

```bash
echo KEYMAP=la-latin1 > /etc/vconsole.conf;echo FONT=ter-132n >> /etc/vconsole.conf
```

```bash
grub-install --efi-directory=/boot/efi --bootloader-id="Void" --target=x86_64-efi
```

```bash
passwd
```

```bash
export USERR=""
```

```bash
useradd -m -g users -G audio,lp,optical,storage,video,wheel,games,power,scanner,polkitd -s /bin/bash $USERR
```

```bash
passwd $USERR
```

```bash
echo 'permit :wheel' > /etc/doas.conf;chmod 0400 /etc/doas.conf
```

```bash
sudo rm /etc/resolv.conf;echo -e 'nameserver 1.1.1.1\nnameserver 8.8.8.8' | sudo tee /etc/resolv.conf > /dev/null
```

```bash
cat > /etc/iwd/main.conf <<EOF 
[General]
EnableNetworkConfiguration=true
EOF
```

```bash
ln -s /etc/sv/iwd       /var/service/
ln -s /etc/sv/bluetooth /var/service/
ln -s /etc/sv/ufw       /var/service/
ln -s /etc/sv/snapper   /var/service/
ln -s /etc/sv/power-profiles-daemon /var/service/
echo "0 2 * * * root fstrim -v /" > /etc/cron.d/fstrim
```

```bash
echo 'GRUB_DISABLE_OS_PROBER=false' >> /etc/default/grub
```

```bash
mkdir -p /etc/dracut.conf.d
cat > /etc/dracut.conf.d/btrfs-amdgpu.conf <<EOF
add_dracutmodules+=" btrfs amdgpu "
compress="zstd"
EOF
```

```bash
KVER=$(ls /lib/modules)
dracut --force --kver "$KVER"
```

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```


```bash
sudo ln -s /usr/share/fontconfig/conf.avail/70-no-bitmaps.conf /etc/fonts/conf.d/
```
```bash
sudo ln -s /usr/share/fontconfig/conf.avail/10-hinting-full.conf /etc/fonts/conf.d/
```
```bash
sudo rm /etc/fonts/conf.d/10-hinting-slight.conf 
```

```bash
cat > /etc/fonts/local.conf <<EOF 
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
  
  <!-- Activar subpixel rendering -->
  <match target="font">
    <edit name="rgba" mode="assign">
      <const>rgb</const>
    </edit>
  </match>

  <!-- Activar antialiasing -->
  <match target="font">
    <edit name="antialias" mode="assign">
      <bool>true</bool>
    </edit>
  </match>

  <!-- Aplicar hinting de alta calidad -->
  <match target="font">
    <edit name="hintstyle" mode="assign">
      <const>hintfull</const>
    </edit>
  </match>

  <!-- Activar autohint para una mejor definición -->
  <match target="font">
    <edit name="autohint" mode="assign">
      <bool>true</bool>
    </edit>
  </match>

  <!-- Desactivar mapas de bits embebidos -->
  <match target="font">
    <edit name="embeddedbitmap" mode="assign">
      <bool>false</bool>
    </edit>
  </match>

</fontconfig>
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
ufw default deny incoming;ufw enable
```
```bash
sudo umount /.snapshots
```
```bash
sudo rm -Rf /.snapshots
```
```bash
sudo snapper -c root create-config /
```
```bash
reboot
```

# Manejo de control de verciones de BTRFS
```bash
sudo snapper -c root list
sudo snapper -c root create --description "base"

sudo snapper -c root undochange 1..0
sudo snapper delete 1
```
