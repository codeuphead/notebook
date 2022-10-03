# jenkins 介绍

## 持续集成（Continuous Intergration）、持续交付（Continuous Delivery）

- 开发：代码提交-自动化打包
- 测试：启动自动化测试脚本
- 运维：启动自动更新文件到线上环境

jenkins 工具将这些结合起来，达到 CI、CD 的目标：

开发人员提交代码至 git/svn 后，jenkins 抓取最新代码，执行脚本进行构建，执行测试，发布到部署环境，输出报告。

## 安装

### 包安装

```bash
https://pkg.jenkins.io/redhat-stable/
https://pkg.jenkins.io/debian-stable/

rpm -ivh [package] //安装
rpm -Uvh [package] //升级

```

### CentOS

```bash
需要 jre
导入 key
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
yum install jenkins
```

### Ubuntu

```bash
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add - deb https://pkg.jenkins.io/debian-stable binary/
sudo apt-get update
sudo apt-get install jenkins

```

### 安装好后

访问 `http://localhost:8080`

## 配置相关

### 插件源

```bash
管理 > 插件管理 > 高级 > 站点 url > https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json

/var/lib/jenkins/updates/default.json 修改配置

:%s/http:\/\/www.google.com/http:\/\/www.baidu.com/g
:%s/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g

```

### 触发构建

如果不使用固定频率的构建方式，则默认使用 webhook 的方式进行构建的触发

添加 gitea 的 webhook url: `http://[ your jenkins server ]/gitea-webhook/post` contentType: `application/json`

### 启动停止

```bash
service jenkins start
service jenkins stop
```

安装目录 `/var/lib/jenkins`

配置文件 `/etc/sysconfig/jenkins`

```bash
JENKINS_HOME="/var/lib/jenkins" #主目录
JENKINS_USER="jenkins" #用户，拥有 JENKINS_HOME /var/log/jenkins 目录的权限
JENKINS_PORT="8080" #端口
# 配置启动参数，可配置证书等
JENKINS_ARGS="--httpPort=-1 --httpsPort=9082 --httpsKeyStore=/var/lib/jenkins/cert/mycert.jks --httpsKeyStorePassword=qAv8z6Uc"
```
