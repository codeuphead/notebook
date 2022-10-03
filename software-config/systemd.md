# SYSTEMD

- systemctl enable/disable xxx
- systemctl start/stop xxx
- systemctl restart xxx
- systemctl status xxx
- systemctl daemon-reload

`/etc/systemd/system`

xxx.service

```bash
[Unit]
Description=cost
After=network.target

[Service]
User=root
ExecStart=/usr/jdk1.8.0_162/bin/java -jar /usr/local/app/cost.jar
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```
