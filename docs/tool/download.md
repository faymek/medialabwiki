# 下载工具

## 命令行下载

you-get 和 youtube-dl 都是基于 Python 的命令行媒体文件下载工具，完全开源免费跨平台。用户只需使用简单命令并提供在线视频的网页地址即可让程序自动进行嗅探、下载、合并、命名和清理，最终得到已经命名的完整视频文件。两者搭配使用几乎可以下载所有主流视频网站的视频及有关资源。在安装使用之前需要安装python3、pip、ffmpeg。

### annie

### you-get

github 项目页：https://github.com/soimort/you-get

```shell
pip install you-get
```

**常用命令：**

下载在线视频

```
you-get [视频网址]
```

查看特定网页所有视频资源格式

```
you-get -i [视频网址]
```

指定本地播放器播放在线视频（墙裂建议 Windows 用户将常用播放器安装目录加入系统环境变量，下载视频之前将当前目录切换到 C 盘以外）

```
you-get -p vlc [视频网址]
```

### youtube-dl

github 项目：https://github.com/rg3/youtube-dl

```
pip install youtube-dl
```

**常用命令：**

下载在线视频

```
youtube-dl [视频网址]
```

查看指定网页所有视频格式

```
youtube-dl -F [视频网址]
```

下载指定格式的媒体文件

```shell
youtube-dl -f [format code] [视频网址]
youtube-dl -f 视频编号+音频编号
```

```
37      :       mp4     [1080x1920]
22      :       mp4     [720x1280]
```

**使用Aria2加速：**

安装aria2c，ffmpeg。需要将aria2c安装目录加入环境变量。

```
scoop install aria2
scoop install ffmpeg
```

使用代理进行下载，aria2c不支持socks，因此：

```shell
// 720p mp4
youtube-dl --proxy "127.0.0.1:1080" --external-downloader aria2c --external-downloader-args "-x 16 -k 1M" -f 22[视频网址]
// best mp4
-f bestvideo[ext=mp4]+bestaudio[ext=m4a] --merge-output-format mp4
// best webm
-f bestvideo[ext=webm]+bestaudio[ext=webm][acodec=vorbis] --merge-output-format webm
```

–external-downloader aria2c //调用外部下载工具
–external-downloader-args //外部下载工具指定参数
-x 16 //启用aria2 16个线程，最多支持16线程
-K 1M //指定块的大小
-f 22 //指定下载格式为720p mp4

或使用，aria2c 解除单服务器线程数限制编译版 ，[52破解原帖](https://www.52pojie.cn/forum.php?mod=viewthread&tid=762859&page=1)

- 64位版本：https://ci.appveyor.com/api/projects/myfreeer/aria2-build-msys2/artifacts/aria2c.7z

- 32位版本：https://ci.appveyor.com/api/projects/myfreeer/aria2-build-msys2/artifacts/aria2c_x86.7z



**5.用法**
使用帮助命令查看其用法：

```
youtube-dl -h
```


一些常用的参数：

youtube-dl --list-extractors #查看支持网站列表
youtube-dl -U #程序升级
youtube-dl --get-format URL #获取视频格式
youtube-dl -F URL #获取所有格式（目前仅支持YouTube）
youtube-dl -f format URL #下载指定格式的视频，这里以下载1080p原画质量的视频格式为例:

```
youtube-dl -f 137 http://www.youtube.com/watch?v=n-BXNXvTvV4
```

**6.代理**
推荐使用SS代理。非全局模式，请在命令后面加  --proxy ‘socks5://127.0.0.1:1080‘  (1080是端口号，我的端口是1080，您的端口请按照您自己设置的填写)

例如： youtube-dl --proxy ‘socks5://127.0.0.1:1080‘ [URL] 

**7.下载YouTube视频**
1) 查看视频所有类型,只看不下载
youtube-dl -F [url]
或者
youtube-dl --list-formats [url]

这是一个列清单参数，执行后并不会下载视频，但能知道这个目标视频都有哪些格式存在，这样就可以有选择的下载啦！

**8.关于音频和视频的合并**
下载指定质量的视频和音频并自动合并
youtube-dl -f [format code] [url]

通过上一步获取到了所有视频格式的清单，最左边一列就是编号对应着不同的格式.
由于YouTube的1080p及以上的分辨率都是音视频分离的,所以我们需要分别下载视频和音频,可以使用137+140这样的组合.
如果系统中安装了ffmpeg的话, youtube-dl 会自动合并下下好的视频和音频, 然后自动删除单独的音视频文件

