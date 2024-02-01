# Setting up K3s cluster with Ansible

I using Proxmox to setup K3s env, so I will show you how to setup K3s cluster with Ansible.

## 1. Setting PVE Sever
```bash
# enable bridge-nf-call-iptables
sysctl -w net.bridge.bridge-nf-call-iptables=1

# disabled swap
sysctl vm.swappiness=0
swapoff -a

# enabled IP forwarding
echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.conf
sysctl --system
```

## 2. Using Proxmox to added 5 CTs

> Please make sure you unchecked the "Unprivileged container" checkbox when you create the LXC and setting swap to 0.

## 3. Install Ansible on the server

install ansible with apt
```bash
sudo apt update
sudo apt install ansible
sudo apt install sshpass
```

add `hosts` file for clients.
```ini
[ubuntu]
<ip address 1>
<ip address 2>
```

for example:
```ini
[ubuntu]
192.168.100.200
192.168.100.201
```

after installed ansible, we can use commands to check ansible's version.
```bash
ansible --version
```

and after that, we can command with module
```bash
ansible -i ./inventory/hosts ubuntu -m ping --user someuser --ask-pass
```

or you can command with playbook
```bash
ansible-playbook ./playbooks/apt.yml --user someuser --ask-pass --ask-become-pass -i ./inventory/hosts
```

some examples: 
`apt.yml`
```yaml
- hosts: "*"
  become: yes
  tasks:
    - name: apt
      apt:
        update_cache: yes
        upgrade: 'yes'
```

`zsh.yml`
```yaml
- name: install latest zsh on all hosts
  hosts: "*"
  tasks:
    - name: install zsh
      apt:
        name: zsh
        state: present
        update_cache: true
      become: true
```

## 4. Install K3s to all CTs with Ansible

Go to see [this](https://technotim.live/posts/k3s-etcd-ansible/#prep)

## Refs

- [LiaoAnn/proxmox-settings (github.com)](https://github.com/LiaoAnn/proxmox-settings)
- [Automate EVERYTHING with Ansible! | Techno Tim](https://technotim.live/posts/ansible-automation/)
- [Fully Automated K3S etcd High Availability Install | Techno Tim](https://technotim.live/posts/k3s-etcd-ansible/)
