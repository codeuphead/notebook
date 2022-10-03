# openwrt

## 安装、升级

服务器在线重新覆盖硬盘。 `dd if=/tmp/upload/xxx   of=/dev/sdb`

`lsblk` 查看分区信息。

loop0 循环设备，逻辑上的虚拟设备，该设备一般挂载 overlay。

16M + 300M sda1 + sda2。

16M 装内核 kernel。

300M 装用户相关数据。

squash 文件系统支持 overlay，openwrt 使用这种格式。

ext4 无法重置，没有 overlay 机制。

---

cfdisk

>/dev/sda1 2048 20973570 20971523 10G Linux filesystem
>/dev/sda2 20975616 468862094 447886479 213.6G Linux filesystem

选择 freespace 创建新的分区，选择大小，primary，保存 write

---

mkfs.ext4 /dev/sda3(刚创建的分区)

mount /dev/sda3 /mnt/sda3

ls /mnt/sda3

---

迁移 overlay 文件

cd /overlay

ls

cp -r /overlay/* /mnt/sda3

ls /mnt/sda3

---

openwrt 系统界面，挂载点，添加，UUID 找到刚才的分区，挂载点选作为外部overlay，保存应用，重启

## 配置

### DHCP 静态配置

/etc/config/dhcp

```bash
config host
  option name 'asus-ap'
  option dns '1'
  option mac 'a8:5e:45:af:82:e8'
  option ip '10.10.10.2'

config host
  option name 'leo-tower'
  option dns '1'
  option mac '70:85:c2:f7:49:4a'
  option ip '10.10.10.3'

config host
  option name 'pc-home'
  option dns '1'
  option mac '0c:9d:92:86:07:67'
  option ip '10.10.10.4'

config host
  option name 'switch'
  option dns '1'
  option mac '48:a5:e7:c4:2e:8b'
  option ip '10.10.10.5'

config host
  option name 'xbox-lan'
  option dns '1'
  option mac '38:56:3d:0a:5f:a8'
  option ip '10.10.10.6'

config host
  option name 'sony-tv'
  option dns '1'
  option mac '0c:96:e6:58:b2:3d'
  option ip '10.10.10.7'

config host
  option name tencent-box
  option dns '1'
  option mac '34:85:11:43:f9:22'
  option ip '10.10.10.8'

config host
  option name hw-mate-40-pro
  option dns '1'
  option mac 'b2:4d:63:1d:6f:4f'
  option ip '10.10.10.21'

config host
  option name 'hw-mate-30-pro'
  option dns '1'
  option mac '72:6a:e1:9a:09:04'
  option ip '10.10.10.22'

config host
  option name 'hw-m6'
  option dns '1'
  option mac 'b2:bc:33:40:b8:08'
  option ip '10.10.10.23'

config host
  option name 'mi-pad'
  option dns '1'
  option mac '7c:03:ab:7d:78:e1'
  option ip '10.10.10.24'

config host
  option name 'mi-11-du'
  option dns '1'
  option mac '4c:02:20:53:a4:45'
  option ip '10.10.10.25'

config host
  option name 'mi-11-lqx'
  option dns '1'
  option mac '9c:bc:f0:8f:46:7b'
  option ip '10.10.10.26'

config host
  option name 'iphone-8-plus'
  option dns '1'
  option mac 'b6:6f:b6:5b:40:34'
  option ip '10.10.10.27'

config host
  option name 'mi-8-lite'
  option dns '1'
  option mac '70:bb:e9:bd:08:a1'
  option ip '10.10.10.28'

config host
  option name 'hw-changxiang-10'
  option dns '1'
  option mac 'e4:19:c1:af:f5:5f'
  option ip '10.10.10.29'

config host
  option name 'hw-laptop'
  option dns '1'
  option mac 'f4:a4:75:4a:66:b8'
  option ip '10.10.10.30'

config host
  option name 'lisa-notebook'
  option dns '1'
  option mac 'b0:7d:64:85:f3:bd'
  option ip '10.10.10.31'

config host
  option name 'kindle'
  option dns '1'
  option mac '74:a7:ea:cc:f8:14'
  option ip '10.10.10.32'

config host
  option name 'hw-camera-1'
  option dns '1'
  option mac '60:6d:3c:b8:a9:b4'
  option ip '10.10.10.50'

config host
  option name 'hw-camera-2'
  option dns '1'
  option mac '60:6d:3c:b8:96:93'
  option ip '10.10.10.51'

config host
  option name 'hw-720-air'
  option dns '1'
  option mac '78:b3:b9:15:80:92'
  option ip '10.10.10.52'

config host
  option name 'mi-light'
  option dns '1'
  option mac '54:48:e6:be:ad:c4'
  option ip '10.10.10.53'

config host
  option name 'mi-switch'
  option dns '1'
  option mac '44:23:7c:dc:90:1a'
  option ip '10.10.10.54'

config host
  option name 'media-wahser'
  option dns '1'
  option mac '34:5b:bb:c8:0b:19'
  option ip '10.10.10.55'

config host
  option name 'media-fridge'
  option dns '1'
  option mac 'f0:c9:d1:62:79:2a'
  option ip '10.10.10.56'

config host
  option name 'media-rice'
  option dns '1'
  option mac '70:86:ce:82:66:e8'
  option ip '10.10.10.57'

config host
  option name 'epson-printer'
  option dns '1'
  option mac 'dc:cd:2f:4b:5e:99'
  option ip '10.10.10.58'

config host
  option name 'mi-story'
  option dns '1'
  option mac '7c:49:eb:b2:e4:50'
  option ip '10.10.10.59'

config host
  option name 'hw-electric'
  option dns '1'
  option mac '88:97:46:2e:1c:fe'
  option ip '10.10.10.60'

config host
  option name 'hw-electric2'
  option dns '1'
  option mac '88:97:46:32:cf:56'
  option ip '10.10.10.61'

config host
  option name 'hw-weight'
  option dns '1'
  option mac 'd8:bf:c0:e4:6b:fd'
  option ip '10.10.10.62'

config host
  option name 'hw-speaker-2e'
  option dns '1'
  option mac 'f8:af:05:1c:88:9d'
  option ip '10.10.10.63'

config domain
  option name 'next.leoupup.cn'
  option ip '10.10.10.3'

config domain
  option name 'emby.leoupup.cn'
  option ip '10.10.10.3'

config domain
  option name 'doc.leoupup.cn'
  option ip '10.10.10.3'
```

---

微信推送

企业ID(corpid)
ww4a1e1dd83095080a

帐号(userid)
@all
群发到应用请填入 @all

应用id(agentid)
1000002

应用密钥(Secret)
qklym9Asb1-5vAM8LpPforuPb2o4h7HEP-dvkhZQFN0

```bash
a8:5e:45:af:82:e8\|70:85:c2:f7:49:4a\|0c:9d:92:86:07:67\|48:a5:e7:c4:2e:8b\|38:56:3d:0a:5f:a8\|0c:96:e6:58:b2:3d\|34:85:11:43:f9:22\|b2:4d:63:1d:6f:4f\|72:6a:e1:9a:09:04\|b2:bc:33:40:b8:08\|7c:03:ab:7d:78:e1\|b6:6f:b6:5b:40:34\|70:bb:e9:bd:08:a1\|e4:19:c1:af:f5:5f\|f4:a4:75:4a:66:b8\|b0:7d:64:85:f3:bd\|74:a7:ea:cc:f8:14\|60:6d:3c:b8:a9:b4\|60:6d:3c:b8:96:93\|78:b3:b9:15:80:92\|54:48:e6:be:ad:c4\|44:23:7c:dc:90:1a\|34:5b:bb:c8:0b:19\|f0:c9:d1:62:79:2a\|70:86:ce:82:66:e8\|dc:cd:2f:4b:5e:99\|7c:49:eb:b2:e4:50\|88:97:46:2e:1c:fe\|88:97:46:32:cf:56\|d8:bf:c0:e4:6b:fd\|f8:af:05:1c:88:9d
```
