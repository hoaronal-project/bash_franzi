# bash_franzi
## Log a complete history of commands typed in Bash for all users.

This is a fork of Francois Scheurer's excellent blog post [Bash Audit / Command Logger](http://www.pointsoftware.ch/en/howto-bash-audit-command-logger/) from 2012. I have made minor fixes and customizations, hopefully have made it simpler to install, and have been testing it for over 3 years.

Here is an excerpt from the original blog post:

> Having a complete history of all typed commands can be very helpful in many scenarios:
> 
> * when several administrators work together on the same server and need to know what was done previously
> * when someone need to redo an older sequence of commands or to understand an undocumented maintenance process
> * for troubleshooting or forensic analysis, by crosschecking the date of an event or of a file with the commands executed at that date
> 
> The standard ‘.bash_history’ file of the shell is unfortunately not written on disk in the case of a crash and it may be deleted by the user.
Another problem is that when many shell sessions are running concurrently, their logging will only occur when they are closed, therefore the commands of the history will not appear in their chronological order.
Furthermore, ‘.bash_history’ will not include essential information like the ‘working directory’ of the command; and by default the repetition or re-edition of commands will not be logged, too.

A quick Google search for _log all bash commands_ will find a lot of tools and instructions which are similar to bash_franzi. I am not claiming that bash_franzi is the best, but it works well for me.

## Sample output

```
Oct 23 18:43:24 trjon12 [audit  as jon] /home/jon: zcat -f `ls -vr /var/log/bash*` |grep '[d]d' |grep urandom
Oct 23 18:44:27 trjon12 [audit  as jon] /home/jon: dd if=/dev/urandom bs=1 count=12 status=none |hexdump -v -e '/1 "%02x"'
Oct 23 18:44:46 trjon12 [audit  as jon] /home/jon: dd if=/dev/urandom bs=1 count=12 status=none |base64 -e
Oct 23 18:44:49 trjon12 [audit  as jon] /home/jon: dd if=/dev/urandom bs=1 count=12 status=none |base64 --encode
Oct 23 18:44:59 trjon12 [audit  as jon] /home/jon: dd if=/dev/urandom bs=1 count=12 status=none |base64
Oct 23 19:03:12 trjon12 [audit  as jon]: #=== session opened ===
Oct 23 19:03:14 trjon12 [audit  as jon] /home/jon: man cd
Oct 23 19:03:19 trjon12 [audit  as jon] /home/jon: man sh
Oct 23 19:03:29 trjon12 [audit  as jon] /home/jon: man bash
Oct 23 19:03:32 trjon12 [audit  as jon]: #=== session closed ===
Oct 23 19:13:20 trjon12 [audit  as jon]: #=== session opened ===
Oct 23 19:13:25 trjon12 [audit  as jon] /home/jon: touch t
Oct 23 19:13:37 trjon12 [audit  as jon] /home/jon: chmod 777 t
Oct 23 19:14:38 trjon12 [audit  as jon] /home/jon: chmod ugo-x go-w o-r t
Oct 23 19:14:44 trjon12 [audit  as jon] /home/jon: chmod ugo-xgo-wo-r t
Oct 23 19:14:50 trjon12 [audit  as jon] /home/jon: man chmod
Oct 23 19:15:53 trjon12 [audit  as jon] /home/jon: chmod ugo-x,go-w,o-r t
Oct 23 19:15:56 trjon12 [audit  as jon] /home/jon: ll t
Oct 23 19:16:06 trjon12 [audit  as jon] /home/jon: chmod 640 t
Oct 23 19:16:07 trjon12 [audit  as jon] /home/jon: ll t
```

## Installation

* _note_--new versions can be installed on top of old ones without uninstalling
* log in as a user with `sudo` privileges
* _(optional)_ copy old log files from archive:
```
cd [directory of old log files]
sudo cp -i bash.log* /var/log/
sudo chown syslog:adm /var/log/bash.log.*
```
* download and install:
```
cd /tmp
wget https://gitlab.com/bitinerant/bash_franzi/raw/master/bash_franzi_install
sudo bash bash_franzi_install
```
* _note_--bash sessions which are already open will not be logged

## Usage

* verify log:
```
tail -f /var/log/bash.log
```
* search all logs (square brackets around 1st letter avoids finding _this_ command):
```
zcat -f $(ls -vr /var/log/bash*) |grep '[y]our-search'
```

## Removal

* edit the following files and remove reference to _/etc/bash-franzi_
```
/etc/skel/.bashrc
/root/.bashrc
/home/*/.bashrc
```
* edit the following file and remove reference to _/etc/bash-franzi-forcecommand.sh_
```
/etc/ssh/sshd_config
```
* delete core files and reload ssh:
```
sudo rm /etc/bash-franzi /etc/bash-franzi-forcecommand.sh /etc/rsyslog.d/45-bash-franzi.conf
sudo service ssh reload
```
* edit the following file and remove reference to _/var/log/bash.log_
```
/etc/logrotate.d/rsyslog
```
* _(optional)_ delete old log files:
```
sudo rm /var/log/bash.log*
```

## Known issues

* the installation script fails when no _.bashrc_ exists for a user
* _.bashrc_ relies on _.profile_ to execute it
* a numeric command typed at the command prompt causes _integer expression expected_ errors
