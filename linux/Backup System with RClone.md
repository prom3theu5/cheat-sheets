## create backup

```bash
cd /system-backups && tar -cvzf backup-$(date +%d.%m.%Y).tar.gz --directory=/ --exclude=systembackups/* --exclude=lost+found --exclude=dev/* --exclude=proc/* --exclude=run/* --exclude=sys/* --exclude=tmp/* --exclude=mnt/* --exclude=media/* .
```

Explanation of the command:  

- tar – tar creates us an archive of our data. In this example with the following parameters: 
- c – (create) creates a new archive 
- v – (verbose) detailed output 
-  z – (gzip) the archive will be compressed with gzip 
-  f – (file) name and location of the archive. Must be the last option passed, since everything after f is treated as a file path.
- /path/filename.tar.gz – path to which the archive will be written.  
- As you can see, i have excluded some of directory’s we don’t want to backup, because of storage space reasons or unwanted big stored files in tmp.

---
## setup [Rclone](tools/rclone/install)

Follow the rclone config flow to setup a remote share

## create backup directory in rclone

```bash
rclone mkdir <configured endpoint>:backups
```

---
## push backup to rclone source

```bash
rclone copy -P /system-backups <configuered endpoint>:backups/<server_name>
```

All data from /system-backups will transferred to your destination bucket.

---
## pull backup from rclone to local machine

```bash
rclone copy -P <configured endpoint>:backups/<server_name> /system-backups
```

To download your backup, you need to switch source and destination and specify which backup you actually want to download.

