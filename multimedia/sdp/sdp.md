
# SDP(会话描述协议 Session Description Protocol)

## SDP介绍

SDP是一个用来`描述多媒体会话` 的应用层控制协议，为会话通知、会话邀请和其它形式的 `多媒体会话初始化等目的`提供了多媒体会话描述；它是一个基于文本的协议，这样就能保证协议的可扩展性比较强，这样就使其具有广泛的应用范围。

SDP 完全是一种`会话描述格式`  ― 它不属于传输协议 ― 它只使用不同的适当的传输协议，包括会话通知协议（SAP）、会话初始协议（SIP）、 实时流协议（RTSP）、 MIME 扩展协议的电子邮件以及超文本传输协议（HTTP）。

SDP 不支持会话内容或媒体编码的协商，所以在流媒体中`只用来描述媒体信息`。媒体协商这一块要用 RTSP 来实现。

SDP 文本信息包括：会话名称和意图； 会话持续时间； 构成会话的媒体； 有关接收媒体的信息（地址等）。

## SDP协议结构

SDP协议的信息是文本信息，采用 UTF-8 编 码中的 ISO 10646 字符集。

SDP描述由许多文本行组成，文本行的格式为`<类型>=<值>`，`<类型>`是一个字母，`<值>`是结构化的文本串，其格式依`<类型>`而定。

```txt
<type>=<value>[CRLF]
```

SDP的文本信息包括（*标注\*符号的表示可选字段*）：

* 会话名称和意图描述
```txt
v=(协议版本)
o=(所有者/创建者和会话标识符)
s=(会话名称)
i=*(会话信息)
u=*(URI 描述)
e=*(Email 地址)
p=*(电话号码)
c=*(连接信息 ― 如果包含在所有媒体中，则不需要该字段)
b=*(0个或多个带宽信息)
z=*(时域信息)
```

* 时间描述
```txt
t=(会话活动时间)
r=*(0或多个重复时间)
``` 

* 媒体描述
```txt
m=(媒体名称和传输地址)
i=*(媒体标题)
c=*(连接信息 — 如果包含在会话层则该字段可选)
b=*(带宽信息)
k=*(加密密钥)
a=*(0 个或多个会话属性行)
```


`<type>`为小写字母并且是不可扩展的。“=”两侧不允许有空格。

SDP解析器遇到不认识的`<type>`字段时，必须忽略它。

会话属性（`a=`）是SDP扩展的主要手段。

某些属性是已经被定义好的，而另外的则是在特定的应用、媒体格式规范中才会被定义，因此SDP解析器必须忽略它不能识别的所有属性。

一个SDP会话描述可能包含URI以指定外部的信息，主要在"u="、"k="和"a="这三个字段中体现。这些URI是某些场合下是非联系的，以避免会话描述自包含。

会话层的连接信息("c=")和属性("a=")都是全局的，除非在媒体层被重写。

SDP可能在"o="、"u="、"e="、 "c="和"a="等字段包含域名。

## SDP各TYPE字段详细解释

### 协议版本("v=")
```txt
v=0
```
"v="字段给出了SDP版本目前为0，没有子版本号。

### 会话源("o=")
```txt
o=<username> <sess-id> <sess-version> <nettype> <addrtype> <unicast-address>
```
"o="字段指定了当前会话的发起者、会话标志和会话版本号等信息。其各项值使用空格隔开。
`<username>`用户名，表示登录源主机的用户名称，如果不能提供，则用'-'表示。用户名中不能包含空格。
`<sess-id>`会话ID，用数字字符串来表示。`<username> <session-id> <nettype> <addrtype> <unicast-address>`这个组合了表示该会话的唯一标识。`<sess-id>`通常建议使用NTP格式的时间戳来表示。
`<sess-version>`会话版本，表示当前会话描述的版本号。如果当前会话的数据被修改时，该版本号会递增。同样建议使用NTP格式的时间戳来表示。
`<nettype>`网络类型，使用文本字符串来表示当前的网络类型。现阶段使用`"IN"`来表示"Internet"，以后可能会定义其它的值。
`<addrtype>`网络地址类型，使用文本字符串来表示当前的网络地址类型。现阶段只使用了`"IP4"`和`IP6`来表示ipv4地址及ipv6地址，未来可能会定义其它的值。
`<unicast-address>`单播地址，使用文本字符串表示创建主机的IP地址或域名。如何使用IP地址作为该值，则需要保证该IP能连通。

