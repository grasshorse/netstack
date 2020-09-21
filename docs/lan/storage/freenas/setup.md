# freeNAS configuration for ghrender

## freeNAS install
1. Use freeNAS install USB thumbdrive
2. Install...

## freeNAS Storage Pool Config
1. Login to FreeNAS (root - What#Time)
2. From FreeNAS Dashboard -> Storage -> Pools -> Create
  - Name: drivepool
  - Type: raidz
  - Add Drives (the 4 disks) -> Set to raidz (default is raidz2)
3. Confirm Create
5. In Storage / Pools -> drivepool -> Add Dataset
  - Name: farm
  - Comments: blener render farm shared drive
5. In Services -> Enable NFS and Start Automatically

## Hindsight Jail
1. From FreeNAS Dashboard -> Jails -> ADD
  - Jail Name: hindsight
  - Release: 11.2-RELEASE
  - NEXT:
  - IPv4 Interface: em0
  - IPv4 Address: 192.168.1.6
  - IPv4 Netmask: 24
  - NEXT:
  - Confirm
```
Jail Summary
Jail Name : hindsight
Release : 11.2-RELEASE
IPv4 Address : em0|192.168.1.6/24
Confirm these settings.
```
2. Confirm 
  - Basic: Auto-Start
  - Jail: allow_set_hostname, allow_raw_sockets
3. Install hindsight - Jail -> hindsight -> shell
  - root@hindsight:~ # pkg install stow
  - root@hindsight:~ # mkdir /usr/local/stow
  - root@hindsight:~ # cd /usr/local/stow/
  - root@hindsight:~ # scp cat@192.168.1.30:/Users/cat/Downloads/hindsight_bsd_stow.tgz .
  - root@hindsight:~ # tar -xzvf hindsight_bsd_stow.tgz
  - root@hindsight:~ # vi /etc/rc.d/hindsight
```
$ cat /etc/rc.d/hindsight
#!/bin/sh
#
# PROVIDE: hindsight 
# REQUIRE: DAEMON
# KEYWORD: shutdown

. /etc/rc.subr

name=hindsight
rcvar=hindsight_enable

command="/usr/sbin/daemon"
command_args="-P /var/run/hindsight.pid -T hindsight /usr/local/bin/hindsight /usr/local/hindsight/hindsight.cfg 7"

load_rc_config $name

#
# DO NOT CHANGE THESE DEFAULT VALUES HERE
# SET THEM IN THE /etc/rc.conf FILE
#
hindsight_enable=${hindsight_enable-"NO"}
pidfile=${hindsight_pidfile-"/var/run/hindsight.pid"}

run_rc_command "$1"
```
  - root@hindsight:~ # chmod 555 /etc/rc.d/hindsight
  - root@hindsight:~ # vi /etc/rc.conf
```
 hindsight_enable="YES"
```
  - root@hindsight:~ # systemclt start hindsight
  - root@hindsight:~ # systemclt status hindsight
  - root@hindsight:~ # lsb_heka_cat /usr/local/hindsight/output/input/0.log
  - syslog collector is running on the standard port 514
  - pfsense collector is on port 4514

## Hindsight update
1. root@hindsight:~ # /etc/rc.d/hindsight status || (service hindsight status)
2. root@hindsight:~ # /etc/rc.d/hindsight stop || (service hindsight stop)
3. root@hindsight:~ # cd /usr/local/
4. root@hindsight:~ # scp cat@192.168.1.30:/Users/cat/Downloads/hs_bsd_stow.tgz .
5. root@hindsight:~ # stow -v -D *
6. root@hindsight:~ # rm -rf stow
7. root@hindsight:~ # tar -zxf hs_bsd_stow.tgz
8. root@hindsight:~ # cd stow
9. root@hindsight:~ # stow -v *
10. root@hindsight:~ # /etc/rc.d/hindsight start || (service hindsight start)
11. root@hindsight:~ # /etc/rc.d/hindsight status || (service hindsight status)
12. root@hindsight:~ # lsb_heka_cat /usr/local/hindsight/output/input/0.log
  
  
## Install new FreeNAS and import existing Pool
1. USB FreeNAS install to new USB
2. Login to FreeNAS firstime set pw (What#Time)
3. Storage -> Pools -> Add 
   - Import an existing pool
   - NEXT:
   - Is the pool encrypted? No
   - NEXT:
   - Pool: drivepool
   - NEXT:
   - Confirm: IMPORT
```
Pool Import Summary
Pool to import : drivepool | 9174643673441383865
Confirm these settings.
```
