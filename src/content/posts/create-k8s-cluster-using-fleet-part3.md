---
title: "Build K8s cluster using fleet (Part 3)"
description: "meta description"
date: 2024-07-14T05:00:00Z
image: "/images/posts/01.jpg"
categories: ["kubernetes"]
authors: ["Balaji Balasundaram"]
tags: ["fleet", "k8s", "kubernetes", "cluster"]
draft: true
---
# Build k8s cluster (part 3): Talos Linux, PXE boot, Fleet, Cilium, Longhorn

Setting up a Kubernetes cluster on bare metal using Talos Linux, PXE boot, Fleet, Cilium, and Longhorn is a great choice for a secure and manageable infrastructure. Talos Linux is a modern, secure, and minimal operating system designed specifically for Kubernetes.

### Prerequisites
- Bare Metal Servers: Ensure you have multiple servers available.
- Network Infrastructure: Ensure all servers are on the same network and can communicate with the PXE server.
- DHCP and TFTP Server: Required for PXE booting.
- OS Images: Talos Linux image.
- PXE Boot Configuration: PXE boot setup on the bare metal servers.

## Steps
### 1. Setup PXE Boot Environment
#### Install and configure a DHCP server:

Install DHCP server:
```sh
sudo apt-get install isc-dhcp-server
```
Configure the DHCP server (/etc/dhcp/dhcpd.conf):
```conf
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option routers 192.168.1.1;
    option broadcast-address 192.168.1.255;
    next-server 192.168.1.10;  # IP of the PXE server
    filename "pxelinux.0";
}
```
Install and configure a TFTP server:

Install TFTP server:
```sh
sudo apt-get install tftpd-hpa
```
Configure the TFTP server (/etc/default/tftpd-hpa):
```conf
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/var/lib/tftpboot"
TFTP_ADDRESS="0.0.0.0:69"
TFTP_OPTIONS="--secure"
```
### 3. Install and configure a PXE bootloader:

Install syslinux:
```sh

sudo apt-get install syslinux pxelinux
```
Copy PXE boot files:
```sh
sudo mkdir -p /var/lib/tftpboot/pxelinux.cfg
sudo cp /usr/lib/PXELINUX/pxelinux.0 /var/lib/tftpboot/
sudo cp /usr/lib/syslinux/modules/bios/{ldlinux.c32,libcom32.c32,libutil.c32,menu.c32} /var/lib/tftpboot/
```
Configure PXE boot menu:

Create a default PXE boot configuration (/var/lib/tftpboot/pxelinux.cfg/default):
```conf
DEFAULT menu.c32
PROMPT 0
TIMEOUT 300
ONTIMEOUT local
MENU TITLE PXE Boot Menu

LABEL talos
    MENU LABEL Install Talos Linux
    KERNEL talos-kernel
    INITRD talos-initramfs
    APPEND ip=dhcp
```
Add Talos Linux images:

Download and place Talos Linux kernel and initrd images in /var/lib/tftpboot/.
2. Configure Talos Linux
Generate Talos configuration:

Install talosctl on your local machine:
```sh

curl -Lo talosctl https://github.com/siderolabs/talos/releases/download/v0.13.6/talosctl-linux-amd64
chmod +x talosctl
sudo mv talosctl /usr/local/bin/
```
Generate the Talos cluster configuration:
```sh

talosctl gen config my-cluster https://192.168.1.10:6443
```
Customize the generated configuration files (controlplane.yaml and worker.yaml) as needed.
Set up the Talos configuration files on the PXE server:

Place the generated configuration files in a location accessible by the PXE boot environment, such as /var/www/html/talos/.
Modify the PXE menu to include Talos configuration:

Update the PXE menu entry to include the Talos configuration URL:
conf
Copy code
LABEL talos
    MENU LABEL Install Talos Linux
    KERNEL talos-kernel
    INITRD talos-initramfs
    APPEND ip=dhcp talos.config=http://192.168.1.10/talos/controlplane.yaml
3. Install Kubernetes and Tools
Boot the servers:

Boot the servers using PXE, which will install Talos Linux and configure them as per the Talos configuration files.
Initialize the Kubernetes cluster:

Use talosctl to initialize the Kubernetes cluster:
sh
Copy code
talosctl --nodes <controlplane-node-ip> bootstrap
Install Cilium for Networking:

Once the cluster is up, install Cilium:
sh
Copy code
kubectl create -f https://raw.githubusercontent.com/cilium/cilium/v1.10/install/kubernetes/quick-install.yaml
Install Longhorn for Persistent Storage:

Add the Longhorn Helm repository:
sh
Copy code
helm repo add longhorn https://charts.longhorn.io
helm repo update
Install Longhorn:
sh
Copy code
kubectl create namespace longhorn-system
helm install longhorn longhorn/longhorn --namespace longhorn-system
Install Fleet for GitOps:

Add the Fleet Helm repository:
sh
Copy code
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
helm repo update
Install Fleet:
sh
Copy code
kubectl create namespace fleet-system
helm install fleet-crd rancher-stable/fleet-crd --namespace fleet-system
helm install fleet rancher-stable/fleet --namespace fleet-system
Verifying the Installation
Check the Nodes:

sh
Copy code
kubectl get nodes
Verify Cilium:

sh
Copy code
kubectl get pods -n kube-system -l k8s-app=cilium
Verify Longhorn:

sh
Copy code
kubectl get pods -n longhorn-system
Verify Fleet:

sh
Copy code
kubectl get pods -n fleet-system
This setup provides a secure, automated deployment of a Kubernetes cluster on bare metal servers using Talos Linux, PXE boot, Fleet for GitOps, Cilium for networking, and Longhorn for storage. Adjust configurations based on your specific network setup and requirements.
