Title: Network with Tailscale 
Published: 13/01/2025 10:40
Tags: [Architecture, Network, VPN] 
---

I updated my OpenVPN based network to use Tailscale instead in 2023 and it is game changing. I have used Tailscale ever since. I simply did not update my blog and network diagram. 

![Network](/posts/images/network/network-update.png){ width = 100% }

With [Tailscale](https://tailscale.com/), all my machines appear seamlessly on a single control pane and I can reach any of them from any device.

## Zone Z

Z has a single fiber connection via Swisscom to internet. 

### Inventory

- DeepThread is an AMD Threadripper 1920x running Windows 10. 
- Minis4 is the Beelink MiniS12 N95s running Ubuntu Server 24.10.
- NAS is an an older QNAP TS-269L

## Zone N 

N has two connections, a Starlink (v1 round) with only the powerbrick router and Sosh as a backup DSL provider (with an ADSL Router) both connected to a Ubiquity UDM-PRO-SE in Failover mode.

### Inventory 

- Minis1 is the Beelink MiniS12 N95s running Windows 11, enjoying it VESA mounted behind a screen in the office currently. I originally thought I would also put Ubuntu, but a windows machine is useful.
- Minis2 and Minis3 are  the Beelink MiniS12 N95s running Ubuntu Server  24.10. Currently rackmounted with the UDM-PRO.

### VPN
On the UDM-PRO, a VPN is configured with Ubiquity and I can use the iOS application WifiMan to access the network. It's really a backup of a backup solution to have Wifiman. 


On Minis2 and minis4, a [cloudflared docker](https://github.com/cloudflare/cloudflared) is running, reaching up to Cloudflare and providing an Zero trust tunnel to expose several dockerized websites hosted on it.


I made a [Suno song on how awesome](https://suno.com/song/fb47c594-b22d-4504-83a1-75d8df705194) it is. 
