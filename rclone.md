```bash
[Unit]
Description=jianguoyun
AssertPathIsDirectory=/home/xggorange/jianguoyun
After=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/rclone mount jianguoyun:/ /home/xggorange/jianguoyun \
--allow-non-empty \
--cache-dir /home/xggorange/.jianguoyun \
--vfs-cache-mode writes \
--vfs-cache-max-age 24h \
--vfs-cache-max-size 1G \
--vfs-read-chunk-size 128M \
--vfs-read-chunk-size-limit 1G \
--buffer-size 128M
ExecStop=fusermount -u /home/xggorange/jianguoyun
Restart=on-abort
User=xggorange

[Install]
WantedBy=default.target
```
