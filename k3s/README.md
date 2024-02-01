# K3s (optional)

Because k8s is too heavy, so I use k3s here.
K3s is a lightweight k8s distribution. It is designed for production workloads in unattended, resource-constrained, remote locations or inside IoT appliances.

## Step 1: Setting PVE Server

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

## Step 2: Create K3s Server LXC

Please make sure you unchecked the "Unprivileged container" checkbox when you create the LXC and setting swap to 0.

After the LXC is created, you need to add the following lines to the LXC config file.

```bash
# edit /etc/pve/lxc/<template_id>.conf, here <template_id> is the id of the LXC
nano /etc/pve/lxc/200.conf

# add those line
lxc.apparmor.profile: unconfined
lxc.cgroup.devices.allow: a
lxc.cap.drop:
lxc.mount.auto: "proc:rw sys:rw"
```

Agent LXC is the same as Server LXC.

After that, you can start the LXC.

## Step 3: Setting LXC

rc.local is a script that will be executed at the end of each multiuser runlevel.

To make rc.local work in LXC, you need to do the following steps.

First you need to create rc.local.

```bash
nano /etc/rc.local
```

Then you need to add the following lines to the LXC config file.

```bash
#!/bin/sh -e
# Kubeadm 1.15 needs /dev/kmsg to be there, but it's not in lxc, but we can just use /dev/console instead
# see: https://github.com/kubernetes-sigs/kind/issues/662
if [ ! -e /dev/kmsg ]; then
    ln -s /dev/console /dev/kmsg
fi
# https://medium.com/@kvaps/run-kubernetes-in-lxc-container-f04aa94b6c9c
mount --make-rshared /
```

And give rc.local execution permission.

```bash
chmod +x /etc/rc.local
```

After that, you need to reboot the LXC.

```bash
reboot
```

## Step 4: Install K3s Server

```bash
# update
apt update && apt dist-upgrade -y

# install k3s
curl -sfL https://get.k3s.io | sh -

# get k3s token
cat /var/lib/rancher/k3s/server/node-token

# get k3s server ip
ip a
```

Do the same thing for the K3s Agent LXC.

And the token and the server ip will be used in the next step.

## Step 5: Setting K3s Agent

After the K3s Server and K3s Agent are created, you need to add the following lines to the K3s Agent config file.

Add agent node to the cluster(Remember to replace the `<server_ip>`, `<node_token>` and `<node_name>` with your own)

```bash
curl -fsL https://get.k3s.io | K3S_URL=https://<server_ip>:6443 K3S_TOKEN=<node_token> sh -s - --node-name <node_name>
```

## Step 6: Testing

Apply the example

```bash
kubectl apply -f https://k8s.io/examples/service/load-balancer-example.yaml
```

Enable access to the service

```bash
kubectl expose deployment hello-world --type=LoadBalancer --name=hl
```

Check IP and port

```bash
kubectl describe services 
```

Access the service with the `http://<LoadBalancer Ingress>:<Port>`

## Uninstall K3s

```bash
# for server
/usr/local/bin/k3s-uninstall.sh

# for agent
/usr/local/bin/k3s-agent-uninstall.sh
```

## Some Parctices

I highly recommend you to watch this video: [you need to learn Kubernetes RIGHT NOW!!](https://youtu.be/7bA0gTroJjw?si=mn6UrFMt6n8RqZr6), it will help you understand k8s better.
