# linux-package

- [linux-package](#linux-package)
  - [CentOS](#centos)
    - [yum 切换源](#yum-切换源)
    - [yum 常用命令](#yum-常用命令)
    - [rpm 包直接安装](#rpm-包直接安装)
  - [Ubuntu](#ubuntu)
    - [apt 切换源](#apt-切换源)
    - [常用命令](#常用命令)

## CentOS

### yum 切换源

```bash
# 备份原来的源
cd /etc/yum.repos.d/
mv CentOS-Base.repo CentOS-Base.repo_bak

# 阿里源
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum clean all
yum makecache

# 清华源
https://mirror.tuna.tsinghua.edu.cn/help/centos/

# epel 源
yum -y install epel-release
yum clean all
yum makecache
```

### yum 常用命令

```bash
yum list installed | grep xxx
yum update
yum remove xxx

```

### rpm 包直接安装

```bash
rpm -qa | grep [package] 相关包名列出
rpm -e [文件名]

--force 覆盖
--nodeps

rpm -ivh [package] 安装
rpm -Uvh [package] 更新
rpm -e [package] 卸载
rpm -q [package] 查看是否安装
rpm -qi [package] 查看安装包信息
rpm -ql [package] 列出包文件
rpm -qf 列出文件属于哪个 rpm 包
rpm -qilp 列出未安装的 rpm 包文件中包含有那些文件
```

## Ubuntu

### apt 切换源

```bash
sudo cp /etc/apt/sources.list /etc/apt/sources.list.20190516
sudo nano /etc/apt/sources.list

# 替换字符
security.ubuntu -> mirrors.aliyun
archive.ubuntu -> mirrors.aliyun

# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
```

### 常用命令

```bash
install #安装软件包
remove #移除软件包
purge #移除软件包及配置文件
update #刷新存储库索引
upgrade #升级所有可升级的软件包
autoremove #自动删除不需要的包
full-upgrade #在升级软件包时自动处理依赖关系
search #搜索应用程序
show #显示装细节
```
