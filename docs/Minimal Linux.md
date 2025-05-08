# Objective
My goal with this project is to see how small I can make a functional system, it can be useful on storage constrained embedded devices to create a OS with only the tools needed to do a job and nothing more. This is an attempt to create a base template for such a system and also to further my understanding of computers and Linux.

# Install dependencies

Install standard build tools using apt, zypper, pacman, etc.. (gcc, bison, flex, git, etc..)

# Build Linux

Clone latest stable release of linux kernel

```
git clone --depth=1 --branch=v6.13 git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
```

Make the minimal config included with linux, this config is included by default but it is not quite usable

```
make tinyconfig
```

Then you have the tiniest possible kernel, to make it usable add a few key items.

```
make menuconfig
```

> 64-bit kernel
>
> General setup/Initial RAM filesystem and RAM disk (initramfs/initrd) support
>
> General setup/Configure standard kernel features (expert users)/Enable support for printk
>
> Executable file formats/kernel support for elf binaries
>
> Executable file formats/kernel support for scripts starting with #!
>
> Device drivers/Character devices/Enable TTY
>
> File systems/Pseudo filesystems/proc support
>
> File systems/Pseudo filesystems/sysfs support
>
> Power management and ACPI options / ACPI support
>
> Processor type and features / EFI runtime service support
>
> Processor type and features / EFI runtime service support / EFI stub support
>
> Device drivers / Graphics support / Framebuffer devices / Support for frame buffer device drivers
>
> Device drivers / Graphics support / Framebuffer devices / Support for frame buffer device drivers / EFI-based Framebuffer Support
>
> Device drivers / Graphics support / Console display driver support / Framebuffer Console support

Save the config and then compile the kernel

```
make -j 6
```

# Build busybox

Busybox is a good tool for simplifying the setup of our embedded system as it includes all your tools in a single binary.


```
wget https://busybox.net/downloads/busybox-1.37.0.tar.bz2
tar -xvjf busybox-1.37.0.tar.bz2
cd busybox-1.37.0/
```

Now we make the config, here you can choose what tools to package within busybox

```
make menuconfig
```

> Enable Settings/Build static binary (no shared libs)

```
make -j 6
```

# Build initrd

we create a directory to store our init scripts and copy our busybox

```
mkdir ~/initramfs
mkdir ~/initramfs/bin
cp busybox ~/initramfs/bin/
```

Next we create an init script in \~/initramfs/bin/init

```
#!/bin/busybox sh
/bin/busybox mkdir -p /usr/bin
/bin/busybox mkdir -p /usr/sbin
/bin/busybox mkdir /sbin
/bin/busybox mkdir /proc
/bin/busybox mkdir /sys
/bin/busybox mount -t sysfs none /sys
/bin/busybox mount -t proc none /proc
/bin/busybox --install -s

exec /bin/sh
```

Then make  it executable

```
chmod +x ~/initramfs/bin/init  
```

Now we go to the directory and turn it into an initrd

```
cd ~/initramfs
find . -print0 | cpio --null --create --verbose --format=newc | gzip --best > ../initrd
```

Now we have initrd file in \~/initrd  
You can boot this initrd and kernel with qemu to get a shell
```
qemu-system-x86_64 -kernel arch/x86/boot/bzImage -initrd ~/initrd
```
You should now be greeted with a simple shell that you can interact with your minimal linux system

# Make bootable USB

```
sudo fdisk /dev/sdb
```

Create new partition and set type to EFI System

create fat32 filesystem

```
sudo mkfs.fat -F 32 /dev/sdb1
```

mount /dev/sdb1 /mnt/hda

```
cp ~/initrd /mnt/hda
cp ~linux/arch/x86/boot/bzImage /mnt/hda
```

create startup script startup.nsh

```
fs0:\bzImage initrd=initrd
```
Then copy the startup.nsh to the root of your usb stick
```
cp startup.nsh /mnt/hda
```
Plugin and bootup your minimal linux on real hardware. You may need to navigate the efi shell if this startup.nsh does not get called correctly.

