# Tips and tricks for using ssh

## Local config file \~/.ssh/config
You can make your life easier by adding your commonly accessed remote hosts to your ssh config, it can help you keep track of which keys map to which machines, users, ports, addresses, etc.
### Nonstandard algorithms
Here is an example host configuration I used for my very old rooted kindle:
```
Host kindle   
       HostName 192.168.1.2  
       IdentityFile \~/.ssh/kindle_id_rsa  
       Port  8022  
       User justin  
       KexAlgorithms +diffie-hellman-group1-sha1
```
The kindle runs a very old droppbear ssh that requires the older diffie hellman algo to connect to it. We also store the username, port, ip address, and ssh key file to use
### Proxy Jump
Additionally, it may be useful to use one machine to access another one, we list another useful host configuration here:
```
Host host1
       HostName 192.168.4.2
       User justin 
Host host2  
       HostName 192.168.5.2   
       ProxyJump host1  
       User justin 
```
Here we use host1 to jump to host2, this could be very complicated normally, but defined here we just have to type `ssh host2` and everything else is taken care of
### Port forwarding
Sometimes we just want to access a port running on a remote host, or use a remote host to access a port on another device on its network. See the below example to forward these back to our own localhost port.
```
Host remoteHost
        HostName 192.168.5.2
        LocalForward 9443 192.168.5.3:443  
        LocalForward 8888 localhost:8888
```
Here we forward the web server port on a device we cannot directly communicate with 192.168.5.3 to our localhost:9443. We also forward a jupyter notebook from remoteHost to our local port 8888
## SSH CLI
We won't repeat all the previous commands again, although they can also be run from the CLI. Some additional commands I have found useful
### Run shell commands and return output
```
ssh host1 whoami
```
This one is a bit obvious, although some people who only use ssh for a remote session may not realize you can use it to quickly run a single command.
### Fork the command to background
```
ssh -f host1 whoami
```
For long running commands or asynchronous commands you can do a non-blocking command with -f.
This can be useful in scripting if running several commands on many devices.

