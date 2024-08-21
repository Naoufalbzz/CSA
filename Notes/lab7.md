# Lab 7
## BorgBackup
- Create a folder on the web VM and store the files (e.g. ~/important-files). Use curl with the --location and --remote-name-all options. What do these options do? Why do you need them? Do you really need them? What happens without them?
  - curl --location: Follows HTTP redirects, necessary if the URL might redirect
  - curl --remote-name-all: Saves each file with its original name from the URL
  - Without --location: Fails to follow redirects, potentially missing the correct file
  - Without --remote-name-all: Files are not saved with their original names, requiring manual naming
  - --location: Necessary for handling URL redirects.
  - --remote-name-all: not necessary
- Install borg on both the machine were the files are used, and the machine were the backups will be stored. As borg is only available on linux machines [^1] and we don't want to introduce another VM to burden your laptop further, we will store the active versions of the files on web and the backup db VM. It is important that both machines have borg installed.
  - ram aanpassen naar 1,5gb van web en db
  - `sudo dnf install epel-release -y` om extra packages te enablen
  - `sudo dnf install borgbackup -y`
  - passphrase=`root`
- From the webserver, initialize a backup repository on db. This repository will contain all the created backups. Make sure you use the repokey` encryption mode!
  - `ssh-copy-id vagrant@172.30.0.15` zodat we geen ssh password moeten invoeren
- Export the borg keyfile in a readable format so you can store it on a safe location
  - export keyfile:`borg key export vagrant@172.30.0.15:/home/vagrant/backups /home/vagrant/borg-keyfile.txt`
  - borg keyfile:
```
[vagrant@web ~]$ cat borg-keyfile.txt
BORG_KEY bd1ec5a8d32269521b9aecea2ff8a372d00424d07848d9412becbe40badc2bb8
hqlhbGdvcml0aG2mc2hhMjU2pGRhdGHaAN6W7S2qmbiKdE0tAvzcflXaL1IqAolLSpT1UP
JMVnE1XM+Ow+9+N6mgV9E62BhDTFQ8sA2QdzKrB65cJspQfYOLJ1/fh8B89+bPfk55j5kN
G4TAxRHX5Bh2ykjBG1EKAefAWi4BaLd5hptdbLBy0Qy9wA+LqL+U8Z8O1nskMKIC8aumTL
d9foQEYAnk1m4X591fRrucWTO7grdAJiNqdBGQdr+bDJauKvupXFfm4O5D7uX12zgNtOuB
5xuK5w+OfqVBvEAcvcKONp9ls+4V2AFB3FDCHl72mctK4ULki4WkaGFzaNoAIE8reUntm4
nfeaf5FtWonPebhSJPcHA+SAKOVX3T5I+Bqml0ZXJhdGlvbnPOAAGGoKRzYWx02gAgOfBJ
2EHDc9XguMwn1XEqYiE1IzQyTEpvmJBSpgaDNc2ndmVyc2lvbgE=
```
- Create a backup, make sure it is called first
  -  `borg create vagrant@172.30.0.15:/home/vagrant/backups::first /home/vagrant/important-files`
- List all backups
  - `borg info vagrant@172.30.0.15:/home/vagrant/backups`    
```
[vagrant@web important-files]$ borg info vagrant@172.30.0.15:/home/vagrant/backups
Enter passphrase for key ssh://vagrant@172.30.0.15/home/vagrant/backups:
Repository ID: bd1ec5a8d32269521b9aecea2ff8a372d00424d07848d9412becbe40badc2bb8
Location: ssh://vagrant@172.30.0.15/home/vagrant/backups
Encrypted: Yes (repokey)
Cache: /home/vagrant/.cache/borg/bd1ec5a8d32269521b9aecea2ff8a372d00424d07848d9412becbe40badc2bb8
Security dir: /home/vagrant/.config/borg/security/bd1ec5a8d32269521b9aecea2ff8a372d00424d07848d9412becbe40badc2bb8
------------------------------------------------------------------------------
                       Original size      Compressed size    Deduplicated size
All archives:                3.41 MB              3.41 MB              1.71 MB

                       Unique chunks         Total chunks
