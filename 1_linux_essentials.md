# Linux Essentials

## Installing CentOS 7

create 2 servers: 
* server1.example.com
* server2.example.com  

both with users:
* root / pa55word
* tux / pa55word

set networking:
* adapter 1: NAT Network "NatNetwork" (must first add this through Virtualbox main menu)
* adapter 2: Host-only - in Preferences, ensure host-only network is set-up as follows:
    * Adapter
      * IPv4 Address: 192.168.56.1
      * IPv4 Network Mask: 255.255.255.0
    * DHCP Server
      * Server Address: 192.168.56.100
      * Server Mask: 255.255.255.0
      * Lower Address Bound: 192.168.56.101
      * Upper Address Bound: 192.168.56.254


During install must set Network and hostname:
* Ethernet (enp0s3): on
* Ethernet (enp0s8): on

show IP address
```sh
ip a s
```

network manager command line tool
```sh
nmcli conn show
# to bring up a connection to a specific interface
nmcli conn up enp0s8
# if the network interface is down you will need to bring it up using ifup
ifup enp0s8
```

to ensure interfaces are brought up at boot
```sh
sed -i s/ONBOOT=no/ONBOOT=yes/ /etc/sysconfig/network-scripts/ifcfg-enp0s3
sed -i s/ONBOOT=no/ONBOOT=yes/ /etc/sysconfig/network-scripts/ifcfg-enp0s8
```

update software and install some additional packages, including X (as root) on server 1
```sh
yum update
yum install -y redhat-lsb-core net-tools epel-release kernel-headers kernel-devel
yum groupinstall -y "Development Tools"
yum groupinstall -y "X Window System" "MATE Desktop"
```

service management tool
```sh
# set default run level for graphical environment
systemctl set-default graphical.target
# enter the graphical environment
systemctl isolate graphical.target
```

Reboot server.  This will show an extra option (the latest kernel) at boot time and will use this as default.  Login as "tux"

Within the graphical emulator open terminal then as super user add VBox guest additions (from Device menu select "Install Guest Additions CD image" - this will make image available but will need to be run as root)
```sh
su -
mount # to view devices
/run/media/tux/VBOX{tab-to-auto-complete}/VBoxL{tab-to-auto-complete}
# reboot once complete
reboot
```

Now you should be able to go full-screen, with Host-f

Right-click on VBOX... on virtual desktop and select "Eject"


## Working at the Command Line

From within virtual machine (server1 GUI), press host+fn+f2 (on Mac), otherwise right-ctrl+f2 (on Windows), to open physical console tty2

Commands
```sh
# to see what terminal I am currently logged on to
tty
# to see who is currently logged on
who
# tux    :0      # graphical environment 
# tux    pts/0   # psuedo terminal 0
# tux    tty2    # physical terminal
```

### Listing files

When we run *ls* to list files we get colouring.  This is because it is being aliased - use the following to see what
```sh
type ls
# ls is aliased to `ls --color=auto'

# to show file types
ls -F
# something/  is a directory
# something@ is a symbolic link

# to clear screen
ctrl-l

