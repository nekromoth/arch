#### KEYMAP
    loadkeys de 
    
#### SSH ? 
    (wifi-menu)
    systemctl start sshd
    (systemctl status sshd)
    passwd 
    ip addr
    
#### PARTITIONS
    (lsblk)
    gdisk /dev/sda
        * Part 1  512 MiB   ef00  /boot
        * Part 2  remainder 8300  /
          
#### LUKS ENCRYPTION
    modprobe dm-crypt
    cryptsetup -c aes-xts-plain -s 512 -h sha512 -i 4000 luksFormat /dev/sda2
    cryptsetup open /dev/sda2 *cryroot*

#### FORMAT AND MOUNT ROOT
    mkfs.ext4 /dev/mapper/cryroot
    mount /dev/mapper/cryroot /mnt
    
#### FORMAT AND MOUNT EFI
    mkfs.vfat -F32 /dev/sda1
    mkdir /mnt/boot
    mount /dev/sda1 /mnt/boot

** install base system **
    (wifi-menu)
    pacstrap /mnt base base-devel networkmanager linux-lts gvim man-db man-pages texinfo intel-ucode

** gen filesystem table **
    genfstab -pU /mnt >> /mnt/etc/fstab
        * If you have SSD change relatime on all non-boot partitions to noatime.
            vim /mnt/etc/fstab)

** go into new system ** 
    arch-chroot /mnt

** system clock **
    ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime
    hwclock --systohc

** hostname **
    echo moth > /etc/hostname

** root password **
    passwd 

** locales ** 
    echo LANG=en_US.UTF-8 > /etc/locale.conf
    echo KEYMAP=de-latin1-nodeadkeys > /etc/vconsole.conf
    locale-gen

** install bootloader **
    bootctl install

** configure mkinitcpio **
    vim /etc/mkinitcpio.conf 
        HOOKS=(base udev autodetect keyboard keymap modconf block encrypt filesystems fsck)
    mkinitcpio -p linux-lts

** configure bootloader **
    vim /boot/loader/loader.conf
        default arch
        editor 0
        timeout 0
    1234
    
        title   Archlinux
        linux   /vmlinuz-linux-lts
        initrd  /intel-ucode.img
        initrd  /initramfs-linux-lts.img
        options cryptdevice=UUID=<UUID>:cryroot root=/dev/mapper/cryroot rw
