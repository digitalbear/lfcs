# Linux Operation Essentials

Prev: [Linux Essentials](./1_linux_essentials.md)

### Reading Operating System Data
```sh
cat /etc/system-release # get version of OS
lsb_release -d # another way to get version of OS
uname -r # get kernel version
cat /proc/version # another way to get kernel version
cat /proc/cmdline # arguments passed to kernal at boot
lsblk # list block devices
```

## Starting and Stopping CentOS 7

Message user
```sh
# to send message to individual message
write tux
# or
cat > message <<END
The server is coming down at 5pm
Ensure you save and logoff
END
cat message
# to broadcast message to all users
wall < message
# To turn messages on/off
mesg y
mesg n
```

### Shutdown commands
```sh
halt
poweroff
reboot
# legacy commands
init --help
telinit --help
#
shutdown -h 10 "the system is going down in 10min"
# to cancel:
ctrl-c
shutdown -c
# nologin file is create when time to shutdown is less than 5 minutes
cat /run/nologin
```

### Changing Runlevels
```sh
who -r # show me my current run level - 5 being the graphical target
runlevel # ditto
systemctl get-default
systemctl set-default multi-user.target
# to change the run level while system is running
systemctl isolate multi-user.target # run level 3 - multi-user, same a graphical target level but without the graphics
systemctl isolate rescue.target # single user level 1, will lose network connections
```

### Selection Runlevels at Boot
* Start up virtual instance (must be powered off first)
* click _e_ to edit the default kernal (default entry)
* scroll down to the line starting "linux16"
* navigate to end of line using ctrl-e and append:
  * "1" or preferrably,
  * "systemd.unit=rescue.target"
* ctrl-x (or F10) to exit and continue booting






Next: [Linux User and Group Management](./3_linux_user_and_group_management.md) 
