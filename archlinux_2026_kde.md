## Conexion Wi-Fi (Live USB)

| Paso | Comando / Accion |
|------|------------------|
| 1. Verificar interfaces de red disponibles | `ip link` |
| 2. Habilitar interfaz Wi-Fi | `ip link set <INTERFACE> up` (Sustituir `<INTERFACE>` por el nombre de la interfaz, por ejemplo, `wlan0` o `wlp3s0`) |
| 3. Buscar redes disponibles | `iwctl station <INTERFACE> scan` (Reemplazar `<INTERFACE>` por el nombre de la interfaz, por ejemplo, `wlan0`) |
| 4. Ver redes disponibles | `iwctl station <INTERFACE> get-networks` |
| 5. Conectar a una red Wi-Fi | `iwctl station <INTERFACE> connect <SSID>` (Reemplazar `<SSID>` por el nombre de tu red Wi-Fi) |
| 6. Verificar la conexion | `ping -c 3 google.com` |
| 7. Si se requiere contrasena | Durante el paso 5, se te pedira que ingreses la contrasena de la red Wi-Fi. |

## Esquema de Particiones

###### particiones propuestas
| Disposit.        | Tamano | Tipo             |
| :--------------- | :----- | :--------------- |
| /dev/nvme0n1p1 | 512M   | EFI System       |
| /dev/nvme0n1p2 | MAX    | Linux filesystem |

## Variables
```bash
export DISK="/dev/nvme0n1"
export EFI="${DISK}p1"
export ROOT="${DISK}p2"
```

## Particionado
```bash
sgdisk --zap-all $DISK
sgdisk -n1:0:+512M -t1:ef00 $DISK
sgdisk -n2:0:0 -t2:8300 $DISK

mkfs.vfat -F 32 $EFI
mkfs.btrfs -f -d single -m single -L "ArchRoot" $ROOT
```

## Subvolumenes Btrfs
```bash
mount $ROOT /mnt
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@log
btrfs subvolume create /mnt/@pkg
btrfs subvolume create /mnt/@tmp
btrfs subvolume create /mnt/@snapshots
btrfs subvolume create /mnt/@srv
btrfs subvolume create /mnt/@containers
btrfs subvolume create /mnt/@libvirt
umount /mnt
```

## Montajes optimizados
```bash
mount -o noatime,compress=zstd:3,ssd,discard=async,space_cache=v2,commit=60,subvol=@ $ROOT /mnt
mkdir -p /mnt/{home,var/log,var/cache/pacman/pkg,var/tmp,.snapshots,srv,boot,var/lib/{containers,libvirt}}
mount -o noatime,compress=zstd:3,ssd,discard=async,space_cache=v2,commit=60,subvol=@home $ROOT /mnt/home
mount -o noatime,compress=no,ssd,discard=async,space_cache=v2,commit=60,subvol=@log $ROOT /mnt/var/log
mount -o noatime,compress=no,ssd,discard=async,space_cache=v2,commit=60,subvol=@pkg $ROOT /mnt/var/cache/pacman/pkg
mount -o noatime,compress=no,ssd,discard=async,space_cache=v2,commit=60,subvol=@tmp $ROOT /mnt/var/tmp
mount -o noatime,compress=zstd:9,ssd,discard=async,space_cache=v2,commit=60,subvol=@snapshots $ROOT /mnt/.snapshots
mount -o noatime,compress=zstd:1,ssd,discard=async,space_cache=v2,commit=60,subvol=@srv $ROOT /mnt/srv
mount -o noatime,nodatacow,ssd,discard=async,commit=60,subvol=@containers $ROOT /mnt/var/lib/containers
mount -o noatime,nodatacow,ssd,discard=async,commit=60,subvol=@libvirt $ROOT /mnt/var/lib/libvirt
mount $EFI /mnt/boot

chattr +C /mnt/var/lib/containers
chattr +C /mnt/var/lib/libvirt
```

## Instalacion base
```bash
pacstrap -K /mnt base linux-zen amd-ucode linux-firmware-whence linux-firmware-amdgpu linux-firmware-intel btrfs-progs pipewire wireplumber bluez mesa vulkan-radeon libva-mesa-driver snapper wayland neovim zram-generator hunspell-es_pa fontconfig power-profiles-daemon upower terminus-font sudo sof-firmware nftables networkmanager iwd inter-font ttf-jetbrains-mono noto-fonts noto-fonts-cjk noto-fonts-emoji
```

## Generar fstab
```bash
genfstab -pU /mnt >> /mnt/etc/fstab
```

## Chroot

```bash
arch-chroot /mnt
```

