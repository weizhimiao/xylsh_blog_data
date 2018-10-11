---
title: 浅谈Web客户端追踪[转]
date: 2017-03-29 20:30:00
tags:
- 客户端追踪
categories:
- Web
---
随着互联网络的广泛普及，数以亿计网民的网络行为数据早已成为最宝贵的资源，企业通过五花八门的各种手段了解网民的行为和隐私数据，用于广告投递、用户兴趣分析等，进而作为决策的依据。利用Web客户端对用户行为进行收集和追踪是重要手段之一，文本对主流的Web客户端追踪技术进行了简要分析，并给出相关参考供感兴趣的朋友深入，不喜之处还望大神勿喷。

<!-- more -->

## Web客户端追踪技术概述

Web客户端追踪，主要是指用户使用客户端（通常是指浏览器）访问Web网站时，Web服务器通过一系列手段对用户客户端进行标记和识别，进而关联和分析用户行为的技术。

实际上，只要通过Web进入互联网的海洋，Web客户端追踪几乎无时不刻不在发生。

当你网购时，即便没有登录，关掉浏览器后购物车的物品也不会消失；当你访问其他新闻、娱乐网站时，弹出的广告往往都是近期浏览购物网站的类似商品；稍有意识的用户可能会不定时清空浏览器缓存、使用“无痕浏览”、“隐私保护模式”等，然而仍然不能阻止类似广告的洗脑。

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/CCC2C265-F4A1-4D0E-AEE7-3DAC87997A54.png)</center>

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/4809C3B6-4DAD-49A8-9AE5-3B59C15E44A7.png)</center>

现实世界可通过体貌特征、身份证件、生物特征（如指纹信息）等手段对用户进行唯一性识别，Web世界主要通过Cookies、客户端指纹等技术进行识别。

## 典型追踪技术

### 1. Cookie追踪

#### 1)  Cookie简介

Cookie，中文翻译为小甜饼，有时也用复数形式Cookies，在Web世界中其实际上是用户浏览网站时，网站存储在用户浏览器上的一段信息，并在服务器和浏览器之间传递，用户与辨别用户身份和维持状态。

通常是以cookies:user@domain格式命名的，user是你的本地用户名，domain是所访问的网站的域名。在现有Windows系统中，一般存放位置在

