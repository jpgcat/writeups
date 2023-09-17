## Introduction

VFIO (Virtual Function I/O) is a Linux Kernel subsystem that acts as a framework for directing devices to applications and virtual machines, which allows them to interact with other hardware devices at a low level. VFIO is mainly useful for virtual machines and environments, where you can pass physical devices such as your graphic card, your network interfaces, or more, with near to native performance.

Being a die-hard Linux fan since the first time I started using it, many of my drawbacks were not being able to properly get a VFIO setup working, or working well. Due to my incompetence and not quite doing enough research I failed to keep myself on a Linux operating system and defaulted back to Windows. I am not really a gamer of such, instead using my computer for mostly educational purposes, but I do enjoy the odd game from time to time. Being laser focused on security while also needing many Microsoft applications for my education, VFIO was the best option for me, it allowed me to create virtual environments with my graphics card for near-native performance. I have single virtual machines for education, gaming, and development, but you could do it however you want.

**Why not Dual-boot?** Dual booting is a time consuming process, for example, imagine I wanted to play a game in the midst of writing a report. It is very annoying for you to switch your computer off just to boot into a separate system for singular actions like playing games or doing your work. Linux can be made to work for you so why not spend that extra time tweaking it to suit you?

Please take note that all the steps I provide are from a clean setup with no prior installation, I am going to be taking you from the very start of setting up Arch Linux to the very end of your virtual machine setup. Certain steps may be subject to change through iterations of the Linux kernel or to be tweaked to your liking, please at least have some prior understanding of a terminal before using this. **To fully do this you will need two GPU's, an x86 processor, and a substantial amount of RAM (16 - 32GB+).**

## Process