### Variables
```bash
export HOSTNAME="KL86"
export USERNAME="mihatra"
export LOCALE="es_PA.UTF-8"
export TIMEZONE="America/Panama"
export KEYMAPS="la-latin1"
export DISK="/dev/nvme0n1"
export ROOT="${DISK}p2"
```

### Identidad del sistema

```bash
echo $HOSTNAME > /etc/hostname;
cat >> /etc/hosts <<EOF
127.0.0.1 localhost $HOSTNAME
::1 localhost $HOSTNAME
EOF
```

```bash
ln -sf /usr/share/zoneinfo/$TIMEZONE /etc/localtime;
hwclock --systohc
```

```bash
echo $LOCALE UTF-8 >/etc/locale.gen;echo LANG=$LOCALE >/etc/locale.conf;echo LANG=$LOCALE >>/etc/environment;echo LC_TIME=C >>/etc/environment;echo LC_COLLATE=C >>/etc/environment;locale-gen
```

```bash
echo KEYMAP=$KEYMAPS > /etc/vconsole.conf;echo FONT=ter-132n >> /etc/vconsole.conf
```

### Usuarios y seguridad

```bash
passwd
```

```bash
useradd -m -g users -G wheel -s /bin/bash $USERNAME;
passwd $USERNAME
```

```bash
echo "%wheel ALL=(ALL:ALL) ALL" > /etc/sudoers.d/10-wheel;
chmod 0440 /etc/sudoers.d/10-wheel
```

```bash
echo "$USERNAME:100000:65536" >> /etc/subuid;
echo "$USERNAME:100000:65536" >> /etc/subgid
```

### Bootloader y kernel

```bash
bootctl install
```

```bash
mkdir -p /boot/loader/entries/
```

```bash
cat > /boot/loader/loader.conf <<EOC
default arch.conf
timeout 2
console-mode max
editor no
EOC
```

```bash
cat > /boot/loader/entries/arch.conf <<EOF
title   Arch Linux (Zen MinMax)
linux   /vmlinuz-linux-zen
initrd  /amd-ucode.img
initrd  /initramfs-linux-zen.img
options root=UUID=$(blkid -s UUID -o value $ROOT) rw rootflags=subvol=@ amd_pstate=active nowatchdog quiet loglevel=3
EOF
```

```bash
sed -i 's/MODULES=()/MODULES=(amdgpu)/' /etc/mkinitcpio.conf

sed -i 's/^HOOKS=.*/HOOKS=(systemd autodetect microcode modconf kms keyboard sd-vconsole block filesystems)/' /etc/mkinitcpio.conf
sed -i 's/^#\?#COMPRESSION="zstd"/COMPRESSION="zstd"/' /etc/mkinitcpio.conf

mkinitcpio -P
```

### Ajustes del sistema

```bash
sed -i 's/^#\?NAutoVTs=6/NAutoVTs=2/' /etc/systemd/logind.conf

```

```bash
cat > /etc/systemd/zram-generator.conf <<EOF
[zram0]
zram-size = min(ram / 2, 8192)
compression-algorithm = zstd
EOF
```

```bash
cat > /etc/sysctl.d/99-rendimiento.conf <<EOF
vm.swappiness = 10
vm.vfs_cache_pressure = 50
vm.dirty_ratio = 10
vm.dirty_background_ratio = 5
EOF
```

```bash
sed -i 's/^#\?Storage=persistent/Storage=volatile/' /etc/systemd/journald.conf
sed -i 's/^#\?RuntimeMaxUse=.*/RuntimeMaxUse=15M/' /etc/systemd/journald.conf
```

```bash
# opcional
sed -i 's/^#\?Storage=external/Storage=none/' /etc/systemd/coredump.conf
sed -i 's/^#\?ProcessSizeMax=.*/ProcessSizeMax=0/' /etc/systemd/coredump.conf
```

### Red y firewall

```bash
mkdir -p /etc/NetworkManager/conf.d
cat > /etc/NetworkManager/conf.d/wifi_backend.conf <<EOF
[device]
wifi.backend=iwd
EOF
```

```bash
cat > /etc/nftables.conf <<EOF
flush ruleset

table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;

        # Aceptar trafico localhost
        iif "lo" accept

        # Aceptar trafico de conexiones ya establecidas
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

### Apariencia y entorno

```bash
# Eliminar el hinting completo si lo activaste
rm -f /etc/fonts/conf.d/10-hinting-full.conf;
rm -f /etc/fonts/conf.d/10-hinting-slight.conf;
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

  <alias>
    <family>sans-serif</family>
    <prefer>
      <family>Inter</family>
      <family>Noto Sans</family>
    </prefer>
  </alias>

  <alias>
    <family>system-ui</family>
    <prefer>
      <family>Inter</family>
    </prefer>
  </alias>

  <alias>
    <family>monospace</family>
    <prefer>
      <family>JetBrains Mono</family>
      <family>Noto Sans Mono</family>
    </prefer>
  </alias>

