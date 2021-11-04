---
title: "FFMepg使用笔记"
date: 2017-05-22
tags: ["ffmepg"]
draft: false
---

## 基本使用语法

`ffmepg -i <file> -vcodec h264 -f <type> <output>`
简单命令如下：

    ffmpeg -i out.ogv -vcodec h264 out.mp4
    ffmpeg -i out.ogv -vcodec mpeg4 out.mp4
    ffmpeg -i out.ogv -vcodec libxvid out.mp4
    ffmpeg -i out.mp4 -vcodec wmv1 out.wmv
    ffmpeg -i out.mp4 -vcodec wmv2 out.wmv
    ffmpeg -i out.ogv -s 640x480 -vcodec h264 out.mp4

**主要参数：**
-i: 设定输入流
-f: 设定输出格式
-ss: 开始时间

**视频参数：**
-b 设定视频流量，默认为200Kbit/s
-r 设定帧速率，默认为25
-s 设定画面的宽与高
-aspect 设定画面的比例
-vn 不处理视频
-vcodec 设定视频编解码器，未设定时则使用与输入流相同的编解码器, h264 最佳

**音频参数：**
-ar 设定采样率
-ac 设定声音的Channel数
-acodec 设定声音编解码器，未设定时则使用与输入流相同的编解码器
-an 不处理音频

例如：
`ffmpeg -i out.ogv -s 640x480 -b 500k -vcodec h264 -r 29.97 -b:a 48k -ac 2 out.mp4`
`ffmpeg -i cucumber.mov -c:v libx264 -crf 28 -s 640x480 output.mp4`

## 附录1：Mp4 文件如何在线快速播放

MP4文件不是流媒体文件，采用flash 进行web播放时，有不少mp4文件（特别是录制的视频）的mp4的头文件（索引文件）默认在末尾，会导致整个视频不能**边缓存边播放**，而是整个缓存后才能播放。处理方式要进行转码，但是有时直接转码并不能解决这个问题，需要移动moov头文件才可以，参数的关键是`-movflags faststart`，具体调用方式如下：

`ffmepg -i <file> -movflags faststart -f <type> <output>`

## 参考
> [Fast-Start Enabled Videos with FFMpeg](http://salman-w.blogspot.co.il/2013/08/fast-start-enabled-videos-with-ffmpeg.html)
> [ffmpeg 入门](http://einverne.github.io/post/2015/12/ffmpeg-first.html)



