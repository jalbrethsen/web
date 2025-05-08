Download VM as .ova format

tar -xvf <filename>.ova

qm importovf <vmid> <ovf file> <storage>

ex. qm importovf 122 Parrot-security-6.3.2_amd64.ovf vol_vms

import disk 

qm importdisk 122 Parrot-security-6.3.2_amd64-disk001.vmdk vol_vms

change settings to q35, virtio scsi, ovmf

user: user

pass: parrot