---
layout: post
title: "Software List"
permalink: /arch/software-list
---

* [powerpill](#powerpill)
* ruby
* nodejs
* transmission

***

##[Sound](#sound)

* alsa-utils

***

##[Apps](#apps)

* git
* stow

***

##[Xorg](#xorg)

* xorg-server 
    * xorg-server-utils 
    * xorg-apps 
    * xorg-xinit 
    * mesa-demos 
    * xorg-xterm 
    * xclock
    * nvidia
        * nvidia-libgl
        * lib32-nvidia-libgl

***

##[Desktop](#desktop)

* bspwm 
    * sxhkds
    * slim
    * feh
    * dmenu    
* rxvt-unicode
* udisk2
* thunar
* firefox
* chromium

***

* qemu
	* qemu-arch-extra
* libvirt
    * dmidecode
    * ebtables
    * dnsmasq
    * bridge-utils
    * openbsd-netcat	
    * virt-manager    
    * rpmextract
 
***    
      
## <a name="xorg"></a>Xorg
    
####Install Xorg

    sudo powerpill -S xorg-server xorg-server-utils xorg-apps xorg-xinit mesa-demos xterm xclock

####Install Nvidia
            
    sudo powerpill -S nvidia nvidia-libgl lib32-nvidia-libgl
        
####Configure
 
    cat dotfiles/configs/xorg.keyboard > /etc/X11/xorg.conf.d/00-keyboard.conf
                
    cat dotfiles/configs/xorg.nvidia > /etc/X11/xorg.conf.d/20-nvidia.conf
        
***

## <a name="desktop"></a>Desktop
    
####Install

    sudo powerpill -S bspwm sxhkd slim feh dmenu rxvt-unicode udisk2 thunar firefox chromium
        
####Configure

    cd ~/dotfiles
    stow xinitrc
    stow bspwm
    stow sxhkd
    sudo systemctl enable slim.service
    
***

## <a name="powerpill"></a>Powerpill
 
####Install
    
    sudo cat ~/dotfiles/configs/xyne >> /etc/pacman.conf
    
    sudo pacman -Syy powerpill
    
####Configure

***

## <a name="apps"></a>Apps

####Install

    sudo powerpill -S stow git
    
####Configure

    git pull https://github.com/x01000010/dotfiles    
    rm .bashrc
    cd dotfiles
    stow bash    
    
***

## <a name="sound"></a>Sound

####Install

    sudo powerpill -S alsa-utils
    
####Configure

Run `alsamixer` and unmute the main channel.