Chunk index:                       9                   13
```
- Add a file test.txt with as content Hello world. Create another backup, make sure it is called second
  -  `sudo nano test.txt`
  -  `borg create vagrant@172.30.0.15:/home/vagrant/backups::second /home/vagrant/important-files`
```
[vagrant@web important-files]$ borg list vagrant@172.30.0.15:/home/vagrant/backups
Enter passphrase for key ssh://vagrant@172.30.0.15/home/vagrant/backups:
first                                Sun, 2024-08-11 02:34:21 [11abba33e95b25a36380140c604f580b9ab2b7ba335978ff69e982ad7a60a3b6]
second                               Sun, 2024-08-11 02:38:09 [69c5dac350fbd8ffdc7a20d19c1dec11c7b0f1fe3a913510ba06ecb508e5f4b8]
```
```
[vagrant@web important-files]$ borg list vagrant@172.30.0.15:/home/vagrant/backups::first
Enter passphrase for key ssh://vagrant@172.30.0.15/home/vagrant/backups:
drwxr-xr-x vagrant vagrant        0 Sun, 2024-08-11 02:33:17 home/vagrant/important-files
-rw-r--r-- vagrant vagrant      173 Sun, 2024-08-11 02:33:07 home/vagrant/important-files/bf1f3fb5-b119-4f9f-9930-8e20e892b898-720.mp4
-rw-r--r-- vagrant vagrant      300 Sun, 2024-08-11 02:33:08 home/vagrant/important-files/100.txt
-rw-r--r-- vagrant vagrant      300 Sun, 2024-08-11 02:33:08 home/vagrant/important-files/996.txt
-rw-r--r-- vagrant vagrant  1702187 Sun, 2024-08-11 02:33:08 home/vagrant/important-files/Toreador_song_cleaned.ogg
```
```
[vagrant@web important-files]$ borg info vagrant@172.30.0.15:/home/vagrant/backups
Enter passphrase for key ssh://vagrant@172.30.0.15/home/vagrant/backups:
Repository ID: bd1ec5a8d32269521b9aecea2ff8a372d00424d07848d9412becbe40badc2bb8
Location: ssh://vagrant@172.30.0.15/home/vagrant/backups
Encrypted: Yes (repokey)
Cache: /home/vagrant/.cache/borg/bd1ec5a8d32269521b9aecea2ff8a372d00424d07848d9412becbe40badc2bb8
Security dir: /home/vagrant/.config/borg/security/bd1ec5a8d32269521b9aecea2ff8a372d00424d07848d9412becbe40badc2bb8
------------------------------------------------------------------------------
                       Original size      Compressed size    Deduplicated size
All archives:                3.41 MB              3.41 MB              1.71 MB

                       Unique chunks         Total chunks
