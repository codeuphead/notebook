# HTTPD

centos 7

1.安装 Apache
yum install httpd.x86_64

2.设置服务器开机自动启动 Apache
systemctl enable httpd

若要验证是否自动启动可在重启服务器后在终端键入以下命令来检测 Apache 是否已经启动
systemctl is-enabled httpd

如果看到了 enable 这样的响应，则表示 Apache 已经启动成功 3.手动启动 Apache
systemctl start httpd 在浏览器中输入 IP 地址即可验证是否启动成功

4.手动重启 Apache
systemctl restart httpd

5.手动停止 Apache
systemctl stop httpd

## 端口转发

```bash
<VirtualHost *:80>
    ServerName www.lovehls.cn
    ProxyRequests Off
    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>

<VirtualHost *:80>
    ServerName cost.lovehls.cn
    ProxyRequests Off
    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:8080/hello
    ProxyPassReverse / http://127.0.0.1:8080/hello
</VirtualHost>
```

## SSL

```bash
SSLEngine on
SSLCertificateFile /root/cert/apache/2751507__readoor.cn_public.crt
SSLCertificateKeyFile /root/cert/apache/2751507__readoor.cn.key
SSLCertificateChainFile /root/cert/apache/2751507__readoor.cn_chain.crt
```
