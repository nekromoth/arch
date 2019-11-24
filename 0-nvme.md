#### KEYMAP
    loadkeys de 
    
#### SSH ? 
    (wifi-menu)
    systemctl start sshd
    (systemctl status sshd)
    passwd 
    ip addr
Incase your BACKSPACE wont work try:

    export TERM=vt100
    
#### PARTITIONS
    (lsblk)
    gdisk /dev/nvme0n1
        * Part 1  512 MiB   ef00  /boot
        * Part 2  remainder 8300  /
          
#### LUKS ENCRYPTION
    modprobe dm-crypt
    cryptsetup -c aes-xts-plain -s 512 -h sha512 -i 16000 luksFormat /dev/nvme0n1p2
    cryptsetup open /dev/nvme0n1p2 cryroot

#### FORMAT AND MOUNT ROOT
    mkfs.ext4 /dev/mapper/cryroot
    mount /dev/mapper/cryroot /mnt
    
#### FORMAT AND MOUNT EFI
    mkfs.vfat -F32 /dev/nvme0n1p1
    mkdir /mnt/boot
    mount /dev/nvme0n1p1 /mnt/boot

#### INSTALL THE BASE SYSTEM
    pacstrap /mnt base base-devel networkmanager linux-lts linux-firmware intel-ucode openssh gvim man-db man-pages texinfo

#### GENERATE FILESYSTEM TABLE
    genfstab -pU /mnt >> /mnt/etc/fstab
    
If you have a SSD change ```relatime``` on all **non-boot** partitions to ```noatime```.

    vim /mnt/etc/fstab

#### CHANGE ROOT
    arch-chroot /mnt

#### SYSTEM CLOCK
    ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime
    hwclock -s
    
#### HOSTNAME
    echo arch > /etc/hostname
    vim /etc/hosts
        127.0.0.1   localhost
        ::1         localhost
        127.0.1.1   arch.localdomain arch
Change `arch` to your hostname.

#### ROOT PASSWORD
    passwd 

#### USER
    useradd -m -g users -s /bin/bash moth
    passwd moth
    
#### SUDO 
    EDITOR=vim visudo
    moth ALL=(ALL) ALL
    
#### LOCALES
    vim /etc/locale.gen
        en_US.UTF-8 UTF8
    echo LANG=en_US.UTF-8 > /etc/locale.conf
    echo KEYMAP=de-latin1-nodeadkeys > /etc/vconsole.conf
    locale-gen
    
#### INITRAMDISKFS
    vim /etc/mkinitcpio.conf 
        HOOKS=(base udev autodetect keyboard keymap modconf block encrypt filesystems fsck)
    mkinitcpio -p linux-lts

#### BOOTLOADER
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
        options cryptdevice=UUID=<UUID>:cryroot root=/dev/mapper/cryroot rw
**Use the UUID from /dev/nvme0n1p2 !**
