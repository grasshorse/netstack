[documents index](../../../)
# xcp-ng

## Install server
1. Use Install [xcp-ng Homepage](https://xcp-ng.org/) USB key to boot and install
  - Server Name: nsgXX
  - Special: xxxx#xxxxst
  - IP: DHCP (MAC tied to IP via DHCP later)
2. GUI Admin
  - ssh into system and yum update reboot [LT-video](https://youtu.be/q-jKs62b6Co?t=678)
    ```bash
      cat@cats-Mac-mini ~ % ssh root@192.168.2.38
      root@192.168.2.38's password: 
      Last login: Tue Sep 22 21:34:28 2020 from cats-mac-mini
      [08:19 nsg01 ~]# yum upgrade
      Loaded plugins: fastestmirror
      Loading mirror speeds from cached hostfile
      Excluding mirror: updates.xcp-ng.org
       * xcp-ng-base: mirrors.xcp-ng.org
      Excluding mirror: updates.xcp-ng.org
       * xcp-ng-updates: mirrors.xcp-ng.org
      No packages marked for update
      [08:20 nsg01 ~]# shutdown -r now
    ```
  - Checkout the console
  ```bash
    [08:20 nsg01 ~]# xsconsole
  ```
  - [XOA Appliance install](https://youtu.be/mp-pCgYszqU?t=305)
    ```bash
      bash -c "$(curl -s http://xoa.io/deploy)"
    ```
  - XOA Quick Deploy [https://192.168.2.38](https://192.168.2.38) go to the just installed servers webpage [LT-video](https://youtu.be/q-jKs62b6Co?t=792)
  - [Windows xenadmin Client XOA github](https://github.com/xcp-ng/xenadmin/releases/) NOTE: Not good with 8.1 and is lacking upgrades
  - XOA build from source [Tutorial - How To Build Xen Orchestra From Source Using XenOrchestraInstallerUpdater](https://www.youtube.com/watch?v=lf_tNVomBcE)

## Create local ISO repository [xcp-ng doc](https://github.com/xcp-ng/xcp/wiki/Create-a-local-ISO-repository) 
1. ssh to ng01 server create Local_ISO directory and pull down [small debian](https://www.debian.org/distrib/netinst) [LT-video](https://youtu.be/q-jKs62b6Co?t=903)
    ```bash
      [08:20 nsg01 ~]# cd /
      [08:55 nsg01 /]# mkdir Local_ISO
      [08:55 nsg01 /]# cd Local_ISO/
      [08:55 nsg01 Local_ISO]# pwd
      /Local_ISO
      [09:11 nsg01 Local_ISO]# wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-10.5.0-amd64-netinst.iso
      --2020-09-23 09:14:15--  https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-10.5.0-amd64-netinst.iso
      Resolving cdimage.debian.org (cdimage.debian.org)... 194.71.11.173, 194.71.11.165, 2001:6b0:19::173, ...
      Connecting to cdimage.debian.org (cdimage.debian.org)|194.71.11.173|:443... connected.
      HTTP request sent, awaiting response... 302 Found
      Location: https://caesar.ftp.acc.umu.se/debian-cd/current/amd64/iso-cd/debian-10.5.0-amd64-netinst.iso [following]
      --2020-09-23 09:14:16--  https://caesar.ftp.acc.umu.se/debian-cd/current/amd64/iso-cd/debian-10.5.0-amd64-netinst.iso
      Resolving caesar.ftp.acc.umu.se (caesar.ftp.acc.umu.se)... 194.71.11.142, 2001:6b0:19::142
      Connecting to caesar.ftp.acc.umu.se (caesar.ftp.acc.umu.se)|194.71.11.142|:443... connected.
      HTTP request sent, awaiting response... 200 OK
      Length: 365953024 (349M) [application/x-iso9660-image]
      Saving to: ‘debian-10.5.0-amd64-netinst.iso’

      100%[================================================================================================================================>] 365,953,024 7.18MB/s   in 29s    

      2020-09-23 09:14:47 (12.2 MB/s) - ‘debian-10.5.0-amd64-netinst.iso’ saved [365953024/365953024]

      [09:14 nsg01 Local_ISO]#
    ```
2. Go to [XOA - New - Storage](https://192.168.2.97/#/new/sr) and create Local ISO storage [LT-video](https://youtu.be/q-jKs62b6Co?t=1015)
    - Host: nsg01
    - Name: Local ISO
    - Description: Small ISO storage for server bootstrap
    - Select Storge Type: ISO SR - Local
    - Path: /Local_ISO
    - Click Create
3. Verify iso [XOA - Home - Storages - Local ISO - Disk](https://192.168.2.97/#/srs/ae836939-3ce9-b95a-d7fa-119d9237986d/disks) to verify debian iso

## Add Remote Storage
1. Go to [XOA - New - Storage](https://192.168.2.97/#/new/sr) and create FreeNAS NFS storage [LT-video](https://youtu.be/q-jKs62b6Co?t=1190)
    - Host: nsg01
    - Name: FreeNAS NFS
    - Description: FreeNAS NFS storage on local network
    - Select Storge Type: VDI SR - NFS
    - Server: 192.168.2.83
    - Path: (should query the server) /mnt/nspool/
    - Click Create
 
### Add Local Disks
  - Install a hard drive on a XenServer.
  - Run the following command from the command line interface to display the installed disks:
    ```bash
      fdisk -l
      lsblk
    ```
  - Inspect the disks partitions
    ```bash
      cfdisk /dev/sdc
    ```
  - List the xen server ID
    ```bash
      xe host-list
    ```
  - Run the following command from the command line interface:
    ```bash
      xe sr-create host-uuid<hostid> name-label=<nameLabel> shared=false device-config:device=<devicePath> type=lvm content-type=user
    ```
    - hostID: is shown by xe host-list (just tab if your on the server)
    - nameLabel: is the display name used in XCP-ng
    - devicePath:  shown in fdisk -l or lsblk /dev/sdc
  - Example I ran on hsg04
    ```bash
    xe sr-create host-uuid=7749xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxb0 name-label-"hsg04d2" shared=false device-config=/dev/sdb type=lvm content-type=user
    ```
4. Pass-through Storage
  - Run the following command from the command line interface to display the installed disks:
    ```bash
      lsblk
    ```
  - Inspect the disks partitions delete partitions you want to re-use
    ```bash
      cfdisk /dev/sdc
    ```
  - List the xen server ID
    ```bash
      xe host-list
    ```
  - Make directory with symlinks to devices you want to pass-through
    ```bash
      cd /srv/
      mkdir pass_drives
      cd pass_drives
      ln -s /dev/sdc
    ```
  - Run the following command from the command line interface:
    ```bash
      xe sr-create name-label=Pass_Drives type=udev content-type=disk device-config:location=/srv/pass_drives 
    ```
    - hostID: is shown by xe host-list
    - nameLabel: is the display name used in XCP-ng
    - devicePath:  shown in fdisk -l or lsblk - /dev/sdc
  - fire up vm and connect

## Notes
- [xcp-ng Homepage](https://xcp-ng.org/)
- [xcp-ng Github](https://github.com/xcp-ng/xcp)
- [xen-orchestra - architecture](https://xen-orchestra.com/docs/architecture.html)
- [xcp-ng Guest Tools](https://github.com/xcp-ng/xcp/wiki/Guest-Tools)
- [xcp-ng Graphical Client](https://github.com/xcp-ng/xenadmin/releases/)
- [xcp-ng iso 8.1 Download](http://mirrors.xcp-ng.org/isos/8.1/xcp-ng-8.1.0-2.iso)
- [xcp-ng Best Practices](https://github.com/xcp-ng/xcp/wiki/Best-Practices-Guide)
- [citrix Remove Storate Repository](https://support.citrix.com/article/CTX131328)

### Tutorials
- [Tutorial - XCP-ng quick install - Lawrence System](https://www.youtube.com/watch?v=mp-pCgYszqU)
  - [XOA Appliance install](https://youtu.be/mp-pCgYszqU?t=305)
  ```bash
  bash -c "$(curl -s http://xoa.io/deploy)"
  ```
  - [https://192.168.1.126/ Default user is admin@admin.net with admin as a password ](http://192.168.1.126/)
  - [Xen Orchestra Community build from source](https://xen-orchestra.com/docs/from_the_sources.html)
  - [Xen Orchestra Installer github](https://github.com/ronivay/XenOrchestraInstallerUpdater)
  - [Lawrence hack of Installer - Lawrence System](https://github.com/flipsidecreations/XenOrchestraInstallerUpdater)
- [Tutorial - XCP-ng Install 7.4](https://www.youtube.com/watch?v=bG5enpij0e8&feature=youtu.be)
- [Tutorial - Configuring Citrix XenServer With FreeNAS & ISCSI For Storage](https://www.youtube.com/watch?v=-KmgwQORAX8&list=PLjGQNuuUzvmv1n8W-lDplGiDwlxvSSIcv&index=38)
- [Tutorial - Virtual Lab Setup - Lawrence System](https://www.youtube.com/watch?v=mXwSMh9uk0w)
- [Tutorial - XCP-NG 8.0 HA High Availability Cluster Setup](https://www.youtube.com/watch?v=jvhUY81pBw0)
- [Tutorial - Explaining Resource Pools With Xenserver XCP-NG & Xen Orchestra](https://www.youtube.com/watch?v=imOsGG9AmOk)
- [Tutorial - VM Backups, Disaster Recovery and Continuous Replication with Xen Orchestra Backup](https://www.youtube.com/watch?v=1tJZAc-A4kU)
- [Tutorial - XCP-NG & Xenmotion: Migrating Live VM's Between Servers](https://www.youtube.com/watch?v=5XoXQAIjFH8)
- [Tutorial - How To Load XCP-NG Xenserver PV Drivers via Windows Update & Xen Orchestra](https://www.youtube.com/watch?v=nGfx5upOk8c)
- [Tutorial - Running pfsense on XCP-NG Xenserver & Installing Xenserver tools](https://www.youtube.com/watch?v=hy6RwgDm1p0)
- [Tutorial - XCP-NG Stack - How to load XEN Tools in windows VM's](https://www.youtube.com/watch?v=SsuoPzKXnBA)
- [Tutorial - How to add additional hard disk in xenserver](https://www.youtube.com/watch?v=HgjfQKr6u1w)
- [Tutorial - Xenserver Hard Drive Whole Disk Passthrough with XCP-NG](https://www.youtube.com/watch?v=vSDDMIG6Huk)
- [Tutorial - Citrix Xenserver Adding A Hard Drive](https://www.youtube.com/watch?v=gNLBNUHI1uE)
- [Tutorial - How To Build Xen Orchestra From Source Using XenOrchestraInstallerUpdater](https://www.youtube.com/watch?v=lf_tNVomBcE)


### Setup disk on hsg03
```
root@192.168.1.123's password: 
Last login: Mon May 25 20:20:31 2020 from 192.168.1.120
[17:01 hsg03 ~]# ls
CentOS-7-x86_64-DVD-2003.iso
[17:01 hsg03 ~]# lsblk
NAME                                                            MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sdb                                                               8:16   0 136.7G  0 disk 
├─sdb2                                                            8:18   0   512M  0 part 
├─sdb3                                                            8:19   0 136.2G  0 part 
└─sdb1                                                            8:17   0  1007K  0 part 
sda                                                               8:0    0 136.7G  0 disk 
├─sda4                                                            8:4    0   512M  0 part 
├─sda2                                                            8:2    0    18G  0 part 
├─sda5                                                            8:5    0     4G  0 part /var/log
├─sda3                                                            8:3    0  95.2G  0 part 
│ └─VG_XenStorage--b4394697--21e5--f609--94dc--97817d48b2c3-MGT 253:0    0     4M  0 lvm  
├─sda1                                                            8:1    0    18G  0 part /
└─sda6                                                            8:6    0     1G  0 part [SWAP]
[17:04 hsg03 ~]# xe host-list
uuid ( RO)                : 9d41f57b-2da4-492d-8a7e-79309ec72ac7
          name-label ( RW): hsg03
    name-description ( RW): Default install


uuid ( RO)                : de0ae348-63d3-402f-aeba-04091be1bca9
          name-label ( RW): hsg02
    name-description ( RW): Default install


uuid ( RO)                : 7749899f-1c7b-465f-9123-4e0dc59077b0
          name-label ( RW): hsg08
    name-description ( RW): Default install


uuid ( RO)                : 5f64dbdf-781e-4b86-be03-6be544870402
          name-label ( RW): hsg04
    name-description ( RW): Default install


[17:05 hsg03 ~]# xe sr-create host-uuid=9d41f57b-2da4-492d-8a7e-79309ec72ac7  name-label=hsg03d2 shared=false device-config:device=/dev/sdb type=lvm content-type=user
729d3457-7a10-17d8-0e8f-68bd0042b712
[17:06 hsg03 ~]# 
```
