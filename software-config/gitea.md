# gitea

## 安装

```bash
#下载gitea
chmod +x gitea

#安装 git

#添加用户
useradd --system --shell /bin/bash --comment 'Git Version Control' --user-group --home-dir /home/git git

#设置目录
mkdir -p /var/lib/gitea/{custom,data,log}
chown -R git:git /var/lib/gitea/
chmod -R 750 /var/lib/gitea/
mkdir /etc/gitea
chown root:git /etc/gitea
chmod 770 /etc/gitea

#安装完，进行配置后，将 /etc/gitea 权限设置回来
chmod 750 /etc/gitea
chmod 640 /etc/gitea/app.ini
```

## systemctl 启动

```bash
[Unit]
Description=Gitea (Git with a cup of tea)
After=syslog.target
After=network.target
Requires=mysql.service

[Service]
# Modify these two values and uncomment them if you have
# repos with lots of files and get an HTTP error 500 because
# of that
###
#LimitMEMLOCK=infinity
#LimitNOFILE=65535
RestartSec=2s
Type=simple
User=git
Group=git
WorkingDirectory=/var/lib/gitea/
ExecStart=/opt/gitea web --config /etc/gitea/app.ini -p 9080
Restart=always
Environment=USER=git HOME=/home/git GITEA_WORK_DIR=/var/lib/gitea
# If you want to bind Gitea to a port below 1024 uncomment
# the two values below
###
#CapabilityBoundingSet=CAP_NET_BIND_SERVICE
#AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```

## SSL

配置文件

```bash
[server]
PROTOCOL         = https
DOMAIN           = pineapple.readoor.cn
HTTP_PORT        = 9080
ROOT_URL         = https://pineapple.readoor.cn:9080/
CERT_FILE        = custom/public.pem
KEY_FILE         = custom/key.pem
```

## 坑

systemd 启动的 gitea，必须把 git 放在 在 /bin/git 否则会报错

```bash
ln -s /opt/git/bin/git-upload-pack /bin/git-upload-pack
ln -s /opt/git/bin/git-upload-archive /bin/git-upload-archive
ln -s /opt/git/bin/git-shell /bin/git-shell
ln -s /opt/git/bin/git-receive-pack /bin/git-receive-pack
ln -s /opt/git/bin/gitk /bin/gitk
ln -s /opt/git/bin/git-cvsserver /bin/git-cvsserver
ln -s /opt/git/bin/git /bin/git
```