__If you are starting from fresh__ please make sure to back up any important files (Eg. [KeepassXC](https://github.com/keepassxreboot/keepassxc) Databases, Text Files, Important Browser Data, Screenshots) and store them, or zip them up and forget about them, in a separate drive that you can access later on. 

First, get the application named [Ventoy](https://github.com/ventoy/Ventoy) from its [official download page](https://www.ventoy.net/en/download.html), Ventoy allows you to create boot-able USB's which can store as much ISO's as your USB can handle in storage. You can also use different tools but I recommend this one for starters. First, download the installation package which will be named something like `ventoy-x.x.xx-windows.zip` and extract it.
Now run `Ventoy2Disk.exe`, select the USB you want to use and click `Install`. After this has completed you will find your USB now has 2 partitions, find it in the Windows file manager and check that you have a `Ventoy` disk.

![Image](https://raw.githubusercontent.com/jpgcat/writeups/main/Arch%20Linux%20VFIO%20(Windows)/Pasted%20image%2020230916063348.png)

Secondly, get the latest Arch Linux ISO from the [official download page](https://archlinux.org/download/) and store it onto the `Ventoy` disk partition. Once it has all installed double check to make sure you have saved every important file, and also check that the USB can be safely removed before continuing (if you pull it out too early the data might have not been saved fully yet which will lead to a lot of confusion later on). Next, reboot your computer and select the USB stick you installed Ventoy into as your boot device and select Arch Linux when Ventoy has loaded.

![Image](https://raw.githubusercontent.com/jpgcat/writeups/main/Arch%20Linux%20VFIO%20(Windows)/Pasted%20image%2020230916063800.png)

If you are familiar with terminals this is your chance for your skills to shine, if not then don't worry, it is not as scary as it looks. Assuming you have the latest Arch Linux ISO run the following command in the terminal `archinstall`, this will bring up a GUI-like interface for you to install Arch Linux through. I am not going to walk you through installing Arch Linux using [archinstall](https://wiki.archlinux.org/title/archinstall) (there are plenty of thorough tutorials online and it is not that hard to navigate at all), instead I am going to tell you to make sure you do a few things during the install. **Please make sure you are [encrypting the drive](https://en.wikipedia.org/wiki/Disk_encryption#Full_disk_encryption) you are installing Linux to, this will add an extra layer of security and frankly should be a norm. Please also make sure you are choosing [proprietary NVIDIA drivers over open-source NVIDIA drivers](https://github.com/NVIDIA/open-gpu-kernel-modules/discussions/457) as proprietary just works better. Then finally choose [Pipewire](https://wiki.archlinux.org/title/PipeWire) for the audio drivers, [systemd-bootctl](https://wiki.archlinux.org/title/Systemd-boot) for the boot-loader, and [GNOME](https://wiki.archlinux.org/title/GNOME) as your desktop environment, these 3 steps will make your install experience a lot more easier and you can swap these out later if you prefer.**

![Image](https://raw.githubusercontent.com/jpgcat/writeups/main/Arch%20Linux%20VFIO%20(Windows)/Pasted%20image%2020230916064943.png)

Once Arch Linux is fully installed and you can access your system, it is time to tweak your VFIO setup so you can passthrough your second GPU. First check your devices using `lspci -nnk` in the Terminal, this will allow you to check your groups and get the IDs retaining to your GPU. Once you have found the correct GPU, keep that Terminal open and open a second Terminal and enter the command `vim /boot/loader/entries/*-*_linux.conf` you may need [Vim](https://www.freecodecamp.org/news/vim-beginners-guide/) which you can install through `sudo pacman -S vim`. In the config file find the line that starts with `options` and enter the following parameters to the end of the file `iommu=1 amd_iommu=on vfio-pci.ids= quiet splash` (swap out `amd_iommu` for `intel_iommu` if you are using an Intel CPU). Go back to the terminal that you ran `lspci -nnk` on and find the IDs. They should look something like this;  ![[Pasted image 20230916070359.png]]
Once you have found the correct IDs go back to the Terminal with the boot loader config open and append the IDs to the end of `vfio-pci.ids=` separated by commas (example: `vfio-pci.ids=10de:1aed,10de1aec,10de:1aeb,10de:2184`) and then save the file.

Now run the command `vim /etc/mkinitcpio.conf` and find the line that starts with `MODULES=(` and enter this into the brackets `vfio_pci vfio vfio_iommu_type1` then find the line that starts with `HOOKS=(` and check that the brackets have `modconf` inside of them somewhere if it does then save the file.

Now run the command `sudo mkinitcpio -p linux` ([mkinitcpio](https://wiki.archlinux.org/title/mkinitcpio)) this will regenerate your [initramfs](https://wiki.archlinux.org/title/Arch_boot_process#initramfs). You have now completed adding VFIO to your Linux system.

Now run these following commands one after the other ([libvirt](https://wiki.archlinux.org/title/libvirt), [qemu](https://wiki.archlinux.org/title/QEMU));
`sudo pacman -Syu qemu libvirt virt-manager qemu-arch-extra dnsmasq bridge-utils`
`sudo systemctl enable libvirtd`
`sudo gpasswd -a $USER libvirt`
`sudo gpasswd -a $USER kvm`
and then reboot your system.

***Upon rebooting you may notice a completely black screen instead of a passphrase screen, this is fine it is just a bug, if this happens just enter your password as normal and press enter, your computer should start booting up soon after.***

Once your computer has started, check to make sure your GPU is now using the correct drivers with the `lspci -nnk` command it should say the following under the GPU name `Kernel driver in use: vfio-pci`. If not I recommend defaulting to the [Arch Wiki](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF) with references from this write-up and checking that you have not misread anything.

Once you have double checked that your GPU is using the correct drivers, install the correct packages for [Looking Glass](https://looking-glass.io/) onto your system through this following command;
```bash
pacman -Syu cmake gcc libgl libegl fontconfig spice-protocol make nettle pkgconf binutils \
            libxi libxinerama libxss libxcursor libxpresent libxkbcommon wayland-protocols \
            ttf-dejavu libsamplerate
```
Now download Looking Glass Source from the [official site](https://looking-glass.io/artifact/stable/source), first `unzip` the archive and then `cd` into it and run the following commands;
`mkdir client/build`
`cd client/build`
`cmake ../`
`make`
`make install`
There may be a few (extra) packages that you are required to install if this happens search the package name up followed by Arch Linux.

We will now need to set permissions for Looking Glass we can do this by creating a Looking Glass config file. Run the command `vim /etc/tmpfiles.d/10-looking-glass.conf` and enter the following into it;
```shell
# Type Path               Mode UID  GID Age Argument

f /dev/shm/looking-glass 0660 user kvm -
```
Now reboot your system.

Once your system is back, open Virtual Machine Manager ([virt-manager](https://wiki.archlinux.org/title/virt-manager)) and go to Preferences and enable XML Editing. Now we can start creating our Virtual Machine, first make sure you have a Windows ISO, if you are using a VPN consider getting your ISO from [Massgravel](https://massgrave.dev/genuine-installation-media.html) instead of the [official Microsoft download page](https://www.microsoft.com/en-us/software-download).

Once you have your ISO, go through and create a VM using virt-manager. Select your ISO through the Browse Local button, set the memory to be at least 8GB, your disk size to be at least 120GB and leave the vCPU config as is. Configure your virtual machine before installing and remove the network device, this will allow you to skip Microsofts account setup during the Windows install. Also choose `OVMF_CODE.secboot.fd` as your CPU firmware under the Overview tab, then click Begin Installation.

![Image](https://raw.githubusercontent.com/jpgcat/writeups/main/Arch%20Linux%20VFIO%20(Windows)/Pasted%20image%2020230916073114.png)

Once the VM has booted up into the Windows setup page select `English (International)` as your region, this will remove 'most' bloatware and 'most' advertising while going through driver setups like NVIDIA. Follow through with the install and select Windows (your version) Pro just so you have the most flexibility while using Windows. Windows should tell u that your computer does not meet the requirements, from here press the **Shift + F10** keys and follow the below;
- Type in `regedit` to the command prompt.
- Navigate to `HKEY_LOCAL_MACHINE \ System \ Setup` in [Registry Editor](https://en.wikipedia.org/wiki/Windows_Registry).
- Left click on the list and add a new Key named `LabConfig`.
- Left click in the new folder you added named `LabConfig` and add the following values as separate `DWORD (32-bit)` values; `BypassTPMCheck`, `BypassRAMCheck`, `BypassSecureBootCheck` and set them all to `1` instead of `0`.
- Exit Registry Editor and Command Prompt.
Now you can click the back button and re-select Windows (your version) Pro and it should start the install process.

On your first reboot during the setup it will take a while to load the [OOBE](https://en.wikipedia.org/wiki/Out-of-box_experience) page then throw an error, this is fine, just do not click retry. Now follow through with the install till the end. While the Windows setup is running we should install the [VirtIO](https://www.linux-kvm.org/page/Virtio) drivers from the [install page](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso).

Once Windows has successfully booted, shutdown the VM and add a network card to it through Add Devices. Now create a 0.1GB VirtIO CD-ROM disk with nothing in it and then create a Floppy Disk utilizing the VirtIO ISO you downloaded earlier.

Now start-up your VM again, once Windows has booted go to Files and look for the VirtIO Disk and Install the `virtio-win-gt-x64` drivers.

![Image](https://raw.githubusercontent.com/jpgcat/writeups/main/Arch%20Linux%20VFIO%20(Windows)/Pasted%20image%2020230916075848.png)

Now it is best to make sure your Windows system is up to date, run the Windows Update tool and update your Windows system. Once updating has completed reboot your system again.

Now we are ready to install the three most critical programs on our Windows host.
- [The latest NVIDIA drivers](https://www.nvidia.com/download/index.aspx) - this will allow the host to use the GPU effectively.
- [Looking Glass (Host)](https://looking-glass.io/artifact/stable/host) - this will connect the host to your client.
- [SPICE Guest Tools](https://www.spice-space.org/download/windows/spice-guest-tools/spice-guest-tools-latest.exe) - this will allow for tools like clipboard synchronization between the host and the client.

You can now shutdown your VM, tweak it to your liking, then use it however you like.

Remember to remove the QXL display from the VM and add the IVSHMEM drivers to your Windows VM if you haven't already you can do this by adding;
```xml
<shmem name='looking-glass'>
  <model type='ivshmem-plain'/>
  <size unit='M'>32</size>
</shmem>
```

## Extras

If you often play games with [Easy Anticheat](https://easy.ac/en-us/) or [Hyperion/Byfron](https://web.archive.org/web/20220729202008/https://byfron.com/) you can bypass VM detection through a very simple [tweak](https://www.youtube.com/watch?v=Iass2FMHHng).

If Windows security is an issue you can harden your Windows VM using [Hardentools](https://github.com/hardentools/hardentools) which turns off features not commonly used that have gaping security issues.

If you do not have a Windows activation key you can open Powershell and enter `irm https://massgrave.dev/get | iex` this will allow you to activate your system through [Massgravel](https://github.com/massgravel/Microsoft-Activation-Scripts) a free and open source Microsoft activation tool.

## Conclusion

And we are done, being someone who used to distrohop quite a lot I understand the feeling of trying to find and setup the perfect system, getting frustrated, then restarting. This guide is a comprehensive write-up through a lot of my own hardships getting VFIO working and finalizing the choice between Arch, Debian, or Fedora.

While Linux development in the past few years has come a long way we still have a long way more to go before companies and developers start putting in the extra work to provide for Linux systems. That is why it is best to turn to options like VFIO that are able to provide you a near-native performance Windows experience with ease.

***Please note: In no way shape of form is this meant to be a fully professional or great to follow write-up. I am simply trying to improve on my report writing skills, if it irks you that any part of this write-up is factually incorrect then please; go away. If you are following this write-up and you would like me to explain something in more detail or need extra help with a part please do not hesitate to reach out to me.***

## References

- [Arch Linux Wiki](https://wiki.archlinux.org/) for most things related to PCI passthrough.
- [Whonix Wiki (KVM)](https://www.whonix.org/wiki/KVM) for a simple and complete libvirt installation.
- [TailsOS Install](https://tails.net/install/linux/index.en.html) for the Boot Menu image.
- [VirtIO-Win Github (README)](https://github.com/virtio-win/virtio-win-pkg-scripts/blob/master/README.md) for links to VirtIO drivers.
- [Ventoy](https://www.ventoy.net/en/index.html) for info on Ventoy.
- [VFIO Reddit (post from Databauer)](https://www.reddit.com/r/VFIO/comments/12nfck3/what_is_vfio/) for help explaining VFIO.
- [AverageLinuxUser](https://averagelinuxuser.com/) for the archinstall image.
- [Looking Glass Install Documentation](https://looking-glass.io/docs/B6/install/) for looking glass info.
- [Spice Space](https://www.spice-space.org/download.html) for SPICE guest tools info.
- [Massgravel](https://massgrave.dev/index.html) for Windows installation and activation info.
- [Wikipedia](https://en.wikipedia.org/wiki/) for anything no one else can explain.
