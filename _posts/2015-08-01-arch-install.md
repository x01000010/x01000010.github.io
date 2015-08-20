---
layout: post
title: "Arch Install"
date: 2015-08-01
permalink: /linuxadmin/linux-install
---


###preinstall


    keyboard
        loadkeys dvorak
    partition
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




###Install

<body>
		<h1>Install</h1>
		<ul>
			<li>install<ul>
					<li>
						<code>pacstrap /mnt base base-devel</code>
					</li>
				</ul>
			</li>
			<li>fstab<ul>
					<li>
						<code>genfstab -U /mnt > /mnt/etc/fstab</code>
					</li>
				</ul>
			</li>
			<li>chroot<ul>
					<li>
						<code>arch-chroot /mnt /bin/bash</code>
					</li>
				</ul>
			</li>
		</ul>
	</body>

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
    dhcp
        ip link
        systemctl enable dhcpcd@interface_name.service
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


    yaourt
    bspwm
    feh
    dmenu
    libvirt
    /etc/polkit-1/rules.d/49-org.libvirt.unix.manager.rules /* Allow users in kvm group to manage the libvirt daemon without authentication */ polkit.addRule(function(action, subject) { if (action.id == "org.libvirt.unix.manage" && subject.isInGroup("kvm")) { return polkit.Result.YES; } });
    tint2
    slim

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

###BSPWM
    Install
        sudo powerpill -S bspwm sxhkd
        nano ~/.xinitrc
            comment out all exec calls
            sxhkd &
            exec bspwm
    Configuration
        mkdir -p ~/.config/bspwm
        mkdir -p ~/.config/sxhkd
        cd
        wget https://raw.githubusercontent.com/baskerville/bspwm/master/examples/bspwmrc
        mv bspwmrc .config/bspwm
        chmod +x ~/.config/bspwm/bspwmrc
        wget https://raw.githubusercontent.com/baskerville/bspwm/master/examples/sxhkdrc
        mv sxhkdrc .config/sxhkd
    Multi-monitor setup
        The example bspwmrc configures ten desktops on one monitor like this: bspc monitor -d I II III IV V VI VII VIII IX X You will need to change this line and add one for each monitor, similar to this: bspc monitor DVI-I-1 -d I II III IV bspc monitor DVI-I-2 -d V VI VII bspc monitor DP-1 -d VIII IX X You can use `xrandr -q` or `bspc query -M` to find the monitor names. The total number of desktops were maintained at ten in the above example. This is so that each desktop can still be addressed with 'super + {1-9,0}' in the sxhkdrc.

###Slim

    Install
        sudo pacman -S slim
    Configure
        sudo systemctl enable slim.service


###yauort

    setup
        mkdir AUR
        cd AUR
        pacman -S wget
    install
        wget https://aur.archlinux.org/packages/pa/package-query/package-query.tar.gz
        wget https://aur.archlinux.org/packages/ya/yaourt/yaourt.tar.gz
        tar -xvf package-query.tar.gz
        tar -xvf yaourt.tar.gz
        cd package-query
        makepkg -sri
        cd ../yaourt
        makepkg -sri




###TODO

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



			password prompt githum off to right side
			
			automount usb drive
			wine
			
			zsh
			irssi
			vim
			
			btsync
			mouse pointer delay in showing up
			brindge newrok
			internal network