`C:\Users\user\AppData\Local\Microsoft\Windows\TemporaryInternet Files\`文件夹下。

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/90ADED7C-F517-4460-AAF4-4052AF293B63.png)</center>


以添加购物车为例，Cookies的大致利用过程可表示为：
- ①、用户第一次访问购物网站：

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/5FD30CF8-5A10-454A-924A-0E51431A95FB.png)</center>

- ②、用户第二次访问网站：

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/1F01FC18-7FAB-4DE7-88DF-4BCBDE1F5979.png)</center>

- ③、浏览器查看Cookies如下：

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/909A1F2B-3584-49CA-9446-8BD5E9685565.png)</center>


#### 2)  Evercookie

用户可以通过清空浏览器缓存等方式，清除已保存的Cookie，Evercookie将Cookie通过多种机制保存到系统多个地方，如果用户删除其中某几处的Cookie， Evercookie仍然可以恢复Cookie，如果开启本地共享对象(Local Shared Objects)，Evercookie甚至可以跨浏览器传播（详见参考地址[1]）。

主要的存储机制如下图，开源地址：

https://github.com/samyk/evercookie

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/3D9E61BC-FFE2-4FDF-8755-2BC7F293ABD4.png)</center>

#### 3)  Cookie同步

Cookie同步是指用户访问某A网站时，该网站通过页面跳转等方式将用户的Cookie发送到B网站，使得B网站获取到用户在A网站的用户隐私信息，然后通过Ad Network等一系列平台进行有效的广告推送服务。

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/81117524-6CBE-4EBF-B2E8-F6CFDED2C069.png)</center>

研究人员通过访问了Alexa排名前1500网站，发现两个追踪者进行Cookie同步以后，可以把数据完全共享，就像是一个追踪者一样。（详见参考地址[2]）

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/13E3C6AC-ACC4-4D12-B33E-F14FF233F9D7.png)</center>

Cookie越来越受限制，不少安全工具甚至是浏览器都允许或者引导关闭追踪Cookie，浏览器指纹追踪渐渐成为了Web追踪的重要技术手段。

### 2. 浏览器指纹

类似人的外貌和指纹，Web客户端（这里主要指浏览器）也有多种“外貌”信息和“指纹”信息，将这些信息综合分析计算后，可对客户端唯一性识别，进而追踪、了解网民行为和隐私数据。

#### 1)  基本指纹

基本指纹是任何浏览器都具有的特征标识，比如硬件类型（Apple）、操作系统（Mac OS）、用户代理（Useragent）、系统字体、语言、屏幕分辨率、浏览器插件 (Flash,Silverlight, Java, etc)、浏览器扩展、浏览器设置(Do-Not-Track, etc)、时区差（BrowserGMT Offset）等众多信息。

可以在该网址进行查看测试，https://www.whatismybrowser.com/

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/761F97FB-C31C-49ED-91F9-0361F0DF4A26.png)</center>

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/BE609BB4-53EE-4115-BA93-40CD9C243972.png)</center>

#### 2)  高级指纹

基本指纹就像是人的外貌特征，外貌可以用男女、身高、体重之分，然而这些特征不能对某个人进行唯一性标识。基于HTML5的诸多高级指纹对此提供了新思路。

- ①、Canvas指纹

Canvas（画布），是HTML5中一种动态绘图的标签，可以使用其生成高级图片，官网有众多绘画事例，如下图。

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/AAC806FA-2B19-4735-932F-80D47366E009.png)</center>

2014年9月，ProPublica报道：新型的Canvas指纹追踪正在被用到“上到白宫，下到YouPorn”等众多网站，众多重要网站都部署了Canvas指纹追踪。

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/4EFB75A4-9BF9-4E43-A9B4-D0F17BC3984D.png)</center>

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/BD836DFD-178E-446F-A71F-C2DF37A2DD15.png)</center>

利用Canvas进行追踪的过程大致如下：

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/B8554485-29B1-4B8A-89AB-8550371CE09A.png)</center>

基于Canvas绘制特定内容的图片，使用canvas.toDataURL()方法获得图片内容的base64编码（对于PNG格式的图片，以块(chunk)划分，最后一块是32位CRC校验）作为唯一性标识，如下图。

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/1BDA6AFA-B37C-400E-AE2F-71B13F76E4F9.png)</center>

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/7B8514B0-881B-4880-B297-918CB3B72A4C.png)</center>

***Canvas指纹的原理大致如下：***

相同的HTML5Canvas元素绘制操作，在不同操作系统、不同浏览器上，产生的图片内容不完全相同。

在图片格式上，不同浏览器使用了不同的图形处理引擎、不同的图片导出选项、不同的默认压缩级别等。

在像素级别来看，操作系统各自使用了不同的设置和算法来进行抗锯齿和子像素渲染操作。即使相同的绘图操作，产生的图片数据的CRC检验也不相同。

在线测试地址：

https://www.browserleaks.com/canvas

可查看浏览器的Canvas唯一性字符串。

- ②、AudioContext指纹

HTML5提供给JavaScript编程用的AudioAPI则让开发者有能力在代码中直接操作原始的音频流数据，对其进行任意生成、加工、再造，诸如提高音色，改变音调，音频分割等多种操作，甚至可称为网页版的Adobe Audition。

***AudioContext指纹原理大致如下：***

方法一：生成音频信息流(三角波)，对其进行FFT变换，计算SHA值作为指纹，音频输出到音频设备之前进行清除，用户毫无察觉。

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/38DBB916-AAF4-42E1-B7B4-11A3F9DC3F39.png)</center>


方法二：生成音频信息流（正弦波），进行动态压缩处理，计算MD5值。

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/038BF410-893D-4D20-B62A-24FECB62A794.png)</center>


AudioContext指纹基本原理：
主机或浏览器硬件或软件的细微差别，导致音频信号的处理上的差异，相同器上的同款浏览器产生相同的音频输出，不同机器或不同浏览器产生的音频输出会存在差异。

从上可以看出AudioContext和Canvas指纹原理很类似，都是利用硬件或软件的差异，前者生成音频，后者生成图片，然后计算得到不同哈希值来作为标识。

音频指纹测试地址：https://audiofingerprint.openwpm.com/

#### 3)  硬件指纹

硬件指纹主要通过检测硬件模块获取信息，作为对基于软件的指纹的补充，主要的硬件模块有：GPU’sclock frequency、Camera、Speakers/Microphone、Motion sensors、GPS、Battery等。

更多细节请参考：https://arxiv.org/pdf/1503.01408v3.pdf

#### 4)  综合指纹

Web世界的指纹碰撞不可避免，将上述基本指纹和高级指纹综合起来，计算哈希值作为综合指纹，可以大大降低碰撞率。

测试地址：https://panopticlick.eff.org/

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/2959BA38-D4E7-4E17-BDE4-3A562ABB1D21.png)</center>

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/C748D9A2-26B7-4205-9A39-652AE9DAD763.png)</center>

### 3. 跨浏览器指纹

上述指纹都是基于浏览器进行的，同一台电脑的不同浏览器具有不同的指纹信息。

这样造成的结果是，当同一用户使用同一台电脑的不同浏览器时，服务方收集到的浏览器指纹信息不同，无法将该用户进行唯一性识别，进而无法有效分析改用户的的行为。

近期有学者研究了一种跨浏览器的浏览器指纹，其依赖于浏览器与操作系统和硬件底层进行交互进而分析计算出指纹，这种指纹对于同一台电脑的不同浏览器也是相同的。

更多技术细节请参考：

http://yinzhicao.org/TrackingFree/crossbrowsertracking_NDSS17.pdf

### 4. WebRTC

WebRTC（网页实时通信，Web Real Time Communication），是一个支持网页浏览器进行实时语音对话或视频对话的API，功能是让浏览器实时获取和交换视频、音频和数据。基于WebRTC可以实现浏览器上，通过Javascript就可以达到实时通讯的能力。

基于WebRTC的实时通讯功能，可以获取客户端的IP地址，包括本地内网地址和公网地址。其原理是利用到RTCPeerConnection的API，大致函数如下：

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/369C0E01-4122-4B75-BCAA-00C6026C1378.png)</center>

<center>![img](http://n.sinaimg.cn/games/3ece443e/20170329/701F38B8-C7A0-49AE-8676-6053F8F0EF4D.png)</center>


利用WebRTC能做的事情还远不止这些，比如使用其探测扫描内网信息，进行语音、视频交流，更多技术细节请参考：

http://net.ipcalf.com

https://diafygi.github.io/webrtc-ips/

## 防客户端追踪措施

### 1. 浏览器设置

- ①、使用隐身模式，目前主流的浏览器都支持该模式。

- ②、禁用Cookie和JavaScript（此项可能导致页面显示不正常，慎用）

- ③、禁用WebRTC，如Firefox浏览器：打开about:config，找到media.peerconnection.enabled的项，设置成 false
- ④、禁用Geolocation，Firefox浏览器：打开about:config，找到geo.enabled的值，设置其值为 false。Chrome 点击设置（Settings），从显示高级设置（Show advanced settings）上，找到隐私（Privacy）并且点击内容设置（Content settings），在窗口里找到定位（Location）并设置选项不允许任何网站追踪你的物理位置（Do not allow any site to track your physical location）

- ⑤、限制API访问文件资源时序信息，恶意网站会通过检测浏览器缓存的时序信息，包括访问和忽略第三方网站的资源，来判断使用者是否访问过第三方网站。Firefox浏览器：打开about:config，将dom.enable_resource_timing, dom.enable_user_timing 和dom.performance.enable_user_timing_logging设置为 false，来阻止这些 API 运行。

### 2. 插件

推荐几个较好的插件来阻止第三方广告追踪和广告：
Ghostery，官网地址：
https://www.ghostery.com/try-us/download-browser-extension

Privacy Badger，官网地址：
https://www.eff.org/privacybadger/

uMatrix（仅Chrome和FireFox）：
https://addons.mozilla.org/en-us/firefox/addon/umatrix/

NoScript（仅FireFox）：
https://addons.mozilla.org/en-US/firefox/addon/noscript/#

Chameleon（仅Chrome）：
https://github.com/ghostwords/chameleon


## 参考资料

[1] http://samy.pl/evercookie/

[2] http://freedom-to-tinker.com/2014/08/07/the-hidden-perils-of-cookie-syncing/

[3] https://securehomes.esat.kuleuven.be/~gacar/persistent/index.html

[4] http://cseweb.ucsd.edu/~hovav/papers/ms12.html

[5] https://arxiv.org/pdf/1503.01408v3.pdf

[6] https://eprint.iacr.org/2015/616.pdf

> 作者：ArkTeam/Wellee
>
> 来源：FreeBuf