</fontconfig>
EOF
```

```bash
cat >> /etc/environment <<EOF
# Forzar Wayland nativo universal (Aislamiento de X11)
MOZ_ENABLE_WAYLAND=1
ELECTRON_OZONE_PLATFORM_HINT=wayland
NIXOS_OZONE_WL=1
QT_QPA_PLATFORM=wayland;xcb
SDL_VIDEODRIVER=wayland
EOF
```

### Servicios

```bash
systemctl enable bluetooth.service
systemctl enable upower.service
systemctl enable NetworkManager.service
systemctl enable power-profiles-daemon.service
systemctl enable nftables.service
systemctl enable iwd.service
```

```bash
# opcional
systemctl disable man-db.timer
systemctl disable shadow.timer
```

## Salir y reiniciar

```bash
exit
```
```bash
umount -R /mnt
```
```bash
reboot
```

## Post-instalacion

### Snapper

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

### KDE Plasma (Wayland Puro - Misión Crítica)

```bash
# Instalacion del nucleo absoluto (cero bloatware, cero meta-paquetes)
sudo pacman -S sddm plasma-desktop kwin qt6-wayland systemsettings plasma-workspace-wallpapers xdg-desktop-portal-kde bluedevil dolphin konsole filelight ksystemlog plasma-systemmonitor kwrite merkuro okular gwenview kamoso haruna elisa kcalc discover flatpak firefox firefox-i18n-es-mx plasma-nm plasma-pa

# Configurar SDDM para ejecutarse nativamente bajo Wayland (KWin) y descartar X11 en el login
sudo mkdir -p /etc/sddm.conf.d
sudo cat > /etc/sddm.conf.d/10-wayland.conf <<EOF
[General]
DisplayServer=wayland
GreeterEnvironment=QT_WAYLAND_SHELL_INTEGRATION=layer-shell

[Wayland]
CompositorCommand=kwin_wayland --drm --no-lockscreen --no-global-shortcuts --locale1
EOF

# Castración de indexadores (Baloo) a nivel de sistema
sudo mkdir -p /etc/xdg
echo -e "[Basic Settings]\nIndexing-Enabled=false" > /etc/xdg/baloofilerc

# Habilitar el gestor de sesiones
sudo systemctl enable sddm.service

# Crear script de configuración de usuario (ejecutar manualmente al iniciar sesión por primera vez)
export USERNAME=$(grep '/home' /etc/passwd | grep '/bin/bash' | head -1 | cut -d: -f1)
cat > /home/$USERNAME/kde-setup.sh <<'SCRIPT'
#!/bin/bash
CONF_DIR="$HOME/.config"
mkdir -p "$CONF_DIR"

# 1. Castrar Discover (Evitar que busque actualizaciones en segundo plano y consuma RAM)
mkdir -p "$CONF_DIR/systemd/user"
ln -s /dev/null "$CONF_DIR/systemd/user/plasma-discover-update.service"
ln -s /dev/null "$CONF_DIR/systemd/user/plasma-discover-update.timer"

# 2. Deshabilitar DrKonqi (reportes de fallos de KDE)
mkdir -p "$CONF_DIR/autostart"
cp /etc/xdg/autostart/org.kde.drkonqi.desktop "$CONF_DIR/autostart/" 2>/dev/null || true
echo "Hidden=true" >> "$CONF_DIR/autostart/org.kde.drkonqi.desktop"

# 3. Kdeglobals: Apagar animaciones globales y forzar renderizado inmediato
cat >> "$CONF_DIR/kdeglobals" <<EOF
[KDE]
AnimationDurationFactor=0

[KCrash]
AutoRestart=false
EOF

# 4. Baloo (Indexador): Destruir cualquier intento de indexación
cat > "$CONF_DIR/baloofilerc" <<EOF
[Basic Settings]
Indexing-Enabled=false
EOF

# 5. Dolphin: Sin Baloo, sin miniaturas, sin pestañas recordadas
cat > "$CONF_DIR/dolphinrc" <<EOF
[General]
ShowSpaceInfo=false
RememberOpenedTabs=false

[PreviewSettings]
Plugins=
EOF

# 6. Elisa: Prohibir escaneo automático al iniciar
cat > "$CONF_DIR/elisarc" <<EOF
[General]
ScanAtStartup=false
EOF

# 7. KSystemLog: Solo journal actual, sin históricos
cat > "$CONF_DIR/ksystemlogrc" <<EOF
[General]
MaxLines=1000
EOF

# 8. Okular: Bajo consumo de memoria
cat > "$CONF_DIR/okularpartrc" <<EOF
[Core Performance]
MemoryLevel=Low
EnableCompositing=false
EOF

