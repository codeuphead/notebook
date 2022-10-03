# Basic

下载已经编译好的二进制程序，包括三个模块：ffmpeg ffprobe ffplay

基本转换步骤：

1. 解封装 (Demuxing)
1. 解码 (Decoding)
1. 编码 (Encoding)
1. 封装 (Muxing)

## 编译相关

### 定制

先进行 configure 配置，再进行 make 编译，再执行 make install 安装即可。

configure 命令查看各种支持的封装格式、编码格式、流媒体传输协议。

对于 FFmpeg 不支持的格式，可以通过 `configure --help` 查看所需要的第三方库。

例如需要支持 H.264 视频与 AAC 音频编码，调整配置项 `configure --enable-libx264 --enable-libfdk-aac --enable-gpl --enable-nonfree`

> 2016 年初开始，FFmpeg 自身的 AAC 编码器质量逐步好转，现在 FFmpeg 源代码中，已经没有 libfaac

- 查看支持的编码器 `configure --list-encoders`
- 查看支持的解码器 `configure --list-decoders`
- 查看支持的封装 `configure --list-muxers`
- 查看支持的解封装 `configure --list-demuxers`
- 查看支持的通信协议 `configure --list-protocols`

## 简介

### `ffmpeg -h` 显示帮助信息

### `ffmpeg -version` 显示版本信息

### `ffmpeg -formats` 支持的文件格式

- 第一列 `D.` 代表支持封装 `.E` 代表支持解封装
- 第二列 文件格式
- 第三列 格式详细说明

### `ffmpeg -encoders` 支持的编码器

### `ffmpeg -decoders` 支持的解码器

- 第一列
  - 第 1 位 V A S 分别代表 视频、音频、字母
  - 第 2 位 帧级多线程支持
  - 第 3 位 分片级多线程支持
  - 第 4 位 改编码为试验版本
  - 第 5 位 draw horiz band 模式的支持
  - 第 6 位 直接渲染模式的支持
- 第二列 编码格式
- 第三列 说明

### `ffmpeg -filters` 支持的滤镜

- 第一列
  - 时间轴支持
  - 分片线程处理支持
  - 命令支持
- 滤镜名
- 转换方式，如音频转音频，视频转视频，创建音频，创建视频等
- 作用说明

### 查看特定格式

- `ffmpeg -h -muxer=flv` 查看 FLV 封装器的参数支持
- `ffmpeg -h -demuxer=flv` 查看 FLV 解封装器的参数支持
- `ffmpeg -h -encoder=h264` 查看 H.264(AVC) 的编码参数支持
- `ffmpeg -h -decoder=h264` 查看 H.264(AVC) 的解码参数支持
- `ffmpeg -h filter=colorkey` 查看 colorkey 滤镜的参数支持

### 参数查看

该功能包含在 AVFormat 模块中，通过 libavformat 库进行 mux 和 demux 操作

通过 `ffmpeg --help full`

AVFormatContext 参数部分，为封装转换可使用的参数

AVCodecContext 参数，部分为编解码可使用的参数

### ffprobe 常用命令

`ffprobe --help` 查看详细帮助信息

`ffprobe file.mp3` 查看文件信息

`ffprobe -show_packets input.flv` 查看多媒体数据包信息

### ffplay 常用命令

## 简单命令示例

```bash
//普通转换
ffmpeg -i input.flv -c copy -f mp4 output.mp4

//将 mp4 的 moov 移动到 mdat 之前
ffmpeg -i input.flv -c copy -f mp4 -movflags faststart output.mp4

//音频转换 从AC3 到 AAC 或 mp3
ffmpeg -i input_ac3.mp4 -vcodec copy -acodec aaac -f flv output.flv

//生成带关键帧索引的 FLV
ffmpeg -i input.mp4 -c coyp -f flv -flvflags add_keyframe_index output.flv
```

## ffmepg 常用参数

```bash
-y 覆盖输出文件
-i 输入文件
-vcodec h264 视频编码格式
-b 设置比特率 默认200 kb/s
-r 设置帧率 默认 25
-s 设置分辨率
-acodec 音频编码格式
-ar 设置音频采样率 44100
-ab 设置音频比特率 128 96 64 32
-ac 设置声道数 默认 1
-vn 忽略视频流
-an 忽略音频流
-vol 音量
-c copy 使用源文件的编码格式，不进行转码，如需转码，设置对应参数即可
-bsf:v h264_mp4toannexb 如果输入文件是 MP4 格式，将 MP4 中的 H.264 数据转换为 H.264 AnnexB 标准编码，如果源文件为 FLV、TS 等流媒体格式，则不需要这个参数
-hls_time 设置每个分片时长，该切片规则是从关键帧处开始切片，时间不一定均匀
-hls_playlist_type vod 设置为点播式列表
-hls_list_size 0 分片个数，0 为保持所有的分片
-hls_base_url 设置分片前置路径，需要设置为待存储的 oss 路径
-hls_segment_filename 设置分片文件名模板
-use_localtime 以时间戳为切片文件名
-hls_key_info_file 设置加密文件路径
```

### 音频采样率

```bash
48KHz DVD 品质
44.1KHz CD 品质
22.05KHz FM 广播品质
```

### 音频比特率

每秒钟，用多少个比特来表示

`采样率 * 采样大小 * 声道数`

128K 的比特率是 `44.1KHz * 16bit * 2channel = 1411.2Kbps = 176KBs`

