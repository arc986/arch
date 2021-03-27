pacstrap /mnt base base-devel grub ntfs-3g gvfs efibootmgr htop openssh linux linux-headers linux-firmware vim amd-ucode zsh netctl wpa_supplicant dialog dhcpcd xf86-video-amdgpu vulkan-radeon lib32-mesa lib32-vulkan-radeon libva-mesa-driver lib32-libva-mesa-driver mesa-vdpau lib32-mesa-vdpau xorg-server xorg-xinit pulseaudio pulseaudio-alsa alsa-utils alsa-plugins alsa-lib ttf-liberation ttf-bitstream-vera ttf-dejavu ttf-droid ttf-freefont ttf-font-awesome ttf-ubuntu-font-family hunspell-es_pa terminus-font xorg-xdm xorg-xset xorg-xrdb f2fs-tools neovim git python-pip xorg-xrdb xorg-xset xdg-utils rxvt-unicode



go npm lxappearance dmenu ranger mutt python-neovim  

```
loadkeys la-latin1
```

```
echo en_US.UTF-8 UTF-8 >/etc/locale.gen
echo LANG=en_US.UTF-8 >/etc/locale.conf
echo LANG=en_US.UTF-8 >>/etc/environment
echo LC_TIME=en_US.UTF-8 >>/etc/environment
echo export LANGUAGE=en_US.UTF-8 >> ~/.bashrc
echo export LANG=en_US.UTF-8 >> ~/.bashrc
echo export LC_ALL=en_US.UTF-8 >> ~/.bashrc
echo export LANGUAGE=en_US.UTF-8 >> ~/.zshrc
echo export LANG=en_US.UTF-8 >> ~/.zshrc
echo export LC_ALL=en_US.UTF-8 >> ~/.zshrc
locale-gen
```



```
export USERR="kraleon"
useradd -m -g users -G audio,lp,optical,storage,video,wheel,games,power,scanner -s /bin/zsh $USERR
passwd $USERR
```

```
Instale el paquete powertop.
/etc/systemd/system/powertop.service
[Unit]
Description=Powertop tunings

[Service]
Type=exec
ExecStart=/usr/bin/powertop --auto-tune
RemainAfterExit=true

[Install]
WantedBy=multi-user.target

powertop --calibrate
```

echo 5 > /sys/class/backlight/amdgpu_bl0/brightness 

sudo pacman -S crda

sudo crda

```
sudo pacman -S crda
sudo crda
sudo iw reg set PA
or 
sudo nvim /etc/conf.d/wireless-regdom
```

Packages (5) swaybg-1.0-3 wlroots-0.10.1-1 xcb-util-errors-1.0-3 xcb-util-image-0.4.0-3 sway-1:1.4-9

```ls
sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs \
       https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'
```