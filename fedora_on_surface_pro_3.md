# Notes on Fedora setup specific to Surface Pro 3

## Wifi drop out

### First fix

This increased stability on return from suspend.

Create ``/etc/pm/config.d/modules`` with the contents below, to unload the module before suspend and reload on resume.
```
SUSPEND_MODULES="mwifiex"
```

### Second fix

Outcome to be evaluated.

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
> ```
> [connection]
> # Values are 0 (use default), 1 (ignore/don't touch), 2 (disable) or 3 (enable).
> wifi.powersave = 2
> ```
> 
> To /etc/NetworkManager/NetworkManager.conf seems to have done the trick. It may have to do with the fact that I am using NetworkManager now that I think of it. I am not sure what is the default connection manager for Ubuntu.

## Suspend and hibernate

I changed pretty much anything which was configured to use ``suspend`` to ``suspend-then-hibername``, which became available in Fedora 29. Prior to that, there was 'hybrid-sleep' which chose a direct 'hibernate' if it was available, otherwise would simple suspend.

Using ``suspend`` on it's own is fine, but still consumes a significant amount of power. 

This works OK with LUKS (which I configure at disk level).

These are the lines I changed in `/etc/systemd/logind.conf`:

```
HandleSuspendKey=suspend-then-hibernate
HandleHibernateKey=suspend-then-hibernate
HandleLidSwitch=suspend-then-hibernate
HandleLidSwitchExternalPower=suspend-then-hibernate
HandleLidSwitchDocked=suspend-then-hibernate
```
See `man /etc/systemd/logind.conf`

## ssd disk trim

```
dnf install hdparm
sudo hdparm -I /dev/sda | grep TRIM
sudo systemctl enable --now fstrim.timer
```
## video accelleration
```
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

```
dnf install libva-intel-hybrid-driver libva-intel-driver
dnf install intel-media-driver
```

### Firefox

To change compositing from 'basic' to 'OpenGL' (see about:support):

about:config > 

layers.accelleration.force-enabled > true
layout.display-list.retain.chrome > true
