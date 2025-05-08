tips and tricks for using ssh

# Config file \~/.ssh/config

Host host1   
       HostName 192.168.1.2  
              IdentityFile \~/.ssh/home_id_rsa  
       Port  8022  
       User justin  
              KexAlgorithms +diffie-hellman-group1-sha1  
Host host2   
       HostName 192.168.5.2   
       ProxyJump host1  
       Port  18022  
       User justin  
              LocalForward 9443 192.168.5.3:9443  
              LocalForward 8888 localhost:8888

# CLI

## forward port to local

ssh host1 -L 8889:[localhost:8889](http://localhost:8889)

## Run shell commands and return output

ssh host1 whoami

## Now fork the command to background

ssh -f host1 whoami

## 