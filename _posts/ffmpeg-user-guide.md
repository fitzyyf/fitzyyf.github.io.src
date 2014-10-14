title: "ffmpeg相关使用说明"
date: 2013-05-24 10:26
tags:
- ffmpeg
categories: 
- 开发笔记
---
最近使用ffmpeg 来处理下视频相关的信息，整理如下：

## ffmpeg 查看支持的格式
	# yfyang at mumu-mbp in ~/Movies/sublime text 2 [10:38:10]
	$ ffmpeg -formats less
    File formats:
     D. = Demuxing supported
     .E = Muxing supported
     --
      E 3g2             3GP2 (3GPP2 file format)
      E 3gp             3GP (3GPP file format)
     D  4xm             4X Technologies
      E a64             a64 - video for Commodore 64
     D  aac             raw ADTS AAC (Advanced Audio Coding)
     DE ac3             raw AC-3
     D  act             ACT Voice file format
     ...

## ffmpeg 获取视频信息及含义

	# yfyang at mumu-mbp in ~/Movies/sublime text 2 [10:44:17]
	$ ffmpeg -i 10-Package-Control.mov
	Input #0, mov,mp4,m4a,3gp,3g2,mj2, from '10-Package-Control.mov':
	  Metadata:
	    major_brand     : qt
	    minor_version   : 537199360
	    compatible_brands: qt
    creation_time   : 2012-09-21 22:00:29
	  Duration: 00:03:56.67, start: 0.000000, bitrate: 349 kb/s  
>  Duration:播放时间	,start:开始时间,bitrate:码率 单位 kb
	  
	  Stream #0:0(eng): Audio: aac (mp4a / 0x6134706D), 44100 Hz, stereo, fltp, 94 kb/s
> Audio:aac (mp4a / 0x6134706D):音频编码 ,44100 Hz:音频采样频率
	  
	    Metadata:
	      creation_time   : 2012-09-21 22:00:29
	      handler_name    : Apple Alias Data Handler
	    Stream #0:1(eng): Video: h264 (Main) (avc1 / 0x31637661), yuv420p, 1152x720, 252 kb/s, 15 fps, 15 tbr, 1500 tbn, 3k tbc

> Video: h264 (Main) (avc1 / 0x31637661):编码格式, yuv420p:视频格式, 1152x720:分辨率, 252 kb/s, 15 fps, 15 tbr, 1500 tbn, 3k tbc
	    
	    Metadata:
	      creation_time   : 2012-09-21 22:00:29
	      handler_name    : Apple Alias Data Handler

## ffmpeg 截图视频
1. 截取视频默认分辨率的图片
	
		ffmpeg -i 10-Package-Control.mov -vframes 1 -f image2 -y -ss 00:01:38 -t 0.01 ss.jpg
	
	> 将`10-Package-Control.mov`中第一分38秒的视频截图为ss.jpg
	
	> 这里建议截取为jpg，png的话要比jpg大不少，大约在3倍左右。

2. 指定分辨率截取

		ffmpeg -i 10-Package-Control.mov  -vframes 1 -f image2 -y -ss 38 -t 0.01 -s 100x100 ss.png
		
	> -s 100x100 截取100x100的分辨率

## ffmpeg 转换视频
1. 转换指定格式文件到flv格式

		ffmpeg -i 10-Package-Control.mov test.flv -ab 64 -acodec mp3 -ac 2 -ar 22050 -b 230 -r 24 -y
		
## Java调用ffmpeg处理视频工具
