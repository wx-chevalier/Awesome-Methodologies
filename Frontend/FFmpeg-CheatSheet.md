# FFmpeg CheatSheet

# 基础使用

## 参数

```sh
# Common switches
-codecs          # list codecs
-c:v             # video codec (-vcodec) - 'copy' to copy stream
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
$ ffmpeg -f dshow -i video="Integrated Webcam":audio="Microphone (Realtek Audio)" -profile:v high -pix_fmt yuvj420p -level:v 4.1 -preset ultrafast -tune zerolatency -vcodec libx264 -r 10 -b:v 512k -s 640x360 -acodec aac -ac 2 -ab 32k -ar 44100 -f mpegts -flush_packets 0 udp://192.168.1.4:5000?pkt_size=1316
```
