# BASE SYSTEM 

**KEYMAP**
   
    loadkeys de 

---

**SSH TARGET**

    (wifi-menu)
    passwd 
    systemctl start sshd
    (systemctl status sshd)
    ip addr
    
**SSH HOST**
    
    ssh root@ip.address.of.target
    
In case your *backspace* doesnt work try: `export TERM=vt100`

---

**PARTITIONS**

    (lsblk)
    gdisk /dev/nvme0n1
    
* `Part 1  512 MiB   ef00  /boot`
* `Part 2  rest 8300  /`
          
**LUKS ENCRYPTION**

    modprobe dm-crypt
    cryptsetup -c aes-xts-plain -s 512 -h sha512 -i 16000 luksFormat /dev/nvme0n1p2
    cryptsetup open /dev/nvme0n1p2 cryroot

**FORMAT AND MOUNT ROOT**

    mkfs.ext4 /dev/mapper/cryroot
    mount /dev/mapper/cryroot /mnt
    
**FORMAT AND MOUNT BOOT EFI**

    mkfs.vfat -F32 /dev/nvme0n1p1
    mkdir /mnt/boot
    mount /dev/nvme0n1p1 /mnt/boot

**INSTALL THE BASE SYSTEM**

    pacstrap /mnt base base-devel networkmanager linux-lts linux-firmware intel-ucode openssh gvim man-db man-pages zsh

**GENERATE FILESYSTEM TABLE**
    
    genfstab -pU /mnt >> /mnt/etc/fstab
    
Change `relatime` on **non-boot** partition (`nvme0n1p2` aka `/dev/mapper/cryroot`) to `noatime`.

    vim /mnt/etc/fstab

**GOTO NEW ARCH-INSTALL**

    arch-chroot /mnt

**SYSTEM CLOCK**

    ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime
    hwclock -l
    
**HOSTNAME**

    echo arch > /etc/hostname
    vim /etc/hosts
        127.0.0.1   localhost
        ::1         localhost
        127.0.1.1   arch.localdomain arch

**ROOT PASSWORD**
    
    passwd 

**USER**
    
    useradd -m -g users -s $(which zsh) moth
    passwd moth
    
**ENABLE SUDO FOR USER**

    EDITOR=vim visudo
    moth ALL=(ALL) ALL
    
**LOCALES**

    vim /etc/locale.gen
        en_US.UTF-8 UTF8
    echo LANG=en_US.UTF-8 > /etc/locale.conf
    echo KEYMAP=de-latin1-nodeadkeys > /etc/vconsole.conf
    locale-gen
    
**INITRAMDISKFS**
    
    vim /etc/mkinitcpio.conf 
        HOOKS=(base udev autodetect keyboard keymap modconf block encrypt filesystems fsck)
    mkinitcpio -p linux-lts

**BOOTLOADER**
    
    bootctl install
    
    vim /boot/loader/loader.conf
        default arch
        editor 0
        console-mode max
        timeout 0
        
    lsblk -f >> /boot/loader/entries/arch.conf
    
    vim /boot/loader/entries/arch.conf
        title   Archlinux
        linux   /vmlinuz-linux-lts
        initrd  /intel-ucode.img
        initrd  /initramfs-linux-lts.img
        options cryptdevice=UUID=>>UUID<<:cryroot root=/dev/mapper/cryroot rw

> Use the UUID from /dev/nvme0n1p2 !
Rembember: `>>UUID<<` is a placeholder as a whole.

**REBOOT** 

    exit
    reboot




# DESKTOP ENVIROMENT

**INTERNET**

    systemctl enable NetworkManager
    systemctl start NetworkManager
    nmtui
    
**AUTOLOGIN**

    sudo mkdir /etc/systemd/system/getty@tty1.service.d
    sudo vim   /etc/systemd/system/getty@tty1.service.d/override.conf
        [Service]
        ExecStart=
        ExecStart= -/usr/bin/agetty -a moth %I $TERM

**DOTFILES**

**DESKTOP SYSTEM**
    
    sudo pacman -S xorg xorg-xinit i3-gaps firefox ranger git terminus-font gufw polkit-gnome gnome-keyring rxvt-unicode pulseaudio pavucontrol dosfstools feh w3m

**XORG KEYBOARD and i3**
    
    localectl --no-convert set-x11-keymap de pc105 nodeadkeys
    
    vim ~/.xinitrc
        xset r rate 250 60 
        exec i3
       
**FIREWALL**
    
    systemctl start ufw.service
    systemctl enable ufw.service
