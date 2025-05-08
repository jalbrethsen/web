sudo fdisk /dev/sda

delete partition to resize (/dev/sda2)

recreate partition and redefine the end of partition

do not delete ext4 signature (not sure if it makes a difference)

write to disk  
sudo resize2fs /dev/sda2