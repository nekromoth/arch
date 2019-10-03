### Keyboard
    loadkeys de

### Partitions
    gdisk /dev/nvme0n1

### Filesystem
    mkfs.vfat -F 32 -n EFI /dev/nvme0n1p1
    mkfs.ext4 -L ROOT /dev/nvme0n1p2

### Mount partitions
    mount /dev/nvme0n1p2 /mnt
    mkdir /mnt/boot
    mount /dev/nvmen1p1 /mnt/boot

### Install arch
    pacstrap /mnt base base-devel bash-completion vim

### Generate filesystem table
    genfstab -Lp /mnt >> /mnt/etc/fstab

### Goto new installed system
    arch-chroot /mnt

### Configure mirrors
    vim /etc/pacman.d/mirrorlist
* add `mirrorlist-manager` AUR

### Install Bootloader
    bootctl install
    vim /boot/loader/loader.conf
        default arch
        timeout 0
    vim /boot/loader/entries/arch.conf
        title   Archlinux
        linux   /vmlinuz-linux
        initrd  /initramfs-linux.img
        options root=LABEL=ROOT rw

### Timezone
    ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime
    hwclock -s

### Language
    vim /etc/locale.gen
        en_US.UTF-8 UTF8
    locale-gen
    vim /etc/locale.conf
        LANG=en_US.UTF-8
    
### Keyboard
    vim /etc/vconsole.conf
        KEYMAP=de-latin1-nodeadkeys

### Host
    vim /etc/hostname
        arch
    vim /etc/hosts
        127.0.0.1   localhost
        ::1         localhost
        127.0.1.1   home.localdomain home

### Root password
    passwd

### User
    useradd -m -g users -s /bin/bash moth
    passwd moth

### sudo 
    visudo
        moth ALL=(ALL) ALL

### Internet
    pacman -S networkmanager
    systemctl enable NetworkManager

### Autologin
    mkdir /etc/systemd/system/getty@tty1.service.d
    vim   /etc/systemd/system/getty@tty1.service.d/override.conf
        [Service]
        ExecStart=
        ExecStart= -usr/bin/agetty -a moth %I $TERM

### REBOOT, LOGIN
    exit
    reboot


### Xorg, i3, sound, misc, fonts
    sudo pacman -S xorg xorg-xinit
    sudo pacman -S i3-gaps rofi rxvt-unicode
    sudo pacman -S alsa-utils
    sudo pacman -S firefox ranger feh git neofetch w3m
    sudo pacman -S terminus-font
    **download polybar from aur**
 
### Xorg Keyboard
    localectl --no-convert set-x11-keymap de pc104 nodeadkeys
    vim ~/.xinitrc
        xset r 250 60 

### Xorg autostart
    vim ~/.xinitrc
        xset r rate 250 60
        exec i3
    vim ~/.bash_profile
        [[ -f ~/.bashrc ]] && . ~/.bashrc
        exec startx
    
    
### install shit with AUR
    git clone https://aur.archlinux.org/polybar.git
    makepkg -si
    
### spotify
    if "one or more pgp signatures could not be verified!"
    gpg --recv-keys 2EBF997C15BDA244B6EBF5D84773BD5E130D1D45
    to spotify directory
    
### Firewall
    sudo pacman -S gufw
    systemctl start ufw.service
    systemctl enable ufw.service
    
### Password authentication popup
    sudo pacman -S polkit-gnome gnome-keyring
    vim .config/i3/config
        exec --no-startup-id /usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1 & eval $(gnome-keyring-daemon -s --components=pkcs11,secrets,ssh,gpg) &
