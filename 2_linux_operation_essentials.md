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

### Selecting Runlevels at Boot
* Start up virtual instance (must be powered off first)
* click _e_ to edit the default kernel (default entry)
* scroll down to the line starting "linux16"
* navigate to end of line using ctrl-e and append:
  * "1" or preferrably,
  * "systemd.unit=rescue.target"
* ctrl-x (or F10) to exit and continue booting

## The Boot Process

### Managing GRUB Recovery

```sh
vi /etc/default/grub
# set GRUB_DISABLE_RECOVERY="false" to enable recovery mode
grub2-mkconfig -o /boot/grub2/grub.cfg
reboot
```

### Recover Lost Root Passwords
* Reboot machine
* At boot option inout _e_ at kernel entry to edit
* scroll down to the line starting "linux16"
* navigate to end of line using ctrl-e and:
  * remove: rhgb quiet  # so we can see the boot process
  * append: rd.break enforcing=0
* ctrl-x
```sh
switch_root:/# mount -o remount,rw /sysroot # to remount root filesystem - currently set as readonly - need it read-right so we can set the password
chroot /sysroot # to set a false root which points through to the sysroot directory
# we are no working with real filesystem, which was previously mounted into sysroot
passwd # to reset password
# assuming no error...
exit # get out of chroot environment
# up arrow to retrieve mount command and edit
mount -o remount,ro /sysroot
exit # to exit and continue with boot process
# logon using the new password
restorecon /etc/shadow # since password was set outside of normal security context
setenforce 1 # selinux 
```

## Managing GRUB2
GRUB - GRand Unified Bootloader

### Re-installing GRUB
```sh
# as root
grub2-install /dev/sda # install into the master boot record - /dev/sda on Centos 7
```

### Manage GRUB2 Defaults
```sh
vi /etc/default/grub
# make changes and exit - then run
grub2-mkconfig -o /boot/grub2/grub.cfg
```

### Manage GRUB2 with grubby
```sh
grubby --default-kernel
grubby --set-default /boot/vmlinuz-3.10.0-514.el7.x86_64
grubby --default-kernel
grubby --info=ALL
grubby --info /boot/vmlinuz-3.10.0-514.el7.x86_64
grubby --remove-args="rhgb quiet" --update-kernel !$
reboot
```

### Password Protect GRUB2
Cleartext password
```sh
cp /etc/grub.d/01_users .
cd /etc/grub.d
vi 01_users # replace contents with the following (minus hash symbols)
# #!/bin/sh -e
# cat << EOF
#    set superusers="andrew"
#    password andrew L1nux
# EOF
grub2-mkconfig -o /boot/grub2/grub.cfg
reboot
# when we click _e_ to edit boot option this will now ask for a user and password
```
Encrypted password
```sh
grub2-mkpasswd-pbkdf2 # this will output a password hash - copy this to the clipboard
vi /etc/grub.d/01_users
# #!/bin/sh -e
# cat << EOF
#     set superusers="andrew"
#     password_pbkdf2 andrew grub.pbkdf2.sha512.10000.DDF4EAC30F5A9BB6809BF9E02C384E1B335CA469584CE6855069CE8FF991CDE85E114A18AA27D1EE9ED269F5B8C357A9CDFD886B03F4D80FB69565EBEE1F0A53.F7E4269A1746100DD4254B73AC9872A17EA0D1DAF1CFD282D9F99A99B13DC88C1B5579653831BA54894DCD3794C96405EFD646E52FF12D9A7A043C364F394B6C
# EOF
grub2-mkconfig -o /boot/grub2/grub.cfg
reboot
```

### Custom GRUB2 Entries
```sh
vi /etc/grub.d/40_custom
# add details as required
grub2-mkconfig -o /boot/grub2/grub.cfg
reboot
```

## Managing Linux Processes

### Listing Processes with ps
```sh
ps
# BSD format 
ps aux
# UNIX format
ps -f # full listing
ps -F # extra full listing
ps -l # long listing
ps -e # list everything
pstree
```

### The /proc directory and the $$ variable
```sh
# $$ variable show the process id of the current process (the bash shell)
ps -p $$ -f
# double check
ps
echo $$
# we can see this also in the /proc directory
cd /proc
cd $$
```

### Send Signals with kill
```sh
stty -a # to view keyboard shortcuts for sending signals
kill -l # to see the signals that can be sent to a process
kill -15 PID # terminate
kill -9 PID # signal cannot be caught so will always execute
```

### Shortcuts with pgrep, pkill, and top
```sh
pgrep sshd # list process id of sshd processes
ps -F -p $(pgrep sshd)
# above is actually better than "ps -eF | grep sshd" as it includes headers and doesn't incude itself
# create some background sleep processes
sleep 100&
sleep 100&
sleep 100&
pgrep sleep # to list these
pkill sleep # to kill these
```

```sh
top
# "f" to show menu of what to sort by
# arrow up or down then "s" and escape to select
# "q" to quit
```

## Process Priority

### Backgrounding Tasks
```sh
sleep 1000&
jobs
sleep 1000 # without the ampersand
(ctrl-z) # to move task to background, suspended
jobs # this will show both our sleep jobs - the most recently one with a "+" symbol next to it - this means it has focus
bg # to resume the job in the background - the job that has focus
jobs
fg # to bring a currently in focus background job into the foreground
fg <nr> to bring job <nr> to the foreground
```

### Configuring Process Priority using nice
```sh
ps -l
# Columns PRI (priority) and NI (nice) should show 80 and 0 respectively for our sleep jobs
# Process Priority can be between 60 (high) and 99 (low) - default 80
# Nice can be between -20 and +19 - default 0
# manually set the nice value
nice -n 1 sleep 1000 &
renice -n 10 -p 2930 # where -p is PID
# only root can set priority lower than current value or nice value less than zero
# edit default values in the following file
vi /etc/security/limits.conf
# and add the following line
# tux - priority 10 # for a group add @ at start, or use * for all users
su - tux # to start a new shell
sleep 250&
ps -l # see new processes showing default priority of 10
```







Next: [Linux User and Group Management](./3_linux_user_and_group_management.md) 
