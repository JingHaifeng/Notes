# iOS 系统架构

由上至下，操作系统分层
|macOS|iOS|
|:---|:---|
|Cocoa|Cocoa Touch|
|Media|Media|
|Core Service|Core Service|
|Core OS|Core OS|

## Core OS

Core OS 是使用 FreeBSD 和 Mach 改写的 Darwin，提供了系统需要的最基础功能。
> 开源、符合POSIX标准的一个Unix核心

* 硬件驱动
* 内存管理
* 程序管理
* 线程管理（POSIX）
* 文件系统
* 网络(BSD Socket)
* 安全模块
* 电源管理
* 标准输入输出

部分的系统底层功能，应用层可通过 LibSystem 库来访问，如下接口集：

* 线程(POSIX)
* 网络(BSD Socket)
* 文件系统访问
* 标准 I/O
* Bonjour 和 DNS 服务
* Local Information 本地化信息
* 内存分配
* 数字计算

> 许多 Core OS 技术的头文件位于 /usr/include/，是 iPhone SDK 安装目录。

## Core Service

这一层在 Core OS 之上，提供了更加丰富的功能，包含：Foundation.Framework 和 Core Foundation.Framework 等。Foundation 提供了一系列如 字符串处理，排列组合，日历时间等基础功能。

*Foundation是属于Objective-C的API，Core Fundation是属于C的API*

此外还有比较重要的框架：

### 电话本
电话本框架（AddressBook.framework）提供了保存在手机设备中的电话本编程接口。开发者能使用该框架访问和修改存储在用户联系 人数据库里的记录。例如，一个聊天程序可以使用该框架获得可能的联系人列表，启动聊天的进程（Process），并在视图上显示这些联系人信息等。

### 核心基础框架
核心基础框架（CoreFoundation.framework）是基于C语言的接口集，提供iPhone应用的基本数据管理和服务功能。该框架 支持如下功能：

* Collection数据类型（Arrays、 Sets等）
* Bundles
* 字符串管理
* 日期和时间管理
* 原始数据块管理
* 首选项管理
* URL和Stream操作
* 线程和运行循环（Run Loops）
* 端口和Socket通信。

>核心基础框架与基础框架是紧密相关的，它们为相同的基本功能提供了Objective-C接口。如果开发者混合使用Foundation Objects 和Core Foundation类型，就能充分利用存在两个框架中的"toll-free bridging"。toll-free bridging意味着开发者能使用这两个框架中的任何一个的核心基础和基础类型，例如Collection和字符串类型等。每个框架中的类和数据类型的 描述注明该对象是否支持toll-free bridged。如果是，它与哪个对象桥接（toll-free bridged）。

### CFNetwork
 CFNetwork框架（CFNetwork.framework）是一组高性能的C语言接口集，提供网络协议的面向对象的抽象。开发者可以使用 CFNetwork框架操作协议栈，并且可以访问低层的结构如BSD Sockets等。同时，开发者也能简化与FTP和HTTP服务器的通信，或解析DNS等任务。使用CFNetwork框架实现的任务如下所示：

* BSD Sockets
* 利用SSL或TLS创建加密连接
* 解析DNS Hosts
* 解析HTTP协议，鉴别HTTP和HTTPS服务器
* 在FTP服务器工作
* 发布、解析和浏览Bonjour服务。

### 核心位置框架（Core Location Framework）
核心位置框架（CoreLocation.framework）主要获得手机设备当前的经纬度，核心位置框架利用附近的GPS、蜂窝基站或 Wi-Fi 信号信息测量用户的当前位置。iPhone 地图应用使用这个功能在地图上显示用户的当前位置。开发者能融合这个技术到自己的应用中，给用户提供一些位 置信息服务。例如可以提供一个服务：基于用户的当前位置，查找附近的餐馆、商店或设备等。

### 安全框架（Security Framework）
iPhone OS除了内置的安全特性外，还提供了外部安全框架（Security.framework），从而确保应用数据的安全性。该框架提供了管理证书、公钥/私 钥对和信任策略等的接口。它支持产生加密安全的伪随机数，也支持保存在密钥链的证书和密钥。对于用户敏感的数据，它是安全的知识库（Secure Repository）。CommonCrypto接口也支持对称加密、HMAC和数据摘要。在iPhone OS里没有OpenSSL库，但是数据摘要提供的功能在本质上与OpenSSL库提供的功能是一致的。

### SQLite
iPhone应用中可以嵌入一个小型SQL数据库SQLite，而不需要在远端运行另一个数据库服务器。开发者可以创建本地数据库文件，并管理这些 文件中的表格和记录。数据库SQLite为通用的目的而设计，但仍可以优化为快速访问数据库记录。访问数据库SQLite的头文件位 于\<iPhoneSDK>/usr/include/sqlite3.h，其中\<iPhoneSDK>是SDK安装的目标路径。

### 支持XML
基础框架提供NSXMLParser类，解析XML文档元素。libXML2库提供操作XML内容的功能，这个开放源代码的库可以快速解析和编辑 XML数据，并且转换XML内容到HTML。访问libXML2库的头文件位于目录\<iPhoneSDK>/usr/include /libxml2/，其中\<iPhoneSDK>是SDK安装的目标目录。

## 媒体层

