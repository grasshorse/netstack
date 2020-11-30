# backup using zfs snapshot and replication

Backing up freenas we use zfs snapshots.  We will use the backup server to drive the replication.

# On the backup freenas

1. Create replication task
  - Tasks -> Replication Tasks
  - Source Location: On a Different System
  - Destination Location: On the System -> ghpool/backup/projects (not recursive)
  - SSH Connection
    - Source Server: 192.168.254.6
    - User: root
    - Generate SSH Key
