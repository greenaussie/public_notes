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


