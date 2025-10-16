Title: Omarchy - Multi-monitor HDMI and Audio 
Published: 16/10/2025
Tags: [Omarchy, Hyprland, Audio] 
---

# Configuring multi-monitor and choosing HDMI sound output in Omarchy

I added a vertical monitor to my Omarchy desk. I have my BenQ horitontal and have added a Samsung as a vertical screen on the side. 

## Video setup

Listing my monitors gives me the following: 

``` bash
hyprctl monitors

Monitor HDMI-A-1 (ID 0):
	3840x2160@60.00000 at 0x0
	description: Samsung Electric Company LS27A800U HNMW500048
	make: Samsung Electric Company
	model: LS27A800U
	physical size (mm): 600x340
	serial: -----------
	active workspace: 1 (1)
	special workspace: 0 ()
	reserved: 0 26 0 0
	scale: 1.50
	transform: 1
	focused: yes
	dpmsStatus: 1
	vrr: false
	solitary: 0
	solitaryBlockedBy: windowed mode,missing candidate
	activelyTearing: false
	tearingBlockedBy: next frame is not torn,user settings,missing candidate
	directScanoutTo: 0
	directScanoutBlockedBy: user settings,missing candidate
	disabled: false
	currentFormat: XRGB8888
	mirrorOf: none
	availableModes: 3840x2160@60.00Hz ...

Monitor HDMI-A-2 (ID 1):
	3840x2160@60.00100 at -2560x0
	description: BNQ BenQ EL2870U TBK01026SL0
	make: BNQ
	model: BenQ EL2870U
	physical size (mm): 620x340
	serial: -----------
	active workspace: 2 (2)
	special workspace: 0 ()
	reserved: 0 26 0 0
	scale: 1.50
	transform: 0
	focused: no
	dpmsStatus: 1
	vrr: false
	solitary: 0
	solitaryBlockedBy: not opaque
	activelyTearin#pactl list short sinks
#pactl list cardsg: false
	tearingBlockedBy: next frame is not torn,user settings,missing candidate
	directScanoutTo: 0
	directScanoutBlockedBy: user settings,missing candidate
	disabled: false
	currentFormat: XRGB8888
	mirrorOf: none
	availableModes: 3840x2160@60.00Hz ...

```



I edited the monitors file to have `~/.config/hypr/monitors.conf`

```bash
monitor=HDMI-A-2,preferred,auto-left,1.5
monitor=HDMI-A-1,preferred,auto-right,1.5, transform, 1
```

so that the A-1 (Samsung) has the vertical transform. I might switch it to -1 and turn the monitor the other way, because I have less of a bezel on the top of the monitor, but this is the current setup. I'm still not sure which auto- command I need at minimum, but with these two commands - the screens align the way I want.



## Audio setup

The next issue I have is that the audio output is connected from the back of the BenQ monitor to my speaker system, taking the audio along the HDMI channel. Sometimes, depending on which order the screens wake up, the audio output  switches to the Samsung. For the moment, I haven't yet fully figured out how to lock the audio output to a single HDMI output, so I have two scripts to run - checking which outputs where. I'll appreciate the help if someone has suggestions. 

 I used these commands to list the cards and outputs: 

```
pactl list short sinks
pactl list cards
```

And depending on which screen woke up first, it's either running this one: 

```
pactl set-card-profile alsa_card.pci-0000_00_1f.3 output:hdmi-stereo-extra1
pactl set-default-sink alsa_output.pci-0000_00_1f.3.hdmi-stereo-extra1
```

or this one: 

```
pactl set-card-profile alsa_card.pci-0000_00_1f.3 output:hdmi-stereo
pactl set-default-sink alsa_output.pci-0000_00_1f.3.hdmi-stereo
```



I'm hoping to figure out how to pin the audio to the BenQ monitor soon. 
