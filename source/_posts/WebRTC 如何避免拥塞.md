---
title: WebRTC 如何避免拥塞？
date: 2018-06-03 19:59:25
tags: WebRTC
---

## WebRTC 如何避免拥塞？

### 1. WebRTC 网络传输背景

#### 1.1. WebRTC && 多数直播

**大多数视频直播平台**

| 子项   | 说明                                                                      |
|:---- |:----------------------------------------------------------------------- |
| 传输协议 | RTMP（Real Time MEssaging Protocol)                                      |
| 优点   | 使用RTMP， 其基于TCP传输，跟flash等流媒体服务支持比较好，同时CDN支持良好                            |
| 缺点   | 直播音视频数据量大，实时性要求比较高，TCP的重传机制和拥塞机制不适用于实时传输。传输控制依赖于TCP本身协议控制机制，网络成本较大不够灵活。 |

音视频对网络丢包有一定程度天然容忍性（常见优酷，爱奇艺都有缓冲）。使用UDP传输是不错到选择。

<!--more-->

**webRTC**

| 子项   | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|:---- |:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 传输协议 | RTP: 针对流媒体传输的基础协议，定义Internet上传输音视频数据包。 <br> RTCP：负责流媒体到传输质量保证，提供流量控制和拥塞控制等机制。![](https://gss1.bdstatic.com/-vo3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike80%2C5%2C5%2C80%2C26/sign=57e79b5fb0de9c82b268f1dd0de8eb6f/4bed2e738bd4b31c42beebdd84d6277f9f2ff8d2.jpg)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| 优点   | 包括ICE、STUN、TURN等信令交互、网络大洞（P2P)，主流浏览器支持（chrome, firefox,opera, safari,edge等）<br> ![](https://gss2.bdstatic.com/-fo3dSag_xI4khGkpoWK1HF6hhy/baike/w%3D268%3Bg%3D0/sign=54395c9081d4b31cf03c93bdbfed4042/2cf5e0fe9925bc3142b4464b54df8db1ca137073.jpg) ![](https://gss0.bdstatic.com/94o3dSag_xI4khGkpoWK1HF6hhy/baike/s%3D220/sign=bb31c6af3812b31bc36cca2bb61a3674/54fbb2fb43166d2287f64f7c472309f79152d251.jpg)![](https://gss0.bdstatic.com/-4o3dSag_xI4khGkpoWK1HF6hhy/baike/w%3D268%3Bg%3D0/sign=1db5f93247c2d562f208d7ebdf2af7d2/f9198618367adab4ee604c1183d4b31c8701e434.jpg)![](https://gss1.bdstatic.com/-vo3dSag_xI4khGkpoWK1HF6hhy/baike/w%3D268%3Bg%3D0/sign=5838ca994410b912bfc1f1f8fbc69b3e/500fd9f9d72a6059b0046c382b34349b033bbaf8.jpg)![](https://gss1.bdstatic.com/9vo3dSag_xI4khGkpoWK1HF6hhy/baike/w%3D268%3Bg%3D0/sign=37f66727093387449cc5287a6934bec4/d53f8794a4c27d1e13c902b31ed5ad6edcc438dd.jpg) |

### 2. WebRTC 处理

#### 2.1. NACK

NACK( negative acknowledge character)， 就是否定应答。与之对应的是TCP中的ACK( acknowledge character)。

> 在TCP中，接收端对于收到的包都要进行应答即发送ACK包，发送端通过接受ACK包，来确定发送的包已经被成功接收，以此来保证网络包的传输可靠。

RTP协议不保证传输的可靠性，所以接收端也就不会发送ACK包。如果这个包比较重要，可以给对端发送NACK包，来告诉这个包我没有收到，你如果‘还有’的话，就给我重新发送一遍（Retransmission）。

前面说到的丢包，都加了引号，这个“丢包”，有可能是真的丢了，也有可能是顺序乱了。我们知道网络包的到达，有时候不是一定严格按照包序到达的，我收到了很多较新的包了，某个旧的包还没有收到，我就认为它丢了，也没准儿一会儿它又到了。怎么判断“丢包”呢？ 我们通过：

1. **乱序的偏移来决定发起重传**

2. **根据预计到达时间已经超过一定时间了来发起重传 (一般是等待一个rtt＋jitter的时长)。**

更多丢包判断参见如下连接：[WebRTC中丢包重传NACK实现分析](https://www.jianshu.com/p/a7f6ec0c9273)

#### 2.2. 前向纠错编码( FEC )

FEC（Forward Error Correction，前向纠错码），通过增加冗余来增强容错性到一种方法。如果没有FEC，接受端发现丢包时，需要通过发送NACK 来发起重传，但重传会影响延时性。

- FEC则是在发送端增加冗余数据，接受端在数据丢失到情况下可以重建丢失的数据。

- 增加FEC冗余数据占据了有效带宽

上面两点涉及到取舍，冗余的数据与减少丢包导致的延时。不过，FEC到冗余度是可以根据网络状况动态调整。

#### 2.3. 抖动缓冲( Jitterbuffer )

网络传输中总是存在着抖动，导致网络包不是均匀顺序到达，WebRTC利用一个缓冲区，进行等待与排序，这就是jitterbuffer。他能动态根据网络情况调整缓冲区的大小。

主流的实时音视频框架基本都会实现jitterbuffer功能，诸如WebRTC、doubango等。WebRTC的jitterbuffer按照功能分类的话，可以分为jitter和buffer。

`jitterbuffer = jitter + buffer`



`buffer`

buffer主要对**丢包、乱序、延时到达等异常**情况进行处理，还会和NACK、FEC（前向纠错码）、FIR(关键帧请求))等QOS（质量服务）相互配合。

![](http://www.ucpaas.com/u/allimg/1706/8-1F602094T0135.jpg)

buffer 主要分为如下几种type：

- freeframes

- incompleteframes

- decodableframes

| type             | remark                                                                                                                                                                                                                                                                         |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| freeframes       | 新到的数据包会根据时间戳在incompleteframes 和decodableframes 中寻找，相同放置到相应队列，否则从freeframes弹出空frame                                                                                                                                                                                             |
| incompleteframes | 至少有一个数据包，帧数据不完整                                                                                                                                                                                                                                                                |
| decodableframes  | 可解码，此队列中有decodable和complete 状态。complete 状态帧可被解码线程取出解码，完成后将buffer 重新push 到freeframes 队列。<br><br> decodable会根据decode\_error\_mode 有不同的规则，QOS的不同策略会设置不同的decode\_error\_mode ，包含kNoErrors、kSelectiveErrors以及kWithErrors。decode\_error\_mode 就决定了解码线程从buffer中取出来的帧是否包含错误，即当前帧是否有丢包。 |

jitterbuffer与QOS策略联系紧密，比如，incompleteframes和decodable队列清除一些frame之后，需要FIR（关键帧请求），根据包序号检测到丢包之后要NACK（丢包重传）等。



`jitter`

网络延迟带来的抖动会让音视频的播放不平稳，如音频的颤音，视频的忽快忽慢。那么如何对抗jitter呢？增加延时。

jitter主要根据当前帧的大小和延时评估出**jitterdelay**，再结合**decode delay、render delay以及音视频同步延时，得到render time**，来控制**平稳的渲染视频帧**。

![](http://www.ucpaas.com/u/allimg/1706/8-1F602094Z4a1.jpg)

其中:

- freeDelayMS: 两笔RTP1, RTP2 之间时间差

- frameSizeBytes: 当前帧数据大小

- incompleteFrame: 是否微完整帧

- UpdateEstimate: 用卡尔曼滤波对帧间延迟进行滤波（具体算法这里不进行讨论）

在得到jitterdelay之后，通过jitterdelay+ decodedelay +renderdelay，再确保大于音视频同步的延时，加上当前系统时间得到rendertime，这样就可以控制播放时间。控制播放，也就间接控制了buffer的大小。

当然，仅仅通过动态的jitterbuffer 是无法完全解决网络拥塞的问题，根本上还是应该调整发送的码率。

#### 2.4. 带宽自适应

带宽自适应是指在音视频的收发过程中，根据网络带宽的变化，自动的来调整发送码率，来适应带宽的变化。在带宽足够的情况下，增加帧率和码率，提高音视频的质量，带来更好的通信体验。在带宽不足的情况下，主动降低码率或者帧率，保证通信的流畅性和可用性，也是带来更好的通信体验。

带宽自适应的核心：**如何准确的估计带宽**。WebRTC在实现带宽自适应时采用了Google提出一个称为REMB（Receiver Estimated Max Bitrate，最大接收带宽估计）的带宽估计算法。

大致算法是：

- 接受端维护状态机（根据丢包率或延时情况）

- 根据丢包率或延时情况修改remb 值

- 将remb 值通过RTCP 发送给发送端

- 发送端根据remb 值调整码率

补充：[码流 / 码率 / 比特率 / 帧速率 / 分辨率 / 高清的区别](https://blog.csdn.net/xiangjai/article/details/44238005)

### 参考资源:

[webrtc 中到网络反馈于控制](https://blog.csdn.net/mantis_1984/article/details/53572822)

[WebRTC视频JitterBuff](https://blog.csdn.net/u012635648/article/details/72953237)
