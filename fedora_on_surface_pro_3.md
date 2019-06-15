# Notes on Fedora setup specific to Surface Pro 3

## And then it broek again....

Update from kernel 5.0.17-300 (no problems) to 5.1.7-300 and 5.1.7-300, I cannot connect to some wifi networks (mwifiex driver, marvel firmware). Both current and
previous firmware (from linux-firmware, which was experimenally downgraded with `dnf downgrade linux-firmware`) have this problem. Booting with 5.0 kernel
fixes the problem, so it can be surmised an update to the kernel or the kernel module is involved. Yet to report this problem. `journalctl -u NetworkManager -u wpa_supplicant` seem
to gather some logs at `info` level but not much there other than authentication seems to fail. See [log here](log.txt) of an attempt to switch networks. WPA suplicant at least probably needs to be started in debug mode.

## Wifi drop out

### First fix

> **Update to Fedora 30**. This fix requires the `pm-utils` package which is no longer available for Fedora.

This increased stability on return from suspend.

Create ``/etc/pm/config.d/modules`` with the contents below, to unload the module before suspend and reload on resume.

```text
SUSPEND_MODULES="mwifiex"
```

### Wifi dropout

This fixed my wifi drop out issues.

https://github.com/jakeday/linux-surface/issues/84#issuecomment-363072395

> I experienced another wifi drop since the last time I posted, but it was after a restart. So I found the command to show the interface power_save status at startup, and here it is for me:
> 
> ```
> sudo iw dev wlp2s0 get power_save
> Power save: on
> ```
> 
> So 2 things: in my case I have only seen the drop when in "Power save mode on", so setting it off does indeed seem to help. And next it is not enabled by default on my system. Maybe this is a feature of the AUR branch.
> I am now looking for a way to enable it at startup, like it should. However, I think we can close this issue. Or rename it "wifi power save wrongly enabled at startup".
> 
> Edit : adding
> 
> ```text
> [connection]
> # Values are 0 (use default), 1 (ignore/don't touch), 2 (disable) or 3 (enable).
> wifi.powersave = 2
> ```

> To /etc/NetworkManager/NetworkManager.conf seems to have done the trick. It may have to do with the fact that I am using NetworkManager now that I think of it. I am not sure what is the default connection manager for Ubuntu.

## Wifi not available when waking from a sleep mode:

It transpires that Gnome 3 invokes systemd to carry out commands such as `systemctl suspend-then-hibernate` and `systemctl suspend`. See `man systemd-sleep` for details. As a result, the following hack is used to unload the wifi adaptor modules when the system sleeps, and restores then whem the system wakes up.

/usr/lib/systemd/system-sleep/remove-wifi-adapter-modules.sh:

```bash
#!/bin/bash
[ "$1" = "post" ] && exec /usr/sbin/modprobe mwifiex_pcie mwifiex
[ "$1" = "pre" ] && exec /usr/sbin/rmmod mwifiex_pcie mwifiex
exit 0
```

```bash
chmod +x /usr/lib/systemd/system-sleep/remove-wifi-adapter-modules.sh
systemctl daemon-reload
```

## Suspend and hibernate

Gnome will determine the deepest sleep mode possible, and invoke that to make your system sleep. It is not clear if this includes pressing of power button, or shutting the lid.

You can work out which modes are supported by trying them out, but you cannot make gnome use them:

systemctl suspend # does something
systemctl hybrid-sleep # not available
systenctl suspend-then-hibernate # seems to map to above.

## ssd disk trim

```bash
dnf install hdparm
sudo hdparm -I /dev/sda | grep TRIM
sudo systemctl enable --now fstrim.timer
```

## Video accelleration

```bash
[root@green2 ~]# lspci
00:00.0 Host bridge: Intel Corporation Haswell-ULT DRAM Controller (rev 09)
00:02.0 VGA compatible controller: Intel Corporation Haswell-ULT Integrated Graphics Controller (rev 09)
00:03.0 Audio device: Intel Corporation Haswell-ULT HD Audio Controller (rev 09)
00:14.0 USB controller: Intel Corporation 8 Series USB xHCI HC (rev 04)
00:16.0 Communication controller: Intel Corporation 8 Series HECI #0 (rev 04)
00:1b.0 Audio device: Intel Corporation 8 Series HD Audio Controller (rev 04)
00:1c.0 PCI bridge: Intel Corporation 8 Series PCI Express Root Port 3 (rev e4)
00:1f.0 ISA bridge: Intel Corporation 8 Series LPC Controller (rev 04)
00:1f.2 SATA controller: Intel Corporation 8 Series SATA Controller 1 [AHCI mode] (rev 04)
00:1f.3 SMBus: Intel Corporation 8 Series SMBus Controller (rev 04)
01:00.0 Ethernet controller: Marvell Technology Group Ltd. 88W8897 [AVASTAR] 802.11ac Wireless
[root@green2 ~]#
```

A bit of research suggested the high CPU i was experiences with video playback might be
fixed with some packages. Small impact. Perhaps. Noting like the performance I get from
Raspberry Pi....

```bash
dnf install libva-intel-hybrid-driver libva-intel-driver
dnf install intel-media-driver
```

### Firefox

To change compositing from 'basic' to 'OpenGL' (see about:support):

about:config >

layers.accelleration.force-enabled > true
layout.display-list.retain.chrome > true

## Phantom touch input

Random input happenings. Found [this](https://www.youtube.com/watch?v=MdmnIG5ZVM4). Solution seems to be working, and strangely, the touch screen still seems to be working. So one would assume there are two drivers involved and this simply avoids contention.

/usr/share/X11/xorg.conf.d/10-evdev.conf:

```text
Section "InputClass"
        Identifier "evdev touchscreen catchall"
        MatchIsTouchscreen "on"
        MatchDevicePath "/dev/input/event*"
        Driver "evdev"

        # Add this line to control the phantom input
        Option "Ignore" "on"
EndSection
```
