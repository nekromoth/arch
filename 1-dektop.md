#### INTERNET
    systemctl enable NetworkManager
    systemctl start NetworkManager
    nmtui
    
#### SSH ? 
    systemctl start sshd
    (systemctl status sshd)
    ip addr
Incase your BACKSPACE wont work try:

    export TERM=vt100  
    
#### AUTOLOGIN
    sudo mkdir /etc/systemd/system/getty@tty1.service.d
    sudo vim   /etc/systemd/system/getty@tty1.service.d/override.conf
        [Service]
        ExecStart=
        ExecStart= -/usr/bin/agetty -a moth %I $TERM

#### DESKTOP SYSTEM
    sudo pacman -S xorg xorg-xinit i3-gaps rofi ranger conky firefox ranger git w3m terminus-font gufw polkit-gnome gnome-keyring terminus-font bash-completion rxvt-unicode pulseaudio pavucontrol 
    
Laptop ?: add ```acpi light```

It would to be a good idea to reboot after this.
    
    
Install ```polybar``` and ```terminus-ttf``` from AUR. 
    mkdir AUR 
    cd AUR
    git clone https://aur.archlinux.org/polybar.git
    git clone https://aur.archlinux.org/terminus-font-ttf.git
    
```cd``` into downloaded folder and ```makepkg -si```

#### XORG KEYBOARD and i3
    localectl --no-convert set-x11-keymap de pc104 nodeadkeys
    vim ~/.xinitrc
        xset r 250 60 
        exec i3

#### BACKLIGHT
    sudo EDITOR=vim visudo
        moth = ALL = NOPASSWD: /usr/bin/light
        
Now you can execute with ```sudo light <operation>``` without entering the password.

#### FIREWALL
    systemctl start ufw.service
    systemctl enable ufw.service

