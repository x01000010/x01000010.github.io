---
layout: post
title: "Arch Install"
date: 2015-08-01
permalink: /linuxadmin/linux-install
---

* [ToDo](#todo)


***

###PreInstall

Change keyboard layout to dvorak

    loadkeys dvorak

Set up partitions

 wipefs -af /dev/sda
 wipefs -af /dev/sdb
        fdisk /dev/sdb
            g
            n
            <enter>
            <enter>
            +(total size - 15G)G
            n
            <enter>
            <enter>
            <enter>
            w
    format
        mkfs.btrfs -d single /dev/sda /dev/sdb1
        mkswap /dev/sdb2
        swapon
    mount
        mount /dev/sda /mnt
        btrfs subvolume create /mnt/boot
        btrfs subvolume create /mnt/home
        btrfs subvolume create /mnt/var
        btrfs subvolume create /mnt/data
    mirror list
        reflector
            pacman -S reflector
            reflector --verbose --country 'United States' -l 20 -p http --sort rate --save /etc/pacman.d/mirrorlist

***

###Install

pacstrap /mnt base base-devel
genfstab -U /mnt > /mnt/etc/fstab
arch-chroot /mnt /bin/bash

***

###Configure

    disable pc speaker
        rmmod pcspkr
        echo "blacklist pcspkr" > /etc/modprobe.d/nobeep.conf
    locale
        locale-gen
            nano /etc/locale.gen
            en_US.UTF-8 UTF-8
            locale-gen
            echo LANG=en_US.UTF-8 > /etc/locale.conf
        console keymap and font
            nano /etc/vconsole.conf
            KEYMAP=dvorak
            FONT=lat9w-16
    btrfs tools
        pacman -Syy brtfs-progs
    time zone
        ln -sf /usr/share/zoneinfo/US/Arizona /etc/localtime
    Clock
        hwclock --systohc --utc
    hostname
        echo capricorn13 > /etc/hostname
        nano /etc/hosts
    
    mkinitcpio
        mkinitcpio -p linux
    root password
        passwd
    bootloader
        pacman -S grub
        grub-install --recheck /dev/sda
        grub-mkconfig -o /boot/grub/grub.cfg
    exit
    umount /mnt
    reboot


###Software


  

###Post

    log in as root
        user
            useradd -m -G wheel -s /bin/bash x01000010
            edit sudoers
            exit
    log in as user
    powerpill
        sudo nano /etc/pacman.conf
            [xyne-x86_64]
            # A repo for Xyne's own projects: http://xyne.archlinux.ca/projects/
            # Packages for the "x86_64" architecture.
            # Note that this includes all packages in [xyne-any].
            SigLevel = Required
            Server = http://xyne.archlinux.ca/repos/xyne
            uncomment multilib
        sudo pacman -Syy powerpill
    sudo powerpill -S wget
    xorg
        Install Xorg
            sudo powerpill -S xorg-server xorg-server-utils xorg-apps xorg-xinit mesa-demos xterm xclock
        Install Nvidia
            sudo powerpill -S nvidia nvidia-libgl lib32-nvidia-libgl
        Configure
            nano /etc/X11/xorg.conf.d/00-keyboard.conf
                Section "InputClass"
                Identifier "system-keyboard"
                MatchIsKeyboard "on"
                Option "XkbLayout" "dvorak"
                Option "XkbModel" "pc104"
                EndSection
            nano /etc/X11/xorg.conf.d/20-nvidia.conf
                Section "Device"
                Identifier "Nvidia Card"
                Driver "nvidia"
                VendorName "NVIDIA Corporation"
                Option "NoLogo" "true"
                EndSection
        check /usr/share/X11/xorg.conf.d/
        cp /etc/X11/xinit/xinitrc ~/.xinitrc
        test: startx

dhcp
        ip link
        systemctl enable dhcpcd@interface_name.service





***

### <a name="todo"></a>TODO

* programs to use with bspwm
    * compton/librgba
    * conky/archy
    * panels/tint2/lemonbar
    * feh/hsetroot 
    * xbindkeys
* tab complete with sudo
* dotfile
* dhcp not starting
* audio
    * alsa-utils alsamixer
* ntp setup
* password prompt githum off to right side
* automount usb drive
* wine
* zsh
* irssi
* vim
* automount usb drive
* btsync
* mouse pointer delay in showing up
* yaourt
* change xrvt font
			