### 会话名字("s=")
```txt
s=<sess-name>
```
`<sess-name>`表示会话名字，每个会话描述有且只有一个"s="字段，"s="字段不能为空，并且应该为ISO 10646 字符格式。如果一个会话没有有意义的名字，则该字段值为一个空格。

### 会话信息("i=")

```txt
i=<session description>
```
`"i="`字段提供会话的文本信息。在会话层和媒体层最多只能出现一次。如果有`"a=charset"`字段，它标志了`"i="`字段使用的字体集合；如果没有`"a=charset"`字段，则必须使用UTF-8编码的IOS 10646字体集。
`"i="`也可以用于每个媒体层，这是用来标志每个媒体轨的。特别在两个媒体轨的媒体类型都是相同的情况下，这个字段就显得特别有意义。
这个字段主要是用来方便人类阅读的，并不适合自动解析。

### 统一资源描述("u=")
```txt
u=<URI>
```

`<URI>`是统一资源描述符，用于表示有关会话信息的外部信息。
这个字段是可选的。如果出现，就必须先于第一个媒体轨出现，并且不能多于一个。

### 电子邮件和电话("e="和"p=")
```txt
e=<email-address>
p=<phone-number>
```
`<email-address>`和`<phone-number>`这两个字段描述了会议拥有者的联系方式，为可选字段。
如果需要设置这两个字段，则必须在第一个媒体轨前面设置。

### 连接信息("c=")
```txt
c=<nettype> <addrtype> <connection-address>
```

一个会话描述必须在每个媒体层包含`c=`字段或或在会话层包含一个`c=`字段。如果会话层和媒体层都含有`c=`字段，那么媒体层`c=`字段信息会覆盖掉会话层出现的`c=`字段。

`<nettype>`网络类型，同`"o="`字段中的`<nettype>`，为文本字符串。现阶段只定义了"IN"，表示"Internet"。

`<addrtype>`地址类型，同`"o="`字段中的`<addrtype>`，为文本字符串。现阶段只使用了`"IP4"`和`IP6`来表示ipv4地址及ipv6地址。

`<connection-address>`连接地址，其值定义取决到`<addrtype>`。
当`<addrtype>`为`IP4`和`IP6`时，连接地址定义如下：
* 当会话为多播时，连接地址为多播地址；当会话为单播时，连接地址为单播地址，并且为媒体数据的源地址。
* 如何地址类型是`IPv4`的，则还需要给定TTL值，TTL表示包的生存时间，范围为0~255。

会话的TTL将使用斜杠作为前缀附加到地址后面，如`c=IN IP4 224.2.36.42/127`，该连接信息中的TTL为127。

另外分层编码器通常把媒体流分为多层的媒体数据，接收方可以根据自己的能力或需要只接收其它的某些层。这种编码通常以多个多播地址的方式来发送，通常设计如下：
```txt
<base multicast address>[/<ttl>]/<number of addresses>
```
`<base multicast address>`为组播基地址，`<ttl>`为生存周期，`<number of addresses>`为地址数量，如果地址数据没给出来，则默认为1。
如：
```txt
c=IN IP4 224.2.1.1/127/3
```
表示`224.1.1.1`、`224.1.1.2`和`224.1.1.3`。
这个在语法上等价于：
```txt
c=IN IP4 224.2.1.1/127
c=IN IP4 224.2.1.2/127
c=IN IP4 224.2.1.3/127
```

这种多个地址的表示法只能用于媒体层，而不能在会话层使用，而且也不能用于单播地址。

