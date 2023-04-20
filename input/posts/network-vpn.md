Title: Network planning and VPN 
Published: 20/04/2023 18:40
Tags: [Architecture, Network, VPN] 
---

I am in the process of setting up my homelab network between my two locations.

![Network](/posts/images/network/network.png){ width = 100% }

## Zone Z

Z has a single fiber connection via Swisscom to internet. 

### Inventory
- DeepThread is an AMD Threadripper 1920x running Windows 10. 
- Minis3 and Minis4 are the Beelink MiniS12 N95s running Ubuntu Server 23.04.
- NAS is an an older QNAP TS-269L

### VPN
With a L2TP VPN connection configured to allow remote access onto the network, so I can get Red (Surface Laptop) and Xr (iPhone) onto the network in case.

## Zone N 

N has two connections, a Starlink (v1 round) with only the powerbrick router and Sosh as a backup DSL provider (with an ADSL Router) both connected to a Ubiquity UDM-PRO-SE in Failover mode. Getting a VPN to N is a little more involved, since the UDM is behind a separate router on each WAN. 

### Inventory 
- Minis1 is the Beelink MiniS12 N95s running Windows 11. Planned to Switch to Ubuntu 23.04, but enjoying it VESA mounted behind a screen in the office currently. 
- Minis2 is the Beelink MiniS12 N95s running Ubuntu Server 23.04. Currently rackmounted with the UDM-PRO.

### VPN
On the UDM-PRO, a VPN is configured with Ubiquity and I can use the iOS application WifiMan to access the network.
On Minis2, a [cloudflared docker](https://github.com/cloudflare/cloudflared) is running, reaching up to Cloudflare and providing an Zero trust tunnel to expose several dockerized websites hosted on it.

# The issue at hand

I would like the N Minis1 & Minis2 to be able to access the Z NAS, ideally with a relatively simple connection that I can leave running all the time, to be able to pull files from the NAS and ideally also access the NAS's front-end application from inside my N location. I could connect to the SwisscomVPN every time I do something that requires connectivity to the NAS, but I would really ideally like a more permanent solution where I make the Z NAS "visible" in the N network. Or go full and establish a site-to-site VPN and simply make the two areas N and Z communicate seamlessly while still having local connectivity.

Do you have any suggestions as to how best to accomplish this? 
