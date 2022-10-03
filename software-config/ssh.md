# SSH

## 生成密钥

```bash
ssh-keygen -t rsa -b 4096 -C "xxx"
```

参数

```bash
-b：指定密钥长度
-e：读取openssh的私钥或者公钥文件
-C：添加注释
-f：指定用来保存密钥的文件名
-i：读取未加密的ssh-v2兼容的私钥/公钥文件，然后在标准输出设备上显示openssh兼容的私钥/公钥
-l：显示公钥文件的指纹数据
-N：提供一个新密语
-P：提供（旧）密语
-q：静默模式
-t：指定要创建的密钥类型
-p：改密码
```

查看密钥指纹

```bash
ssh-keygen -lf <key-path>
ssh-keygen -E md5 lf <key-path>
```

## 服务器 sshd 配置

```bash
cd ~/.ssh
cat id_rsa.pub >> authorized_keys
chmod 600 authorized_keys
chmod 700 ~/.ssh

/etc/ssh/sshd_config

RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile %h/.ssh/authorized_keys

PermitRootLogin yes
PasswordAuthentication yes

systemctl restart sshd
```

## ssh agent 密钥代理

git bash 用，用来使用带密码的 key

`eval "$(ssh-agent -s)"` 确认是否运行

```bash
ssh-agent [-c | -s] [-d] [-a bind_address] [-t life] [command [arg ...]]
ssh-agent [-c | -s] -k

-a bind_address：bind the agent to the UNIX-domain socket bind_address
-c 生成C-shell风格的命令输出
-d 调试模式
-k 把ssh-agent进程杀掉
-s 生成Bourne shell 风格的命令输出
-t life：设置默认值添加到代理人的身份最大寿命
```

运行 ssh-agent `ssh-agent`

### ssh-add 添加密钥

将密钥添加至 ssh agent

```bash
ssh-add ~/.ssh/id_rsa //添加 key
ssh-add -d ~/.ssh/id_rsa.pub //删除密钥
ssh-add -l //显示

-D：删除ssh-agent中的所有密钥
-d：从ssh-agent中的删除密钥
-e pkcs11：删除PKCS#11共享库pkcs1提供的钥匙
-s pkcs11：添加PKCS#11共享库pkcs1提供的钥匙
-L：显示ssh-agent中的公钥
-l：显示ssh-agent中的密钥
-t life：对加载的密钥设置超时时间，超时ssh-agent将自动卸载密钥
-X：对ssh-agent进行解锁
-x：对ssh-agent进行加锁
```

注：

`ssh-add -K /path/to/key` `-K` 是苹果系统版本的特殊参数，其会在添加 ssh key 至 ssh agent 时，将密码存入 keychain，以后再次添加的时候，就不用输入密码了

如果使用 macOS Sierra 10.12.2 或更高的版本，需要设置 ssh 配置文件来自动加载 key 到 ssh agent，并且通过 keychain 保存密码

设置配置文件：

```bash
Host *
   UseKeychain yes
   AddKeysToAgent yes
   IdentityFile ~/.ssh/id_rsa
   IdentityFile ~/.ssh/xxx
```

## 拷贝密钥到主机

```bash
ssh-copy-id [-i [identity_file]] [user@]machine

ssh-copy-id user@server
ssh-copy-id -i ~/.ssh/id_rsa.pub user@server
```

## Windows 使用 ppk 类型的 key

通常 PuTTY 或者其他 SSH 程序都带有额外的插件程序，包括 PuTTY Key Generator、Pageant 等

通过 PuTTYGen 可以将生成的 key 的私钥转换成 ppk 类型，以供 Pageant 等程序使用（windows 平台常用的一种格式）

运行-`shell:startup`打开开机启动文件夹-新建 bat 批处理程序

输入以下内容

```bash
cd "C:\Program Files\TortoiseGit\bin"
start .\pageant.exe C:\Users\startiasoft\.ssh\aliyun-lisa-work.ppk C:\Users\startiasoft\.ssh\gitea-work.ppk  C:\Users\startiasoft\.ssh\ecd.ppk C:\Users\startiasoft\.ssh\liuyifei.ppk
exit
```

## windows 使用 ssh config 命令行登录

```bash
Host 247
   HostName 47.101.169.247
   User root
   Port 9292
   IdentityFile C:\Users\startiasoft\.ssh\lyf-247
```

通配符 “\*”表示任意字符串，“?”表示任意单个字符

Get-Service -Name ssh-agent | Set-Service -StartupType Automatic

## vscode 使用 ssh agent

如果 vscode 不能使用 ssh agent 中的 key，设置 GIT_SSH 环境变量为 plink.exe(putty 提供的 app，tortoise git 也有)
