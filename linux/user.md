# 用户

```
adduser xxx
usermod -aG sudo xxx
```

## 文件

- 用户名 UID `/etc/passwd -rw-r--r--`
- 组 GID `/etc/group -r--------`
- 口令 `/etc/shadow -rw-r--r--`
- 组口令 `/etc/gshadow -r--------`

```bash
/etc/passwd 每一行一条记录，每条记录7个字段
username:password:UID:GID:username?:homeDir:shell
```

```bash
/etc/shadow 每一行一条记录，每条记录9个字段
用户名:口令:最后一次修改时间:最小时间间隔（更改密码天数）:最大间隔:警告时间:不活动时间:失效时间:标志
```

```bash
/etc/group
组名:口令:组ID:组列表
```

```bash
/etc/shadow

组名:组口令:管理账号:组列表
```

## 命令

```bash
用户管理
useradd
usermod
userdel

组管理
groupadd
groupdel
groupmod

批量管理
newusers
chpasswd

组成员管理
gpasswd -a [用户账号名] [组账号名] //添加用户到组
gpasswd -d [用户账号名] [组账号名] //从组删除用户
usermod -G [组账号名] [用户账号名] //添加用户到组，会离开其他组
usermod -a -G gName uName //添加用户到组，不离开组

口令维护
passwd [用户账号名] //设置口令
passwd -l [用户账号名] //禁用账户口令
passwd -S [用户账号名] //查看账户口令
passwd -u [用户账号名] //恢复账户口令
passwd -d [用户账号名] //清除账户口令

id //显示当前用户 id gid
groups //显示指定用户所属组列表
whoami //显示当前用户名
w/who //显示用户相关信息
newgrp //转换用户的组到指定组
```

## `useradd [options] username`

```bash
-e 账户过期日期
-f 账户密码不活动期
-g 指定用户组 id 或名称
-m 创建用户目录
-M 不创建用户目录
-N 不创建与用户名同名的用户组
-o 允许创建重复的 UID
-p 指定密码
-r 创建系统账户
-s 指定用户的登录shell
-u 用户号
-U 创建和用户名相同的组
```

## `userdel [options] username`

```bash
-f 强制
-r 移除 home 目录和 mail spool
```

## `usermod [options] username`

## 口令设置

`passwd [options] username`

```bash
-l 锁定口令，禁用账号
-u 解锁口令
-d 使账号无口令
-f 强迫下次登录修改口令
```