### 带宽("b=")
```txt
b=<bwtype>:<bandwidth>
```
`<bwtype>:<bandwidth>`带宽类型和带宽值，这个字段的意思是本会话或者媒体需要占用的带宽。
SDP RFC文档中制定了两种带宽类型，分别为`CT`和`AS`。
`CT`表示这个会话所占的所有带宽的大小，当用于RTP会话时，表示所有的RTP会话所占用的带宽。
`AS`表示针对特定应用的，通常表示某应用所占用的最大带宽。当用于RTP会话时，表示单一RTP会话所占用带宽。
通常`CT`表示的是所有媒体轨所占用的带宽，`AS`表示单一媒体轨所占用的带宽。

如果在`<bwtype>`前加上`X-`前缀，则表示这些带宽类型只是在实验中使用，如：`b=X-YZ:128`。在这里并不推荐使用`X-`前缀的用法，应该采用在IANA注册的形式来增加带宽类型。

SDP解析器应该忽略不认识的带宽类型。带宽类型是字母+数字的形式，并没有长度限制，但一般都很短。

`<bandwidth>`缺省单位为`kilobits`，即`kb`。在定义带宽类型的时候也会定义这个单位。

### 时间信息("t=")
```txt
t=<start-time> <stop-time>
```
`<start-time>`和`<stop-time>`分别表示会话的开始与结束时间，如果会话在不规则的多个时间段内有效，则可能出现多个`t=`字段。如果时间段是规则的，则应该使用`r=`字段。

时间格式均采用NTP格式。转换为UNIX时间时，则需要减去`2208988800`，因为该时间格式计时是从1900年开始算的，而UNIX时间是从1970年开始的。

如果`<stop-time>`为0，则表示会话时长不受限，不过会话需要在`<start-time>`以后才有效；如果`<start-time>`为0，则表示会话一直有效。

### 重复信息("r=")
```txt
r=<repeat interval> <active duration> <offsets from start-time list>
```
`r=`字段表示会话的重复时间，`<repeat interval>`表示重复间隔，`<active duration>`表示持续时长`<offsets from start-time list>`开始偏移时间链表。

如：如果一个会话有效时间为每个周一上午10和周二上午11点，每次持续1个小时，总共持续3个月。则`t=`字段中的`<start-time>`应该为上午10点，`<stop-time>`应该为三个月后；`r=`字段中的 `<repeat interval>`为1个星期，`<active duration>`为1个小时，`<offsets from start-time list>`则为0和25小时，因此则其表示如下：
```txt
t=3034423619 3042462419
r=604800 3600 0 90000
```
通常，为了更紧凑的表示，也可以使用天、小时、分钟和秒为单位。
```txt
d - days (86400 seconds)
h - hours (3600 seconds)
m - minutes (60 seconds)
s - seconds (allowed for completeness)
```
则上面的`r=`字段可以换成`r=7d 1h 0 25h`来表示。


### 时区("z=")
```txt
z=<adjustment time> <offset> <adjustment time> <offset> ....
```
`z=`字段用来表示时区信息，`<adjustment time>`表示基准时区信息，`<offset>`表示需要设置的偏移量。允许发送方指定NTP时间区域调整发生时和会话时间的偏移量。
例如：
```txt
z=2882844526 -1h 2898848070 0
```
表示在时刻2882844526，基准时刻就需要往前调整1个小时；而在时刻2898848070，则不需要调整

### 加密密钥("k=")
```txt
k=<method>
k=<method>:<encryption key>
```
当在安全的通道中传输SDP信息时，则可以通过SDP来传递密钥，`k=`字段则表示协商密钥，现已不推荐使用该方式。
当`k=`字段在所有媒体轨之前，这表示应用于所有媒体轨；当`k=`字段处理某媒体轨时，则表示该密钥仅用户该媒体轨。

SDP RFC规范中定义`k=`字段方式如下：
* `k=clear:<encryption key>` 密钥为明文密钥。
* `k=base64:<encryption key>` 密钥经过base64编码。
* `k=uri:<URI>` 密钥通过URI获取。
* `k=prompt` 双方已经明确的指定密钥。

*注：除非保证传输SDP的通道完全安全，否则不建议使用`k=`字段。*

