# Netstack Windows 10 pro workstation on 1U-Dell


## Build hswin10pro
1. nswin10 
  - BIOS Config
    1. Name: hswin10pro VMID: 303 Server: pm01
    2. Windows 10/2016
    3. CD/DVD: Storage: hsfreenas ISO: Win10_1909_English_x64.iso
    4. Hard Disk: Bus: VirtIO 0 Storage: local-lvm Disk size: 32GB Cache: Write back
    5. 6 cores, 8192 (8GB), Enable Numa
    6. Network: Bridge: vmbro Model: VirtIO Firewall: UNCHECKED <disable>
    7. Add second CD/DVD: Storage: hsfreenas ISO: virt-win-0.1.171.iso
  - Boot Win10 Install USB
    1. Proceed with windows installation as normal. Custom: Install Windows only (advanced)
    2. Where to Install setup will not find any drives.
        - Select "Load driver"
        - Browse to CD: viostor > w10 > amd64
        - Click OK should load the Red Hat VirtIO SCSI controller
        - Click NEXT to partion and continue the installation
    3. Finish Windows Install remove Install USB and Reboot
  - Start nswin10
    1. Login on Console with User
    2. Enable "Microsoft Remote Connection"
    3. Turn off Windows Firewall
  - Reboot
  - Setup Resources
    1. [Windows 10 iso - Download link](https://www.microsoft.com/en-us/software-download/windows10ISO)
    2. [tbd]()
2. Run Windows Updates
  - Keep updating and rebooting until is stops complaining
3. Basic Installs
  - Download and install Chrome Downloads/ChromeSetup
  - Download and install FireFox Downloads/Firefox Installer
  - Download and install Edge Downloads/MicrosoftEdgeSetup
4. Connect [NFS Storage](https://graspingtech.com/mount-nfs-share-windows-10/)
  - CD/DVD: Storage: nsfreenas ISO: Win10_1909_English_x64.iso
  - Install NFS Client
    1. Open "Control Panel"
    2. Turn ON windows features
      - Services for NFS -> Client for NFS
      - Windows Subsystem for Linux
    3. Enable Write Permissions for the Anonymous User
      - On cmd run "mount" should see UID=-2 and GID=-2
      - Open "regedit"
      - Browse: HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\ClientForNFS\CurrentVersion\Default
      - Add New DWORD (32-bit): AnonymousUid  Value: 0x000000 (0)
      - Add New DWORD (32-bit): AnonymousGid  Value: 0x000000 (0)
    4. Reboot (or restart NFS client)
  - Connect to nspool
    ```
    C:\Users\nsadmin>mount -o anon \\192.168.128.2\mnt\nspool N:
    ```
  - Verify mount
    ```
    C:\Users\nsadmin>mount

    Local    Remote                                 Properties
    -------------------------------------------------------------------------------
    H:       \\192.168.128.2\mnt\nspool               UID=0, GID=0
                                                    rsize=131072, wsize=131072
                                                    mount=soft, timeout=6.4
                                                    retry=1, locking=yes
                                                    fileaccess=755, lang=ANSI
                                                    casesensitive=no
                                                    sec=sys

    C:\Users\nsadmin>
    ```
5. Install Ubuntu 20.04 LTS app via Play store
  - Start session with hsadmin - normalpw
  - Where is [file system root in windows](https://askubuntu.com/questions/759880/where-is-the-ubuntu-file-system-root-directory-in-windows-subsystem-for-linux-an)
  - Did not get working but... [nfs mount from windows subsystem](https://superuser.com/questions/1128634/how-to-access-mounted-network-drive-on-windows-linux-subsystem/1261563)
  - Install nsf-common
    1. sudo apt install nfs-common
  - NFS Mount the 192.168.1.2:/mnt/hspool to /mnt/hspool
    1. sudo mkdir -p /mnt/hspool
    2. sudo mount 192.168.1.2:/mnt/hspool /mnt/hspool -vvv
    3. Verify: df -h
    4. sudo touch /mnt/hspool/thistest.txt
    5. ls -l /mnt/hspool/thistest.txt
6. Install [Visual Studio Code](https://code.visualstudio.com/)
7. Shutdown backup: tbd (backup before Key Activation)
8. Activate Windows Pro Key
9. Shutdown backup: tbd (backup after Key Activation)
  
## Notes
1. [Windows Sandbox](https://www.theverge.com/2018/12/19/18147991/microsoft-windows-sandbox-security-safety-isolation-standalone-apps)
2. [Windows Sandbox - mstech](https://techcommunity.microsoft.com/t5/windows-kernel-internals/windows-sandbox/ba-p/301849#)
3. [Windows Containers - msdocs](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/)
4. [Windows k8s - msdocs](https://docs.microsoft.com/en-us/virtualization/windowscontainers/kubernetes/getting-started-kubernetes-windows)
5. [NFS mount on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-18-04)
