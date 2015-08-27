---
layout: post
title: "Arch Install"
date: 2015-08-01
permalink: /linuxadmin/linux-install
---

* [ToDo](#todo)


***

##PreInstall

####Change keyboard layout to dvorak

    loadkeys dvorak

####Set up partitions

    wipefs -af /dev/sda
    wipefs -af /dev/sdb

    parted /dev/sdb mktable gpt
    parted /dev/sdb mkpart primary linux-swap 1MiB 20GiB
    parted /dev/sdb mkpart primary  ext3 20GiB 100%

####Format, mount, and create subvolumes

    mkfs.btrfs -d single /dev/sda /dev/sdb2
    mkswap /dev/sdb1
    swapon /dev/sdb1

    mount /dev/sda /mnt
    btrfs subvolume create /mnt/boot
    btrfs subvolume create /mnt/home
    btrfs subvolume create /mnt/var
    btrfs subvolume create /mnt/data

####Setup mirrors

    pacman -Syy reflector --noconfirm
    reflector --verbose --country 'United States' -l 20 -p http --sort rate --save /etc/pacman.d/mirrorlist

***

##Install

####Install base and dev tools

    pacstrap /mnt base base-devel
    
####Create fstab

    genfstab -U /mnt > /mnt/etc/fstab
    
####Chroot into system    
    
    arch-chroot /mnt /bin/bash

***

##Configure

####Disable pc speaker

    rmmod pcspkr
    echo "blacklist pcspkr" > /etc/modprobe.d/nobeep.conf

####Setup locale

    nano /etc/locale.gen
    
uncomment `en_US.UTF-8 UTF-8`
    locale-gen
    
    echo LANG=en_US.UTF-8 > /etc/locale.conf

####Setup console keymap and font
                        
    echo KEYMAP=dvorak > /etc/vconsole.conf
    echo FONT=lat9w-16 >> /etc/vconsole.conf
    
####Set the time zone and clock
    
    ln -sf /usr/share/zoneinfo/US/Arizona /etc/localtime
    hwclock --systohc --utc
    
####Set the hostname
    echo capricorn13 > /etc/hostname
    
add the hostname to `/etc/hosts`
    
####Install needed apps    
    
    pacman -Syy brtfs-progs grub
    
####mkinitcpio

    mkinitcpio -p linux
    
####Set root password

    passwd
    
####Set up the bootloader
    
    grub-install --recheck /dev/sda
    grub-mkconfig -o /boot/grub/grub.cfg
    
    exit
    umount /mnt
    reboot


***

##Post

log in as root

####set up user

    useradd -m -G wheel -s /bin/bash x01000010

####edit sudoers

    nano /etc/sudoers

    exit

log in as user
   
####setup dhcp
    
Get the inerface name with `sudo ip link`

    systemctl enable dhcpcd@interface_name.service

####Pacman

Run `sudo nano /etc/pacman.conf` and uncomment multilib

Update the system with `sudo pacman -Syyu`

***

##Software

* [apps]({% post_url 2015-08-09-software-list %}#apps)
* [powerpill]({% post_url 2015-08-09-software-list %}#powerpill)
* [sound]({% post_url 2015-08-09-software-list %}#sound)
* [Xorg]({% post_url 2015-08-09-software-list %}#xorg)
* [Desktop]({% post_url 2015-08-09-software-list %}#desktop)

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
			
