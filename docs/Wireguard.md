# Wireguard VPN in network namespace
This project is meant to enable multiple simultaneous Wireguard VPNs running on the same device by keeping them in separate network namespaces. You can get some free Wireguard VPNs with a Proton account.

## Define configurables
If you want several VPNs you would want to wrap this in a script, for now we walk through a single example.
```
NETNS_NAME=container-0   
WG_IFACE_NAME=wg-0   
CMD="curl http://ifconfig.me"   
```
Additionally you can get your WG_PRIV_KEY, WG_PEER, and WG_ENDPOINT from your VPN config downloaded from Proton.
## Setup namespace and create interface
Now we create a new network namespace, new interface, and bind the interface to the namespace
```  
sudo ip netns add {NETNS_NAME}   
sudo ip link add {WG_IFACE_NAME} type wireguard   
sudo ip link set {WG_IFACE_NAME} netns {NETNS_NAME}   
```
## Configure Wireguard
Now you can use the info from your wireguard config to configure the interface
```
sudo ip -n {NETNS_NAME} addr add {WG_ADDR} dev {WG_IFACE_NAME}   
sudo ip netns exec {NETNS_NAME} wg set {WG_IFACE_NAME} private-key {WG_PRIV_KEY} peer {WG_PEER} allowed-ips 0.0.0.0/0 endpoint {WG_ENDPOINT}   
```
## Test it out
Now we bring the interface up, add a default route through the wireguard interface and test your public IP address.
```
sudo ip -n {NETNS_NAME} link set {WG_IFACE_NAME} up   
sudo ip -n {NETNS_NAME} route add default dev {WG_IFACE_NAME   
sudo ip netns exec {NETNS_NAME} {CMD}   
```
Now you should see your public IP is from whatever Wireguard server you are connected to.
