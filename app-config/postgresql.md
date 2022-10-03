# 安装配置

```bash
yum install https://download.postgresql.org/pub/repos/yum/10/redhat/rhel-7-x86_64/pgdg-centos10-10-2.noarch.rpm

//删除之前的 postgresql
yum remove postgresql*
rm -rf /var/lib/pgsql
rm -rf /usr/pgsql-10
userdel postgres

//安装 postgresql
yum install postgresql10
yum install postgresql10-server
/usr/pgsql-10/bin/postgresql-10-setup initdb
systemctl enable postgresql-10
systemctl start postgresql-10
```

## 修改用户密码

```bash
su - postgres
psql -U postgres
ALTER USER postgres WITH PASSWORD 'xxx'
```

## 创建用户

```psql
CREATE USER dbuser WITH PASSWORD 'password';
CREATE DATABASE exampledb OWNER dbuser;
GRANT ALL PRIVILEGES ON DATABASE exampledb to dbuser;
```

## 修改认证方式、端口

```bash
/var/lib/pgsql/10/data/pg_hba.conf
/var/lib/pgsql/10/data/postgresql.conf
```

## client 命令

```psql
createdb mydb
createuser
dropdb mydb
```

## psql 命令

```psql
\du 所有用户
\l 所有db
\q
```