### 属性("a=")
```txt
a=<attribute>
a=<attribute>:<value>
```
`a=`字段是SDP扩展属性的主要方式。
属性有会话层属性和媒体层属性，会话层属性应用于整体会话的过程（包含所有媒体轨），媒体层的属性只应用于该媒体轨。
属性有两种方式：
* 特定属性，以`a=<attribute>`表示，如`"a=recvonly"`
* 值属性，以`a=<attribute>:<value>`表示， 如`"a=orient:landscape"`

`<attribute>`必须使用US_ASCII子集，属性的值`<value>`则为ISO-10646中的字体串集。

如果SDP解析器有不认识的属性字段，则应该直接忽略它。


### 媒体信息("m=")
```txt
m=<media> <port> <proto> <fmt> ...
m=<media> <port>/<number of ports> <proto> <fmt> ...
```
一个会话描述可能包含多个媒体描述。每个媒体描述都是以`"m="`字段开始的，结束于下一个`"m="`或者至整体会话结束。

`<media>`表示媒体类型，有视频、音频、文本、应用、消息等。

`<port>`指发送媒体流的端口，这个字段的意义依赖于`c=`字段。对于多个分层的码流，如果采用单播地址，则必须使用端口来区分这些码流；当端口为连续时，则可使用`/<number of ports>`方式来设置连续的端口数量，如果端口为不连续的端口，则需要使用`a=`字段来指明端口。
如RTP协议，默认情况下使用偶数端口来发送RTP数据，对应加1的端口来发送RTCP数据；如果有明确设定不同的端口来发送RTCP数据，则需要使用`a=rtcp:`来指明。
如在`c=`字段中采用了多地址，在`m=`了字段中采用多端口，那么这些地址和端口默认是一一对应的。

`<proto>`表示传输协议，传输协议的意义依赖于`c=`字段,如`c=`字段中的`IP4`则表示在IPv4上的协议，SDP RFC4566文档中规定了如下几个协议：
* udp，udp上的一个未指定的协议
* RTP/AVP，指RTP协议
* RTP/SAVP，指SRTP协议
 
`<fmt>`表示媒体格式类型，这个参数可能是一个链表，表示多个媒体格式的类型，该字段依赖于`<proto>`协议字段。
如果`<proto>`字段为`RTP/AVP`或`RTP/SAVP`，则媒体格式表示RTP负载格式的编号。当出现第一个链表的时候，表示链表中的媒体格式都可用于当前媒体轨，第一个格式为其默认值。
如果`<proto>`字段为`udp`，则媒体格式指定媒体类型为音频、视频、文本、应用或者消息，这些媒体类型定义了数据包的传输格式。


## SDP属性

SDP中属性`a=`字段，作为SDP扩展的主要方式，在RFC4566中主要定义了如下扩展属性：

### 分类属性 `a=cat:<category>`

这个属性给出点分层次式会话分类号,供接收方筛选会话。这是会话层属性，并且它不依赖于字符集。

### 关键字属性`a=keywds:<keywards>`

根据关键字隔离相应的会话， 供接收方筛选会话。会话层属性，不依赖于字符集，默认字符集为ISO 10646/UTF-8。

### 工具集属性`a=tool:<name and version of tool>`

这个属性给出了创建会话描述的工具的名字处版本号。会话层属性。

### 媒体包时长`a=ptime:<packet time>`

这个属性给出每个媒体包的时长，以毫秒为单位。通常只用于音频数据。对于解码来说，不是必须属性，通常只作为编码/打包的时长建议。媒体层属性。

### 媒体包最大时长`a=maxptime:<packet max time>`

这个属性给出了每个媒体包的最大时长，以毫秒为单位。如果是基于帧编码，则这个通常是帧长的整数倍。通常只用于音频数据。媒体层属性。基于RFC2327的才出现的属性，如果没有实现，则会忽略该属性。

### 媒体负载信息属性`a=rtpmap:<payload type> <encodeing ame>/<clock rate> [/<encoding parameters>]`

这个属性是表示媒体流传输协议的RTP具体内容。rtpmap:是rtp map即RTP参数映射表。媒体层属性。该属性可能存在多个，表示多个媒体流信息。

`<payload type>`：负载类型，对应表示RTP包中的音视频数据负载类型，比如RTP的数据类型是H.264，那么这里就是96。

