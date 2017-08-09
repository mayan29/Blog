## 第一章 AVFoundation 入门



### 1.1 AV Foundation 简介

AV Foundation 是处理基于时间的视听数据的高级框架。其处于 AVKit 和 Core Audio / Core Media / Core Animation 框架之间。 

![AV Foundation for iOS](https://github.com/Mayan29/ReadingNotes/blob/master/04.《AV%20Foundation%20开发秘籍》读书笔记/DATA/pic00.png)

> #### Core Audio
> Core Audio 由多个框架整合在一起的总称，为音频和 MIDI 内容的录制、播放和处理提供相应接口。Core Audio 也提供高层级的接口，比如通过 Audio Queue Services 框架，处理基本的音频播放和录音功能。同时也提供相对低层级的接口，尤其是 Audio Units 接口，提供了针对音频信号进行完全控制的功能，并通过 Audio Units 构建一些复杂的音频处理模式。如果想详细了解这一框架，建议阅读由 Chirs Adamson 和 Kevin Avila 撰写的 Learning Core Audio 一书。
> 
> #### Core Video
> Core Video 为其相对的 Core Media 提供图片缓存和缓存池支持，提供了一个能够对数字视频逐帧访问的接口。
> 
> ### Core Media 
> Core Media 是 AV Foundation 所用到的低层级媒体管道的一部分。它提供针对音频样本和视频帧处理所需的低层级数据类型和接口。Core Media 还提供了基于 CMTime 数据类型的时基模型。
> 
> ### Core Animation
> Core Animation 是合成及动画相关框架，主要功能就是提供美观、流程的动画效果。使用 Core Animation 对于视频内容的播放和视频捕获，AV Foundation 提供了硬件加速机制来对整个流程进行优化。AV Foundation 还可以利用 Core Animation 让开发者能够在视频编辑和播放过程中添加动画标题和图片效果。



### 1.2 AV Foundation 解析

AV Foundation 框架包含 100 多个类，如果将其按照功能单元进行分解，就会变得比较容易理解：

#### 1.2.1 音频播放记录

上图中，AV Foundation 方框右上角有一个小方格被单独标记为音频专用类，这是由 AV Foundation 提供的关于音频处理的一些早期功能。AVAudioPlayer 和 AVAudioRecorder 可以提供一种更简单的整合音频播放和记录的功能，但是并不是 AV Foundation 用于播放和记录音频的唯一方式，却是最容易学习、功能最强大的方法。

#### 1.2.2 媒体文件检查

AV Foundation 提供检查正在使用的媒体文件的功能，比如是否可以用于回放或者是否可以被编辑和导出，还可以获取内容持续时间，创建日期，首选播放音量。此外，该框架还基于 AVMetadataItem 类提供强大的元数据支持，允许开发者读写关于媒体资源的描述信息，比如唱片簿和艺术家信息。

#### 1.2.3 视频播放

这是 AV Foundation 提供的最常用的功能，这一部分的核心类是 AVPlayer 和 AVPlayerItem。

#### 1.2.4 媒体捕捉

AV Foundation 提供了丰富的 API 集可以对摄像头等设备进行精密控制。摄像头捕捉的核心类是 AVCaptureSession，其作为所有活动的汇集点来接收摄像头设备由各路流发过来的视频和图片。

#### 1.2.5 媒体编辑

AV Foundation 可以将多个音频和视频资源进行组合，允许修改和编辑独立的媒体片段、随时修改音频文件的参数以及添加动画标题和场景切换效果。

#### 1.2.6 媒体处理

使用 AVAssetReader 和 AVAssetWritere 类来执行更高级的媒体处理任务，这些类可以直接访问视频帧和音频样本。



### 1.3 数字媒体压缩

现实生活中，看到的和听到的通过模拟信号传递给我们，信号的频率和强度是在不断变化的。但是数字世界的信号是离散的，由 1 和 0 组成。将模拟信号转换成我们能够存储并传输的数字信号，要进过模数转换，我们将这个过程称为采样。

视频文件 1 秒钟所能展现的帧数成为视频的帧率，并用 FPS 作为单位进行测量。常见的帧率是 24 FPS、25 FPS、30 FPS。

#### 1.3.1 色彩二次抽样

与我们熟知的 RGB 类似，YUV 也是一种颜色编码方法，主要用于电视系统和模拟视频领域，将亮度 Y 和色度 UV 分离。图片所有细节都保存在 Y 通道中，如果除去 Y 信息，剩下的就是一幅灰度图片；如果除去 UV 信息，则变成黑白影像。因为我们的眼睛对亮度的敏感度要高于颜色，所以我们可以大幅减少存储在每个像素中的颜色信息，不至于图片的质量严重受损，这个减少颜色数据的过程就称为色彩二次抽样。

YUV 主流的采样方式有三种：4:4:4、4:1:1、4:2:2、4:2:0，这些值就是设备所使用的色彩二次抽样的参数。iPhone 摄像头通常以 4:2:0 方式进行拍摄。

- 4:4:4  水平和垂直方向，YUV 数量相同
- 4:1:1  水平方向每四个点共用一个 UV 数据，垂直方向 YUV 数量相同
- 4:2:2  水平方向每两个点共用一个 UV 数据，垂直方向 YUV 数量相同
- 4:2:0  水平方向和垂直方向，每两个点共用一个 UV 数据

#### 1.3.2 有损压缩

人类可以听到的音频范围是 20 Hz - 20000 Hz，但是我们真正敏感的频率区间是 1 kHz - 5 kHz，使用过滤技术来减少或消除特定频率，从而减少媒体内容中的冗余数据。

#### 1.3.3 视频编解码器

AV Foundation 提供有限的编解码器集合，主要归结为 H.264 和 Apple ProRes

##### H.264

H.264 规范是 MPEG（Motion Picture Experts Group，运动图像专家组）所定义的 MPEG-4 的一部分，遵循早期的 MPEG-1 和 MPEG-2 标准，用于流媒体文件和移动设备及视频摄像头。

H.264 和其他形式的 MPEG 压缩一样，通过空间（帧内压缩）和时间（帧间压缩）两个维度缩小视频文件的尺寸。

> 帧内压缩：通过消除视频帧内的色彩和结构中的冗余信息来进行压缩，在不降低图片质量的情况下尽可能缩小尺寸，同 JEPG 压缩原理类似，通过这一过程创建的帧称为 I-frames
>
> 帧间压缩：很多帧组合在一起作为一组图片（简称 GOP），对于 GOP 所存在的时间维度的冗余可以被消除。比如行驶的汽车，背景环境通常是固定的，就代表一个时间维度上的冗余，可以通过压缩方式消除。
>
> - I 帧，关键帧。包含创建完整图片需要的所有数据，每组都会有一个 I 帧。由于它是一个独立帧，其尺寸最大，但是解压最快；
> - P 帧，差别帧。表示这一帧和之前的 I 帧（或 P 帧）的差别，解码时需要用之前缓存的画面叠加上本帧定义的差别，生成最终画面；
> - B 帧，双向帧。记录本帧和前后帧的差别。不仅要取得之前的缓存画面，还有解码之后的画面，通过前后画面与本帧数据的叠加取得最终的画面。几乎不需要存储空间，但是解压比较耗时，因为它依赖于周围其他的帧。

H.264 一共有三种压缩标准，Baseline、Main、High

> - Baseline：提供最低效的压缩，因此压缩后文件仍较大。只支持 I/P 帧，一般用于视频通话、手机视频；
> - Main：计算强度比 Baseline 高，可以达到比较高的压缩率。支持 I/P/B 帧，用于 mp4、PSP、iPod；
> - High：高标准得到最高质量的压缩效果。计算复杂度最高。用于蓝光影片、高清电视。

##### Apple ProRes

Apple ProRes 被认为是一个中间件或中间层编解码器，其独立于帧的，意味着只有 I 帧可以被使用，其更适用于内容编辑上。但是 ProRes 编解码器只在 MAC OS 上可用，iOS 只能使用 H.264

ProRes 是有损编解码器，但是它具有最高的编解码质量。Apple ProRes 422 使用 4:2:2 色彩二次抽样和 10 位的采样深度。Apple ProRes 4444 使用 4:4:4:4 色彩二次抽样，4 个表示支持无损 alpha 通道和高达 12 位的采样深度。

#### 1.3.4 音频编解码器

AAC（高级音频编码）是 H.264 标准相应的音频处理方式，目前是最主流的编码方式。这种格式比 MP3 格式有着显著的提升，可以在低比特率的前提下提供更高质量的音频，是 Web 上发布和传播中最理想的音频格式。而且相对 MP3 来说，AAC 没有来自证书和许可方面的限制。

> 注意：AV Foundation 和 Core Audio 支持 MP3 数据解码，但是不支持编码。



### 1.3 文字转语音

```objc
#import <AVFoundation/AVFoundation.h>

// 语音表达
AVSpeechUtterance *utterance = [AVSpeechUtterance speechUtteranceWithString:@"哎呦我擦。你吓死我了。"];
utterance.voice              = [AVSpeechSynthesisVoice voiceWithLanguage:@"zh-CH"];  // 语言
utterance.rate               = 0.4f;  // 语速（0 ~ 1）
utterance.pitchMultiplier    = 1.0f;  // 音调（0.5 ~ 2.0）
utterance.postUtteranceDelay = 0.1f;  // 播放下一句有短暂的暂停
 
// 语音合成器
AVSpeechSynthesizer *synthesizer = [[AVSpeechSynthesizer alloc] init];
[synthesizer speakUtterance:utterance];
```

