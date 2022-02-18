# Vivi's Homelab

<!-- ANCHOR: introduction -->
###### Status: Work in Progress
![last commit](https://img.shields.io/github/last-commit/viv-codes/homelab)
![license](https://img.shields.io/github/license/viv-codes/homelab)


# Introduction
---
This repo hosts all of the files and documentation from the creation and operation of my homelab. A homelab is a system or collection of systems used for learning about hardware, networking, and software, with the secondary goal of hosting services. This is the case for my homelab, and as a result of this project I hope to deepen my knowledge of DevOps techniques and tools, specifically Proxmox, Nginx, and Pfsense.

## Overview
My homelab consists of a two servers, with separate proxmox installations on each. My systems are named Elips and Minerva. Elips is less powerful, but has more space for storage expansion in the future, which is why it will be used for secondary backups in the future. Eplis is used as my dev server, where I try and learn about new configurations before deploying them to Minerva for prod. Minerva is my more powerful node, and is used to host the majority of my services via PCT containers (and eventually kubernetes). Minerva also will contain a private network accessible via openvpn, which I intend to use for higher security applications and more sensetive storage.  
#### Elips
![Online](https://img.shields.io/uptimerobot/status/m790187873-2619a6e8222a7cd184383f39)
![Uptime](https://img.shields.io/uptimerobot/ratio/7/m790187873-2619a6e8222a7cd184383f39?label=Uptime%20-%20week)
![Uptime](https://img.shields.io/uptimerobot/ratio/m790187873-2619a6e8222a7cd184383f39?label=Uptime%20-%20Month)
* Lenovo Thinkserver TD340
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
* SuperMicro XXXX chassis with H8QG6 Motherboard
* RAM - 256Gb DDR3 Registered ECC Memory
* CPU - 48 x AMD Opteron 6180 SE, 4 sockets, 12 cores @2.5GHz
* Networking - 2 x 1Gbe
* Storage
  - 2TiB - ZFS Mirror (Boot drive and VM Backups)
  - 2TiB - ZFS Mirror (VM Storage)
  - 8TiB - ZFS Mirror (Bulk storage)

![A graph of my homelab network](/readme_assets/network.png)


# Backend
---
![Proxmox](https://a11ybadges.com/badge?logo=proxmox)
![Docker](https://a11ybadges.com/badge?logo=docker)
![NGINX](https://a11ybadges.com/badge?logo=nginx)
![pfSense](https://a11ybadges.com/badge?logo=pfsense)
<!-- ![OpenVPN](https://a11ybadges.com/badge?logo=openvpn)
 -->
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
![A screenshot of minerva's proxmox management page](/readme_assets/minerva.png)

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
Routing within my systems is handled by Nginx Proxy Manager. This is a frontend for NGINX that allows for easy remote configuration of proxy hosts, redirection hosts, streams, and 404 hosts. I primarily use NGINX Proxy Manager for it's proxy host functionality, as it routes incoming requests based on my CNAME DNS records to my various service containers. Nginx Proxy Manager also assists with the generation of SSL certificates for some services, and can secure sites behind group-based authentication.

Nginx Proxy Manager has a clean, uncomplicated interface that makes management super easy
![A screenshot of nginx proxy manager's webpage](/readme_assets/npm.png)
This is the proxy-hosts page, where you can see all of my container hosts.
![A screenshot of nginx proxy manager's webpage](/readme_assets/npm1.png)



<!-- ## OpenVPN
 -->
# Services
---
<!-- ![WordPress](https://a11ybadges.com/badge?logo=wordpress) -->
![GitLab](https://a11ybadges.com/badge?logo=gitlab)
![Nextcloud](https://a11ybadges.com/badge?logo=nextcloud)
![Overleaf](https://a11ybadges.com/badge?logo=overleaf)
![Jellyfin](https://a11ybadges.com/badge?logo=jellyfin)

## Services

#### Heimdall - https://dash.vhafener.com
Heimdall is the dashboard I use, which makes it easier and faster to access all of my services. I set this to the default homepage of firefox on all of my machines for easy access. 
![A screenshot of heimdall](/readme_assets/heimdall.png)

#### Nextcloud - https://drive.vhafener.com
This is by far my most used service. This is the primary place where all of my school, personal, and archival documents are hosted. I have a total of 1.6TB currently stored in my Nextcloud account. I use the Nextcloud desktop client to sync my frequently used documents across all of my devices, so that I have local access, while keeping nextcloud up-to-date on my changes. I also configured my Nextcloud instance with a built in Collabora Online Development Environment (CODE) server, which enables real-time sharing and collaboration of documents on my Nextcloud, with very similar performance to google docs. 
![A screenshot of Nextcloud](/readme_assets/nextcloud.png)

#### Gitlab - https://git.vhafener.com
I use this GitLab instance to hold internal documentation and some repos that I don't want on GitHub, even in private repos. Eventually I plan on integrating this with k8s to handle CD/CI operations. 
![A screenshot of gitlab](/readme_assets/gitlab.png)

#### Overleaf - http://overleaf.vhafener.com
I use overleaf to edit and store my resumes, as well as to typeset important papers or documents
![A screenshot of overleaf](/readme_assets/overleaf.png)

#### Atheos - http://ide.vhafener.com
I don't use this often, but host it as a redundant system in case my laptop fails. This way I always have access to an ide that I can go back to later, in case I have classwork or something. I came within minutes of needing it on Wednesday, February 16th. 
![A screenshot of atheos](/readme_assets/ide.png)


#### Jellyfin - http://watch.vhafener.com

#### Kavita - http://read.vhafener.com
Kavita is a redundant system in case I'm bored and want something to read, but don't have my kindle or any or my other devices that have access to nextcloud. It's 
![A screenshot of kavita](/readme_assets/read.png)


## Documentation referenced:
* Proxmox Installation - https://pve.proxmox.com/wiki/Installation
* pfSense Installation - https://docs.netgate.com/pfsense/en/latest/install/download-installer-image.html
