# FFmpeg CheatSheet

# 基础使用

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
$ ffmpeg -framerate 1 -pattern_type glob -i '*.bmp' \\n  -c:v libx264 -r 30 -pix_fmt yuv420p out.mp4
```

# 摄像头

```sh
# 查找设备名
$ ffmpeg -list_devices true -f dshow -i dummy

# 输出为 MP4
$ ffmpeg -f dshow -rtbufsize 1000000k -s 640×480 -r 30 -i video=”1714-INOGENI 4K2USB3″ -an -c:v libx264 -q 0 -f h264 – | ffmpeg -f h264 -i – -an -c:v copy -f mp4 file.mp4 -an -c:v copy -f h264 pipe:play | ffplay -i pipe:play

# 输出为 UDP 流
$ ffmpeg -f dshow -i video="Integrated Webcam":audio="Microphone (Realtek Audio)" -profile:v high -pix_fmt yuvj420p -level:v 4.1 -preset ultrafast -tune zerolatency -vcodec libx264 -r 10 -b:v 512k -s 640x360 -acodec aac -ac 2 -ab 32k -ar 44100 -f mpegts -flush_packets 0 udp://192.168.1.4:5000?pkt_size=1316
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
