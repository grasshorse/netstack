# backup proceedure for netstack ops

## rsync
Use rsync on windows via bash to move nstestShare (on c:) to network volume
```
cat@catSurface:~$ rsync -avzP /mnt/c/nstestShare 'ssh -p22' buadmin@192.168.128.3:/volume1/bg/rsync
```

### Resources
- [rsync on synology](https://www.synology.com/en-global/knowledgebase/DSM/help/DSM/AdminCenter/application_backupserv_sharedfoldersync)
- [rsync from windows 10](https://www.linux.com/training-tutorials/most-useful-linux-commands-you-can-run-windows-10/)
- [Setup ssh access on windows 10](https://www.hanselman.com/blog/HowToSSHIntoAWindows10MachineFromLinuxORWindowsORAnywhere.aspx)
- [Could not chdir to home dir /var/services/homes/buadmin](https://www.linuxquestions.org/questions/linux-newbie-8/%27could-not-chdir-to-home-directory-home-%5Buser%5D-permission-denied%27-780328/)