**9.下载字幕**
youtubd-dl --write-sub [url] //这样会下载一个vtt格式的英文字幕和mkv格式的1080p视频下来
youtube-dl --write-sub --skip-download [url] //下载单独的vtt字幕文件,而不会下载视频
youtube-dl --write-sub --all-subs [url] //下载所有语言的字幕(如果有的话)
youtube-dl --write-auto-sub [url] //下载自动生成的字幕(YouTube only)

**10.关于youtube的字幕接口**
获取所有语言的字幕列表：‘http://video.google.com/timedtext?hl=en&v=hRfHcp2GjVI&type=list‘
获取字幕，在视频有官方字幕的情况下：‘http://www.youtube.com/api/timedtext?lang=%s&v=%s&name=%s‘

「上传字幕」和「机器字幕」是不互相「兼容」的，有「上传字幕」的视频是没有「机器字幕」的，当然一个视频也可能「上传字幕」和「机器字幕」都没有

**11.webvtt字幕转srt字幕方法**
直接修改后缀名：直接将*.vtt文件的后缀名改为*.srt。然后删除最前方的类型标识符：WEBVTT，保存即可载入srt字幕正常使用。
附上知乎讨论贴：https://www.zhihu.com/question/29789259

 **12.直接使用youtubd-dl转字幕的方法**

youtubd-dl 自带一个 --convert-subs FORMAT ` `参数。

具体用法见例子：

```
youtube-dl -f 137+140  --convert-subs "srt" --all-subs https://www.youtube.com/watch?v=hRfHcp2GjVI
```

需要注意的是，不能带上 --skip-download 参数。

详见：https://github.com/rg3/youtube-dl/issues/9073

**13.下载视频列表**
youtube-dl -f [format code] [palylist_url] //这种方式可以下载制定清晰度的mp4视频
youtube-dl [playlist_url] //下载视频列表,这种方式下载的视频可能是mkv格式或者webm格式
youtube-dl -cit [playlist_url] //下载视频列表,这种方式下载的视频可能是mkv格式或者webm格式
youtube-dl --yes-playlist [url] //当链接为视频列表,则下载该列表视频,跟上面的一样,可能是mkv或者webm格式



### annie

### you-get

### youtube-dl 

## 视频下载

### 浏览器插件

- 油猴插件：Local YouTube Downloader 
- Stream Video Downloader

- Video Download Professional https://chrome.google.com/webstore/detail/elicpjhcidhpjomhibiffojpinpmmpil

### 网站下载

- https://en.savefrom.net/

### 流媒体下载

先在网页的F12，找到.m3u8或.mpd的链接，然后可以：

- `ffmpeg -i m3u8_uri "save_video.mp4"`
- VSO Downloader

## 字幕下载

下载带字幕的Youtube视频：KeepVid和ClipConverter 

下载YouTube字幕：https://downsub.com/， https://zhuwei.me/y2b/

## 通用下载

### IDM

 https://www.appinn.com/idm-lizhi-201904/

只有纯 HTTP 下载，不支持BT 

### XDM 

链接: https://pan.baidu.com/s/1mjsdTOk 密码: fz5k

https://www.jb51.net/softs/597062.html

 IDM和XDM 对比 分析 与 实验

https://blog.csdn.net/qq_32768743/article/details/85412299

### FDM

https://www.appinn.com/xtreme-download-manager/

### 讨论

实测还是IDM多线程速度更快更稳，两者都在用，超过10G的大文件用FDM下，IDM合并文件能把人搞疯，有次定时关机下一个46G的原盘，第二天开机下到99%没下完，然后重下光是合并文件就花了1个多小时，任务管理器硬盘读写速度在五六十M左右，照理十几分钟就完事了，搞不懂怎么这么恶心，还搞得我访问缓存硬盘内容卡的要死

新版本的FDM不支持批量下载，就弃用了。

主要是idm用来抓网页视频真乃神器，除了个别网站有协议不能爬，连ts的视频流都能抓下来整合好。另外，对于一些资源他的速度真的很快，我一度怀疑他有经服务器中转加速。

抓视频实现方法太多，随便一个浏览器插件都能做的很好，不足以成为付费卖点。速度快有待商榷，所谓速度不外乎受限于带宽线路和服务器，服务器速度一定，没有离线中转服务器什么工具都一样，现在的所有下载工具基本都是分块提速，分的块越多提速越快，这不是idm才有的独门秘技。除了界面丑哭已经放弃治疗以外，我看不出这个下载工具有什么独特付费卖点，假如它是开源项目我倒是可以理解

抓视频要看网站的吧，不知这个能否满足需要https://www.downloadhelper.net/

下载存档直播流，我测试过很多软件，只有idm可以捕获并且完整下载

 FDM和XDM和IDM都用过一段时间,IDM>FDM>XDM 

### Jdownloader

### Motirx