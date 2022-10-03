# git

- [git](#git)
  - [安装最新版 git](#安装最新版-git)
    - [ubuntu](#ubuntu)
    - [centOS](#centos)
  - [配置](#配置)
  - [使用](#使用)
  - [忽略文件](#忽略文件)
  - [git 服务器](#git-服务器)
  - [pageant](#pageant)
  - [gitolite](#gitolite)

## 安装最新版 git

### ubuntu

```bash
add-apt-repository ppa:git-core/ppa
apt update
apt install git
```

### centOS

```bash
# 安装依赖
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel
yum install  gcc perl-ExtUtils-MakeMaker

# 删除旧版本
yum remove git

# 下载最新源码包
wget https://github.com/git/git/archive/v2.23.0.tar.gz

# 解压
tar -xzvf v2.9.2.tar.gz -C /opt/

# 进入文件夹
make prefix=/usr/local/git all
make prefix=/usr/local/git install

# 添加环境变量
echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/bashrc
source /etc/bashrc # 实时生效

git --version
```

## 配置

用户配置文件在用户目录下的 .gitconfig 文件

必填用户名和邮箱

```bash
git config --global user.name "xxx"
git config --global user.email "xxx"
```

```bash
gpg --full-generate-key
gpg --armor --export {key_id}
gpg --list-keys
git config --global user.signingkey {key_id}
git config --global gpg.program "C:/Program Files (x86)/GnuPG/bin/gpg.exe"
git config --global commit.gpgsign true
git commit -S -m "..."
```

## 使用

```bash
git init //初始化仓库
git status //查看仓库状态

git add <filename> //添加单个文件、目录至暂存区
git add . //添加目录下所有文件至暂存区

git diff //查看所有文件更改内容
git diff <filename> //查看单个文件更改内容

git commit -m "xxx" //提交
git checkout <filename> //撤销未提交到暂存区的修改
git reset <filename> //撤销提交暂存区的操作

git log //查看log
git log <commit id> -1 //查看指定 id 的 log，并且只输出一行
git log -p //参数，显示修改内容

git clone <远程地址> <本地目录名>
git clone -o //指定远程主机名

git remote //列出远程主机，origin 是默认的远程主机名
git remote -v //查看远程主机地址
git remote show <主机名> //查看详细信息
git remote add <主机名> <网址> //添加远程主机
git remote rm <主机名> //删除远程主机
git remote rename <原主机名> <新主机名>

git fetch <远程主机名> //将更新取回本地
git fetch <远程主机名> <分支名> //取指定分支

git branch
git branch -r //查看远程分支
git branch -a //查看所有分支


git checkout -b newBranch origin/master

git merge origin/master

git rebase origin/master

git branch --set-upstream master origin/next //指定 master 分支追踪 origin/master 分支

git pull <远程主机名> <远程分支名>:<本地分支名> //相当于先 fetch，再 merge。本地分支为当前分支，可以省略。如果当前分支与远程分支存在追踪关系，可以省略远程分支名。如果当前分支只有一个追踪分支，那么主机名也可以省略。
git pull --rebase <远程主机名> <远程分支名>:<本地分支名> //使用 rebase 模式

git pull //当前分支自动与唯一一个追踪的分支进行合并

git push --set-upstream master origin

```

## 忽略文件

代码仓库目录下的`.gitignore`文件，在提交的时候，会进行读取进行匹配。

## git 服务器

yum install git //安装 git
adduser git //创建 git 用户
公钥导入 /home/git/.ssh/authorized_keys
gisut init --bare /srv/sample.git //初始化仓库
chown -R git:git /srv/sample.git //修改 owner

禁用 shell git 用户登录：编辑 /etc/passwd
git:x:1001:1001:,,,:/home/git:/bin/bash -> git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell

/usr/local/git/bin/git-shell

这样用户可以通过 ssh 使用 git 但是无法登录 shell

git clone git@server:/srv/sample.git

## pageant

环境变量添加 GIT_SSH 指定到 plink.exe
可以令 VSCODE 等客户端通过 plink 使用 ssh key（ppk 格式的）

## gitolite

`useradd --system --shell /bin/bash --create-home git`
