# VXLAN

used in proxmox to have all VM's appear on same layer 2 network

## Commands to run on Linux

sudo ip link add vxlan-1 type vxlan id 100 dev butler dstport 4789   
sudo ip addr add 192.168.210.2/24 dev vxlan-1   
sudo bridge fdb append to 00:00:00:00:00:00 dst 10.200.100.1 dev vxlan-1   
sudo bridge fdb append to 00:00:00:00:00:00 dst 10.200.100.8 dev vxlan-1   
sudo ip link set vxlan-1 up

# Geneve

intended to replace proliferation of linux tunnels including vxlan and gre

uses udp encapsulation

## Commands to run on Linux

sudo ip link add gen0 type geneve id 12345 remote 192.168.50.158 tos 48   
sudo ip addr add 192.168.4.2/24 dev gen0   
sudo ip link set gen0 up