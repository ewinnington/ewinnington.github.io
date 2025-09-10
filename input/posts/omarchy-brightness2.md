Title: Omarchy - Integrating screen brightness via key binds
Published: 10/09/2025
Tags: [Omarchy, Linux] 
---

# Controlling External Monitor Brightness (DDC/CI) in Hyprland with a Real OSD

As a follow-up to the previous entry on how to get the [brightness adapted by ddcutil](https://ewinnington.github.io/posts/omarchy-brightness), I actually asked [OpenAI's Codex](https://openai.com/codex/) to wire it up in my keyboard bindings. Once it succeeded and get the osd wired up, I asked it to document the process. Here is the Codex generated documentation. 

As a side note I created a small script to increase or decrease the brightness on the command line, that is executable. 

```bash
#!/bin/bash
# brightness up/down script using ddcutil
STEP=10
case "$1" in
  up)   ddcutil setvcp 10 +$STEP ;;
  down) ddcutil setvcp 10 -$STEP ;;
  get)  ddcutil getvcp 10 ;;
  *)    echo "Usage: $0 {up|down|get}" ;;
esac
```

# OpenAI Codex steps and explanations 

- Environment: Hyprland (Omarchy on Arch), SwayOSD, ddcutil
- Goal: Make hardware brightness keys and Alt+F1/F2 control HDMI monitor brightness via DDC/CI, with a correct on-screen display (OSD).

## The Problem

- Omarchy’s default media bindings show the OSD and call brightnessctl, which targets laptop backlights—not external HDMI displays.
- My script ~/bin/hdmi-brightness already adjusts HDMI brightness using ddcutil, but Hyprland wasn’t calling it from brightness keys.
- Bonus ask: show an OSD reflecting the real HDMI brightness level.

## Solution Summary

- Unbind default brightness keys.
- Bind brightness keys and Alt+F1/F2 to the hdmi-brightness script.
- After each adjustment, read the real brightness via ddcutil getvcp 10 and display an OSD using SwayOSD’s custom-progress mode.

## Keybindings

- File: ~/.config/hypr/bindings.conf
- Unbind defaults:
    - unbind = , XF86MonBrightnessUp
    - unbind = , XF86MonBrightnessDown
    - unbind = ALT, XF86MonBrightnessUp
    - unbind = ALT, XF86MonBrightnessDown
- Bind to DDC/CI script + OSD:
    - `bindeld = , XF86MonBrightnessUp, HDMI Brightness up, exec, bash -lc "~/bin/hdmi-brightness raise; read P R <<< $(ddcutil getvcp 10 2>/dev/null | awk
'BEGIN{FS=\"[=,]\"} /current value/ {cv=$2+0; mv=$4+0; if(mv<=0){p=0;r=0}else{p=int((cv*100+mv/2)/mv); r=cv/mv}; if(r<0) r=0; if(r>1) r=1; printf(\"%d %.2f\", p, r)}');
$osdclient --custom-icon display-brightness --custom-progress-text \"Brightness: $P%\" --custom-progress \"$R\""`
    - `bindeld = , XF86MonBrightnessDown, HDMI Brightness down, exec, bash -lc "~/bin/hdmi-brightness lower; read P R <<< $(ddcutil getvcp 10 2>/dev/null | awk
'BEGIN{FS=\"[=,]\"} /current value/ {cv=$2+0; mv=$4+0; if(mv<=0){p=0;r=0}else{p=int((cv*100+mv/2)/mv); r=cv/mv}; if(r<0) r=0; if(r>1) r=1; printf(\"%d %.2f\", p, r)}');
$osdclient --custom-icon display-brightness --custom-progress-text \"Brightness: $P%\" --custom-progress \"$R\""`
    - `bindeld = ALT, F2, HDMI Brightness up, exec, [same as above with raise]`
    - `bindeld = ALT, F1, HDMI Brightness down, exec, [same as above with lower]`
- $osdclient comes from Omarchy’s defaults and shows OSD on the focused monitor:
    - `swayosd-client --monitor "$(hyprctl monitors -j | jq -r '.[] | select(.focused == true).name')"`

## OSD Integration

- ddcutil getvcp 10 returns a line containing “current value” and “maximum value”.
- We parse it to get:
    - P (percent, e.g., 72)
    - R (ratio 0.00–1.00 for a progress bar)
- SwayOSD’s --custom-progress shows a clean bar without changing any backlight devices:
    - --custom-progress "$R"
    - --custom-progress-text "Brightness: $P%"
    - --custom-icon display-brightness

## Testing

- Ensure the script is executable: chmod +x ~/bin/hdmi-brightness
- Reload Hyprland: hyprctl reload
- Press brightness keys and Alt+F1/F2:
    - External monitor brightness changes (DDC/CI)
    - OSD shows an accurate bar and percentage
- If no OSD appears:
    - systemctl --user enable --now swayosd
    - Keep a window on the monitor you want the OSD (defaults to focused)

## Troubleshooting

- No brightness change: confirm ddcutil works (ddcutil detect, ddcutil getvcp 10), user in i2c group, and i2c-dev is loaded.
- Different keycodes: use wev to check actual keysyms and update the binds accordingly.
- OSD on the wrong output: we can pin $osdclient to a specific monitor name (e.g., --monitor "HDMI-A-1").

## Why This Works

- It replaces backlight-centric controls with DDC/CI, which external monitors use.
- The OSD is decoupled from any system backlight and directly reflects DDC/CI state, so it’s always accurate.
