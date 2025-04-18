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
| /dev/nvme0n1p1 | 550M   | EFI System       |
| /dev/nvme0n1p2 | MAX    | Linux filesystem |

###### si *NO existen* las particiones creadas
```bash
mkfs.vfat -F 32 /dev/nvme0n1p1
```
```bash
mkfs.btrfs -f -d single -m single -L "root" /dev/nvme0n1p2
```

```bash
mount /dev/nvme0n1p2 /mnt
btrfs subvolume create /mnt/@root
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@snapshots
umount /mnt
```

```bash
mount -o noatime,compress=zstd:3,space_cache=v2,ssd,autodefrag,subvol=@root /dev/nvme0n1p2 /mnt
mkdir -p /mnt/{home,boot/efi}
mount -o noatime,compress=zstd:3,space_cache=v2,ssd,autodefrag,subvol=@home /dev/nvme0n1p2 /mnt/home
mount /dev/nvme0n1p1 /mnt/boot/efi
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

```bash
nvim /etc/default/grub

GRUB_CMDLINE_LINUX_DEFAULT="quiet splash loglevel=3 amdgpu.dpm=1 amdgpu.dc=1 amdgpu.noretry=0 amdgpu.vm_fragment_size=9 amdgpu.powerplay=1"
```

```bash
mkinitcpio -p linux-zen;grub-mkconfig -o /boot/grub/grub.cfg
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
ufw default deny incoming;ufw enable
```

```bash
snapper -c root create --description "Backup antes de actualización"
snapper -c root list
snapper -c root rollback <ID>

sudo pacman -S waybar sway ly alacritty swaybg swaylock swayidle xdg-desktop-portal-wlr clipman mako wf-recorder light nnn wofi grim slurp
```

```bash
nvim ~/.config/sway/config
```

```bash
#
# Configuración de Sway (Solo Wayland, Repos Oficiales, Laptop, Eficiente)
# Usando waybar, alacritty, wofi, etc., sin dependencias a X11.
# Guarda este archivo como ~/.config/sway/config
#

# --- Variables ---
set $mod Mod4 # La tecla Super (Windows)
set $term alacritty # Tu terminal Wayland (Alacritty)
set $menu wofi # Tu lanzador Wayland (Wofi)
set $filemgr $term -e nnn # Gestor de archivos en terminal (nnn)

# Ruta a tu fondo de pantalla (¡CAMBIA ESTO!)
set $wallpaper $HOME/wallpaper.jpg # <<< CAMBIA ESTA RUTA >>>

# --- Configuración de Salida (Monitores Wayland) ---
# Sway maneja multimonitor Wayland automáticamente. Configúralos aquí.
# Encuentra los nombres de tus salidas Wayland con 'swaymsg -t get_outputs'.
# Ejemplo (adapta a tus nombres de salida y resoluciones):
# output eDP-1 resolution 1920x1080 position 0,0 # Monitor de la laptop
# output DP-1 resolution 2560x1440 position 1920,0 # Monitor externo a la derecha
# output HDMI-A-1 disable # Deshabilitar si no la usas

# Configura el fondo de pantalla (usando el comando Wayland swaybg)
exec swaybg -i $wallpaper -m fill # Ajusta '-m fill' si quieres otro modo

# --- Configuración de Entrada (Teclado, Mouse, Touchpad con Libinput Wayland) ---
# Disposición del teclado
input "type:keyboard" {
    xkb_layout "latam" # <<< AJUSTA A TU DISPOSICION >>>
    # Si usas múltiples: xkb_layout "us,latam"; xkb_options "grp:alt_shift_toggle"
}

# Configuración del Touchpad (usando Libinput, el estándar en Wayland/wlroots)
input "type:touchpad" {
    dwt enabled       # Deshabilitar al escribir
    tap enabled       # Tocar para clickar
    natural_scroll enabled # Scroll "natural"
    # scroll_factor 0.7 # Ajusta velocidad
    # accel_profile adaptive # Perfil aceleración

    # Gestos básicos (scroll con 2 dedos, click medio con 3) funcionan con Libinput.
    # Gestos complejos como swipe con 3/4 dedos para cambiar de escritorio NO son nativos de Sway/Libinput básico
    # y requieren herramientas adicionales que a menudo NO están en repos oficiales o son más complejas.
    # Esta configuración se limita a lo que Libinput en Wayland ofrece directamente a Sway.
}

# --- Apariencia (Opcional) ---
# default_border pixel 2
# gaps inner 10
# font pango:monospace 8

# --- Keybindings (Atajos de Teclado) ---
# Atajos estándar usan $mod (Super)

