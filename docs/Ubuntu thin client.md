# Setup nfsroot

*loosely based on [https://help.ubuntu.com/community/DisklessUbuntuHowto](https://help.ubuntu.com/community/DisklessUbuntuHowto)*

#### Install requirements

```
sudo apt install nfs-kernel-server -y
sudo systemctl enable --now nfs-kernel-server.service
```

#### Make directory and add it to exports file

```
sudo mkdir /nfsroot
sudo vim /etc/exports
```

> /nfsroot 192.168.205.0/24(rw,no_root_squash,async,insecure)

#### Sync the nfs exports

```
sudo exportfs -rv
```

#### Enter working system (client) with fresh install

```
sudo cp /boot/vmlinuz-`uname -r` ~
sudo vim /etc/initramfs-tools/initramfs.conf  
```

> BOOT=nfs   
> MODULES=netboot

```
mkinitramfs -o ~/initrd.img-`uname -r`
```

#### Mount nfsroot on working system (client)

```
sudo apt install -y nfs-common
mount -t nfs -onolock 192.168.205.10:/nfsroot /mnt
cp -ax /. /mnt/.
cp -ax /dev/. /mnt/dev/.
```

# Setup pxe boot and tftp

#### Install requirements

```
sudo apt-get install isc-dhcp-server tftpd-hpa syslinux pxelinux initramfs-tools
```

#### Setup dhcp server /etc/dhcp/dhcpd.conf

```
allow booting;
allow bootp;

subnet 192.168.2.0 netmask 255.255.255.0 {
  range 192.168.2.xxx 192.168.2.xxx;
  option broadcast-address 192.168.2.255;
  option routers 192.168.2.xxx;
  option domain-name-servers 192.168.2.xxx;

  filename "pxelinux.0";
  next-server 192.168.3.10;
}


# force the client to this ip for pxe.
# This is only necessary assuming you want to send different images to different computers.
host pxe_client {
  hardware ethernet xx:xx:xx:xx:xx:xx;
  fixed-address 192.168.2.xxx;
}
```

#### Set DHCP server interface (must be in the correct dhcp range)

```
#/etc/default/isc-dhcp-server
INTERFACESv4=ens18
```

#### Restart DHCP Server using the command

```
sudo service isc-dhcp-server restart
```

#### On mine I had vyos point to my tftp server and nfsroot

```
configure
set service dhcp-server shared-network-name 'vxlan' subnet 192.168.205.0/24 option bootfile-name pxelinux.0
set service dhcp-server shared-network-name 'vxlan' subnet 192.168.205.0/24 option bootfile-server 192.168.205.10
commit
save
```

#### Configure tftpd

```
# /etc/default/tftpd-hpa 
RUN_DAEMON="yes" 
TFTP_ADDRESS="0.0.0.0:69" 
TFTP_USERNAME="tftp" 
TFTP_DIRECTORY="/tftpboot" 
TFTP_OPTIONS="-l -s"
```

#### Copy files from server (may also work with client fresh install)

```
sudo mkdir -p /tftpboot/pxelinux.cfg
sudo mkdir -p /tftpboot/boot
sudo cp /usr/lib/PXELINUX/pxelinux.0 /tftpboot
sudo cp -r /usr/lib/syslinux/modules/bios /tftpboot/boot/isolinux
```

#### Create /tftpboot/pxelinux.cfg/default

```
LABEL linux
DEFAULT linux
KERNEL vmlinuz-5.4.0-195-generic
APPEND root=/dev/nfs initrd=initrd.img-5.4.0-195-generic nfsroot=192.168.205.10:/nfsroot ip=dhcp rw console=tty0 console=ttyS0,115200n8
```

Get tftp started 

```
sudo chmod -R 777 /tftpboot
sudo /etc/init.d/tftpd-hpa start
```

# Put it all together

#### Add kernel and initrd to tftpboot folder from the fresh install copied to nfsroot

```
sudo cp /nfsroot/home/ubuntu/vmlinuz-5.4.0-195-generic /tftpboot/
sudo cp /nfsroot/home/ubuntu/initrd.img-5.4.0-195-generic /tftpboot/
```

#### Configure /nfsroot/etc/fstab, this is important to avoid parallel clients locking each other out

```
none /var/run tmpfs defaults 0 0
none /var/log tmpfs defaults 0 0
none /var/lock tmpfs defaults 0 0
none /var/tmp tmpfs defaults 0 0
proc /proc proc defaults 0 0
none /media tmpfs defaults 0 0
none /var/lib/dhcp tmpfs defaults 0 0
none /tmp tmpfs defaults 0 0
```

#### Comment out exec update-grub in /nfsroot/etc/kernel/postinst.d/zz-update-grub

#### Clear out entries from client netplan to avoid messing with interface, it already has dhcp setup

may have to disable ipv6 if issues bringing up iface