[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Raspberry CheatSheet | 树莓派资料索引

# 配置

- 博通 BCM2837B0 SoC，集成四核 ARM Cortex-A53（ARMv8）64 位@ 1.4GHz CPU，集成博通 Videocore-IV GPU
- 内存：1GB LPDDR2 SDRAM
- 有线网络：千兆以太网（通过 USB2.0 通道，最大吞吐量 300Mbps）
- 无线网络:2.4GHz 和 5GHz 双频 Wi-Fi，支持 802.11b/g/n/ac
- 蓝牙：蓝牙 4.2&低功耗蓝牙（BLE）
- 存储：Micro-SD
- 其他接口：HDMI，3.5mm 模拟音频视频插孔，4x USB 2.0，以太网，摄像机串行接口（CSI），显示器串行接口（DSI），MicroSD 卡座，40pin 扩展双排插针
- 尺寸：82mmx 56mmx 19.5mm，50 克

## 系统

我们首先需要准备 SD 卡并且将系统烧录到其中，在 MAC 系统中，可以通过 df 命令查看分区情况：

```sh
$ df -lh

Filesystem     Size   Used  Avail Capacity iused               ifree %iused  Mounted on
/dev/disk1s1  234Gi  170Gi   60Gi    75% 5903759 9223372036848872048    0%   /
/dev/disk1s4  234Gi  3.0Gi   60Gi     5%       3 9223372036854775804    0%   /private/var/vm
/dev/disk4s1   15Gi  2.4Mi   15Gi     1%       0                   0  100%   /Volumes/NO NAME

# 可选，格式化磁盘
$ sudo diskutil eraseDisk FAT32 CAM_STORE MBRFormat /dev/disk4s1

# 如果提示 Resource Busy，则可进行卸载
$ diskutil unmount /dev/disk4s1

# 烧写系统
$ sudo dd bs=4m if=rpi_35_v6_1_2_3_jessie_kernel_4_4_50.img of=/dev/disk4s1

# 卸载系统
$ diskutil unmountDisk /dev/disk4s1
```

## WiFi

# GPIO

![](https://imgsa.baidu.com/exp/pic/item/9304c888d43f8794e438169fd51b0ef41ad53a78.jpg)

```py
import RPi.GPIO as GPIO
import time

def setup():
    GPIO.setmode(GPIO.BOARD)
    GPIO.setup(11, GPIO.OUT)
    GPIO.setup(13, GPIO.OUT)
    GPIO.output(11, GPIO.LOW)
    GPIO.output(13, GPIO.LOW)

def destroy():
    GPIO.output(11, GPIO.LOW)
    GPIO.output(13, GPIO.LOW)
    GPIO.setup(11, GPIO.IN)
    GPIO.setup(13, GPIO.IN)

def openLed():
    setup()
    GPIO.output(13, GPIO.HIGH)
    for i in range(2):
        GPIO.output(11,GPIO.HIGH)
        time.sleep(1)
        GPIO.output(11, GPIO.LOW)
        time.sleep(1)
#    destroy()
    GPIO.cleanup()

if __name__=="__main__":
    openLed()
```