# Lanzar terminal (Alacritty Wayland)
bindsym $mod+Return exec $term

# Lanzar menú (Wofi Wayland - solo aplicaciones .desktop)
bindsym $mod+d exec $menu --show drun # <--- Lanza Wofi para apps instaladas

# Lanzar gestor de archivos (nnn en Alacritty)
bindsym $mod+Shift+Return exec $filemgr

# Cerrar ventana enfocada
bindsym $mod+Shift+q kill

# Manejo de ventanas (Puedes añadir los atajos estándar de i3/Sway aquí)
# bindsym $mod+h focus left
# bindsym $mod+Shift+h move left
# bindsym $mod+f fullscreen toggle
# bindsym $mod+space floating toggle
# bindsym $mod+r mode "resize" {...} # Modo redimensionar

# Workspaces
# bindsym $mod+1 workspace number 1
# bindsym $mod+Shift+1 move container to workspace number 1
# ... hasta el 10

# Scratchpad (opcional)
# bindsym $mod+Shift+minus move scratchpad
# bindsym $mod+minus scratchpad show

# --- Keybindings de Sistema y Multimedia (Wayland/Consola) ---
# Control de Volumen (usando pactl de Pipewire)
bindsym XF86AudioRaiseVolume exec pactl set-sink-volume @DEFAULT_SINK@ +5%
bindsym XF86AudioLowerVolume exec pactl set-sink-volume @DEFAULT_SINK@ -5%
bindsym XF86AudioMute exec pactl set-sink-mute @DEFAULT_SINK@ toggle

# Control de Brillo (usando light, utilidad de consola)
bindsym XF86MonBrightnessUp exec light -A 10
bindsym XF86MonBrightnessDown exec light -U 10

# Captura de Pantalla (usando grim y slurp, herramientas nativas Wayland/wlroots)
# Necesitas grim y slurp instalados (sudo pacman -S grim slurp)
# Captura pantalla completa:
bindsym Print exec grim $HOME/Pictures/screenshot-$(date +%Y%m%d-%H%M%S).png # <<< AJUSTA LA RUTA >>>
# Captura región seleccionada:
bindsym $mod+Print exec slurp | grim -g - $HOME/Pictures/screenshot-area-$(date +%Y%m%d-%H%M%S).png # <<< AJUSTA LA RUTA >>>
# Captura ventana enfocada: Requiere jq (sudo pacman -S jq), que es de consola. No depende de X11.
# bindsym $mod+Shift+Print exec swaymsg -t get_tree | jq -r '.. | select(.focused?) | .rect | "\(.x),\(.y) \(.width)x\(<span class="math-inline">\.height)"' \| slurp \-g \- \| grim \-g \- </span>HOME/Pictures/screenshot-window-$(date +%Y%m%d-%H%M%S).png # Ejemplo

# Grabar Pantalla (usando wf-recorder, nativo Wayland/wlroots)
# Iniciar grabación de una región seleccionada (requiere slurp):
# bindsym $mod+Shift+r exec slurp | wf-recorder -g - -f $HOME/Videos/screencast-$(date +%Y%m%d-%H%M%S).mp4 # <<< AJUSTA LA RUTA >>>
# Nota: Detener wf-recorder requiere matar el proceso.

# --- Modo de Sistema (Salir, Reiniciar, Apagar) ---
# Entrar en modo de sistema ($mod+Shift+e)
bindsym $mod+Shift+e mode "system" {
    set $mode_system_prompt "Modo Sistema: (l)ock (e)xit (r)eboot (s)hutdown"
    swaynag -t warning -m "$mode_system_prompt" # Necesitas swaynag (parte de sway)

    bindsym l exec swaylock -f ; mode "default" # Bloquear pantalla (swaylock es nativo Wayland)
    bindsym e exec swaymsg exit # Salir de Sway (comando nativo Wayland)
    bindsym r exec systemctl reboot # Reiniciar el sistema (comando systemd)
    bindsym s exec systemctl poweroff # Apagar el sistema (comando systemd)

    # Salir del modo sistema
    bindsym Escape mode "default"
    bindsym Return mode "default"
}

# --- Configuración de Autoinicio (Ejecutar programas al iniciar Sway) ---
# Todos estos son programas Wayland-nativos o de consola.

exec waybar # La barra de estado Wayland
exec mako # Demonio de notificaciones Wayland
exec clipman daemon # Gestor de portapapeles Wayland
exec xdg-desktop-portal-wlr # Portal de escritorio para Wayland (requerido por muchas apps)
exec swayidle -w \ # Gestor de inactividad Wayland
    timeout 300 'exec swaylock -f' \
    timeout 600 'exec systemctl suspend' \
    before-sleep 'exec swaylock -f'