# show just the directory, not the contents of that directory
ls -ld
```

### File Types

Everything is a file, even devices such as psuedo terminal

```sh
ls -l /dev/pts/1
# is the same as
ls -l $(tty)
# list block devices (disks and partitions)
lsb
```

Wildcard searching
```sh
# any number of characters
ls -l /dev/sda*
# only one character
ls -l /dev/sda?
# value 1 or 2
ls -l /dev/sda[12]
```

Linux version
```sh
cat /etc/centos-release
cat /etc/redhat-release
lsb_release -d
```

Using rpm package manager
```sh
which lsb_release
# -q query, -f file
rpm -qf /usr/bin/lsb_release
rpm -qf $(which lsb_release)
```

### Working with Files

### Working with Directories

```sh
mkdir one two
touch one/file{1..5}
cp -R one two
# you may need to install tree command: sudo yum install tree
tree two
```
```sh
mkdir -m 777 d1
mkdir -m 700 d2
ls -ld d1 d2
```

### Working with Links

Hard link count - the following outputs a number that identifies the number of subdirectories (minus 2 for the . and .. directories)
```sh
ls -ld /etc
```

Directory entry or inode number - filesystem metadata
```sh
ls -ldi /etc
```

```sh
echo hello > f1
ls -l f1 # shows hard link count of 1
# create hard link between f1 and f2
ln f1 f2
ls -li f1 f2 # shows hard link count of 2 but identical inode numbers
# create a symbolic link between f1 and f3
ln -s f1 f3
ls -li f1 f2 f3 # shows hard link count of 1 and a different inode number
```
Hard links can only be done with files in the same file system

## Reading Files

### Regular Expressions and grep

```sh
sudo yum install ntp
cat /etc/ntp.conf
wc -l !$ 
# where !$ is last argument, i.e. /etc/ntp.conf
cp !$ .
grep '\bserver\b' ntp.conf
# where \b is a word boundary
# using extended grep -E (for enhanced regular expressions)
grep -E 'ion$' /usr/share/dict/words
grep -E '^po..ute$' /usr/share/dict/words
grep -E '[aeiou]{5}' /usr/share/dict/words
```

### Using sed to Edit Files

```sh
sed '/^#/d ; /^$/d' ntp.conf # will write to standard output
sed -i '/^#/d ; /^$/d' ntp.conf # -i option will do an in-place edit
```

Create a function (in memory)
```sh
function clean_file {
  sed -i '/^#/d ; /^$/d' $1 # -i option will do an in-place edit.  $1 is the first input parameter
}
clean_file ntp.conf
```

### Comparing Files

```sh
cp ntp.conf ntp.new
echo new >> ntp.new
diff ntp.conf ntp.new
```

Comparing binary files
```sh
md5sum /usr/bin/passwd
# ssh to server2 and run the same command
md5sum /usr/bin/passwd
```

To verify an installed package
```sh
rpm -V ntp
```

### Using Find

*/usr ("UNIX system resources" directory)*

```sh
find /usr/share/doc -name '*.pdf'
# is the same as including "-print" option
find /usr/share/doc -name '*.pdf' -print 
# execute a copy on the output from the find - to copy all docs to the current directory
find /usr/share/doc -name '*.pdf' -exec cp {} . \;
# use -iname for case-insensitive search
find . -iname '*.pdf'
# delete based on output from find
find . -name '*.pdf' -delete
# find file of type link
find /etc -type l
# limit to just that directory, not below
find /etc -maxdepth 1 -type l
# look for large files
df -h /boot
find /boot -size +20000k -type f
find /boot -size +20000k -type f -exec du -h {} \;
```

## Using the vim Text Editor

```sh
touch newfile
ls -l newfile
stat newfile
# create file with different date
touch -d '10 February 2017' newfile
```

Search command history
```sh
!s # for last command starting with "s"
```

Using nano
```sh
sudo yum install -y nano
nano newfile
# use ctrl-x to exit and save
```

Learning vi
```sh
vimtutor # for a getting started tutorial
(or)
vi then :help # to get into the help menu
```

Setting options from within vi
```
:set number # turn on line numbering
:set nonumber # turn off line numbering
```

Control file (~/.vimrc)
```
set showmode nonumber
set hlsearch # highlight search
set ai ts=4 expandtab # auto-indent, tab spaces = 4, tabs will be converted to spaces
abbr _sh #!/bin/bash # abbreviate: type "_sh" then tab or esc and it will be replaced by the shebang
nmap <C-N> :set invnumber<CR> # key mapping: ctrl-n => toggle line numbering
```

The 3 modes of vi:  
1. command mode  
2. insert mode  
3. last line mode  

Useful "last line mode" commands
```
:e! # discard changes but to not exit (like :q!)
:6,8w newfile2 # write lines 6-8 to newfile2
:r newfile2 (will copy in file at cursor)
```

Useful "command mode" commands
```
I - insert at beginning of line
A - append at end of line
p - paste after line
P - paste before line
u - undo
g~~ - change case of whole line (Upper to lower and vice versa)
gUU - change case to UPPER for whole line
~ - change case of current character
d$ - delete to end of line
dG - delete to end of file
```

## Piping and Redirection

### Redirecting STDOUT

Shell options - noclobber
```sh
set -o # to view
set -o noclobber # to turn on noclobber
# can be set in .bashrc logon script 
set +o noclobber # to turn off
```

```sh
df -h 1> file1 # using 1> to explicitly redirect to STDOUT (standard output)
date +%F > file1 # throws an error because noclobber is on: -bash: file1: cannot overwrite existing file
# but we can still append to the file
date +%F >> file1
# but we can override this using the |
date +%F >| file1
```

Search command history
```sh
ctrl-r then start typing
```

### Redirecting STDERR

```sh
ls /etcw > err
ls /etcw 2> err # throws an error because of noclobber
ls /etcw 2>| err
find /etc -type l 2> /dev/null
# to include STDERR and STDOUT in same file use &
find /etc -type l &> err.txt
```

### Redirecting STDIN

```sh
df -hlT > diskfree
mail -s "Disk Free" tux < diskfree
mail
1 # to select message
d # to delete message
q # to quit
```

### HERE documents

```sh
cat > mynewfile <<END
> this is a little file
> that we can create
> even with scripts
> END
cat mynewfile
```

### Command Pipelines

Using Unamed Pipes
```sh
ls | ws -l
head -n1 /etc/passwd
cut -f7 -d: /etc/passwd
cut -f7 -d: /etc/passwd | sort
cut -f7 -d: /etc/passwd | sort | uniq
cut -f7 -d: /etc/passwd | sort | uniq | wc -l
```

Named Pipes  
```sh
mkfifo mypipe
ls -l !$
# prw-rw-r--. 1 tux tux 0 Mar 14 23:27 mypipe
# Usually a named pipe appears as a file, and generally processes attach to it for inter-process communication
# Now, in a second terminal type the following - this will 'hang'
ls > mypipe
# back in the original terminal
wc -l < mypipe
# this will display data from the pipe and free up the second terminal
```

### Using the command tee

Using the redirect to send output to a file means we will not see the output on the screen.  The tee command will do both
```sh
ls > f89
ls | tee f89
```

## Archiving Files

### The tar command
```sh
du -sh .
tar -cvf /tmp/$USER.tar $HOME
# view contents 
cd /tmp
tar -tf tux.tar
# expand archive
mkdir test
cd test
tar -xvf ../tux.tar
```

### Using Compression
```sh
gzip tux.tar
file tux.tar.gz
gunzip tux.tar.gz
# using bunzip for better compression
bzip2 tux.tar
bunzip2 tux.tar.bz2
```

```sh
# check the time it takes
time tar -cvf tux.tar $HOME
# include gzip into the tar command
time tar -cvzf tux.tar.gz $HOME
# include bzip2 into the tar command
time tar -cvjf tux.tar.bz2 $HOME
# to expand a zipped archive
tar -xzf tux.tar.gz
```

### Working with CPIO
```sh
cd /usr/share/doc
find -name '*.pdf'| cpio -o > /tmp/pdf.cpio
cd
mkdir pdf
cd !$
cpio -id < /tmp/pdf.cpio
ls
```

Ramdisk
```sh
cd /tmp
sudo cp /boot/initramfs-3.10.0-514.10.2.el7.x86_64.img .
sudo chown tux. initramfs-3.10.0-514.10.2.el7.x86_64.img
mkdir work
cd work
cpio -id < ../initramfs-3.10.0-514.10.2.el7.x86_64.img
ls
```

### Imaging with dd (disk duplicator)

From Virtualbox menu: Devices => Optical Drives => Choose Disk Image.  Selecting an ISO file will mount this on the cd-rom drive /dev/sr0
```sh
# take an image of the optical drive
dd if=/dev/sr0 of=netinstall.iso
# take an image of the boot disk
dd if=/dev/sda1 of=boot.img
# backup first 512 bytes of master boot record
su -
fdisk -l
dd if=/dev/sda of=sda.mbr count=1 bs=512
# overwrite master boot record...
dd if=/dev/zero of=/dev/sda count=1 bs=512
# conform that partition tables no longer exist
fdisk -l
# restore partition table
dd if=sda.mbr of=/dev/sda
```

## Accessing Command Line Help

### Accessing Man Pages

The 'man' page for 'man' (```man man```) shows the section numbers of the manual followed by the types of pages they contain.

    1 * Executable programs or shell commands
    2   System calls (functions provided by the kernel)
    3   Library calls (functions within program libraries)
    4   Special files (usually found in /dev)
    5 * File formats and conventions eg /etc/passwd
    6   Games
    7   Miscellaneous (including macro packages and conventions), e.g. man(7), groff(7)
    8 * System administration commands (usually only for root)
    9   Kernel routines [Non standard]

e.g.
```sh
man crontab # to see command details
man 5 crontab # to see file format details
```

### Working with Info Pages

```sh
info ls
# page down to item below * Menu: and hit enter
# hit 'b' to go back to top of page
# hit 'u' to go up a level
```

### Using version controlwith RCS

Create simple script hello.sh, then
```sh
ci hello.sh # ci for 'c'heck 'i'n
# enter a comment, followed by a dot on new line to end comment
>> initial version
>> .
# view history
rlog -b hello.sh
# to edit use 'c'heck 'o'ut
co -l hello.sh
# make a change then 'c'heck back 'i'n
ci hello.sh
>> modified message
>> .
!$
# to checkout a particular version
co -l -r1.1 hello.sh
```

## Understanding File Permissions

Linux ACLs (access control lists) asre built-in to Centos7

### Listing permissions
```sh
ls -l hello.sh,v
stat hello.sh,v
stat -c %A hello.sh,v
stat -c %a hello.sh,v
```

### Managing default permissions

Use umask to specify permissions to be **removed/blocked** from the default permissions, which are 666 for files and 777 for directories
```sh
umask 27
touch file1 # should be rw-r-----
umask 77
touch file4 # should be -rw-------
```

### Setting permissions
```sh
chmod 764 file1 # octal notation
chmod u=rwx,g=rw,o=r file1 # symbolic notation
chmod a+x file2 # give everyone execute permission
chmod o= file2 # remove all permissions for other
```

Sticky bit
```sh
chmod +t /usr/local/tmp # set the sticky bit on a directory
chmod 1777 /usr/local/tmp # same as above, using octal notation
```

### Managing File Ownership
```sh
newgrp wheel # set the primary group of the current user.  NB this creates a new shell and logs user in.  Input exit to get back.
cp -a file2 /tmp/file2a # to keep the file permissions
```

## Access the Root Account

### Using su
"su' stands for *substitute user* (not super user)  

Dollar prompt usually indicates standard (non-root) user   
```sh
su # without any parameters will logon as root - home directory and environment variables will not change 
id # to view details associated with logged on user
exit
su - # (short for su -l) will logon as root - home directory and environment variables will change as for root user
```
Hash prompt associated with root user

### Delegation Using sudo

Does not require divulging the root password to user  

Access is delegated via the sudoers file.  This is not edited directly with vi, but rather with visudo
```sh
sudo visudo # file is located at /etc/sudoers - we do not need to specify
```

Add entry
```sh
tux ALL=(root) ALL
# tux is the user.  If this was prefixed with % this would refer to the tux group
# first "ALL" refers to any host that the connection originates from
# sudo 'as' (root)
# second "ALL" refers to the commands that user tux is allowed to run
```
Aliases
```sh
# Can be created for users or commands.  Case sensitive.  Alias names must be uppercase
User_Alias HELPDESK = user1, user2
Cmnd_Alias HELPCOMMANDS = /usr/sbin/useradd, /usr/bin/passwd
HELPDESK ALL=(root) HELPCOMMANDS # all helpdesk users can execute these specific commands
```

### Restricting root access to SSH

Edit ssh daemon config
```sh
vi /etc/ssh/sshd_config
# uncomment and set: PermitRootLogin no
# save and exit
# restart ssh daemon
systemctl restart sshd
```  
_warning: do not do this unless you have some other way of logging in to your server_

## Accessing Servers with SSH

### Configuring the SSH Client
Create config file in ~/.ssh directory
```sh
vi ~/.ssh/config
# add the following lines:
Host server2
  HostName 192.168.56.102
  User root
  Port 22