即 1 秒钟采样 44.1KHz，采样大小为 16 bit，双声道，一秒需要 176KB 控件，一分钟约为 10.34M

VBR 动态比特率 ABR 平均比特率 CBR 固定比特率

16 bit 为基本，CD 则是 20-24 bit

## HLS 转封装

M3U8 文件格式介绍

- `EXTM3U`

必须要包含的标签，必须在第一行

- `EXT-X-VERSION`

文件版本，通常为 3，目前至少为 7

- `EXT-X-TARGETDURATION`

每一个分片都会有自己的 duration，该标签是最大的分片的近似值

- `EXT-X-MEDIA-SEQUENCE`

直播时，直播切片序列，当播放打开 M3U8 时，以这个标签的值为参考，播放对应的序列号的切片。
分片必须是动态的，序列不能相同，必须是增序。
当 M3U8 列表中，没有 EXT-X-ENDLIST 标签时，播放从倒数第三片播放。
如果前一分片与后一分片不连续，需要使用 EXT-X-DISCONTINUITY 标签来解决这个错误。

- `EXTINF`

为 M3U8 列表中每一个分片的 duration，还可以包含可选的描述信息，使用逗号分开
EXTINF 下面的信息，为具体的分片信息，分片存储路径等信息

- `EXT-X-ENDLIST`

表明该 M3U8 文件不会再产生更多切片，播放分片到该标签后结束。点播类型的 M3U8 均为此

- `EXT-X-STREAM-INF`

主要出现在多级 M3U8 文件中时，例如主 M3U8 包含子 M3U8 列表，或者主 M3U8 包含多码率时，该标签后需要跟随一些属性

BANDWIDTH：播放 M3U8 时最大的带宽
AVRAGE-BANDWIDTH：
CODECS：声明 M3U8 里面的编码信息
RESOLUTION：视频宽高信息
RFRAME-RATE：视频帧率

例子

```bash
#EXTM3U8
#EXT-X-STREAM-INF:BANDWIDTH=1280000,AVERAGE-BANDWIDTH=1000000
http://example.com/low.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=2560000,AVERAGE-BANDWIDTH=2000000
http://example.com/mid.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=7680000,AVERAGE-BANDWIDTH=6000000
http://example.com/high.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=65000,CODECS="mp4a.40.5"
http://example.com/audio-only.m3u8

```

### 常用参数

```bash
hls_time 设置每一片时长，从关键帧处切片，所以时间并不均匀，如果先转码，在切片会比较规律
hls_list_size 设置分片个数
hls_base_url 设置分片前置路径
hls_segment_filename 设置分片文件名模板
hls_key_info_file 设置加密文件路径
hls_playlist_type 设置 M3U8 列表为事件或点播列表
method 设置 HTTP 属性
```

### 其他参数

```bash
start_number 设置 M3U8 列表中的第一片序列数
hls_wrap 设置切片索引回滚边界值
hls_allow_cache 设置 M3U8 中 EXT-X-ALLOW-CACHE 的标签
hls_subtitle_path 设置字幕路径
hls_flags
  single_file 生成一个媒体文件索引与字节范围
  delete_segments 删除 M3U8 文件中不好汗的过期的 TS 切片文件
  round_durations 设置 duration 为整数
  discount_start 设置 M3U8 的 discontinuity 标签
  omit_endlist 设置 M3U8 不追加 endlist 标签
  split_by_time 通过时间切片，每个切片时间一致，容易造成问题
use_localtime 设置 M3U8 文件序号为本地时间戳
use_localtime_mkdir 根据本地时间戳生成目录
```

`-bsf:v h264_mp4toannexb` 将 MP4 中的 H.264 数据转换为 H.264 AnnexB 标准编码，常用于直播传输流的视频，如果源文件为 FLV、TS 等流媒体格式，则可以不需要这个参数

`-re` read input at native frame rate

`ffmpeg -re -i input.mp4 -c copy -f hls -baf:v h264_mp4toannexb -hls_time 10 output.m3u8`

### 普通转换

示例：

```bash
ffmpeg -y -i .\911Mothers_2010W-480p.mp4 -hls_time 10 -hls_playlist_type vod -hls_segment_filename output-%d.ts output.m3u8
```

### 加密转换

HLS 仅支持 AES-128，并在 CBC 模式下使用 AES，加密前需要生成 AES-128 密钥与 IV（初始化向量）

```bash
openssl rand 16 > enc.key //生成随意的16字节值，对应密钥长度128位
openssl rand -hex 16 > iv.txt
```

使用 `-hls_key_info_file` 来传递给加密信息，格式为

```bash
Key URI //指定将写入 M3U8 的 URI
Path to key file //密钥路径
IV //初始化向量
```

示例

```bash
http://113.200.67.189:8082/hls-demo/enc.key
enc.key
f36311dda0f718ca3e83490c557678db
```

转换

示例：

```bash
ffmpeg -i input.mp4 -c copy -bsf:v h264_mp4toannexb -hls_time 10 -hls_playlist_type vod -hls_list_size 0 -hls_key_info_file key.info output.m3u8
```

生成的 M3U8 文件内容为：

```bash
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:11
#EXT-X-MEDIA-SEQUENCE:0
#EXT-X-KEY:METHOD=AES-128,URI="http://113.200.67.189:8082/hls-demo/enc.key",IV=0xf36311dda0f718ca3e83490c557678db
#EXTINF:10.666667,
output0.ts
...
#EXTINF:9.428333,
output5.ts
#EXT-X-ENDLIST
```
