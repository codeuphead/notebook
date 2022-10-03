# tomcat

## 安装

### 下载 tomcat 压缩包，解压至指定文件夹

### 设置环境变量

```bash
/etc/profile.d/path.sh 如果不存在则创建该文件，加入以下内容

export JAVA_HOME=/usr/lib/jvm/jre-1.8.0-openjdk
export CLASS_PATH=.:$JAVA_HOME/lib
export CATALINA_HOME=/usr/local/tomcat
export CATALINA_BASE=/usr/local/tomcat
export PATH=$PATH:$JAVA_HOME/bin:$CATLINA_HOME:/bin

然后执行

source /etc/profile
```

### 设置 tomcat 启动参数

需要设置一个 tomcat 的 pid 文件

```bash
在 tomcat/bin 目录下，新建 setenv.sh 文件，catelina.sh 文件在启动时会调用，配置参数如下：
#add tomcat pid
CATALINA_PID="$CATALINA_BASE/tomcat.pid"
#add java opts
JAVA_OPTS="-server -XX:MetaspaceSize=256M -XX:MaxMetaspaceSize=1024m -Xms512M -Xmx1024M -XX:MaxNewSize=256m"
```

### 创建 tomcat 用户（有需要的话）

```bash
getent group tomcat || groupadd -r tomcat
getent passwd tomcat || useradd -r -d /opt -s /bin/nologin -g tomcat tomcat

# 将 tomcat 目录权限给用户
chown -R tomcat:tomcat /usr/local/apache-tomcat-8.0.36
```

### 设置 systemd 自动启动

```bash
cd /etc/systemd/system

#创建文件 tomcat.service 内容如下

[Unit]
Description=Apache Tomcat 8
After=syslog.target network.target

[Service]
Type=forking
PIDFile=/usr/local/tomcat/tomcat.pid
ExecStart=/usr/local/tomcat/bin/startup.sh
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true
User=【填写用户】
Group=【填写组】

[Install]
WantedBy=multi-user.target

# 使用 systemd

systemctl enable tomcat
systemctl daemon-reload
systemctl start tomcat
systemctl status tomcat
```