# --- Otras configuraciones ---
# Recargar la configuración de Sway
bindsym $mod+Shift+c reload

# --- Modo por defecto ---
mode "default"

```

```bash
~/.config/waybar/config
```

```json
{
    // Puedes tener múltiples barras configurando más objetos aquí.
    // Este objeto configura una sola barra.
    "layer": "top", // "bottom" si prefieres la barra abajo
    "position": "top", // Posición de la barra: "top" o "bottom"
    "height": 32, // Altura de la barra en píxeles (ajustable)
    // "output": "eDP-1", // Opcional: si quieres que la barra aparezca solo en un monitor, especifícalo aquí (usa el nombre real de tu salida)

    // Módulos alineados a la izquierda
    "modules-left": [
        "sway/workspaces" // Muestra los espacios de trabajo de Sway
    ],

    // Módulos alineados a la derecha, en orden tipo macOS
    "modules-right": [
        "network",     // Estado de la red (Wi-Fi/Ethernet)
        "pulseaudio",  // Control de volumen
        "battery",     // Estado de la batería
        "clock",       // Hora y fecha
        "tray"         // Iconos de área de estado (AppIndicators)
    ],

    // --- Configuración Detallada de Cada Módulo ---
    // Define cómo se comporta y qué muestra cada módulo

    "sway/workspaces": {
        "format": "{}", // Muestra el número o nombre del workspace
        "display-format": "{}", // Formato más simple para display
        "tooltip": false // No mostrar tooltip para los workspaces
    },

    "network": {
        "format-wireless": "{}", // Formato minimalista para Wi-Fi
        "format-ethernet": "{}", // Formato minimalista para Ethernet
        "format-disconnected": " offline", // <-- Reemplaza 'offline' con un icono de "sin red" si tu fuente tiene uno
        "tooltip": true, // Mostrar tooltip con más detalles
        "tooltip-format-wireless": "{essid} ({signalStrength}%)", // Essid y señal en tooltip
        "tooltip-format-ethernet": "Ethernet ({ipaddr})",
        "tooltip-format-disconnected": "Desconectado",
        "interval": 5 // Actualizar cada 5 segundos
    },

    "pulseaudio": {
        "format": "{}", // Formato minimalista (se estiliza con CSS)
        "format-bluetooth": "{}",
        "format-muted": "{}", // Formato si está silenciado
        "format-icons": {
             // Iconos de volumen (asegúrate que tu fuente los tenga, estos son ejemplos de Font Awesome 6)
            "default": [" ", " ", " "]
        },
        "tooltip": true,
        "tooltip-format": "{volume}%", // Porcentaje de volumen en tooltip
        "on-click": "pactl set-sink-mute @DEFAULT_SINK@ toggle", // Silenciar/desilenciar con click
        "on-scroll-up": "pactl set-sink-volume @DEFAULT_SINK@ +5%", // Subir volumen con scroll
        "on-scroll-down": "pactl set-sink-volume @DEFAULT_SINK@ -5%", // Bajar volumen con scroll
        "scroll-step": 5 // Incremento del scroll
    },

    "battery": {
        "format": "{icon}", // Mostrar solo el icono de batería
        "format-charging": "🔋", // Icono cuando carga (emoji o glifo)
        "format-plugged": "🔌",  // Icono cuando enchufado (carga completa)
         // "{percentage}" y "{time}" pueden usarse en tooltip o format-alt
        "format-alt": "{percentage}%", // Opcional: muestra porcentaje al lado del icono al pasar el ratón
        "format-icons": [" guage_empty", " guage_low", " guage_mid", " guage_full"], // <-- Iconos de nivel (ejemplos)
        "tooltip": true, // Mostrar tooltip con info detallada
        "tooltip-format": "Nivel: {percentage}%\nTiempo restante: {time}",
        "critical-at": 15, // Nivel crítico para CSS/animación
        "warning-at": 25, // Nivel advertencia para CSS
        "interval": 60 // Actualizar cada 60 segundos
    },

    "clock": {
        "format": "{:%I:%M %p}", // Formato de hora 12h con AM/PM (ej. 10:00 PM)
        "format-tooltip": "{:%A, %B %d, %Y}", // Fecha completa en tooltip
        "interval": 1 // Actualizar cada segundo
    },

    "tray": {
        "icon-size": 18, // Tamaño de los iconos del tray (ajustable)
        "spacing": 5,    // Espacio entre iconos del tray
        "tooltip": true
        // "output": "eDP-1" // Opcional: si quieres el tray solo en un monitor
    }

    // Puedes añadir otros módulos aquí si los necesitas (ej. cpu, memory)
    // y luego estilizarlos con CSS para que se vean minimalistas.
    // Ejemplo:
    // "cpu": { "format": "CPU: {}%", "tooltip": false, "interval": 10 }
}

