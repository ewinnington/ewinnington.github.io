Title: Raspberry pi
Published: 23/01/2013
Tags: [Migrated, Rpi] 
---

Since I've finally had time to do something with my [Raspberry Pi](http://www.raspberrypi.org/), I've set it up as a file server for macs and iTunes server for the house. I'm using a 7 port USB hub, powered, that uses 2 4-port hubs inside. As a drive, I used a Toshiba 500 gig USB powered HDD that supports USB3. The Raspberry is powered directly from the USB hub, so there is only one power supply for the setup with 3 USB cables (1 hub to pi for power supply, 1 pi to hub data, 1 hub to hdd data cable) and 1 ethernet cat5 cable.

The distribution is a standard Raspbian that I keep up to date using _sudo apt-get update_ and _sudo apt-get upgrade_. (_Linux raspberrypi 3.2.27+ #250 PREEMPT Thu Oct 18 19:03:02 BST 2012 armv6l_)

To mount the Toshiba HDD, I first used **dmesg** to watch for where the USB drive appeared when connected to the USB hub. For me it was sda : sda1. So I mounted it using **mount** and after verifying it worked, edited the_ /etc/fstab_ to make the drive auto mount on boot. I added the line: _/dev/sda1 /srv/toshiba auto defaults 0 0_

To act as a file server for mac, I used the software [**netatalk**](http://netatalk.sourceforge.net/). It's quick to install and requires practically no configuration and it worked out of the box. My raspberry appeared on the mac within seconds with the home drive shared, then it was a simple share of the extra folders.

To Install: _apt-get install netatalk_ To configure which folders are shared: _vi /etc/netatalk/AppleVolumes.default_ (remember to check the rights on these folders if you have a problem) To start/stop/restart the service: _sudo /etc/init.d/netatalk start_ (or _stop_ or _restart_)

To act as an [iTunes server](http://elinux.org/RPiForked-Daapd), I used **[forked-daapd](https://github.com/jasonmc/forked-daapd)**. Again, worked out of the box with just a touch of configuration. The config file is very easy. I didn't used the remote function of daapd yet, using the box only as a server, not as a playback device.

To install: _apt-get install forked-daapd_ To configure where the music is at: _sudo vi /etc/forked-daapd.conf_ To start/stop the service: _sudo /etc/init.d/forked-daapd start_ (or _stop_)

With that installed, I had "Music on Raspberry pi" showing up in my iTunes side panel. It just worked.

The only problem I was getting was that the playback of certain songs was getting interrupted in midstream and when I changed songs in iTunes, there was a very long "Opening URL" message box coming up. The Toshiba hard drive where the music is located was running really slow and any access loaded the CPU to 100% for far too long. [Similar problems had been reported on the raspberry pi forum](http://www.raspberrypi.org/phpBB3/viewtopic.php?f=28&t=10355). To diagnose what was the issue, I used **hdparm** to check the performance of the HDD and **lsusb **to find out the tree structure of the USB hub.

Run a performance test of drive sda: _hdparm -tT /dev/sda_

My first test gave me a speed of ~280 KB/s. Abysmal performance. After checking the logs with **dmesg**, I saw many USB reset messages with respect to the USB drive. With **lsusb**, I confirmed the HDD was connected to the second internal usb hub of my powered hub, so I changed port until I found one that is connected to the first hub. The performance difference was instantaneous: 26 MB/s. Connecting it directly to the raspberry pi proved to be too much of a power drain for the pi's usb port.

List the tree of USB hubs and devices connected: _sudo lsusb -t_

A couple more things remained before I could leave my pi headless in the closet next to the router: I changed the default password (_raspberry_) to another one using the command **passwd**.

Then I installed [**tightvncserver**](http://www.tightvnc.com/) on the pi to [serve the desktop over the network](http://elinux.org/RPi_VNC_Server) to a mac or a windows computer. I don't launch it automatically on logging in, I only launch a graphical environment occasionally, so it can live on the command line for the time being.

To install: _sudo apt-get install tightvncserver_ To launch: _tightvncserver_ To create a display on port 0 with a 1024x768 resolution and 24 bit colour depth: _vncserver :0 -geometry 1024x768 -depth 24_

Over the network I could now SSH into the pi (On windows I use [**Putty**](http://www.chiark.greenend.org.uk/~sgtatham/putty/), on mac I just use SSH from the command shell) and launch the GUI on another computer.

Finally, I connected to my home network and forwarded the SSH (port 22) service to my raspberry pi.

Next time, I want to set-up a dynamic dns for the Pi, so that I can always gain access to it, even if my home router reboots and changes ip. I would like it to use a subdomain of my cloudplush address, but I don't know if my hosting provider iPower supports that. Another option would be to email or tweet the ip address of my home router every time it changes.

I also saw an interesting [AirPi project](http://jordanburgess.com/post/38986434391/raspberry-pi-airplay) online, where the pi works as an airplay receiver, this would be an interesting addition to my pi. For this, I need to get a wireless adapter.

I want to investigate using the pi with a LAMP stack on it and [**OwnCloud**](https://owncloud.com/) to serve as a private owned dropbox.

Finally, I really should stop relying on passwords with my SSH connection and start using an SSH keyfile, Especially if it will be more and more internet facing.

The command **history** to helped me remember all the things I did.