Chunk index:                       9                   13
```
- With which bash command can you see the size of the folder with the files on the webserver?
  -  `du -sh /home/vagrant/important-files` (1,7M)
  -  du -sh --si /home/vagrant/important-files (1,8M)
  -  `--si` uses powers of 1000 (e.g., 1 KB = 1000 bytes) instead of 1024 (e.g., 1 KiB = 1024 bytes)
- Now check the size of the backups folder on the database server.
  -   `du -sh /home/vagrant/backups` (1,8M)
- What is the difference between Original size, Compressed size and Deduplicated size?
  - Original Size: Total size of files before any processing
  - Compressed Size: Size after compression is applied, reducing storage needed
  - Deduplicated Size: Size after removing duplicate data chunks   
  - original size is gwn origineel, compressed is na compressie, borg compressed om storage te verminderen en dan deduplicated size is na duplicate chunks of data te verwijderen
- What are chunks?
  -  Chunks are pieces of data that Borg breaks files into for storage
-  It is necessary to periodically check the integrity of the borg repository. With which command can this be done? When should I use the --verify-data option? Tip: use --verbose to see more information
  - borg check vagrant@172.30.0.15:/home/vagrant/backups 
  - The --verify-data option is used to perform an additional check on the actual data within the repository, not only metadata, use it when you need to check repo structure and make sure data is not corrupt or missing
```
[vagrant@web important-files]$ borg check --verbose vagrant@172.30.0.15:/home/vagrant/backups
Remote: Starting repository check
Remote: finished segment check at segment 9
Remote: Starting repository index check
Remote: Index object count match.
Starting archive consistency check...
Remote: Finished full repository check, no problems found.
Enter passphrase for key ssh://vagrant@172.30.0.15/home/vagrant/backups:
Analyzing archive first (1/2)
Analyzing archive second (2/2)
Archive consistency check complete, no problems found.
```
- Restore the original files using the first backup on the database server (without the test.txt file) to the same place on web so it seems like nothing has happened. --strip-elements may come in handy here as borg uses absolute paths inside backups. You should see a similar output after restoring the backup:
  -  `borg extract --strip-components=2 vagrant@172.30.0.15:/home/vagrant/backups::first /home/vagrant/important-files`
```
[vagrant@web ~]$ borg extract --strip-components=2 vagrant@172.30.0.15:/home/vagrant/backups::first /home/vagrant/important-files^C
[vagrant@web ~]$ ls
borg-keyfile.txt  important-files
[vagrant@web ~]$ ll important-files/
total 1676
-rw-r--r--. 1 vagrant vagrant     300 Aug 11 02:33 100.txt
-rw-r--r--. 1 vagrant vagrant     300 Aug 11 02:33 996.txt
-rw-r--r--. 1 vagrant vagrant 1702187 Aug 11 02:33 Toreador_song_cleaned.ogg
-rw-r--r--. 1 vagrant vagrant     173 Aug 11 02:33 bf1f3fb5-b119-4f9f-9930-8e20e892b898-720.mp4
```
- Automate the backups and set an appropriate retention policy
  - `sudo nano backup.sh`
    ```
    #!/bin/bash

    # Define variables
    REPO="vagrant@172.30.20.15:~/backups"
    BACKUP_NAME="backup-$(date +%Y-%m-%d_%H-%M-%S)"
    SOURCE_DIR="/home/vagrant/important-files"

    # Create the backup
    borg create --stats --compression lz4 $REPO::$BACKUP_NAME $SOURCE_DIR

    # Retention policy (e.g., keep daily backups for 7 days, weekly for 4 weeks, and monthly for 6 months)
    borg prune -v --list $REPO --prefix 'backup-' --keep-daily=7 --keep-weekly=4 --keep-monthly=6
    ``` 
  - `chmod +x backup.sh`
  - `sudo nano /etc/systemd/system/backup.service`
    ```
    [Unit]
    Description=Run BorgBackup

    [Service]
    Type=simple
    ExecStart=/home/vagrant/backup.sh
    ```
  - `sudo nano /etc/systemd/system/backup.timer`
    ```
    [Unit]
    Description=Run BorgBackup every 5 minutes

    [Timer]
    OnBootSec=5min
    OnUnitActiveSec=5min
    Unit=backup.service

    [Install]
    WantedBy=timers.target

    ```
  - `sudo systemctl daemon-reload`
  - `sudo systemctl enable backup.timer`
  - `sudo systemctl start backup.timer`


- What is the retention policy here?
  -  retention policy in the script is defined by the `borg prune` command
  -  `--keep-daily=7`: Keep the last 7 daily backups.
  -  `--keep-weekly=4`: Keep the last 4 weekly backups
  -  `--keep-monthly=6`: Keep the last 6 monthly backups 
- Have you ever heard of the Grandfather-Father-Son policy?
  - GFS policy allows for a structured retention strategy, where more frequent backups (sons) are kept for a short period, and less frequent backups (fathers and grandfathers) are kept for longer periods
  - Son: Daily backups
  - Father: Weekly backups
  - Grandfather: Monthly backups
- What does the `borg compact` command do?
  -  used to optimize and clean up the Borg repository
  -  Removing Unused Chunks
  -  Reclaiming Space
  -  `borg compact /path/to/repository`
- can I use tools like borg to backup an active database? Why (not)?
  -  Yes, but it’s not ideal
  -  Consistency Issues: Backup might be inconsistent if the database is active during backup.
  -  Locking Problems: Changes during backup can lead to corrupt data
- Should I take any extra measures to do this safely?
  -  Use database-specific backup tools
  -  temporarily deactivate the database or use database snapshot features
- There is a tool that has been built on top of borg called borgmatic. What does it do? Could it be useful to you? Why (not)?
  - Borgmatic automates and simplifies BorgBackup with easier configuration, scheduling, and notifications
  - It simplifies backup management and automation
- Try lab with Restic
  - Restic is an alternative backup tool offering efficient deduplication, encryption, and support for various storage backends 