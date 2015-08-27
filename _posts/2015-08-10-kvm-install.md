---
layout: post
title: "KVM Install"
date: 2015-08-10
permalink: /linuxadmin/kvm
---

Description: Install KVM Hypervisor


<div style="display: inline-block">
<div class ="panel panel-default" style="border-style: solid">
<div class="panel-heading">Contents</div>
<div class="panel-body">
<a href="#install">Install</a><br />
<a href="#configure">Configure</a><br />
<a href="#network">Network</a><br />
<a href="#todo">ToDo</a><br />
<a href="#bonus">Bonus</a>
</div>
</div>
</div>

For my KVM Hypervisor I decided to use Arch linux with KVM+QEMU and Libvirt for a front end.

My Arch setup [here](/arch).

***

## <a name="install"></a>Install

     sudo powerpill -S qemu libvirt dmidecode ebtables dnsmasq bridge-utils openbsd-netcat virt-manager

***

## <a name="configure"></a>Configure

    sudo usermod -a -G kvm <user>
    
    sudo nano /etc/polkit-1/rules.d/49-org.libvirt.unix.manager.rules 
    
    /* Allow users in kvm group to manage the libvirt daemon without authentication */
    polkit.addRule(function(action, subject) {
        if (action.id == "org.libvirt.unix.manage" && 
            subject.isInGroup("kvm")) { 
                return polkit.Result.YES; 
        } 
    });
    
####Test

    virsh -c qemu:///system
    virsh -c qemu:///session

***

##<a name="network"></a>Network

* [source](http://wiki.libvirt.org/page/VirtualNetworking)
* [source2](http://wiki.libvirt.org/page/Networking)

***

## <a name="todo"></a>ToDo

* network setup
	* bridge
	* host only
* kvm windows no re activate

***

### <a name="bonus"></a>Bonus: Passthrough video card

Try not passing the audio device through

flash uefi to card

match cpus `info cpus`

* [arch wiki: passthrough](https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF)
* [Alex's Blog](http://vfio.blogspot.com)
* [arch forum post](https://bbs.archlinux.org/viewtopic.php?id=162768)
* [uefi support for gpu](https://bbs.archlinux.org/viewtopic.php?pid=1504683#p1504683)
* [archive and other sources](http://arseniyshestakov.github.io/vfio-archive/)



***

### <a name="sources"></a>Sources

* [arch wiki: qemu](https://wiki.archlinux.org/index.php/QEMU)
* [arch wiki: libvirt](https://wiki.archlinux.org/index.php/Libvirt)
