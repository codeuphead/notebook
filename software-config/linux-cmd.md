# Linux

## 常用

```bash
where is xxx //查找
which xxx
pkill -9 xxx //杀进程
killall -9 xxx
kill -9 xxx

ps -ef | grep firefox //进程
ps -ef | grep --color firefox //进程 高亮
pgrep firefox
pidof firefox
ps aux | grep java //查看 java 进程
ps aux //查看所有进程

netstat -apn | grep java/protno //端口占用
netstat -tln | grep 8080 //端口占用
lsof -i :8080 //端口属于哪个程序

df -h 磁盘信息
```

## 环境变量

在 `~/.bash_profile` 定义每个用户的环境变量

在 `/etc/profile.d/custom.sh` 定义所有用户的环境变量

### PS1

PS1='\[\e[33;1m\][\[\e[32;1m\]\u\[\e[31;1m\]@\[\e[32;1m\]\h\[\e[34;1m\]:\[\e[36;1m\]\w\[\e[33;1m\]]\[\e[34;1m\]\$ \[\e[m\]'

## 文件含义

- `/bin/bash` bash 可执行目录
- `/etc/profile` 系统范围，用户第一次登录执行，从 `/etc/profile.d` 中搜集 shell 设置
- `/etc/bashrc` 系统范围，通常定义函数和别名，为每一个 bash shell 执行
- `~/.bash_profile` 用户范围，用户登陆时执行，会执行 `~/.bashrc`，login shell
- `~/.bashrc` 个人交互式 shell，并且每次打开 shell 执行。会执行 `/etc/bashrc`
- `~/.bash-logout` 个人 login shell 清理文件，当 login shell 退出的时候
- `~/.profile` Debian 中替代 `.bash_profile`

## 目录含义

- `/bin` 通常所有用户都会用到的必要命令，例如 ls pwd 等
- `/sbin` 通常是管理系统的命令，例如 shutdown ifconfig 等
- `/usr/bin` 通常是一些非必要的，但是普通用户和超级用户都可能使用到的命令，例如 gcc，ldd 等。包括自己安装的软件，可能再次建立一个链接，指向执行文件
- `/usr/sbin` 通常是一些非必要的，由系统管理员来使用的管理系统的命令，例如 crond，httpd 等等
- `/usr/local/bin` 通常是用户后来安装的软件，可能被普通用户或超级用户使用
- `/usr/local/sbin` 通常是用户后来安装的软件，一般是用来管理系统的，被系统管理员使用

## 测速

iperf 测试带宽

speedtest 测试速度

```bash
curl -Lo speedtest-cli https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py
chmod +x speedtest-cli
```

## 测试设备性能

```bash
wget -qO- https://raw.githubusercontent.com/oooldking/script/master/superbench.sh | bash
```
