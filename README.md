# My Proxmox Settings

After installing Proxmox, there are a few settings that I like to change.

## Basic Settings

### Step 1: Update

```bash
# edit /etc/apt/sources.list
nano /etc/apt/sources.list

# add this line to the end of the file
deb http://download.proxmox.com/debian bullseye pve-no-subscription

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

## Some Other Settings

- [LXC with Docker-Compose](./lxc/)
- [FRP](./frp/)
- [K3s](./k3s/)
