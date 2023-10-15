# My Proxmox Settings

After installing Proxmox, there are a few settings that I like to change.

## Basic Settings

### Step 1: Update

```bash
# edit /etc/apt/sources.list
nano /etc/apt/sources.list

# add this line to the end of the file
deb http://download.proxmox.com/debian buster pve-no-subscription

# edit /etc/apt/sources.list.d/pve-enterprise.list
nano /etc/apt/sources.list.d/pve-enterprise.list

# comment out the line
# deb https://enterprise.proxmox.com/debian/pve buster pve-enterprise

# update
apt update && apt dist-upgrade -y
```

### Step 2: IOMMU

```bash
# edit /etc/default/grub
nano /etc/default/grub

# comment out the line
# GRUB_CMDLINE_LINUX_DEFAULT="quiet"

# uncomment(or add) the line for intel
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
# or uncomment(or add) the line for amd
GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on"

# update grub
update-grub

# edit /etc/modules
nano /etc/modules

# add these lines (make sure you have those lines)
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd

# reboot
reboot
```

### LXC (optional)

#### Step 1: Download LXC Template

##### Step 1.1: Click on the "Create CT" button

##### Step 1.2: Click on the "Templates" button

##### Step 1.3: Select the template you want to download (I will use Debian 11 here)

##### Step 1.4: Click on the "Download" button and waiting for the download to finish

#### Step 2: Create LXC

##### Step 2.1: Click on the "Create CT" button

##### Step 2.2: Select the template you want to use (I will use Debian 11 here)

##### Step 2.3: Enter the hostname

##### Step 2.4: Enter the password and confirm the password

##### Step 2.5: Uncheck the "Unprivileged container" checkbox

##### Step 2.6: Click on the "Next" button

##### Step 2.7: Select the storage you want to use (Highly recommend size >= 20GB)

##### Step 2.8: Select the network you want to use (Highly recommend using static IP, like: 192.168.100.10/24)

##### Step 2.9: Click on the "Next" button

##### Step 2.10: Click on the "Finish" button and waiting for the LXC to be created

#### Step 3: Setting up LXC

##### Step 3.1: Close the protection of the LXC

```bash
# edit /etc/pve/lxc/<template_id>.conf, here <template_id> is the id of the LXC
nano /etc/pve/lxc/100.conf

# add those line
lxc.cgroup.devices.allow: a
lxc.cap.drop:
```

##### Step 3.2: Go to the "Options" tab

##### Step 3.3: Select the "Features" tab

##### Step 3.4: Select the "Enable nesting" checkbox

##### Step 3.5: Start the LXC

##### Step 3.6: Enter the LXC

#### Step 4: Install Docker

```bash
# changed sources
mv /etc/apt/sources.list /etc/apt/sources.list.bk
nano /etc/apt/sources.list

# add these lines
deb http://httpredir.debian.org/debian bullseye main non-free contrib
deb-src http://httpredir.debian.org/debian bullseye main non-free contrib

deb http://deb.debian.org/debian-security/ bullseye-security main contrib non-free
deb-src http://deb.debian.org/debian-security/ bullseye-security main contrib non-free

# update
apt update && apt dist-upgrade -y

# install docker and docker-compose
apt install docker.io docker-compose -y

# start docker
systemctl start docker

# enable docker
systemctl enable docker
```

## FRP (optional)

If you want to access your Proxmox server from outside of your local network but you don't have a public IP address, you can use FRP to do that.

### FRP Server

Here FRP Server is a VPS.
Like your own server or a server from a cloud service provider.

#### Step 1: Setting up FRP Server

```bash
# create a new folder for frp
mkdir /etc/frp

# create frps config file
nano /etc/frp/frps.toml

# add these lines
[common]
# basic
bind_addr = 0.0.0.0
bind_port = 7000
# logging
log_file = ./frps.log
log_level = info
log_max_days = 7
# auth
authentication_method = token
token = 12345678
# dashboard
dashboard_addr = 0.0.0.0
dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = admin
```

#### Step 2: Setting up FRP Service using Docker Compose

```bash
# create docker-compose.yml
nano /etc/frp/docker-compose.yml

# add these lines
version: "3"
services:
  frps:
    image: "snowdreamtech/frps:latest"
    container_name: "frps"
    restart: always
    network_mode: host
    volumes:
      - "/etc/frp:/etc/frp"
```

#### Step 3: Start FRP Server

```bash
# start frp server
docker-compose -f /etc/frp/docker-compose.yml up -d
```

### FRP Client

Here FRP Client is the Proxmox server.

#### Step 1: Setting up FRP Client

```bash
# create a new folder for frp
mkdir /etc/frp

# create frpc config file
nano /etc/frp/frpc.toml

# add these lines
[common]
# basic
server_addr = x.x.x.x
server_port = 7000
# logging
log_file = ./frpc.log
log_level = info
log_max_days = 7
# auth
authentication_method = token
token = 12345678

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 2288
# [ssh] is the name of the service, you can change it to whatever you want
# type is the type of the service, here we use tcp
# local_ip is the ip of the service on the proxmox server
# local_port is the port of the service on the proxmox server
# remote_port is the port of the service on the frp server

[pve web]
type = tcp
local_ip = 127.0.0.1
local_port = 8006
remote_port = 8006
```

#### Step 2: Setting up FRP Service using Docker Compose

```bash
# create docker-compose.yml
nano /etc/frp/docker-compose.yml

# add these lines
version: "3"
services:
  frpc:
    image: "snowdreamtech/frpc:latest"
    container_name: "frpc"
    restart: always
    network_mode: host
    volumes:
      - "/etc/frp:/etc/frp"
```

#### Step 3: Start FRP Client

```bash
# start frp client
docker-compose -f /etc/frp/docker-compose.yml up -d
```