```


```css
/* Archivo: ~/.config/waybar/style.css */
/* Formato: CSS */

/* Estilo General de la Barra */
#waybar {
    background-color: rgba(30, 30, 30, 0.9); /* Fondo oscuro y semi-transparente (90% opaco) */
    color: #ffffff; /* Color de texto por defecto (blanco) */
    font-family: 'Inter Regular', 'Roboto Regular', sans-serif; /* <-- AJUSTA A TUS FUENTES INSTALADAS */
    font-size: 14px; /* Tamaño de la fuente */
    border: none; /* Sin borde para un look limpio */
    /* border-radius: 8px; /* Descomenta y ajusta si quieres esquinas redondeadas (requiere soporte del compositor/tema GTK) */
    /* margin: 5px; /* Descomenta para añadir espacio alrededor de la barra */
}

/* Espaciado y padding */
#waybar .modules-left > widget:first-child { margin-left: 5px; } /* Espacio al inicio de los módulos de la izquierda */
#waybar .modules-right > widget:last-child { margin-right: 5px; } /* Espacio al final de los módulos de la derecha */

#waybar .modules-left > widget,
#waybar .modules-center > widget,
#waybar .modules-right > widget {
    padding: 0 5px; /* Espacio horizontal entre los módulos */
}

/* Estilos de Workspaces (Minimalista, como el número de escritorio) */
#workspaces button {
    padding: 0 3px; /* Padding muy ajustado */
    background-color: transparent;
    color: #888888; /* Color de los workspaces inactivos */
    border: none;
    box-shadow: none;
    min-width: 0; /* Permite que el botón sea solo del tamaño del número/nombre */
}

#workspaces button:hover {
    background-color: rgba(255, 255, 255, 0.1); /* Efecto sutil al pasar el ratón */
}

#workspaces button.active {
    color: #ffffff; /* Color del workspace activo */
    font-weight: bold; /* Hacer el número activo un poco más notorio */
}

#workspaces button.urgent {
    color: #ff0000; /* Rojo para workspaces urgentes */
}

/* Estilos de los Módulos de Estado (Icono-céntricos a la derecha) */
/* Asegura que el padding sea consistente para todos los módulos de estado */
#network, #pulseaudio, #battery, #backlight {
    padding: 0 5px; /* Espacio entre los módulos de estado */
    color: #ffffff; /* Color del icono/texto (blanco) */
}

/* Estilos para estados específicos de los módulos */
#battery.charging { color: #26A65B; } /* Verde */
#battery.warning:not(.charging) { color: #FF7F00; } /* Naranja */
#battery.critical:not(.charging) {
    color: #FF0000; /* Rojo */
    /* Animación opcional de parpadeo para batería crítica */
    animation-name: blink;
    animation-duration: 0.5s;
    animation-timing-function: linear;
    animation-iteration-count: infinite;
    animation-direction: alternate;
}
#pulseaudio.muted { color: #e74c3c; } /* Rojo para icono de volumen silenciado */
#network.disconnected { color: #e74c3c; } /* Rojo para icono de red desconectada */


/* Animación de parpadeo (usada en batería crítica) */
@keyframes blink {
    to { color: #ffffff; /* Parpadea a blanco */ }
}

/* Estilo del Módulo de Hora */
#clock {
    padding: 0 10px; /* Un poco más de espacio alrededor de la hora */
    font-weight: normal; /* Sin negrita por defecto */
}

/* Estilo del Módulo Tray (AppIndicators) */
#tray {
    padding: 0 5px; /* Padding alrededor del área del tray */
    /* El tamaño y espaciado de los iconos dentro del tray se controla en el archivo config */
}
/* Estilos opcionales para iconos en estados pasivos o de atención */
#tray > .passive { -gtk-icon-effect: dim; } /* Atenuar iconos inactivos */
#tray > .needs-attention { -gtk-icon-effect: highlight; } /* Resaltar iconos que necesitan atención */

/* Estilo opcional para el Título de la Ventana (si lo activas en config) */
/* #window { font-style: italic; padding: 0 10px; } */


/* Si añades módulos como CPU/Memoria, estilizalos aquí */
/* #cpu, #memory { padding: 0 5px; color: #cccccc; } */

```
