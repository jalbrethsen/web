NETNS_NAME=container   
WG_IFACE_NAME=wg0   
CMD="curl [ifconfig.me](http://ifconfig.me)"   
# from proton wireguard config   
WG_ADDR=10.2.0.2/32   
# save privkey from config to file   
WG_PRIV_KEY=/home/justin/proton-wg/privkey-1   
WG_ENDPOINT=103.125.235.18:51820   
WG_PEER=agoivyLoPqor8MxA/s6UWJSMcA2pMl+ajO3vy/q3oWQ=   
  
sudo ip netns add {NETNS_NAME}   
sudo ip link add {WG_IFACE_NAME} type wireguard   
sudo ip link set {WG_IFACE_NAME} netns {NETNS_NAME}   
sudo ip -n {NETNS_NAME} addr add {WG_ADDR} dev {WG_IFACE_NAME}   
sudo ip netns exec {NETNS_NAME} wg set {WG_IFACE_NAME} private-key {WG_PRIV_KEY} peer {WG_PEER} allowed-ips 0.0.0.0/0 endpoint {WG_ENDPOINT}   
sudo ip -n {NETNS_NAME} link set {WG_IFACE_NAME} up   
sudo ip -n {NETNS_NAME} route add default dev {WG_IFACE_NAME   
sudo ip netns exec {NETNS_NAME} {CMD}   
  
# example   
sudo ip netns add container   
sudo ip link add wg0 type wireguard   
sudo ip link set wg0 netns container   
sudo ip -n container addr add 10.2.0.2/32 dev wg0   
sudo ip netns exec container wg set wg0 private-key /home/justin/proton-wg/privkey-1 peer agoivyLoPqor8MxA/s6UWJSMcA2pMl+ajO3vy/q3oWQ= allowed-ips 0.0.0.0/  
0 endpoint 103.125.235.18:51820   
sudo ip -n container link set wg0 up   
sudo ip -n container route add default dev wg0   
sudo ip netns exec container curl [ifconfig.me](http://ifconfig.me)