Title: Omarchy: Setting screen brightness over HDMI by terminal
Published: 10/09/2025
Tags: [Omarchy, Linux] 
---

# Omarchy

Witht he coming arrival of the end of Windows10, I installed [Omarchy](https://omarchy.org/) on one my [Beelink MiniS12 N95](https://www.bee-link.com/products/beelink-mini-s12-n95), fully expecting just to play with it and revert back to Windows11 on the machine. Win11 was slow on the machine, but a decent cheap desktop to have connected to a screen. *Omarchy* on the Beelink, even on the tiny hardware, has been absolutely flying. To the point that it's now my main desktop for everything right now. 

One thing I needed to do is to handle the brightness of the screen, from the command line, so I could toggle the screen brightness from the command line. I discovered I could use `ddcutil` which I installed using the package manager on Omarchy.  

```
sudo ddcutil detect
Display 1
   I2C bus:  /dev/i2c-0
   DRM_connector:           card1-HDMI-A-2
   EDID synopsis:
      Mfg id:               BNQ - UNK
      Model:                BenQ EL2870U
      Product code:         31049  (0x7949)
      Serial number:        58M02252SL0
      Binary serial number: 21573 (0x00005445)
      Manufacture year:     2021,  Week: 34
   VCP version:         2.2
```

My screen is connected by HDMI, do I was able to get the information on it. 

```
~ ❯ sudo usermod -aG i2c $USER
```

So I didn't want to sudo all the time for my screen display information, I added my user the control of the i2c bus. This could be a security weakening, there's other ways to do it, but for my case it's fine.

```
~ ❯ ddcutil detect
Display 1
   I2C bus:  /dev/i2c-0
   DRM_connector:           card1-HDMI-A-2
   EDID synopsis:
      Mfg id:               BNQ - UNK
      Model:                BenQ EL2870U
      Product code:         31049  (0x7949)
      Serial number:        58M02252SL0
      Binary serial number: 21573 (0x00005445)
      Manufacture year:     2021,  Week: 34
   VCP version:         2.2
```

Then getting and setting the brightness was done with the following commands: `ddcutil getvcp 10` and `ddcutil setvcp 10 20`

```
~ ❯ ddcutil getvcp 10
VCP code 0x10 (Brightness                    ): current value =    40, max value =   100

~ ❯ ddcutil setvcp 10 20
```

To get a listing of what codes the screen supports, you can use `ddcutil capabilities`. 

```
ddcutil capabilities
Model: EL2870U
MCCS version: 2.2
Commands:
   Op Code: 01 (VCP Request)
   Op Code: 02 (VCP Response)
   Op Code: 03 (VCP Set)
   Op Code: 07 (Timing Request)
   Op Code: 0C (Save Settings)
   Op Code: E3 (Capabilities Reply)
   Op Code: F3 (Capabilities Request)
VCP Features:
   Feature: 02 (New control value)
   Feature: 04 (Restore factory defaults)
   Feature: 05 (Restore factory brightness/contrast defaults)
   Feature: 08 (Restore color defaults)
   Feature: 0B (Color temperature increment)
   Feature: 0C (Color temperature request)
   Feature: 10 (Brightness)
   Feature: 12 (Contrast)
   Feature: 14 (Select color preset)
      Values:
         04: 5000 K
         05: 6500 K
         08: 9300 K
         0b: User 1
   Feature: 16 (Video gain: Red)
   Feature: 18 (Video gain: Green)
   Feature: 1A (Video gain: Blue)
   Feature: 52 (Active control)
   Feature: 60 (Input Source)
      Values:
         0f: DisplayPort-1
         11: HDMI-1
         12: HDMI-2
   Feature: 62 (Audio speaker volume)
   Feature: 72 (Gamma)
      Invalid gamma descriptor: 50 64 78 8c a0
   Feature: 7D (Unrecognized feature)
      Values: 00 01 02 (interpretation unavailable)
   Feature: 7E (Trapezoid)
      Values: 03 0F 10 11 12 (interpretation unavailable)
   Feature: 7F (Unrecognized feature)
   Feature: 80 (Keystone)
      Values: 01 02 03 (interpretation unavailable)
   Feature: 86 (Display Scaling)
      Values:
         01: No scaling
         02: Max image, no aspect ration distortion
         05: Max vertical image with aspect ratio distortion
         0c: Unrecognized value
         10: Unrecognized value
         11: Unrecognized value
         13: Unrecognized value
         14: Unrecognized value
         15: Unrecognized value
         16: Unrecognized value
         17: Unrecognized value
   Feature: 87 (Sharpness)
   Feature: 8D (Audio mute/Screen blank)
      Values: 01 02 (interpretation unavailable)
   Feature: AC (Horizontal frequency)
   Feature: AE (Vertical frequency)
   Feature: B2 (Flat panel sub-pixel layout)
   Feature: B6 (Display technology type)
   Feature: C0 (Display usage time)
   Feature: C6 (Application enable key)
   Feature: C8 (Display controller type)
   Feature: C9 (Display firmware level)
   Feature: CA (OSD/Button Control)
      Values:
         01: OSD disabled, button events enabled
         02: OSD enabled, button events enabled
   Feature: CC (OSD Language)
      Values:
         01: Chinese (traditional, Hantai)
         02: English
         03: French
         04: German
         05: Italian
         06: Japanese
         07: Korean
         09: Russian
         0a: Spanish
         0b: Swedish
         0d: Chinese (simplified / Kantai)
         0e: Portuguese (Brazil)
         0f: Arabic
         12: Czech
         14: Dutch
         1a: Hungarian
         1e: Polish
         1f: Romanian
   Feature: D6 (Power mode)
      Values:
         01: DPM: On,  DPMS: Off
         05: Write only value to turn off display
   Feature: DA (Scan mode)
      Values:
         00: Normal operation
         02: Overscan
   Feature: DC (Display Mode)
      Values:
         04: User defined
         05: Games
         0b: Unrecognized value
         0c: Unrecognized value
         0e: Unrecognized value
         0f: Unrecognized value
         12: Unrecognized value
         13: Unrecognized value
         21: Unrecognized value
   Feature: DF (VCP Version)
```

This was what my BenQ exposes.

## Ease of use 

Discovered that ddcutil allows relative up downs:

```
~ ❯ ddcutil setvcp 10 + 10
~ ❯ ddcutil setvcp 10 - 10
```

## Next steps

I need to figure out how to wire up these commands to the brightness up / down key commands on Omarchy - so I can control the brightness on the keyboard. Still not sure how to get that configuration working, since it doesn't work out of the box with my screen with the default tooling. 
