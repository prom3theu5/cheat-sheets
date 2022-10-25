Syncing a folder is done as below

```bash
rclone sync "source" "target" --checkers 50 --transfers 500 --use-server-modtime --progress
```

Its one  way meaning if `source` is a bucket from config and `target` is a local folder then its a downoad, but if `source` is a local folder and `target` is a remote bucket, then files will be pushed.


> :warning: Be aware, using SYNC is a destructive operation, if you write files to a folder, then use sync to pull the folder - if the files dont exist in the source, they will be removed from the target.
> Use COPY instead if you want a safe copy.