`<encoding name>`:编码器名称，这里主要指的RTP承载音视频编码数据类型，当然可以是标准数据也可以私有数据，如VP8 VP9 H.264等。

`<clock  rate>`:采样率，音视频里面都有时间戳的概念，所以这里表示的音视频的采样率，对音视频同步非常重要。比如视频的90000，音频的8000、48000等。

`<encoding parameters>`：编码参数，可以表示视频的分辨率，帧率，音频的单声道双声道等信息。

### 只接收属性 `a=recvonly`

表示应用端只接受（收流端）只用于媒体，不用于控制协议

### 发送接收属性 `a=sendrecv`

表示应用端发送和接收模式，交互式会话中，该属性为必须值。

### 只发送属性 `a=sendonly`

表示应用端只发送（发流端）只用于媒体，不用于控制协议

### 非活跃属性 `a=inactive`

表示应用端应使用非激活模式，交互式会话中，当一个应用让另一个应用等待时，必须使用该属性。

### 方向属性 `a=orient:<orientation>`

这个属性只用于白板或演示工具，标志了工作区在屏幕的位置。媒体层属性。
其值固定为如下几种：`portrait`、`landscape`、`seascape`。

### 会议类型属性 `a=type:<conference type>`

这个属性标志了会议的类型，会话层属性。
建议值为 `broadcast`、`meeting`, `moderated`、`test`、和`H332`。
对于`a=type:broadcast`类型，`a=recvonly`是缺省的默认值。 

### 字符集属性`a=charset:<character set>`

这个属性表示会话名字和会话信息的字符集。默认为ISO-10646/UTF-8字符集。会话层属性。

该属性值采用US-ASCII字符串，大小写敏感。如果解析器不认识该字符串，则忽略该属性值，同时并把它所影响的区域当成字符串处理。

*注意：字符集禁止使用0x00、0x0a、0x0d这三个字符，如果需要使用，需要采用转义字符来表示。*

### SDP语言属性`a=sdplang:<language tag>`
在会话层，这个属性会影响所有的媒体层的SDP信息；在媒体层，这个属性只会影响该媒体的SDP信息。

其重要度根据出场顺序来决定，最重要的最先出现，最不重要的最后出现。

通常不建议包含多个SDP信息，而是发送一个SDP采用一个SDP信息。

