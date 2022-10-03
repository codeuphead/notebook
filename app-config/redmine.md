# Redmine 相关

## 安装

### ruby

```bash
先安装 RVM-Ruby Version Manager
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB

安装rvm
curl -sSL https://get.rvm.io | bash -s stable

添加用户到 rvm 组

rvm reload

rvm install 2.6.0

rvm list
rvm remove [version]

ruby -v
gem -v
```

ruby 完成后，使用 gem 安装 rails 和 rake

```bash
更换 gem 源
gem source -r https://rubygems.org/
gem source -a https://gems.ruby-china.com/
gem source -l

gem install rake
gem install rails
```

### MySQL 配置

```bash
CREATE DATABASE redmine CHARACTER SET utf8mb4;
CREATE USER 'redmine'@'localhost' IDENTIFIED BY 'my_password';
GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost';
```

`cd redmine` 目录

`cp config/database.yml.example config/database.yml`

`yum install mysql-devel`

设置数据库配置

```bash
production:
  adapter: mysql2
  database: redmine
  host: localhost
  username: redmine
  password: "my_password"
  encoding: utf8mb4
```

### 安装 bundler

```bash
gem install bundler
```

### 安装 依赖

配置数据库 `config/database.yml` 后 重新安装依赖

```bash
cd redmine
bundle install --without development test
```

### 生成会话密钥

`bundle exec rake generate_secret_token`

### 初始化数据库

```bash
bundle exec rake db:migrate RAILS_ENV=production
bundle exec rake redmine:load_default_data RAILS_ENV=production
```

### 文件系统权限

运行程序的用户（如果是 apache 运行的话，需要设置 apache），对于以下目录需要写权限

files log tmp tmp/pdf public/plugin_assets

```bash
mkdir -p tmp tmp/pdf public/plugin_assets
sudo chown -R redmine:redmine files log tmp public/plugin_assets
sudo chmod -R 755 files log tmp public/plugin_assets
```

### 测试

使用 webrick 测试，正式使用需要设置 Passenger、FCGI、Rack server 等。

```bash
bundle exec rails server webrick -e production
```

### 配置

配置在 `config/configuration.yml` 中，这些这是可能每个 Rails 环境中都有配置 production development test

### Apache

gem install passenger

yum install -y httpd-devel libcurl-devel apr-devel apr-util-devel mod_ssl
cd \$REDMINE
gem install passenger

passenger-install-apache2-module

安装完成 passenger 会提示把这段话放到 vim /etc/httpd/conf.d/passenger.conf 中

redmine.conf

```bash
<VirtualHost *:3000>
  ServerName 47.101.169.247
  DocumentRoot "/usr/local/bin/redmine/public"

  RailsEnv production
  ErrorLog logs/error_redmine
  LogLevel warn

  <Directory "/usr/local/bin/redmine/public">
    Options Indexes ExecCGI FollowSymLinks
  Require all granted
    AllowOverride all
  </Directory>

</VirtualHost>
Listen 3000
```

## 插件

```bash
bundle exec rake redmine:plugins:migrate NAME=redmine_knowledgebase RAILS_ENV=production
bundle exec rake redmine:plugins:migrate RAILS_ENV=production
```

删除插件

```bash
bundle exec rake redmine:plugins:migrate NAME= VERSION=0 RAILS_ENV=production
plugins 删除文件
重启 redmine
```

## 备份

```bash
#!/bin/bash
# Database
/usr/bin/mysqldump -h rm-uf6y6vd4539l0q0b0.mysql.rds.aliyuncs.com redmine | gzip > /var/redmine_backup/db/redmine_`date +%Y-%m-%d-%H-%M`.gz

# Attachments
rsync -a /usr/local/bin/redmine/files /var/redmine_backup

find /var/redmine_backup/db/ -mtime +7 -type f -name "*" -exec rm -rf {} \;
```

目录添加配置文件用来填写 mysql 账号密码

```bash
# mysqldump 命令添加参数 --defaults-extra-file=/path/to/config.cnf

# 文件内容
[mysqldump]
user = whatever
password = whatever
host = whatever
```
