# Fedora setup notes

## Docker

To make docker work for normal users:

```
sudo group add docker
sudo usermod -a -G docker $USER
sudo reboot
```

Note the :z or :Z options used with docker volumes, to avoid SELinux errors.

Note that `newgrp` does not seem to to the trick, possibly a full log out and log in would...

## Ruby

Install system version but also need others versions

## bundler
```
gem install bundler
```
### Ruby version manager (rvm)

https://rvm.io/rvm/install

### Stackup

gem install stackup

## AWS

```
dnf install aws-cli
```

## SELinux

I installed most of the packages listed at https://docs.fedoraproject.org/en-US/Fedora/25/html/SELinux_Users_and_Administrators_Guide/chap-Security-Enhanced_Linux-Working_with_SELinux.html#sect-Security-Enhanced_Linux-Working_with_SELinux-SELinux_Packages.


## Grub config

/etc/default/grub.conf

| Line | Description |
|--------------------|-------------------------------------------------------------------------------|
| GRUB_CMDLINE_LINUX | Note the blacklisting of nouveau driver and the omission of the word 'splash' |
| GRUB_GFXMODE= | Note the screen resolution command |

```text
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="resume=/dev/mapper/fedora-swap rd.lvm.lv=fedora/root rd.luks.uuid=luks-fee5da6d-a698-4937-a143-ff950542f30b rd.lvm.lv=fedora/swap quiet rd.driver.blacklist=nouveau"
GRUB_DISABLE_RECOVERY="true"
GRUB_GFXPAYLOAD_LINUX="keep"
GRUB_VIDEO_BACKEND="vbe"
GRUB_TERMINAL_OUTPUT="gfxterm"
GRUB_FONT_PATH="/boot/grub2/fonts/unicode.pf2"
GRUB_GFXMODE="3440x1440x16"
```
Then

```bash
sudo grub2-mkconfig
```

## Repositories

```text
_copr_phracek-PyCharm.repo
fedora-updates-testing-modular.repo
rpmfusion-nonfree.repo
dl.bintray.com_tvheadend_fedora_linux_4.2_fc28_x86_64_.repo
fedora-updates-testing.repo
rpmfusion-nonfree-steam.repo
fedora-cisco-openh264.repo 
google-chrome.repo
rpmfusion-nonfree-updates.repo
fedora-modular.repo
rpmfusion-free.repo
rpmfusion-nonfree-updates-testing.repo
fedora.repo
rpmfusion-free-updates.repo
vscode.repo
fedora-updates-modular.repo
rpmfusion-free-updates-testing.repo
fedora-updates.repo
rpmfusion-nonfree-nvidia-driver.repo
```

tvheadend may be changed now to https://dl.bintray.com/tvheadend/fedora/:bintray-tvheadend-fedora-4.2-stable.repo, but that actually tvheadend is from rpmfusion, therefore probably that is not required.

```
dnf copr enable phracek/PyCharm
```

## Upgrade Fedora 29 to Fedora 30

### OS

https://fedoramagazine.org/upgrading-fedora-29-to-fedora-30/

### Post upgrade
