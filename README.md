# RaspberryPi Cluster
Background for the project is to document all of the details of the setup. I want a container solution for PiHole, Homebridge and other nifty stuff for home usage. The setup contains of three Raspberry Pi 4 with 4 GB RAM. I must emphasize that this is not a how-to/DIY thing, but if you find some of this useful - Awesome!

## Setup Raspberry Pi (This part is for all of the PIes
1. I decided i just stick with Raspberry Pi OS Lite. https://www.raspberrypi.org/software/operating-systems/
2. ++++ Burn it over to a SD card and all of that ++++
3. Add a ssh-file to the root of the filesystem.
4. Setup static IP. Run following in bash
```bash
sudo nano /etc/dhcpcd.conf
```
Add followning under interface eth0. I´m Using Google DNS because I use PiHole, in other usecases I always use the most local DNS host (Router in most cases).
```bash
interface eth0
static ip_address=192.168.86.192/24
static routers=192.168.86.1
static domain_name_servers=8.8.8.8 8.8.4.4 fd51:42f8:caae:d92e::1
```
... Then reboot to make changes complete
```bash
sudo reboot
```
5. Add following to enable memory cgroups
```bash
sudo sed -i '$ s/$/ cgroup_enable=memory cgroup_memory=1/' /boot/cmdline.txt
```
6. .. And since this is is rpi4, you should always activate (in mye opinion)
```bash
sudo nano /boot/config.txt
```
Find the [pi4] section and add the following line:
```bash
arm_64bit=1
```
... Then reboot (you know the drill).
