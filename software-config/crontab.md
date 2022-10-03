# Crontab

cat /etc/crontab

前四行是任务的运行环境变量

所有用户定义的 crontab 保存在 /var/spool/cron

## 格式

minute hour day month week username command

星号 `*`：代表所有可能的值，例如 month 字段如果是 `*` ，则表示在满足其它字段的制约条件后每月都执行该命令操作。

逗号 `,`：可以用逗号隔开的值指定一个列表范围，例如 1,2,5,7,8,9

中杠 `-`：可以用整数之间的中杠表示一个整数范围，例如 `2-6` 表示 2,3,4,5,6

正斜线 `/`：可以用正斜线指定时间的间隔频率，例如 `0-23/2` 表示每两小时执行一次。同时正斜线可以和星号一起使用，例如 `*/10`，如果用在 minute 字段，表示每十分钟执行一次

## 命令

```bash
yum install crontabs
ps -ax | grep cron //查看是否执行
tail -f /var/log/cron //查看执行日志
service crond status //crond 服务
crontab -l //查看用户的 cron 任务
crontab -e //编辑用户的 cron 任务
crontab -u //设定用户的 cron 任务
crontab -r //删除用户的 cron 任务
```
