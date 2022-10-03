# rclone 配置

## 安装

linux `sudo -v ; curl https://rclone.org/install.sh | sudo bash`

windows `https://downloads.rclone.org/v1.59.2/rclone-v1.59.2-windows-amd64.zip`

## systemd 配置

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

## windows service 配置

使用 nssm 进行配置，注意需要指定配置文件位置。

path 项：`C:\xxx\rclone.exe`

startup directory 项：`C:\xxx\`

arguments 项：`mount jianguoyun:/ X: --allow-non-empty --cache-dir x:/x/jianguoyun --vfs-cache-mode writes --vfs-cache-max-age 24h --vfs-cache-max-size 1G --buffer-size 128M --config C:\Users\xxx\AppData\Roaming\rclone\rclone.conf --log-file x:/temp/tmp.txt`
