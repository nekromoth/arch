#### INTERNET
    systemctl enable NetworkManager
    systemctl start NetworkManager
    nmtui
    
#### USER
    useradd -m -g users -s /bin/bash moth
    passwd moth
    
#### SUDO 
    visudo
    moth ALL=(ALL) ALL
    
#### AUTOLOGIN
    mkdir /etc/systemd/system/getty@tty1.service.d
    vim   /etc/systemd/system/getty@tty1.service.d/override.conf
        [Service]
        ExecStart=
        ExecStart= -usr/bin/agetty -a moth %I $TERM

#### DESKTOP SYSTEM
    pacman -S xorg xorg-xinit i3-gaps rofi ranger conky alsa-utils firefox ranger git w3m terminus-font gufw polkit-gnome gnome-keyring
    
Install ```pamac``` and ```terminus-ttf``` from AUR. 

#### XORG KEYBOARD
    localectl --no-convert set-x11-keymap de pc104 nodeadkeys
    vim ~/.xinitrc
        xset r 250 60 
        
#### FIREWALL
    systemctl start ufw.service
    systemctl enable ufw.service

