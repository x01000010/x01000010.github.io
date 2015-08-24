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

### <a name="install"></a>Install

Install

     powerpill -S qemu libvirt dmidecode ebtables dnsmasq bridge-utils openbsd-netcat virt-manager

***

### <a name="configure"></a>Configure

Configure

    sudo usermod -a -G kvm <user>
    
    sudo nano /etc/polkit-1/rules.d/49-org.libvirt.unix.manager.rules 
    
    /* Allow users in kvm group to manage the libvirt daemon without authentication */
    polkit.addRule(function(action, subject) {
        if (action.id == "org.libvirt.unix.manage" && 
            subject.isInGroup("kvm")) { 
                return polkit.Result.YES; 
        } 
    });
    
Test

    virsh -c qemu:///system

    virsh -c qemu:///session

***

### <a name="todo"></a>ToDo

* network setup
	* bridge
	* host only
* kvm windows no re activate

***

### <a name="bonus"></a>Bonus: Passthrough video card

[passthrough](https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF)

####Installation

Install rpmextract.
Install edk2.git-ovmf-x64 from Gerd Hoffman's repository.
Extract that archive to /usr:
# rpmextract.sh edk2.git-ovmf-x64-0-20150223.b877.ga8577b3.noarch.rpm
# cp -R ./usr/share/* /usr/share
Ensure /usr/share/edk2.git/ovmf-x64 contains these files:
$ ls /usr/share/edk2.git/ovmf-x64/*pure*.fd
/usr/share/edk2.git/ovmf-x64/OVMF-pure-efi.fd
/usr/share/edk2.git/ovmf-x64/OVMF_VARS-pure-efi.fd
/usr/share/edk2.git/ovmf-x64/OVMF_CODE-pure-efi.fd
This will be the BIOS that the VM will use. Non-UEFI users may need to use i440fx without OVMF, and the i915 vga arbiter patch for Intel graphics as host, see this forum thread. For users that do have a UEFI compatible motherboard but a UEFI incompatible graphics card, look at this post.


Enabling IOMMU

Ensure that AMD-VI/VT-D is enabled in your BIOS settings.
If your processor is Intel, add intel_iommu=on to your bootloader kernel options. 

After rebooting, check dmesg to confirm IOMMU is enabled:
dmesg|grep -e DMAR -e IOMMU
[    0.000000] ACPI: DMAR 0x00000000BDCB1CB0 0000B8 (v01 INTEL  BDW      00000001 INTL 00000001)
[    0.000000] Intel-IOMMU: enabled
[    0.028879] dmar: IOMMU 0: reg_base_addr fed90000 ver 1:0 cap c0000020660462 ecap f0101a
[    0.028883] dmar: IOMMU 1: reg_base_addr fed91000 ver 1:0 cap d2008c20660462 ecap f010da
[    0.028950] IOAPIC id 8 under DRHD base  0xfed91000 IOMMU 1
[    0.536212] DMAR: No ATSR found
[    0.536229] IOMMU 0 0xfed90000: using Queued invalidation
[    0.536230] IOMMU 1 0xfed91000: using Queued invalidation
[    0.536231] IOMMU: Setting RMRR:
[    0.536241] IOMMU: Setting identity map for device 0000:00:02.0 [0xbf000000 - 0xcf1fffff]
[    0.537490] IOMMU: Setting identity map for device 0000:00:14.0 [0xbdea8000 - 0xbdeb6fff]
[    0.537512] IOMMU: Setting identity map for device 0000:00:1a.0 [0xbdea8000 - 0xbdeb6fff]
[    0.537530] IOMMU: Setting identity map for device 0000:00:1d.0 [0xbdea8000 - 0xbdeb6fff]
[    0.537543] IOMMU: Prepare 0-16MiB unity mapping for LPC
[    0.537549] IOMMU: Setting identity map for device 0000:00:1f.0 [0x0 - 0xffffff]
[    2.182790] [drm] DMAR active, disabling use of stolen memory
Isolating the GPU

Find your target card's PCI location:

lspci -nn|grep -iP "NVIDIA|Radeon"
01:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Cayman PRO [Radeon HD 6950] [1002:6719]
01:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Cayman/Antilles HDMI Audio [Radeon HD 6900 Series] [1002:aa80]
04:00.0 VGA compatible controller [0300]: NVIDIA Corporation G73 [GeForce 7600 GS] [10de:0392] (rev a1)

In this case, the three locations we're after are 1002:6719 1002:aa80 10de:0392. Make note of any locations you intend to pass through to the VM.
To allow the VM access to your passthrough'd devices, you'll need to claim it before the host drivers do. This can be achieved with either one of two kernel modules, vfio-pci or pci-stub.
vfio-pci is available in kernel v4.1+, and is the recommended option if your kernel supports it. You can check if this module is available by running:
$ modprobe vfio-pci
If there is no output, you're good to go. If instead you recieve modprobe: FATAL: Module vfio-pci not found, use the guide further down for pci-stub instead.
vfio-pci
Ensure the vfio-pci driver is loaded on bootup through modprobe.d. It is necessary to tell the vfio-pci driver which PCI devices to bind. If you were adding all three of the PCI devices listed above, your modprobe.d launch config would have the following contents:
/etc/modprobe.d/vfio.conf
options vfio-pci ids=1002:6719,1002:aa80,10de:0392
Next we'll want to ensure the kernel loads the necessary modules for vfio-pci when starting up:
/etc/mkinitcpio.conf
...
MODULES="vfio vfio_iommu_type1 vfio_pci vfio_virqfd"
...
Save the changes to the initial ramdisk environment:
# mkinitcpio -p linux
Note: If you're using a non-standard kernel, replace "linux" with whichever kernel you intend to use.
Finally, we need to tell grub to load the vfio-pci module first:
/etc/mkinitcpio.conf
...
GRUB_CMDLINE_LINUX_DEFAULT="... rd.driver.pre=vfio-pci ..."
...
Reload the grub configuration:
# grub-mkconfig -o /boot/grub/grub.cfg
Reboot and check dmesg output for successful assignment of the device(s) to vfio-pci:
dmesg | grep -i vfio
...
[    0.456472] VFIO - User Level meta-driver version: 0.3
[    0.470269] vfio_pci: add [10de:13c2[ffff:ffff]] class 0x000000/00000000
[    0.483631] vfio_pci: add [10de:0fbb[ffff:ffff]] class 0x000000/00000000
[    0.496998] vfio_pci: add [8086:8ca0[ffff:ffff]] class 0x000000/00000000
[    2.420184] vfio-pci 0000:0a:00.0: enabling device (0000 -> 0003)
[   38.590395] vfio_ecap_init: 0000:0a:00.0 hiding ecap 0x1e@0x258
[   38.590413] vfio_ecap_init: 0000:0a:00.0 hiding ecap 0x19@0x900
[   38.606881] vfio-pci 0000:0a:00.1: enabling device (0000 -> 0002)
[   38.620241] vfio-pci 0000:00:1b.0: enabling device (0000 -> 0002)
...
You can cross reference devices id's with lspci -nn's output.


Blacklisting modules
/etc/modprobe.d/modprobe.conf
blacklist radeon


Binding to VFIO
There are many methods to bind the card to vfio, here is one example:
firewing1 webpage. Check the part after grub2-mkconfig.


IOMMU groups

Only complete IOMMU groups can be attached to the guest VM. To see which groups each of your PCI devices are assigned to:
# find /sys/kernel/iommu_groups/ -type l
/sys/kernel/iommu_groups/0/devices/0000:00:00.0
/sys/kernel/iommu_groups/1/devices/0000:00:01.0
/sys/kernel/iommu_groups/1/devices/0000:01:00.0
/sys/kernel/iommu_groups/1/devices/0000:01:00.1
/sys/kernel/iommu_groups/2/devices/0000:00:02.0
/sys/kernel/iommu_groups/3/devices/0000:00:16.0
/sys/kernel/iommu_groups/4/devices/0000:00:1a.0
/sys/kernel/iommu_groups/5/devices/0000:00:1b.0
/sys/kernel/iommu_groups/6/devices/0000:00:1c.0
/sys/kernel/iommu_groups/7/devices/0000:00:1c.5
/sys/kernel/iommu_groups/8/devices/0000:00:1c.6
/sys/kernel/iommu_groups/9/devices/0000:00:1c.7
/sys/kernel/iommu_groups/9/devices/0000:05:00.0
/sys/kernel/iommu_groups/10/devices/0000:00:1d.0
/sys/kernel/iommu_groups/11/devices/0000:00:1f.0
/sys/kernel/iommu_groups/11/devices/0000:00:1f.2
/sys/kernel/iommu_groups/11/devices/0000:00:1f.3
/sys/kernel/iommu_groups/12/devices/0000:02:00.0
/sys/kernel/iommu_groups/12/devices/0000:02:00.1
/sys/kernel/iommu_groups/13/devices/0000:03:00.0
/sys/kernel/iommu_groups/14/devices/0000:04:00.0



QEMU permissions

Give QEMU access to hardware (there may be safer ways of doing this):
/etc/libvirt/qemu.conf
...
user = "root"
group = "root"
clear_emulator_capabilities = 0


QEMU also needs acces to VFIO files. Include every numbered file in /dev/vfio:
ls -1 /dev/vfio
/etc/libvirt/qemu.conf
...
cgroup_device_acl = [
    "/dev/null", "/dev/full", "/dev/zero",
    "/dev/random", "/dev/urandom",
    "/dev/ptmx", "/dev/kvm", "/dev/kqemu",
    "/dev/rtc","/dev/hpet", "/dev/vfio/vfio",
    "/dev/vfio/11"
]
...

Referenced from firewing1's webpage.



***

### <a name="sources"></a>Sources

* [arch wiki: qemu](https://wiki.archlinux.org/index.php/QEMU)
* [arch wiki: libvirt](https://wiki.archlinux.org/index.php/Libvirt)
