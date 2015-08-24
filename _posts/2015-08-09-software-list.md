---
layout: post
title: "Software List"
permalink: /arch/software-list
---

* firefox
* chromium
* powerpill
* git
* ruby
* nodejs
* transmission
* shotwell

***

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

* bspwm 
    * sxhkds
    * [slim](#slim)
    * feh
    * dmenu
    * stow
* rxvt-unicode
* udisk2
* thunar

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
    * ovf
    
* rpmextract


 
***    
    
    
### <a name="bspwm"></a>BSPWM
    
Install

    sudo powerpill -S bspwm sxhkd
        
Configure

    cd ~/dotfiles
    stow xinitrc
    stow bspwm
    stow sxhkd
    
***

### <a name="slim"></a>Slim

Install

    sudo powerpill -S slim
    
Configure

    sudo systemctl enable slim.service
        
***
