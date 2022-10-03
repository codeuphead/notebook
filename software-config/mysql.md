# MySQL

## 安装

```bash
wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
rpm -Uvh mysql57-community-release-el7-11.noarch.rpm
yum repolist all | grep mysql

sudo yum install mysql-community-server
```

## 修改配置文件

```bash
vi /etc/my.cnf

# mysqld config
[mysqld]
port=3306
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
symbolic-links=0
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
character-set-server=utf8

# mysql config
[mysql]
socket=/var/lib/mysql/mysql.sock
default-character-set=utf8

# client config
[client]
socket=/var/lib/mysql/mysql.sock
default-character-set=utf8
```

```bash
[client] 
default-character-set = utf8mb4 
[mysql] 
default-character-set = utf8mb4 
[mysqld] 
character-set-client-handshake = FALSE 
character-set-server = utf8mb4 
collation-server = utf8mb4_unicode_ci 
init_connect='SET NAMES utf8mb4'
```

## 启动MySQL

systemctl start mysqld

## 重置密码

打印默认密码 `cat /var/log/mysqld.log | grep password`

登录 `mysql -u root -p`

创建用户 `CREATE USER '<user>'@'<host>' IDENTIFIED BY '<password>';`

修改当前用户密码 `SET PASSWORD = PASSWORD('<password>');`

修改指定用户密码 `SET PASSWORD FOR '<user>'@'<host>' = PASSWORD('<password>');`

删除用户 `DROP USER '<user>'@'<host>';`

例：`CREATE USER test@'%' IDENTIFIED BY '1234';`

- `%` 不限定登录主机

## 权限管理

`GRANT <privilege> ON <database>.<table> TO '<user>'@'<host>';`

例：`GRANT ALL ON cost.* TO 'orange'@'%';`

- 第一个 `*` 标识数据库，`*` 标识数据库的表，`%` 标识任意 IP 登录

`<privilege>` 可以设置为：

- ALL 所有权限
- SELECT/INSERT/UPDATE 等权限，多权限用`,`隔开

撤销权限 `REVOKE <privilege> ON <database>.<table> FROM '<user>'@'<host>';`

配置立刻生效 `FLUSH PRIVILEGES;`

查看权限 `SHOW GRANTS;`

查看指定用户权限 `SHOW GRANTS FOR '<user>'@'<host>';`

重启命令：systemctl restart mysqld
