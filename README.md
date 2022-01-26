# Vivi's Homelab

<!-- ANCHOR: introduction -->
###### Status
![last commit](https://img.shields.io/github/last-commit/viv-codes/homelab)
![license](https://img.shields.io/github/license/viv-codes/homelab)


# Introduction
---
This repo hosts all of the files and documentation from the creation and operation of my homelab. A homelab is a system or collection of systems used for learning about hardware, networking, and software, with the secondary goal of hosting services. This is the case for my homelab, and as a result of this project I hope to deepen my knowledge of DevOps techniques and tools, specifically GitOps, Kubernates, Nginx, Proxmox, and Pfsense.

## Overview
My homelab consists of a two-system proxmox cluster. My nodes are Elips and Minerva. Elips is less powerful, but has more space for storage expansion in the future. For this reason Elips primarily hosts my nextcloud instance. Minerva is my more powerful node, and is used to host the majority of my services via Kubernates. Minerva also contains my private network, which I use for higher security storage and operations. 
#### Elips
![Online](https://img.shields.io/uptimerobot/status/m790187873-2619a6e8222a7cd184383f39)
![Uptime](https://img.shields.io/uptimerobot/ratio/7/m790187873-2619a6e8222a7cd184383f39?label=Uptime%20-%20week)
![Uptime](https://img.shields.io/uptimerobot/ratio/m790187873-2619a6e8222a7cd184383f39?label=Uptime%20-%20Month)
* Thinkserver TD340
* RAM - 64Gb DDR3 Registered ECC Memory
* CPU - 1 x Intel Xeon E5-2420 v2, 12 cores @ 2.20GHz
* Networking - 6 x 1Gbe
* Storage
  - 128GiB SSD (Boot drive)
  - 3 TiB - ZFS Mirror
  - 4 TiB - ZFS Mirror
#### Minerva
![Online](https://img.shields.io/uptimerobot/status/m790466602-0858f891fbe572951f707f6d)
![Uptime](https://img.shields.io/uptimerobot/ratio/7/m790466602-0858f891fbe572951f707f6d?label=Uptime%20-%20Week)
![Uptime](https://img.shields.io/uptimerobot/ratio/m790466602-0858f891fbe572951f707f6d?label=Uptime%20-%20Month)
* RAM
* CPU
* Networking - 4 x 1Gbe
* Storage
  - 10TiB - RAIDZ1
 
# Backend
---
![Proxmox](https://a11ybadges.com/badge?logo=proxmox)
![Kubernetes](https://a11ybadges.com/badge?logo=kubernetes)
![Docker](https://a11ybadges.com/badge?logo=docker)
![NGINX](https://a11ybadges.com/badge?logo=nginx)
![pfSense](https://a11ybadges.com/badge?logo=pfsense)
![OpenVPN](https://a11ybadges.com/badge?logo=openvpn)

## Proxmox
1. Install Proxmox from [the ISO](https://www.proxmox.com/en/downloads/category/iso-images-pve) by following the [official documentation ](https://pve.proxmox.com/wiki/Installation)
2. Configure Networking in System>Network
   - Once the installation is complete, your system should have a single bridge named vmnbr0 that connects to your primary network interface. This is the interface that will be used for management of proxmox. Add an A level DNS record pointing at this interface's IP address with a name like proxmox.domain.com. Additional services can also be attached to emulated NICs on this bridge, but since I don't want to use up all of the IPs on 50-net all service network traffic will be passed through pfsense.
   - Create a new linux bridge and bind it to your secondary network interface. This will be used to pass network traffic into pfsense. Set an A level DNS record pointing at this interface's IP, with the base that you want to use for your services. Format of service URLs will be servicename.thisinterfacename.domain.com.
   - Create a third linux bridge and do not bind it to any network interfaces. Set the ip address to 192.168.1.1/24. This is the networking that will be passed through to pfsense
3. Configure storage
   - Set up your storage in the way you see fit. Due to the drives available to me, I chose ZFS RAID mirrors on Elips and ZFS RAID1 striped RAID on Minerva
4. Add certificates in System > Certificates - generate an SSL certificate using something like certbot. Since I already have a wildcard cert for my domain, I just imported it using the proxmox web interface.
5. Disable the Enterprise repo and enable the no-subscription repo in the Updates > Repository tab. This will mean you get updates but won't have to pay.

## Pfsense
1. Installation
   - Navigate to local storage > ISO Images. Either download the pfsense ISO from [here](https://www.pfsense.org/download/) and re-upload it to proxmox, or use the download from URL feature in proxmox to upload the link generated by the above site.
   - Click the 'Create VM' button on the top right of the proxmox dashboard, and create a new VM with 1 socket and 1 Core, and between 512MiB 2GiB RAM. Attach one network device (net0) to the second bridge you created (vmbr1) and a second one to the third bridge you created (vmbr2)
   - Start the pfsense VM
   - Answer 'n' when the installer asks about VLANs
   - Assign WAN to the net0 interface, and LAN to the net1 interface
   - Complete the install
   - Create a VM with 1 socket, 1 core and 2Gb ram from the Ubuntu Desktop ISO found here. Connect this VM to vmbr2. Complete the ubuntu install process in this VM. Boot this VM and navigate to 192.168.1.1 in your browser. This is where you will manage pfsense. The default username is 'admin' and password is 'pfsense'. Enter these values and configure pfsense.


## NGINX

## Kubernates

## Flux

# Services
---
![WordPress](https://a11ybadges.com/badge?logo=wordpress)
![GitLab](https://a11ybadges.com/badge?logo=gitlab)
![Nextcloud](https://a11ybadges.com/badge?logo=nextcloud)
![Overleaf](https://a11ybadges.com/badge?logo=overleaf)
![Jellyfin](https://a11ybadges.com/badge?logo=jellyfin)

## Services


## Documentation referenced:
* Proxmox Installation - https://pve.proxmox.com/wiki/Installation
* pfSense Installation - https://docs.netgate.com/pfsense/en/latest/install/download-installer-image.html
* K8s on Proxmox - https://github.com/galenguyer/k8s 
