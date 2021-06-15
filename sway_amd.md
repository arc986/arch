pacstrap /mnt base base-devel grub zsh wget ntfs-3g efibootmgr htop openssh linux-lts linux-lts-headers linux-firmware vim amd-ucode networkmanager f2fs-tools git fuse pulseaudio pipewire-alsa pipewire-jack pipewire hunspell-es_pa sudo xf86-video-amdgpu vulkan-radeon libva-mesa-driver mesa-vdpau xf86-input-evdev ufw fzf ttf-dejavu neovim neomutt elinks python-pdftotext firefox firefox-i18n-es-mx kitty sway waybar dmenu swaybg swaylock swayidle grim xdg-desktop-portal-wlr xdg-user-dirs virt-manager qemu vde2 ebtables dnsmasq bridge-utils openbsd-netcat dmidecode podman podman-compose flatpak

genfstab -pU /mnt >> /mnt/etc/fstab

arch-chroot /mnt

export HOMBREPC="" echo $HOMBREPC > /etc/hostname;echo "127.0.0.1 localhost $HOMBREPC" >> /etc/hosts;echo "::1 localhost $HOMBREPC" >> /etc/hosts

ln -sf /usr/share/zoneinfo/America/Panama /etc/localtime

echo es_PA.UTF-8 UTF-8 >/etc/locale.gen;echo LANG=es_PA.UTF-8 >/etc/locale.conf;echo LANG=es_PA.UTF-8 >>/etc/environment;echo LC_TIME=C >>/etc/environment;locale-gen

echo KEYMAP=us > /etc/vconsole.conf echo KEYMAP=la-latin1 > /etc/vconsole.conf

grub-install --efi-directory=/boot/efi --bootloader-id='Arch Linux' --target=x86_64-efi

vim /etc/mkinitcpio.conf MODULES=(f2fs fuse amdgpu radeon)

vim /etc/modprobe.d/amdgpu.conf options amdgpu si_support=1 options amdgpu cik_support=1

vim /etc/X11/xorg.conf.d/20-amdgpu.conf Section "Screen" Identifier "Screen" DefaultDepth 30 EndSection

Section "Device" Identifier "AMD" Driver "amdgpu" Option "TearFree" "true" Option "DRI" "3" EndSection

passwd

export USERR="" useradd -m -g users -G audio,lp,optical,storage,video,wheel,games,power,scanner,kvm,polkitd,libvirt,docker -s /bin/zsh $USERR passwd $USERR

visudo %wheel ALL=(ALL) ALL

systemctl enable systemd-resolved.service systemctl enable NetworkManager systemctl enable bluetooth.service systemctl enable gdm.service systemctl enable libvirtd.service systemctl enable docker.service systemctl enable ufw.service

su $USERR cd /tmp;git clone https://aur.archlinux.org/yay.git;cd yay;makepkg -si;sudo pacman -Rsdnc $(pacman -Qqdt) yay -S upd72020x-fw

mkinitcpio -p linux-lts;grub-mkconfig -o /boot/grub/grub.cfg

exit

umount -R /mnt

reboot
