# Alexa to Wyoming
This project describes how to turn your Amazon Echo Dot 2nd generation into a Wyoming Satellite device to integrate with your HomeAssistant.
# Install postmarketOS
To run custom software on this device we must first install an OS that will give us full control of our device. By default the Echo device is locked down to only the Amazon firmware that it ships with.
## Install required tools
We will use mtkclient, EchoCLI, adb, fastboot, and pmbootstrap to install postmarketOS. I ran this from a Parrot Linux machine, but these should work on any other debian-based Linux.

install adb and fastboot:
```
sudo apt install android-tools-adb android-tools-fastboot
```
install mtkclient
```
git clone https://github.com/bkerler/mtkclient
cd mtkclient
pip3 install -r requirements.txt
```
install EchoCLI
```
git clone https://github.com/Dragon863/EchoCLI
cd EchoCLI
pip3 install -r requirements.txt
```
Most Linux dsitributions ship pmbootstrap with their package manager, but it may not support this device, install pmbootstrap from source to get latest version
```
git clone --depth=1 https://gitlab.postmarketos.org/postmarketOS/pmbootstrap.git
mkdir -p ~/.local/bin
ln -s "$PWD/pmbootstrap/pmbootstrap.py" ~/.local/bin/pmbootstrap
```
## Root the device
We must first gain root access on the device before we will be able to install any other OS. We will make use of Daniel Benge did some fine work to develop a tethered root exploit on this device using a patched little kernel and the kamakiri bootrom exploit. The link to Daniel's writeup is [here](https://danieldb.uk/posts/alexa-2/). To root the device we do the following:
```
cd EchoCLI
python3 main.py
```
This will walk you through the rooting process step by step. It involves opening up your device and shorting a capacitor to interrupt the boot process. I used a peice of aluminum foil and jammed it between the capacitor and the RF shield pictured [here](img/mainboard.jpg). The rooting process will give you a modified preloader file called `preloader_no_hdr.bin`. We can use this to bypass the chain of trust in the Amazon firmware and load our custom OS.

## Enter Fastboot Mode
We use the mtkclient with our modified preloader to get to fastboot mode. Whil you run the below command hold down the Uber (circle) button on top of the device until the LED ring turns green.
```
cd mtkclient
python3 mtk.py plstage --preloader=preloader_no_hdr.bin 
fastboot devices
```
It should show the echo device
## Install postmarketOS
We can use the pmbootstrap commands to install postmarketOS
```
pmbootstrap init
pmbootstrap install
pmbootstrap flasher flash_rootfs
pmbootstrap flasher flash_kernel
```
Running flash_kernel failed for me, so I had to export the images and flash the boot_a and boot_b partitions manually.
```
pmboostrap export
fastboot flash boot_a /tmp/postmarketOS/boot.img
fastboot flash boot_b /tmp/postmarketOS/boot.img
```
## Boot to postmarketOS
Now that you have postmarketOS installed you can either unplug the device and replug in or run `fastboot reboot`.

Because of our earlier rooting step, it will no longer boot without running the following command
```
cd mtkclient
python3 mtk.py plstage --preloader preloader_no_hdr.bin
```
Now the device should boot to postmarketOS and you should have a new usb ethernet interface on your host machine, mine is called `enxde61f3b4c3ed`.

You may need to assign an ip address to this interface first
```
sudo ifconfig enxde61f3b4c3ed 172.16.42.2
```
Now you can ssh into the device using the username and password set during your pmbootstrap install.
```
ssh justin@172.16.42.1
```
## Setup Host
The machine which is tethered to the Echo device we call the host and we will use it to forward traffic from the Echo and add port forwarding for the Wyoming service.
```
sudo -i
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -o {DEFAULT_IFACE} -j MASQUERADE
iptables -A PREROUTING -t nat -p tcp -i {DEFAULT_IFACE} --dport 10700 -j DNAT --to-destination 172.16.42.1:10700
```
where the default iface is the name of the interface used to connect to your HomeAssistant server, mine is `enp6s18`.

## Setup Echo
First ssh into the device
```
ssh justin@172.16.42.1
```
Now set your default route to go through the host device so you can access the internet and your HomeAssistant
```
sudo ip route add default via 172.16.42.2
```
Now you can update your packege list and install required audio packages
```
sudo apk update
sudo apk add pulseaudio pulseaudio-alsa alsa-plugins-pulse
```
I had some trouble getting the audio to work, I ended up doing the following to get it working.
```
sudo rm /etc/asound.conf
alsamixer -c 0
```
From there enable audio_i2s1_setting to enable the speaker device. You can adjust the microphone recording volume with the adc micpga volume A/b/c.

The microphone has 9 channels, you can run record audio with the following command
```
arecord -D hw:0,24 -r 16000 -f S24_3LE -t wav > t.wav -c 9
```
hw 0,24 is the only non-dummy device from arecord -l

Additionally I got the time synchronization to work with the following command
```
sudo chronyc makestep
```

## Setup Wyoming Satellite
Because of the limited storage on the device I resorted to using an sshfs to expand the storage so have room to add more software.
```
sshfs -o default_permissions justin@172.16.42.2:~/biscuit /home/justin/share
```
Next we install wyoming-satellite
```
apk add git
cd /home/justin/share
git clone https://github.com/rhasspy/wyoming-satellite
cd wyoming-satellite
scripts/setup
```
Finally we run the command to start the service using our custom mic-command
```
script/run --name 'echo' --uri 'tcp://0.0.0.0:10700' --mic-command 'arecord -D hw:0,24 -r 16000 -f S24_3LE -t raw -c 9' --snd-command 'aplay -r 22050 -c 1 -f S16_LE -t raw'
```
Now from your HomeAssistant device you will see a wyoming device discoverable and hook it up with your workflow, I use a local Ollama agent to run my local Voice Assistant.

While you can get everything hooked up from here, I pipe it into my local OpenWakeWord service, it never triggers the wake word and I am not able to really use it to this point. 

## Future steps
My best guess as to the wake word problem is the fact that our mic has 9 channels which may cause difficulties for my openwakeword. In the future I would like to verify this and get the last few steps complete.

Additionally I tried out running a snapcast client on the Echo device which worked reasonably well, so you can still use it as a music playing device hooked into your Home Assistant.
