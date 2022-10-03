# WSL

## 安装

以管理员身份运行 powershell，执行命令

```bash
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```

从 MS Store 下载 Linux 发行版

启动发行版，设置常用用户名、密码

- 设置 apt 源

`sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak`

`替换为 http://mirrors.aliyun.com`

- 多个发行版

可以安装多个独立的发行版

wslconfig.exe 查看

## WSL Windows

Windows 下的所有盘符都挂载在 WSL 中的 `/mnt` 目录下，可以直接操作

WSL 中的所有数据则存放于 `C:\Users\{你的用户名}\AppData\Local\Packages\{Linux发行版包名}\LocalState\rootfs` 目录中

Windows 运行 WSL 命令

`wsl ls -all`

WSL 运行 Windows 命令

`ipconfig.exe`

`notepad.exe`

## 手动添加至 Terminal

```bash
{
    "acrylicOpacity" : 0.5,
    "closeOnExit" : true,
    "colorScheme" : "Campbell",
    "commandline" : "wsl.exe -d Ubuntu",
    "cursorColor" : "#FFFFFF",
    "cursorShape" : "bar",
    "fontFace" : "Consolas",
    "fontSize" : 10,
    "guid" : "{2c4de342-38b7-51cf-b940-2309a097f518}",
    "historySize" : 9001,
    "icon" : "ms-appx:///ProfileIcons/{9acb9455-ca41-5af7-950f-6bca1bc9722f}.png",
    "name" : "Ubuntu",
    "padding" : "0, 0, 0, 0",
    "snapOnInput" : true,
    "useAcrylic" : false
}
```

## 美化 shell - oh my zsh

查看自己的 shell `cat /etc/shells`

### 安装 zsh

```bash
sudo apt install zsh

chsh -s /bin/zsh #安装完成后设置当前用户使用 zsh 并重启 wsl
```

- wsl-terminal 需要额外配置：

```bash
.bashrc 开头输入以下内容

if [ -t 1 ]; then
exec zsh
fi
```

安装 oh my zsh

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

### 配置 zsh

切换主题

```bash
vi ~/.zshrc
ZSH_THEME="ys"
source ~/.zshrc
```

## 代理

```bash
pip install genpac

sudo genpac -p "SOCKS5 127.0.0.1:1080" --gfwlist-proxy="SOCKS5 127.0.0.1:1080" --gfwlist-url=https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt --output="autoproxy.pac"
```

```bash
.bashrc
export http_proxy=http://127.0.0.1:1080
export https_proxy=http://127.0.0.1:1080
```
