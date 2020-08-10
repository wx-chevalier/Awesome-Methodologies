# FFmpeg CheatSheet

# 基础使用

FFMPEG 可以使用下面的参数进行基本信息查询。例如，想查询一下现在使用的 FFMPEG 都支持哪些 filter，就可以用 ffmpeg -filters 来查询。详细参数说明如下：

| 命令            | 作用                                 |
| --------------- | ------------------------------------ |
| FFmpeg -version | 显示版本                             |
| -formats        | 显示可用的格式（包括设备）           |
| -demuxers       | 显示可用的 demuxers                  |
| -muxers         | 显示可用的 muxers                    |
| -devices        | 显示可用的设备                       |
| -codecs         | 显示已知的所有编解码器               |
| -decoders       | 显示可用的解码器                     |
| -encoders       | 显示所有可用的编码器                 |
| -bsfs           | 显示可用的比特流 filter              |
| -protocols      | 显示可用的协议                       |
| -filters        | 显示可用的过滤器                     |
| -pix_fmts       | 显示可用的像素格式                   |
| -sample_fmts    | 显示可用的采样格式                   |
| -layouts        | 显示 channel 名称和标准 channel 布局 |
| -colors         | 显示识别的颜色名称                   |

## 参数

```sh
# Common switches
-codecs          # list codecs
-c:v             # video codec (-vcodec): 'copy' to copy stream
-c:a             # audio codec (-acodec)
-fs SIZE         # limit file size (bytes)

# Audio
-aq QUALITY      # audio quality (codec-specific)
-ar 44100        # audio sample rate (hz)
-ac 1            # audio channels (1=mono, 2=stereo)
-an              # no audio
-vol N           # volume (256=normal)

# Bitrate
-b:v 1M          # video bitrate (1M = 1Mbit/s)
-b:a 1M          # audio bitrate

# Video
-aspect RATIO    # aspect ratio (4:3, 16:9, or 1.25)
-r RATE          # frame rate per sec
-s WIDTHxHEIGHT  # frame size
-vn              # no video
```

# 录制

```sh
# 查找设备名
## Windows
$ ffmpeg -list_devices true -f dshow -i dummy
## MAC
$ ffmpeg -f avfoundation -list_devices true -i ""
```

## Windows

```go
$ ffmpeg -f dshow -i video="Virtual-Camera" -preset ultrafast -vcodec libx264 -tune zerolatency -b 900k -f mpegts udp://10.1.0.102:1234

$ ffmpeg -f dshow -i video="screen-capture-recorder":audio="Stereo Mix (IDT High Definition" \
-vcodec libx264 -preset ultrafast -tune zerolatency -r 10 -async 1 -acodec libmp3lame -ab 24k -ar 22050 -bsf:v h264_mp4toannexb \
-maxrate 750k -bufsize 3000k -f mpegts udp://192.168.5.215:48550
```

- `-f fshow`: windows system drivers for capturing video and audio
- `-f v4l2`: linux system drivers for capturing video
- `-f alsa`: linux system drivers for capturing audio
- `-i`: ffmpeg option that defines **input**
- `-vcodec libx264`: raw video from camera will be encoded using H264 video codec
- `-r 10`: video FPS (frames per second)
- `-b:v 512k`: video bitrate Kb/s (kilo bits per second)
- `-s 640x360`: video width and height
- `-acodec aac`: raw audio from microphone will be encoded using AAC audio codec
- `-ac 2`: 2 audio channels (stereo)
- `-ab 32k`: audio bitrate in Kb/s
- `-ar 44100`: audio sampling rate 44.1 KHz
- `-f mpegts`: video and audio will be packed into [MPEG transport stream (MPEG TS)](https://en.wikipedia.org/wiki/MPEG_transport_stream)
- `udp://192.168.1.4:5000`: MPEG transport stream is sent via UDP protocol to computer with IP address 192.168.1.4 on IP port 5000.

```sh
$ ffmpeg -y -loglevel warning -f dshow -i video="screen-capture-recorder" -vf crop=690:388:136:0 -r 30 -s 962x388 -threads 2 -vcodec libx264 -vpre baseline -vpre my_ffpreset -f flv rtmp:///live/myStream.sdp

# 输出为 MP4
$ ffmpeg -f dshow -rtbufsize 1000000k -s 640×480 -r 30 -i video=”1714-INOGENI 4K2USB3″ -an -c:v libx264 -q 0 -f h264 – | ffmpeg -f h264 -i – -an -c:v copy -f mp4 file.mp4 -an -c:v copy -f h264 pipe:play | ffplay -i pipe:play
```

## Mac

```sh
#! /bin/bash
#
# Diffusion bilibili live avec ffmpeg
# Make sure you have FFmpeg installed in your mac

# list avfoundation devices
ffmpeg -f avfoundation -list_devices true -i ""

# change the param after `-i` and `-f flv`

# use 140m mermory
ffmpeg \
  -f avfoundation \
  -re -i "2" \
  -vcodec libx264 \
  -preset ultrafast \
  -acodec aac \
  -ar 44100 \
  -ac 1 \
  -f flv "rtmp://example.com/path?key=xx"

# 560m mermory 90% cpu
ffmpeg \
  -f avfoundation \
  -video_size 1920x1080 \
  -framerate 30 \
  -i "2:0" -ac 2 \
  -vcodec libx264 -maxrate 2000k \
  -bufsize 2000k -acodec libmp3lame -ar 44100 -b:a 128k \
  -f flv "rtmp://example.com/path?key=xx"
```

# 格式转换与压缩

```sh
$ ffmpeg -i in.mp4 out.avi
```

```sh
$ ffmpeg -i foo.mp3 -ac 1 -ab 128000 -f mp4 -acodec libfaac -y target.m4r

# no audio
ffmpeg -i input.mov -vcodec h264   -an -strict -2 output.mp4
ffmpeg -i input.mov -vcodec libvpx -an output.webm
ffmpeg -i input.mov -vcodec h264 -acodec aac -strict -2 output.mp4
ffmpeg -i input.mov -vcodec libvpx -acodec libvorbis output.webm
<video width="320" height="240" controls>
  <source src="movie.mp4" type='video/mp4'></source>
  <source src="movie.webm" type='video/ogg'></source>
</video>
```

# 视频生成

```sh
$ ffmpeg -framerate 1 -pattern_type glob -i '*.bmp' -c:v libx264 -r 30 -pix_fmt yuv420p out.mp4
```

## 视频联接

```sh
:: Create File List
echo file file1.mp4 >  mylist.txt
echo file file2.mp4 >> mylist.txt
echo file file3.mp4 >> mylist.txt

# Windows 下视频连接
:: Concatenate Files
ffmpeg -f concat -i mylist.txt -c copy output.mp4

:: Create File List
for %%i in (*.mp4) do echo file '%%i'>> mylist.txt

:: Concatenate Files
ffmpeg -f concat -safe 0 -i mylist.txt -c copy output.mp4
```
