# freeNAS configuration

## freeNAS install 
1. Create freeNAS install USB thumbdrive [Etcher - download](https://www.balena.io/etcher/k) or [Rufus - download](https://rufus.ie/)
2. Insert freeNAS install USB and a USB boot target
3. Install...
4. Pull freeNAS install USB and reboot

## freeNAS Storage Pool Config
1. Login to FreeNAS (root - yoursecurepassword)
2. View Dashboard check for any notifications
3. From FreeNAS Dashboard -> Storage -> Pools -> Add -> Create new pool
  - Name: nspool
  - Type: raidz2
  - Add Drives (the 4 disks) -> Set to raidz2 (default is raidz2)
4. Confirm Create (will delete all data)
5. Add S.M.A.R.T. Test schedule to drives
  - monthly
6. Add SCRUB schedule to drives
  - monthly

## freeNAS NFS Share Dataset Configuration
1. Check Status of nspool [Storage - Pools - Gear Status](http://192.168.2.83/ui/storage/pools/status/1)
2. Add NFS_ISO_Share [Storage - Pools -> nspool -> Add Dataset](http://192.168.2.83/ui/storage/pools/id/nspool/dataset/add/nspool) [LT-video](https://youtu.be/k_gvwU15EyE?t=95)
  - Name: NFS_ISO_Share
  - Comments: NFS ISO files for LAN Share
  - Sync: Inherit (standard)
  - Leave rest default
  - SAVE
2. Add Projects [Storage - Pools -> nspool -> Add Dataset](http://192.168.2.83/ui/storage/pools/id/nspool/dataset/add/nspool) [LT-video](https://youtu.be/k_gvwU15EyE?t=95)
  - Name: Projects
  - Comments: Projects share directory
  - Sync: Inherit (standard)
  - Leave rest default
  - SAVE
3. Edit Permissions on Projects Dataset [LT-video](https://youtu.be/k_gvwU15EyE?t=101)
  - Give Write to Group and Other
  - Apply Permissions Recursively
  - Save
4. Create NFS share [Sharing - NFS - Add](http://192.168.2.83/ui/sharing/nfs/add) [LT-video](https://youtu.be/k_gvwU15EyE?t=117)
  - Path: /mnt/nspool/NFS_ISO_Share
  - All dirs (checked)
  - Enabled (checked)
  - Save

## Reference
- [TrueNAS 12.0 download](https://download.freenas.org/12.0/MASTER/latest/x64/) 
- [TrueNAS core 12.0 Video](https://www.youtube.com/watch?v=KS6gVJnmy2U)
- [FreeNAS 11.3 download](https://www.freenas.org/download-freenas-release/) 
- [youtube - FreeNAS Smart Tests and Scrub Tests](https://www.youtube.com/watch?v=w0ZKuXN1AVM&vl=en)
- [youtube - How to Check SMART Information in FreeNAS](https://www.youtube.com/watch?v=PyBqahRsXwY)
- [youtube - tbd]()
- [youtube - Lawrence Systems - Open Source File Sync: Getting Started Tutorial With Syncthing on Windows & Linux](https://www.youtube.com/watch?v=O5O4ajGWZz8)
- [youtube - Lawrence Systems - Why We Use Syncthing, The Open Source Private File Syncing Tool instead of NextCloud](https://www.youtube.com/watch?v=bNiiJe8NpEw)
- [youtube - Lawrence Systems - FreeNAS 11 Rsync Server Setup](https://www.youtube.com/watch?v=Qhlp18QTUTo&t=359s)
- [youtube - tbd]()
- [youtube - tbd]()

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
2. Login to FreeNAS firstime set pw (yoursecurepassword)
3. Storage -> Pools -> Add 
   - Import an existing pool
   - NEXT:
   - Is the pool encrypted? No
   - NEXT:
   - Pool: nspool
   - NEXT:
   - Confirm: IMPORT
```
Pool Import Summary
Pool to import : nspool | 9174643673441383865
Confirm these settings.
```
