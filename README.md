#RPi4 Docker Cluster
Background for the project is to document all of the details of the setup. I want a container solution for PiHole, Homebridge and other nifty stuff for home usage. The setup contains of three Raspberry Pi 4 with 4 GB RAM. I must emphasize that this is not a how-to/DIY thing, but if you find some of this useful - Awesome!

## Setup Raspberry Pi (This part is for all of the Pi)
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
5. Update the pi! sudo apt update && sudo apt upgrade
```bash
sudo apt update
sudo apt upgrade
```
7. Add following to enable memory cgroups
```bash
sudo sed -i '$ s/$/ cgroup_enable=memory cgroup_memory=1/' /boot/cmdline.txt
```
8. .. And since this is is rpi4, you should always activate (in mye opinion)
```bash
sudo nano /boot/config.txt
```
Find the [pi4] section and add the following line:
```bash
arm_64bit=1
```
9. .. Rename Pi to something else. Like docker-1 or something.
```bash
sudo raspi-config
System Options > Hostname > Remove the oldname (raspberrypi) and add the new.
```
... Then reboot (you know the drill).

## Setup Gluster
This part was painfully easy with the right guide. Strongly recommend checking out this link: https://github-wiki-see.page/m/nerdily/Raspberry-Pi-Docker-Swarm/wiki/2.-GlusterFS (I only use the internal storage, but the guide shows how to use external)


1. Install Gluster on each of the devices and activate service.
```bash
sudo apt install glusterfs-server
sudo systemctl enable glusterd
```
2. Make the folder that you want to replicate on each node.
```bash
##NODE RAS-1
sudo mkdir -p /mnt/glusterfs
sudo mkdir -p /data/glusterfs/myvol1/brick1/
```
3. From the master-node, add accordingly all of the nodes that is in your cluster
```bash
gluster volume create brick1 replica 2 ras-1:/brick1 ras-2:/brick1 force
gluster volume set brick1 cluster.server-quorum-type server
gluster volume set all cluster.server-quorum-ratio 51%
gluster volume start brick1
```
4. Mount the drive on each node
```bash
sudo echo "ras-1:/brick1 /mnt/glusterfs glusterfs defaults,_netdev,noauto,x-systemd.automount 0 0" >> /etc/fstab
```
## Setup Docker (Swarm)
1. Run following command on each pi
```bash
curl -sSL https://get.docker.com | sh;
```
2. Init docker swarm on the manager of the swarm, and add worker to the swarm. For me 192.168.86.191 is manager.
Run following on the manager
```bash
sudo docker swarm init --advertise-addr 192.168.86.191
```
Run following on the worker(s)
```bash
docker swarm join \
        --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
       192.168.86.191:2377
```
Verify that all of the nodes is added to the swarm
```bash
sudo docker node ls
```
To add more tokens use following command
```bash
sudo docker swarm join-token manager
```
