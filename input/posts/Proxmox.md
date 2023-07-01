Title: Proxmox 8 on sub $200 mini PCs
Published: 01/07/2023 22:00
Tags: [Architecture, Proxmox, Tailscale] 
---


# Installing Proxmox, Tailscale, Win11 VMs and Automation 

This is a Beelink MiniS12 with an Intel N95s.

![](/posts/images/minipc.jpg){width = 80%}
(coffee cup for scale)

Up until Proxmox 8 dropped about a week ago, I was unable to install Proxmox due to compatibility with the graphics driver. Now in 8, that was fixed, so I've been able to install Proxmox on several of my machines.

The procedure is trivial: write the proxmox iso to a usb key via a software that will make the iso bootable. Boot the machine with the usb key inserted and select the correct boot drive, then follow the proxmox installation prompts. 

It all worked out of the box.

## Clustering

I was able to connect to my proxmox installed machine on the port :8006 via a browser on my home network. Next step was enabling the management of multiple machines via a single UI. So as soon as I had two machines with Proxmox installer, I was able to go to my primary, click on Create cluster, confirm. Get the Join token, connect to the socond machine and paste the join token into the "Join Cluster". Worked out of the box. 

![](/posts/images/proxmox/proxmox-clustering.png){ width = 80% }

## Removing the Update Repositories to work on the Free version of Proxmox

To stay within the Free licensing of Proxmox and be able to do apt-get, remember to go for each machine and to remove the non-free repos in the repository list.

![](/posts/images/proxmox/proxmox-remove-repos){ width = 80% }

## Installing Tailscale

I use [Tailscale](https://tailscale.com/) at home to connect across multiple locations and roaming devices. Every time I add a Tailscale device, I am amazed at how easy it is. 

```bash
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up
```

Two lines, one URL to visit and the machines were enrolled.

## VMs and LXCs 

To create VMs and LXCs, you need to add iso images or Templates to your local storage: 

Container templates, you can create your own and upload them or simply click on "Templates" and get a couple of ready made ones for use in your containers. 

![](/posts/images/proxmox/container-templates.png){ width = 80% }

You can now get the official windows ISO from the microsoft website. So download it to your local machine and then Upload it to the ISO images. 

![](/posts/images/proxmox/windows-iso.png){ width = 80% }


### Installing Windows 11 Pro and activating with the Hardware license 

I had previously logged in and linked to my microsoft account on the windows 11 pro licensed version of the OS for each of the machines. This meant that when installing the Windows11 Pro from the ISO onto a VM running on the machines, I was able to activate the machine by referring to the previous activation. #Windows11Pro allows itself to be reactivated with the license that came with the hardware, inside a proxmox8 VM of the same machine as long as you pass the host-cpu - or so it seems to me. 

![Proxmox-shell-mode-issue](/posts/images/proxmox/activated.jpg){ width = 80% }

## Issues I had and solutions

### Console not connecting to LXC containers 

Several times, either while connecting to a container in Proxmox directly, or after a containter migation, I was not able to use the integrated shell. I therefore had to change the Container "Options->Console mode" to "shell" to make it connect every time.
![Proxmox-shell-mode-issue](/posts/images/proxmox/Proxmox-Shell-mode.png){ width = 80% }

### apt-get issue in proxmox containers

The first thing I do upon entering a container is nearly always apt-get update. And sometimes it breaks. I couldn't update or install. Here is my checklist: 

1) Check you gave and ip address to the container in the "Network" section, either a dhcp or a static address. 

2) Check your DNS servers: I had not noticed that after installing Tailscale, my DNS records were only pointing to the talescale DNS resolver. Adding back google (8.8.8.8) and cloudflare (1.1.1.1) to my proxmox hosts helped. 

![Tailscale-dns-issue-proxmox](/posts/images/proxmox/Tailscale-dns-issue-proxmox.png){ width = 80% }

By fixing both of these I was able to get the apt-get running correctly. 

## Automation 
### Installing on a client machine the Proxmox CLI Tools 

I'm planning on checking out automation of deployment on proxmox. Making a note here of the command line installation of the tools

```
sudo pip3 install pve-cli
``` 

I'll also look into Terraform + Ansible for a proxmox deployment, or the Packer LXC to make container templates, but that is for next time. 