echo "Configuración de KDE aplicada. Reinicia la sesión."
SCRIPT
chmod +x /home/$USERNAME/kde-setup.sh
chown $USERNAME:$USERNAME /home/$USERNAME/kde-setup.sh

```


### Virtualizacion (KVM/QEMU)

```bash
# Paquetes de virtualizacion
pacman -S qemu-desktop libvirt edk2-ovmf dnsmasq iptables-nft

# Anadir usuario al grupo de virtualizacion
usermod -aG libvirt,kvm $USERNAME

# Activacion por socket (libvirtd solo arranca bajo demanda)
sudo systemctl enable --now libvirtd.socket
```

### Contenedores (Podman rootless)

```bash
# Podman, stack de red en Rust y soporte para compose
pacman -S podman podman-compose netavark aardvark-dns

# Distrobox para entornos de desarrollo aislados
pacman -S distrobox

# Permisos del subvolumen para rootless
sudo chown -R $USERNAME:$USERNAME /var/lib/containers

# Almacenamiento nativo BTRFS (en lugar de overlayfs)
mkdir -p /etc/containers
cat > /etc/containers/storage.conf <<EOF
[storage]
driver = "btrfs"
graphroot = "/var/lib/containers"
EOF
```

### Firefox

```bash
# --- Configuración de Firefox: Lógica de Hierro y Privacidad ---
# NOTA: Ejecutar este bloque SOLO después de haber abierto Firefox al menos una vez para que genere el perfil.

# 1. Localizar la carpeta del perfil por defecto
FF_PROFILE=$(ls -d ~/.mozilla/firefox/*.default-release | head -n 1)

# 2. Inyectar el archivo user.js de alto rendimiento y seguridad
cat > "$FF_PROFILE/user.js" <<EOF
// --- PRIVACIDAD Y SEGURIDAD ---
user_pref("privacy.trackingprotection.enabled", true);       // Bloqueo de rastreo estricto
user_pref("privacy.trackingprotection.socialtracking.enabled", true);
user_pref("privacy.partition.network_state.ocsp_cache", true); // Evita fugas de caché
user_pref("dom.battery.enabled", false);                     // No rastrear nivel de batería
user_pref("browser.send_pings", false);                      // Bloqueo de balizas de clic
user_pref("network.IDN_show_punycode", true);                // Evita ataques de homógrafo (Phishing)

// --- ELIMINAR TELEMETRÍA Y RUIDO ---
user_pref("datareporting.healthreport.uploadEnabled", false);
user_pref("datareporting.policy.dataSubmissionEnabled", false);
user_pref("toolkit.telemetry.enabled", false);
user_pref("app.shield.optoutstudies.enabled", false);
user_pref("browser.discovery.enabled", false);
user_pref("extensions.pocket.enabled", false);               // Adiós Pocket (bloatware)
user_pref("browser.newtabpage.activity-stream.feeds.telemetry", false);
user_pref("browser.newtabpage.activity-stream.showSponsored", false);

// --- RENDIMIENTO Y OPTIMIZACIÓN (HARDWARE AMD) ---
user_pref("media.ffmpeg.vaapi.enabled", true);               // Decodificación video por hardware
user_pref("image.mem.decode_on_draw", true);                 // Ahorro de memoria en renderizado

// --- COMPORTAMIENTO LIMPIO ---
user_pref("browser.shell.checkDefaultBrowser", false);       // No molestar con "Navegador predeterminado"
user_pref("browser.sessionstore.privacy_level", 2);          // No enviar cookies en restauración
user_pref("network.prefetch-next", false);                   // No conectar a enlaces que no has tocado
user_pref("pdfjs.disabled", false);                          // Mantener visor PDF interno (más seguro que externo)
EOF

echo "Firefox endurecido con éxito en: $FF_PROFILE"
```

### Referencia rapida

#### Distrobox
```bash
# listar
distrobox list

# crear
distrobox create --name dev-arch --image archlinux:latest --home ~/Cajas/arch-home
distrobox create --name dev-alpine --image alpine:latest --home ~/Cajas/alpine-home

# entrar
distrobox enter dev-arch

# salir: exit o Ctrl + D

# detener
distrobox stop dev-go

# eliminar
distrobox rm dev-go

# ejecutar comando sin entrar
distrobox enter dev-go -- go version
```

#### Snapper
```bash
# listar snapshots
sudo snapper -c root list

# crear snapshot manual
sudo snapper -c root create --description "antes de actualizar"

# comparar cambios entre dos snapshots
sudo snapper -c root status 1..2

# revertir cambios entre dos snapshots
sudo snapper -c root undochange 1..2

# eliminar un snapshot
sudo snapper delete 2
```