`<language tag>`值在[RFC3066](https://datatracker.ietf.org/doc/html/rfc3066)中定义。

### 语言属性`a=lang:<language tag>`

在会话层，这个属性会影响所有的媒体层的SDP信息；在媒体层，这个属性只会影响该媒体的SDP信息。

其重要度根据出场顺序来决定，最重要的最先出现，最不重要的最后出现。

通常不建议包含多个SDP信息，而是发送一个SDP采用一个SDP信息。

`<language tag>`值在[RFC3066](https://datatracker.ietf.org/doc/html/rfc3066)中定义。

### 帧率属性`a=framerate:<frame rate>`
这个属性给出了视频帧的最大帧率。通常作为推荐参数。媒体层属性，只能用于视频媒体流。
如果`<frame rate>`为小数，则表示形式为：`<integer>.<fraction>`。
 
### 质量属性`a=quality:<quality>`
这个属性给出编码的质量要求，以整数出现，媒体层属性。
对于视频流而言，取值范围为0~10。10表示质量最好，5为默认值，0表示最差。

### 编解码属性 `a=fmtp:<format> <format specific parameters>`
这个属性标志了具体格式的编解码参数，媒体层属性。

`<format>`媒体格式必须为`m=`属性中定义的某个媒体类型。
`<format specific parameters>` 媒体参数可以为SDP传输的任意值。
SDP本身不解析这两个参数的意义，只是原封不动的将其传递给解码器，且每个媒体类型最多只能有一个这种属性。

## SDP实例
```
// SDP开头4个必填字段
- **v=0**   // SDP版本号，设置为0，这是当前使用的唯一SDP版本

- **o=**- 6976870941538595051 2 IN IP4 127.0.0.1
 // 来源包含一系列字段，其中包括用户、会话ID、版本ID、网络地址、地址类型和地址；这里的地址是一个环回地址（127.0.0.1）

- **s=-**  // 主题，会话名，没有的话使用-代替

- **t=0 0**     // 时间，两个值分别是会话的起始时间和结束时间，这里都是0代表没有限制

// 接下来是SDP的属性用于定义此SDP中使用的媒体流ID语义。
- **a=group:**BUNDLE audio video   // 需要共用一个传输通道传输的媒体，如果没有这一行，音视频，数据就会分别单独用一个udp端口来发送

- **a=msid-semantic:** WMS qovvqrAafFmIE8VZCxLsWvxdW7zQtcDpzzsb
//WMS是WebRTC Media Stream简称，这一行定义了本客户端支持同时传输多个流，一个流可以包括多个track,
//一般定义了这个，后面a=ssrc这一行就会有msid,mslabel等属性

// 接下来，此SDP提议包含三个用于建立数据通道的媒体行（m =）
- **m=audio 9 UDP/TLS/RTP/SAVPF 111 103 104 9 0 8 106 105 13 110 112 113 126**
// 第一个音频媒体行列出了端口号（9，不会被使用，因为实际端口位于a = candidate字段中）、扩展的安全音频/视频配置文件（ UDP/TLS/RTP/SAVPF），
//随后列出可能的编解码器（有效负载类型分别是111 103 104 9 0 8 106 105 13 110 112 113 126），用于发送DTMF
// PCM编解码器使用静态有效负载（0~95），而OPUS和telephone-event则使用动态有效负载（范围96~127）。
// 每个编码器都有一个a=rtpmap属性，但此属性对于静态有效负载而言为可选属性

- **c=**IN IP4 0.0.0.0   //媒体链接数据字段，设置了一个无效的IP地址（人们常说的“黑洞”地址0.0.0.0）
- **a=**rtcp:9 IN IP4 0.0.0.0   // 包括一个RTCP端口和IP地址属性，但后者却设置为无效的IP地址和端口，暂不清楚该属性的用途

- **a=**ice-ufrag:U8F/   //用户名片段
- **a=ice-pwd:**Hfo+6wYJCMeLWahO6CLPwbLZ    // ICE密码。每个m行，将包括不同的用户名和密码。使用时使用第一组
- **a=ice-options:**trickle
- **a=fingerprint:**sha-256 EF:BD:53:BB:A4:E9:28:87:74:A4:15:27:59:B8:2D:05:61:04:CA:76:C8:2D:1C:BF:F0:CC:12:D8:8A:CC:AD:49
// 用于建立DTLS链接的自签名证书的SHA-256散列，因此必须协商确定由哪一方开通链接

- **a=setup:actpass**     //以上这行代表本客户端在dtls协商过程中，可以做客户端也可以做服务端，参考rfc4145 rfc4572
- **a=mid:audio**   //在前面BUNDLE这一行中用到的媒体标识
- **a=extmap:**1 urn:ietf:params:rtp-hdrext:ssrc-audio-level    // 指出我要在rtp头部中加入音量信息，参考 rfc6464
- **a=sendrecv**   // 指示视频通话为双向会话。另外几种类型是recvonly,sendonly,inactive（不收不发）
- **a=rtcp-mux**   // RTP多路复用属性指示将通过用于RTP的同一端口对RTCP进行多路传输
- **a=rtpmap:**111 opus/48000/2
- **a=rtcp-fb:**111 transport-cc
// 以上这行说明opus编码支持使用rtcp来控制拥塞，

- **a=fmtp:**111 minptime=10;useinbandfec=1
// 对opus编码可选的补充说明,minptime代表最小打包时长是10ms，useinbandfec=1代表使用opus编码内置fec特性

// 13个可能的编解码器的rtpmap属性
- a=rtpmap:103 ISAC/16000
- a=rtpmap:104 ISAC/32000
- a=rtpmap:9 G722/8000
- a=rtpmap:0 PCMU/8000
- a=rtpmap:8 PCMA/8000
- a=rtpmap:106 CN/32000
- a=rtpmap:105 CN/16000
- a=rtpmap:13 CN/8000
- a=rtpmap:110 telephone-event/48000
- a=rtpmap:112 telephone-event/32000
- a=rtpmap:113 telephone-event/16000
- a=rtpmap:126 telephone-event/8000

- a=ssrc:1815301143 cname:JsTxr0Ps83IbEYOT
//cname用来标识一个数据源，ssrc当发生冲突时可能会发生变化，但是cname不会发生变化，也会出现在rtcp包中SDEC中，
//用于音视频同步

- a=ssrc:1815301143 msid:qovvqrAafFmIE8VZCxLsWvxdW7zQtcDpzzsb 2e0529d2-d3bd-460f-a325-71d148367b1b
//以上这一行定义了ssrc和WebRTC中的MediaStream,AudioTrack之间的关系，msid后面第一个属性是stream-id,
//第二个是track-ida=ssrc:1815301143 mslabel:qovvqrAafFmIE8VZCxLsWvxdW7zQtcDpzzsb
- a=ssrc:1815301143 label:2e0529d2-d3bd-460f-a325-71d148367b1b
- **m=video 9 UDP/TLS/RTP/SAVPF 96 97 98 99 100 101 102 123 127 122 125 107 108 109 124**
- c=IN IP4 0.0.0.0
- a=rtcp:9 IN IP4 0.0.0.0
- a=ice-ufrag:U8F/
- a=ice-pwd:Hfo+6wYJCMeLWahO6CLPwbLZ
- a=ice-options:trickle
- a=fingerprint:sha-256 EF:BD:53:BB:A4:E9:28:87:74:A4:15:27:59:B8:2D:05:61:04:CA:76:C8:2D:1C:BF:F0:CC:12:D8:8A:CC:AD:49
- a=setup:actpass
- **a=mid:video**
- a=extmap:2 urn:ietf:params:rtp-hdrext:toffset
- a=extmap:3 http://www.webrtc.org/experiments/rtp-hdrext/abs-send-time
- a=extmap:4 urn:3gpp:video-orientation
- a=extmap:5 http://www.ietf.org/id/draft-holmer-rmcat-transport-wide-cc-extensions-01
- a=extmap:6 http://www.webrtc.org/experiments/rtp-hdrext/playout-delay
- a=extmap:7 http://www.webrtc.org/experiments/rtp-hdrext/video-content-type
- a=extmap:8 http://www.webrtc.org/experiments/rtp-hdrext/video-timing
- **a=sendrecv**
- **a=rtcp-mux**
- **a=rtcp-rsize**
- **a=rtpmap:**96 VP8/90000
- **a=rtcp-fb:**96 goog-remb
- **a=rtcp-fb:**96 transport-cc

- **a=rtcp-fb:**96 ccm fir
//ccm是codec control using RTCP feedback message简称，意思是支持使用rtcp反馈机制来实现编码控制，
//fir是Full Intra Request简称，意思是接收方通知发送方发送幅完全帧过来

- **a=rtcp-fb:**96 nack   // 支持关键帧丢包重传,参考rfc4585
- **a=rtcp-fb:**96 nack pli    // 支持关键帧丢包重传,参考rfc4585
- **a=rtpmap:**97 rtx/90000
- **a=fmtp:**97 apt=96
- **a=rtpmap:**98 VP9/90000
- **a=rtcp-fb:**98 goog-remb
- **a=rtcp-fb:**98 transport-cc
- **a=rtcp-fb:**98 ccm fir
- **a=rtcp-fb:**98 nack
- **a=rtcp-fb:**98 nack pli
- **a=rtpmap:**99 rtx/90000
- **a=fmtp:**99 apt=98
- **a=rtpmap:**100 H264/90000
- **a=rtcp-fb:**100 goog-remb   // 支持使用rtcp包来控制发送方的码流
- **a=rtcp-fb:**100 transport-cc
- **a=rtcp-fb:**100 ccm fir
- **a=rtcp-fb:**100 nack
- **a=rtcp-fb:**100 nack pli
- **a=fmtp:**100 level-asymmetry-allowed=1;packetization-mode=1;profile-level-id=42001f 
  //h264编码可选的附加说明
- **a=rtpmap:**101 rtx/90000
- **a=fmtp:**101 apt=100
- **a=rtpmap:**102 H264/90000
- **a=rtcp-fb:**102 goog-remb
- **a=rtcp-fb:**102 transport-cc
- **a=rtcp-fb:**102 ccm fir
- **a=rtcp-fb:**102 nack
- **a=rtcp-fb:**102 nack pli
- **a=fmtp:**102 level-asymmetry-allowed=1;packetization-mode=1;profile-level-id=42e01f  
//h264编码可选的附加说明
- **a=rtpmap:**123 rtx/90000
- **a=fmtp:**123 apt=102
- **a=rtpmap:**127 H264/90000
- **a=rtcp-fb:**127 goog-remb
- **a=rtcp-fb:**127 transport-cc
- **a=rtcp-fb:**127 ccm fir
- **a=rtcp-fb:**127 nack
- **a=rtcp-fb:**127 nack pli
- **a=fmtp:**127 level-asymmetry-allowed=1;packetization-mode=1;profile-level-id=4d0032  
 //h264编码可选的附加说明
- **a=rtpmap:**122 rtx/90000
- **a=rtcp-fb:**122 apt=127
- **a=rtcp-fb:**125 H264/90000
- **a=rtcp-fb:**125 goog-remb
- **a=rtcp-fb:**125 transport-cc
- **a=rtcp-fb:**125 ccm fir
- **a=rtcp-fb:**125 nack
- **a=rtcp-fb:**125 nack pli
- **a=fmtp:**125 level-asymmetry-allowed=1;packetization-mode=1;profile-level-id=640032  
 //h264编码可选的附加说明
- **a=rtpmap:**107 rtx/90000
- **a=fmtp:**107 apt=125
- **a=rtpmap:**108 red/90000
- **a=rtpmap:**109 rtx/90000
- **a=fmtp:**109 apt=108
//以上两行是VP8编码的重传包rtp类型
- **a=rtpmap:**124 ulpfec/90000    //支持ULP FEC，参考rfc5109

- **a=ssrc-group:**FID 2893345836 1252482188     
// 在webrtc中，重传包和正常包ssrc是不同的，上一行中前一个是正常rtp包的ssrc,后一个是重传包的ssrc
- **a=ssrc:**2893345836 cname:JsTxr0Ps83IbEYOT
- **a=ssrc:**2893345836 msid:qovvqrAafFmIE8VZCxLsWvxdW7zQtcDpzzsb f10f7f49-dd82-4916-a517-a28547e2c2ff
- **a=ssrc:**2893345836 mslabel:qovvqrAafFmIE8VZCxLsWvxdW7zQtcDpzzsb
- **a=ssrc:**2893345836 label:f10f7f49-dd82-4916-a517-a28547e2c2ff
- **a=ssrc:**1252482188 cname:JsTxr0Ps83IbEYOT
- **a=ssrc:**1252482188 msid:qovvqrAafFmIE8VZCxLsWvxdW7zQtcDpzzsb f10f7f49-dd82-4916-a517-a28547e2c2ff
- **a=ssrc:**1252482188 mslabel:qovvqrAafFmIE8VZCxLsWvxdW7zQtcDpzzsb
- **a=ssrc:**1252482188 label:f10f7f49-dd82-4916-a517-a28547e2c2ff

**media之前的行是Session级别，也就是对会话的描述，而media及它后面的 a =行称作media行，属于media级别。**
```

## 参考
[RFC3264](https://datatracker.ietf.org/doc/html/rfc3264)
[RFC3264 简书](https://www.jianshu.com/p/2ca09bf99280)
[rfc8866](https://datatracker.ietf.org/doc/html/rfc8866)
[rfc4566](https://datatracker.ietf.org/doc/html/rfc4566)
[rfc4566中文](https://max.book118.com/html/2017/1201/142356242.shtm)
[sdp详细介绍](https://blog.csdn.net/cyq129445/article/details/79714199)
[SDP协议介绍](https://www.jianshu.com/p/94b118b8fd97)
[SDP协议简述](https://www.cnblogs.com/ranson7zop/p/7700165.html)