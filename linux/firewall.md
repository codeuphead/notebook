# 防火墙

## iptables

vim /etc/sysconfig/iptables

service iptables restart

## Firewall

- firewall-cmd --state //查看状态
- firewall-cmd --list-all //查看所有
- firewall-cmd --zone=public --list-all //查看公共
- firewall-cmd --get-service //获取可用服务
- firewall-cmd --get-active-zones //查看激活的zone信息
- firewall-cmd --get-zone-of-interface=eth0 //查看指定接口的 zone 信息
- firewall-cmd --zone=public --list-interface //查看指定级别的接口
- firewall-cmd --reload //更新规则 不重启
- firewall-cmd --complete-reload //更新规则 重启服务

- --add-port --add-service --add-interface 添加端口 添加服务 添加接口

```bash
firewall-cmd --zone=public --add-port=5060-5059/udp --permanent
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --reload
```

### zone

- drop: 丢弃所有进入的包，而不给出任何响应
- block: 拒绝所有外部发起的连接，允许内部发起的连接
- public: 允许指定的进入连接
- external: 同上，对伪装的进入连接，一般用于路由转发
- dmz: 允许受限制的进入连接
- work: 允许受信任的计算机被限制的进入连接，类似 workgroup
- home: 同上，类似 homegroup
- internal: 同上，范围针对所有互联网用户
- trusted: 信任所有连接

### 过滤规则

- source: 根据源地址过滤
- interface: 根据网卡过滤
- service: 根据服务名过滤
- port: 根据端口过滤
- icmp-block: icmp 报文过滤，按照 icmp 类型配置
- masquerade: ip 地址伪装
- forward-port: 端口转发
- rule: 自定义规则

优先级

1. source
1. interface
1. firewalld.conf
