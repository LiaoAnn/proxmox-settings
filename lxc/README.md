# LXC with docker-compose (optional)

If you want to use Docker on Proxmox, you can use LXC to do that.

## Step 1: Download LXC Template

First, go to **local(pve) > CT Templates**, and click Template to download the template you want to use.(I will use Debian 11 here)

## Step 2: Create LXC

Click on the "Create CT" button to create a LXC and choose the template you want to use.
> Note: **Uncheck the "Unprivileged container" checkbox**

## Step 3: Setting up LXC

### Step 3.1: Close the protection of the LXC

```bash
# edit /etc/pve/lxc/<template_id>.conf, here <template_id> is the id of the LXC
nano /etc/pve/lxc/100.conf

# add those line
lxc.cgroup.devices.allow: a
lxc.cap.drop:
```

### Step 3.2: Enable nesting

Go to CT's **Options** tab, and click **Features** tab, then check the "Enable nesting" checkbox.

After that, start the LXC and enter the LXC.

## Step 4: Install Docker

First, you need to update the source list. (below is for Debian 11)

```bash
# changed sources
mv /etc/apt/sources.list /etc/apt/sources.list.bk
nano /etc/apt/sources.list

# add these lines
deb http://httpredir.debian.org/debian bullseye main non-free contrib
deb-src http://httpredir.debian.org/debian bullseye main non-free contrib

deb http://deb.debian.org/debian-security/ bullseye-security main contrib non-free
deb-src http://deb.debian.org/debian-security/ bullseye-security main contrib non-free
```

After that, you can install Docker and Docker Compose.

```bash
# update
apt update && apt dist-upgrade -y

# install docker and docker-compose
apt install docker.io docker-compose -y

# start docker
systemctl start docker

# enable docker
systemctl enable docker

# check docker and docker-compose version
docker -v
docker-compose -v
```
