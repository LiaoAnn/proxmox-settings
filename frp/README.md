# FRP (optional)

If you want to access your Proxmox server from outside of your local network but you don't have a public IP address, you can use FRP to do that.

## FRP Server

Here FRP Server is a VPS.
Like your own server or a server from a cloud service provider.

### Step 1: Setting up FRP Server

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

### Step 2: Setting up FRP Service using Docker Compose

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

### Step 3: Start FRP Server

```bash
# start frp server
docker-compose -f /etc/frp/docker-compose.yml up -d
```

## FRP Client

Here FRP Client is the Proxmox server.

### Step 1: Setting up FRP Client

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

### Step 2: Setting up FRP Service using Docker Compose

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

#### Step 4: Access Proxmox Server

Go to browser and type `https://<frps ip address>:8006` to access Proxmox Web UI.

> Note: If you are using a firewall, you need to open the port you want to use.
