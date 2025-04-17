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
mkfs.btrfs -f -d single -m single -L "root" /dev/nvme0n1p2
```

```bash
mount -o noatime,compress=zstd:3,space_cache=v2,ssd,discard=async,autodefrag /dev/nvme0n1p2 /mnt
btrfs subvolume create /mnt/@root
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@snapshots
umount /mnt
mount -o noatime,compress=zstd:3,space_cache=v2,ssd,subvol=@root /dev/nvme0n1p2 /mnt
mkdir -p /mnt/{home}
mount -o noatime,compress=zstd:3,space_cache=v2,ssd,subvol=@home /dev/nvme0n1p2 /mnt/home
mkdir -p /mnt/boot/efi;mount /dev/nvme0n1p1 /mnt/boot/efi
```

###### Instalación minimal no dev Base clean 
```bash
pacstrap /mnt base grub efibootmgr linux-zen linux-zen-headers linux-firmware amd-ucode iwd wireless-regdb btrfs-progs fuse pipewire pipewire-pulse wireplumber pipewire-alsa mesa vulkan-radeon libva-mesa-driver mesa-vdpau lib32-mesa lib32-vulkan-radeon lib32-libva-mesa-driver lib32-mesa-vdpau amdgpu upower sof-firmware sudo ufw snapper util-linux plymouth xorg-xwayland wayland terminus-font xdg-user-dirs htop neovim hunspell-es_pa
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
sed -i 's/MODULES=()/MODULES=(btrfs amdgpu)/' /etc/mkinitcpio.conf
```

```bash
echo RADV_PERFTEST=aco >> /etc/environment;echo DXVK_ASYNC=1 >> /etc/environment
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
systemctl enable systemd-resolved.service;systemctl enable iwd.service;systemctl enable bluetooth.service;systemctl enable ufw.service;systemctl enable upower.service;sudo systemctl enable fstrim.timer;
```
```bash
echo 'GRUB_DISABLE_OS_PROBER=false' >> /etc/default/grub
```
agregar esto para AMD
quedando algo asi GRUB_CMDLINE_LINUX_DEFAULT=""
```bash
nvim /etc/default/grub

quiet splash loglevel=3 amdgpu.dpm=1 amdgpu.dc=1 amdgpu.noretry=0 amdgpu.vm_fragment_size=9 amdgpu.powerplay=1
```

```bash
mkinitcpio -p linux-zen;grub-mkconfig -o /boot/grub/grub.cfg
```

```bash
ufw default deny incoming;ufw enable
```
```bash
plymouth-set-default-theme -R bgrt
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
snapper -c root create --description "Backup antes de actualización"
snapper -c root list
snapper -c root rollback <ID>
```

