#  OpenBSD FAQ - 多媒体

## 启用录音

出于隐私原因，OpenBSD 中默认禁用录音。你可以使用 `kern.audio.record sysctl` 启用该功能。

```
# sysctl kern.audio.record=1
# echo kern.audio.record=1 >> /etc/sysctl.conf
```

## 配置音频硬件

每个声卡型号都有自己的一组控件。有些声卡根本没有控件，其他可能有一百个或更多控件。你可以以 root 身份运行 [mixctl(8)](https://man.openbsd.org/mixerctl) 列出所有控件和当前设置。

并非每个音频芯片的每个选项都必须连接外部世界。例如，列出的输出可能多于实际连接的输出。音频设备的控件可能会有不同的标签。 通常控件有一个有意义的标签，但有时必须简单地尝试不同的设置来查看每个控件的效果。

以下是要考虑的控件列表：

**静音和电平控制** —— 即使主控制似乎设置正确，信号路径中也可能有多个静音或电平控制。在下面的示例中，设备具有主录音电平和麦克风增益控制：

```
record.adc-0:1=248,248
record.adc-0:1_source=mic
inputs.mic=85,85
```

**外部放大器断电 (External Amplifier Power-Down, EAPD)** —— 此开关通常用于笔记本电脑的省电，可能需要设置以获得输出信号：

```
outputs.spkr_eapd=on
```

**录音源** —— 一些设备有多个麦克风输入。此类控制的示例：

```
record.source=mic
record.adc-0:1_source=mic
```

要使更改在每次重新启动时生效，请编辑 /etc/mixerctl.conf 文件。例如：

```
record.adc-0:1_source=mic
inputs.mic=85,85
```

## 调整音频电平

[sndioctl(1)](https://man.openbsd.org/sndioctl) 实用程序用于以普通用户身份操作音频控件。不带参数运行它会列出所有控件和当前设置。

`output.level` 控件始终存在。它要么对应于硬件控制，要么在软件中模拟。

## 使用 USB 音频接口

dmesg 输出中 USB 音频接口的示例可能如下所示：

```
uaudio0 at uhub2 port 1 configuration 1 interface 1 "ABC C-Media USB Audio Device" rev 1.10/1.00 addr 2
uaudio0: class v1, full-speed, sync, channels: 2 play, 1 rec, 8 ctls
audio1 at uaudio0
```

在大多数系统上，第一个音频设备是内部声卡。连接后，USB 音频设备将成为第二个。在默认的 [sndiod(8)](https://man.openbsd.org/sndiod) 配置中，这两个设备分别被编程为 `snd/0`（默认）和 `snd/1`，并且可以独立使用。设置为使用 `snd/1` 的程序将使用 USB 设备。

[sndiod(8)](https://man.openbsd.org/sndiod) 可以配置为使 `snd/0` 对应于 USB 设备（如果它已连接）或未连接时对应于内部设备：

```
# rcctl set sndiod flags -f rsnd/0 -F rsnd/1
# rcctl restart sndiod
```

当服务器打开设备时，它会首先尝试 USB 设备。如果它不存在，则使用内部设备。如果 USB 设备断开连接，sndiod 会尝试使用内部声卡继续操作。如果再次连接 USB 设备，sndiod 将在下次尝试打开设备时看到它。要强制 sndiod 在设备之间切换，请重新加载服务器：

```
# rcctl reload sndiod
```

## 播放音频文件

OpenBSD 带有 [aucat(1)](https://man.openbsd.org/aucat)，这是一个能够播放未压缩的 WAV、AIFF 和 AU 文件的程序。它可以用于非常简单的情况或测试播放：

```
$ aucat -i filename.wav
```

还有许多其他播放器可作为支持其他音频格式的[软件包](https://www.openbsd.org/faq/faq15.html)。

## 录制音频文件

使用 `kern.audio.record` sysctl 启用录音后，[aucat(1)](https://man.openbsd.org/aucat) 可用于录制未压缩的 WAV、AIFF 和 AU 文件。

```
$ aucat -o file.wav
```

上述命令将开始录制 WAV 格式的文件。按 CTRL+C 完成录制。

要播放文件，请运行：

```
$ aucat -i file.wav
```

如果录音似乎可以正常工作，但录音的播放无声或不符合预期，则混音器可能需要进行一些配置。确保您选择了正确的声源进行录音，并且未静音。

如果需要，可以使用 ports 源码树中的适当程序压缩生成的 WAV 文件。或者可以使用诸如 sox、ffmpeg 或 audacity 之类的 port 来录制、处理和压缩音频文件。

## 录制所有音频播放的监听混音

监控流记录来自所有播放设备的组合音频输出，允许你复制或保存通过音频子系统的任何内容。此功能可用于屏幕录像或任何类型的现场音频混合。

使用以下命令为 [sndiod(8)](https://man.openbsd.org/sndiod) 创建监视器子设备 `mon`：

```
# rcctl set sndiod flags -s default -m play,mon -s mon
# rcctl restart sndiod
```

将你的程序配置为从 `snd/0.mon` 设备录制音频，例如：

```
$ aucat -f snd/0.mon -o file.wav
```

此时，你的系统播放的任何内容都记录在 `file.wav` 中。

## 降低音频延迟

延迟是程序决定播放样本与用户听到样本之间的时间。由于音频数据总是被缓冲，这个延迟与音频缓冲区大小成正比。建议使用以下值：

- 实时合成器：5ms。这是敲击 MIDI 键盘上的键和实际听到音符之间的时间。粗略地说，5ms 对应于声音传播 1.75m 所需的时间。
- 游戏：50ms。这是你看到事件和听到相应声音之间的时间。
- 电影播放器等：500 毫秒及以上。 此类应用程序提前“知道”要播放的声音，并以与相应图片同时播放的方式发送音频数据。

音频缓冲区越小（以实现低延迟），溢出/欠载的可能性就越大。缓冲区溢出/不足会导致声音断断续续。

sndiod(8) 对所有音频应用程序施加了最小延迟，默认延迟为 160 毫秒。如果你计划使用需要较低延迟的应用程序，请使用 -b 选项选择所需的延迟（以帧数表示）。例如，在 48000 个样本/秒（samples/second）时，50 毫秒的延迟对应于：

48000 个样本/秒 × 0.050 秒 = 2400 个样本

然后执行

```
# rcctl set sndiod flags -b2400
```

### 低延迟是否可以改善音视频同步？

不，将音频与视频同步不需要低延迟。同步问题通常是由软件本身引起的。强制应用程序使用较小的缓冲区（通过在低延迟模式下启动 sndiod(8)）可能会在某些情况下隐藏实际问题，并给人一种软件运行更好的感觉，但显然正确的做法是开始搜索相应的错误。

## 使用远程音频硬件

sndiod(8) 可以配置为接受来自网络的连接，允许其他机器也使用声卡。在带有声卡的远程系统上，运行：

```
# rcctl set sndiod flags -L-
```

在本地系统上，将您的程序配置为使用 `snd@hostname/0`，其中“主机名（hostname）”是远程系统的地址。`AUDIODEVICE` 环境变量可以设置为上述值，使远程声卡成为默认的音频设备。

任何能够连接到远程主机的 TCP 端口 11025 的系统都可以使用音频设备。 出于隐私原因，在给定时间，一个系统中只有一个用户可以连接到该系统。 如果多个系统必须同时使用音频设备，则 sndio(7) 授权 cookie 必须相同。例如，将你的 `~/.sndio/cookie` 复制到可能使用音频设备的所有帐户。

为避免故障，端口 11025 上的 TCP 流量可以使用数据包过滤器进行优先级排序。使用默认配置时，sndiod 将消耗大约 200kB/s 的网络带宽。

## 选择默认音频设备

默认音频设备的选择由 `AUDIODEVICE` 环境变量决定。如果未设置，则默认使用 `snd/0`，它是 [sndiod(8)](https://man.openbsd.org/sndiod) 管理的第一个音频设备。 选择默认设备的最灵活方法是导出可能在用户的登录配置文件中的 `AUDIODEVICE`。

另一种更改默认音频输出设备的方法是使所需设备成为 [sndiod(8)](https://man.openbsd.org/sndiod) 管理的第一个设备。例如，要使用外部 DAC 而不是主板的板载音频，只需更改 [sndiod(8)](https://man.openbsd.org/sndiod) 的启动标志以使用该设备：

```
# rcctl set sndiod flags -f rsnd/1
# rcctl restart sndiod
```

这将使第二个音频设备 (`rsnd/1`) 成为默认设备。

## 调试音频问题

如果你在播放音频时没有听到任何声音，则可能是调音台控制调低音量或只是静音。请参阅[本节](https://www.openbsd.org/faq/faq13.html#confaudio)以配置混音器。在报告问题之前，请取消**所有**输入和输出的静音。

如果你认为你的设备应该可以正常工作，但无论出于何种原因不能正常工作，那么是时候进行一些调试了。以下步骤可以确定 DAC 是否正在处理数据。

```
# cat > /dev/audio0 < /dev/zero &
[1] 9926
# audioctl play.{bytes,errors}
play.bytes=3312000
play.errors=0
# audioctl play.{bytes,errors}
play.bytes=7065600
play.errors=0
# audioctl play.{bytes,errors}
play.bytes=9379200
play.errors=0
# kill %1
# fg %1
cat > /dev/audio0 < /dev/zero
Terminated
```

在这里，我们看到每次检查时，处理的数据计数 `play.bytes` 都在增加，所以数据在流动。我们还看到，设备没有少运行任何样本（`play.errors`）。这也是好事。

请注意，即使你在运行上述测试时插入了扬声器，你也不应该听到任何声音。 该测试向设备发送 “zero” 信号，这对于所有当前支持的默认编码是静音的。

由于我们知道设备可以处理数据，因此最好再次检查混音器设置。 确保所有输出和所有输入都未静音且处于合理水平。

如果此时你仍有问题，可能是时候[提交错误报告](https://www.openbsd.org/report.html)了。除了完整的 dmesg 和问题描述等正常的错误报告信息外，还请包括 `mixerctl -v` 的默认输出以及上述 DAC 处理测试的输出。

## 使用 MIDI 乐器

乐器数字接口 (MIDI) 协议提供了一种标准化且有效的方法，可以将音乐表演信息表示为电子数据。MIDI 数据仅包含合成器播放声音所需的指令，而不是实际的声音。

要播放 MIDI 数据，需要一个连接到机器的 MIDI 端口的合成器。同样，要录制 MIDI 数据，需要 MIDI 乐器（例如 MIDI 键盘）。高级 MIDI 乐器可能包含多个子部分（合成器、键盘、控制面等...）。它们在 OpenBSD 上显示为多个 MIDI 端口。

当您已经运行 OpenBSD 时，在 dmesg(8) 命令的输出中查找 MIDI 端口。dmesg 输出中的 MIDI 端口示例是：

```
umidi0 at uhub2 port 2 configuration 1 interface 0 "Roland Roland XV-2020" rev 1.10/1.00 addr 2
midi0 at umidi0: <USB MIDI I/F>
umidi1 at uhub1 port 2 configuration 1 interface 1 "Evolution Electronics Ltd. USB Keystation 61es" rev 1.00/1.25 addr 3
midi1 at umidi1: <USB MIDI I/F>
```
它显示了两个附加的 [midi(4)](https://man.openbsd.org/midi) 驱动程序，程序称为：

- `midi/0` - 通过 USB 连接的合成器
- `midi/1` - MIDI 主键盘

键盘的输出可以连接到合成器的输入，如下所示：

```
$ midicat -q midi/0 -q midi/1
```

现在你可以听到你在合成器上的 MIDI 键盘上演奏的内容。

[sndiod(8)](https://man.openbsd.org/sndiod) 服务器暴露了 MIDI 直通端口，允许程序互相发送 MIDI 数据。例如，如果你没有连接硬件合成器，你可以启动一个软件合成器（如 audio/fluidsynth 端口），然后把它作为 MIDI 输出。

```
$ midicat -q midi/0 -q midithru/0
```

## 使用网络摄像头

### 启用视频录制

出于隐私原因，OpenBSD 中默认禁用视频录制。`kern.video.record` sysctl 可用于启用它。

```
# sysctl kern.video.record=1
# echo kern.video.record=1 >> /etc/sysctl.conf
```

### 受支持的硬件

[uvideo(4)](https://man.openbsd.org/uvideo) 设备驱动程序支持许多遵循 USB 视频类 (UVC) 规范的网络摄像头，并通过 [video(4)](https://man.openbsd.org/video.4) 层向用户公开。

受支持的网络摄像头（或其他视频设备）在 dmesg 中的显示如下所示：

```
uvideo0 at uhub0 port 8 configuration 1 interface 0 "Azurewave Integrated Camera" rev 2.01/69.05 addr 10
video0 at uvideo0
uvideo1 at uhub0 port 8 configuration 1 interface 2 "Azurewave Integrated Camera" rev 2.01/69.05 addr 10
video1 at uvideo1
```

该设备可通过 `/dev/video0` 访问。

一些笔记本电脑还为红外摄像头连接了第二个（无法使用的）视频设备。你可以使用 [video(1)](https://man.openbsd.org/video.1) 命令找到可用的相机设备：

```
$ video -q -f /dev/video0
video device /dev/video0:
  encodings: yuy2
  frame sizes (width x height, in pixels) and rates (in frames per second):
        320x180: 30
        320x240: 30
        352x288: 30
        424x240: 30
        640x360: 30
        640x480: 30
        848x480: 20
        960x540: 15
        1280x720: 10
  controls: brightness, contrast, saturation, hue, gamma, sharpness, white_balance_temperature
$ video -q -f /dev/video1
video: /dev/video1 has no usable YUV encodings
```

默认只允许 root 访问视频设备。必须更改设备的权限才能将其作为普通用户使用：

```
# chown $USER /dev/video0
```

### 录制网络摄像头流

本节使用 `graphics/ffmpeg` 包中的 `ffplay` 和 `ffmpeg`。要查看给定相机的功能，请运行以下命令：

```
$ ffplay -f v4l2 -list_formats all -i /dev/video0
[...]
[video4linux2,v4l2 @ 0x921f8420800] Raw       : yuyv422 : YUYV : 640x480 320x180 320x240 352x288 424x240 640x360 848x480 960x540 1280x720
[video4linux2,v4l2 @ 0x921f8420800] Compressed:   mjpeg : MJPEG : 1280x720 320x180 320x240 352x288 424x240 640x360 640x480 848x480 960x540
```

第一行显示未压缩的 YUYV 格式显示支持的分辨率。这种格式的帧率可能非常低。第二行显示支持的 MJPEG 压缩视频分辨率，它可以提供更高的帧率。

选择其中一种 MJPEG 分辨率并运行以下命令进行测试：

```
$ ffplay -f v4l2 -input_format mjpeg -video_size 1280x720 -i /dev/video0
[...]
Input #0, video4linux2,v4l2, from '/dev/video0':B sq=    0B f=0/0
  Duration: N/A, start: 1599377893.546533, bitrate: N/A
    Stream #0:0: Video: mjpeg (Baseline), yuvj422p(pc, bt470bg/unknown/unknown), 1280x720, 30 fps, 30 tbr, 1000k tbn, 1000k tbc
```

网络摄像头流应与分辨率和帧率一起显示。

如果可行，可以使用 `ffmpeg` 录制视频，如下所示：

```
$ ffmpeg -f v4l2 -input_format mjpeg -video_size 1280x720 -i /dev/video0 ~/video.mkv
```

按 “q” 结束录制。

### 控制网络摄像头设置

网络摄像头通常具有亮度、对比度和其他可使用 [video(1)](https://man.openbsd.org/video.1) 命令更改的控件。

```
$ video -c
brightness=128
contrast=32
saturation=64
hue=0
gamma=120
sharpness=3
white_balance_temperature=auto
```

例如，亮度设置可以更改为 200：

```
$ video brightness=200
brightness: 128 -> 200
```

可以使用 `video -d` 将所有设置恢复为默认值。

如果设置为 “auto” 值，某些设置支持自动调整。

### 在网络浏览器中访问网络摄像头

默认情况下，Chromium 可以访问 `/dev/video`。要允许 Chromium 访问其他视频设备，必须将设备路径添加到 `/etc/chromium/unveil.main` 和 `/etc/chromium/unveil.utility_video`。

默认情况下，Firefox 可以访问 `/dev/video` 和 `/dev/video0`。 要允许 Firefox 访问其他视频设备，必须将设备路径添加到 `/etc/firefox/unveil.main`。