```

### Using Key Based Authentication
Create rsa key pair
```sh
ssh-keygen -t rsa
# default keys generated
#  private: id_rsa
#  public: id_rsa.pub
# Copy public key to remote server
ssh-copy-id -i id_rsa.pub server2
```
Using SSH agent
```sh
ssh-agent bash # open in a new shell
ssh-add # should prompt for passphrase, if we used one, and will cache the credentials in the SSH agent
# update /etc/ssh/sshd_config to only allow root login with keys: PermitRootLogin without-password
```

### Copy Files Securely

Using Aliases (in ~/.ssh/config)
```sh
# scp <local file> <host alias>:<remote directory>
scp /etc/hosts server2:/tmp
# or in the reverse direction
# scp <host alias>:<remote director/file> <local directory>
scp server2:/tmp/hosts .
```

On Windows use WinSCP GUI tool

## Using Screen and Script

### Using Script as Collaboration Tool

This will record all commands and output between typing _script_ and _exit_
```sh
script
ls
ls -a
ls -A
exit
```

Create a named pipe
```sh
mkfifo /tmp/mypipe
script -f /tmp/mypipe
# in a second terminal
cat /tmp/mypipe
# now, any commands entered in first terminal will be immediately visible in second terminal.
# type exit in second terminal to end 
```
Would have been really useful in the past, but probably just do a screenshare now.


### Running Commans Across Your Estate with Screen

```sh
yum install -y screen
screen
# create a screen config file in home directory - to control what screens are set-up when you type screen
vi .screenrc
# with contents
screen -t s1 0 bash
screen -t s2 1 ssh server2
# save and exit, then type:
screen
# this should create screen 0 in current shell, and screen 1 logged on to server2
# ctrl-a n for Next (press ctrl and a together, let go then press n or p or shift-")
# ctrl-a p for Previous
# ctrl-a " to list screens
exit # to get out of screen
```
To install or run a command across multiple servers using screen
```sh
# use .screenrc file to define list of servers
ctrl-a : # (ctrl-a colon) will bring a command prompt to bottom of current screen, type the following:
:at "#" stuff "yum install -y zsh^M"
# "#" for all screens in .screenrc
# "stuff" to stuff commands into screen buffer
# finish off with end-of-line character
```


[Next: Linux Operation Essentials](./2_linux_operation_essentials.md)