### 图像技术（Graphics Technologies）
高质量图像是所有iPhone应用的一个重要的组成部分。任何时候，开发者可以采用UIKit 框架中已有的视图和功能以及预定义的图像来开发iPhone应用。然而，当UIKit 框架中的视图和功能不能满足需求时，开发者可以应用下面描述的技术和方法来制作视图。

#### 1.Quartz
核心图像框架（CoreGraphics.framework）包含了Quartz 2D画图API，Quartz与在Mac OS中采用的矢量图画引擎是一样先进的。Quartz支持基于路径（Path-based）画图、抗混淆（Anti-aliased）重载、梯度 （Gradients）、图像（Images）、颜色（Colors）、坐标空间转换（Coordinate-space Transformations）、pdf文档创建、显示和解析。虽然API是基于C语言的，它采用基于对象的抽象表征基础画图对象，使得图像内容易于保存和复用。

#### 2.核心动画（Core Animation）
Quartz核心框架（QuartzCore.framework）包含CoreAnimation接口，Core Animation是一种高级动画和合成技术，它用优化的重载路径（Rendering Path）实现复杂的动画和虚拟效果。它用一种高层的Objective-C接口配置动画和效果，然后重载在硬件上获得较好的性能。Core Animation集成到iPhone OS 的许多部分，包括UIKit类如UIView，提供许多标准系统行为的动画。开发者也能利用这个框架中的Objective-C接口创建客户化的动画。

#### 3.OpenGL ES
OpenGL ES框架（OpenGLES.framework）符合OpenGL ES v1.1规范，它提供了一种绘画2D和3D内容的工具。OpenGL ES 框架是基于Ｃ语言的框架，与硬件设备紧密相关，为全屏游戏类应用提供高帧率（high frame rates）。开发者总是要使用OpenGL框架的EAGL接口，EAGL接口是OpenGL ES框架的一部分，它提供了应用的OpenGL ES画图代码和本地窗口对象的接口。
### 音频技术（Audio Technologies）

#### 1.核心音频（Core Audio Family）
核心音频框架家族（Core Audio family of frameworks）提供了音频的本地支持，如表16-1所示。Core Audio是一个基于C语言的接口，并支持立体声（Stereo Audio）。开发能采用iPhone OS 的Core Audio框架在iPhone 应用中产生、录制、混合和播放音频。开发者也能通过核心音频访问手机设备的振动功能。

|Framework|Service|
|---|---|
|CoreAudio.framework|定义核心音频的音频数据类型|
|AudioUnit.framework|提供音频和流媒体文件的回放和录制，并且管理音频文件和播放提示声音|
|AudioToolbox.framework|提供使用内置音频单元服务，音频处理模块|

#### 2.OpenAL
iPhone OS 也支持开放音频库（Open Audio Library， OpenAL）。OpenAL是一个跨平台的标准，它能传递位置音频（Positional Audio）。开发者能应用OpenAL在需要位置音频输出的游戏或其他应用中实现高性能、高质量的音频。
由于OpenAL是一个跨平台的标准，采用OpenAL的代码模块可以平滑地移植到其他平台。

### 视频技术（Video Technologies）
iPhone OS通过媒体播放框架（MediaPlayer.framework）支持全屏视频回放。媒体播放框架支持的视频文件格式包括.mov, .mp4,.m4v和.3gp，并应用如下压缩标准：
* H.264 Baseline Profile Level 3.0 video，在30 f/s 的情况下分辨率达到640×480像素。注意：不支持B frames；
* MPEG4规范的视频部分；
* 众多的音频格式，包含在音频技术的列表里，如AAC、Apple Lossless （ALAC）、A-law、IMA/ADPCM（IMA4）、线性PCM、μ-law和Core Audio等。

## Cocoa Touch
最上面一层是Cocoa Touch，它是Objective-C的API, 其中最核心的部分是UIKit.Framework,应用程序界面上的各种组件，全是由它来提供呈现的，除此之外它还负责处理屏幕上的多点触摸事件，文字的输出，图片,网页的显示，相机或文件的存取，以及加速感应的部分等。具体介绍如下:

### UIKit框架
UIKit框架（UIKit.framework）包含Objective-C程序接口，提供实现图形，事件驱动的iPhone应用的关键架构。 iPhone OS中的每一个应用采用这个框架实现如下核心功能：
* 应用管理
* 支持图形和窗口
* 支持触摸事件处理
* 用户接口管理
* 提供用来表征标准系统视图和控件的对
* 支持文本和Web内容
* 通过URL scheme与其他应用的集成

特殊功能
* 加速计数据
* 内建Camera
* 用户图片库

### 基础框架（Foundation Framework）
* Collection数据类型（包括Arrays、Sets）
* Bundles
* 字符串管理
* 日期和时间管理
* 原始数据块管理
* 首选项管理
* 线程和循环
* URL和Stream处理
* Bonjour
* 通信端口管理
* 国际化

### 电话本UI框架（Address Book UI Framework）
电话本UI框架（AddressBookUI.framework）是一个Objective-C标准程序接口，主要用来创建新联系人，编辑和选择 电话本中存在的联系人。它简化了在iPhone应用中显示联系人信息，并确保所有应用使用相同的程序接口，保证应用在不同平台的一致性。