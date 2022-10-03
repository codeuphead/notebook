# Nginx

```bash
/etc/yum.repos.d/nginx.repo
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=1
enabled=1

wget http://nginx.org/keys/nginx_signing.key
rpm --import nginx_signing.key
yum clean all
yum makecache
yum install nginx
```

```bash
/etc/nginx 配置文件
/usr/sbin/nginx 程序文件
/var/log/nginx 日志
/etc/init.d/nginx 启动脚本

参考 /etc/nginx/sites-available 里的配置
/var/www/nginx-default/
/var/www

/etc/initd/ningx start 启动
```
