# HLS(HTTP Live Streaming)

HLS是一个由苹果公司提出的基于HTTP的流媒体网络传输协议。

HLS的工作原理是把整个流分成一个个小的基于HTTP的文件来下载，每次只下载一些。当媒体流正在播放时，客户端可以选择从许多不同的备用源中以不同的速率下载同样的资源，允许流媒体会话适应不同的数据速率。在开始一个流媒体会话时，客户端会下载一个包含元数据的Extended M3U(m3u8) playlist文件，用于寻找可用的媒体流。

HLS只请求基本的HTTP报文，与实时传输协议（RTP）不同，HLS可以穿过任何允许HTTP数据通过的防火墙或者代理服务器。它也很容易使用CDN来传输媒体流。这是HLS应用在直播上的一大优势。

由于传输层协议只需要标准的HTTP协议, HLS可以方便的透过防火墙或者代理服务器, 而且可以很方便的利用CDN进行分发加速, 并且客户端实现起来也很方便.

HLS最主要的问题就是实时性差。由于HLS往往采用6s的切片，所以最小也要有6s的延迟，有时甚至更差。
HLS使用的是HTTP短连接，且HTTP是基于TCP的，所以这就意味着HLS需要不断地与服务器建立连接。TCP每次建立连接时都要进行三次握手，而断开连接时，也要进行四次挥手，基于以上这些复杂的原因，就造成了HLS延迟比较久的局面。

另外HLS协议本身实现了码率自适应，不同带宽的设备可以自动切换到最适合自己码率的视频播放。

HLS支持如下功能：
* 直播和点播功能(Live broadcasts and prerecorded content (video on demand, or VOD))
* 支持多个备选流以及多码率(Multiple alternate streams at different bit rates)
* 可根据不同网络带宽智能切换(Intelligent switching of streams in response to network bandwidth changes)
* 多媒体加密以及用户认证(Media encryption and user authentication)


## HLS 框架说明

HLS直播技术框架图如下(图源于苹果官网):
![a.img](./hls_framework.png)

### Server
Server模块负责获取媒体的输入流并对其进行编码与切片。它将流封装在适合传送的格式中，并将其切分成合适的小媒体文件以方便传输。

Server组件在获取多媒体流后然后经过编码器（media encoder)编码成MPEG-4格式并打包成 MPEG-2的传输流。

传输流经过切片器(stream segmenter)后会被分散为小片段然后保存为一个或多个系列的的媒体文件，同时切片器也会创建一个索引文件(Playlist)，该索引文件会包含这些媒体文件的一个列表，也能包含元数据。该索引文件通常保存为.m3u8格式。

### Distribution
Distribution分发系统是一个网络服务或者一个网络缓存系统，用于通过HTTP向客户端发送媒体文件和索引文件。不用自定义模块发送内容。通常仅仅需要很简单的网络配置即可使用。而且这种配置一般就是限制指定.m3u8文件和多媒体文件的MIME类型。

### client
client负责确定要请求的适当媒体，下载这些资源，然后重新组合它们，以便媒体可以以连续流的形式呈现给用户。

client首先获取索引文件(Playlist)，使用标识流的URL。索引文件(Playlist)依次指定可用media segment、解密密钥和任何可用的备用流的位置。对于选定的流，client按顺序下载每个可用的媒体文件。每个文件包含流的一个连续段。一旦下载了足够数量的数据，客户端就开始向用户呈现重新组合的流。

client负责获取任何解密密钥，验证或呈现允许身份验证的用户界面，并根据需要解密媒体文件。这个过程一直持续到客户机在索引文件(Playlist)中遇到结束(EXT-X-ENDLIST)标记。如果不存在结束标记，则索引文件(Playlist)是正在进行的广播的一部分。在正在进行的广播期间，客户机定期加载index文件的新版本。客户机在更新的索引文件(Playlist)查找新的媒体文件和加密密钥，并将这些url添加到其队列中。

HLS一系列小文件的形式发送音频和视频，通常持续约6秒，称为media segment文件。索引文件(playlist)提供media segment文件的URL的有序列表。client访问索引文件的URL，然后按顺序请求索引文件。

## Playlist(m3u/m3u8)

HLS通过`URI(RFC3986)`指向的一个`Playlist`来表示一个媒体流. `Playlist`文件的格式是起源于 `M3U`，一个`Playlist`可以是一个`Media Playlist`或者`Master Playlist`, 两者都是包含`URI`和`描述性标记`的`UTF-8`文本文件。因此又叫`M3U8`。

`m3u8 Playlist`是一个文本文件，由多个独立的行组成。每一行以 `LF` 或 `CR+LF` 作为结束标识。行可以是一个`URI`，`空行`，或以注释字符`#`开头。空行将会被忽略。空格只能作为特殊指定的元素出现。

一个`URI行`表示一个`Media Segment`媒体文件或者一个`Playlist`列表文件。
`URI`可以是相对地址。一个相对地址的`URI`必须能够被解析，借助于包含它的播放列表文件的`URI`。

`Playlist`必须是`Media Segment`或者`Master Playlist`，所有其它的`Playlist`都是
无效的。
`Media Playlist`中所有的行都是`Media Segments`的`URI`。
`Master Playlist`中所有的行都是`Media Playlist`的`URI`。

以字符`#`开头的行，可能是注释或标签。
标签以`#EXT`开头。除标签以外的以`#`开头的注释行都应该被忽略。

`Media Playlist`文件的`duration`，是它所包含的所有媒体文件的持续时间的总和。

`Media segment`的`rate(比特率)`是该媒体的大小除以它的`EXTINF`中标记的`duration`,即文件大小除以时长。

`Playlist`文件名必须以`.m3u8`作为后缀，并或者使用`application/vnd.apple.mpegurl`作为HTTP中`Content-Type`的值；或者必须以`.m3u`作为后缀，并或者使用`audio/mpegurl`作为HTTP中`Content-Type`的值。

### PlayList AttributeName

`Playlist`中属性表示方法为：
```
AttributeName=AttributeValue
```

`AttributeName`必须为无引号的字符串，其必须从`[A..Z]`, `[0..9]`和`-`中取值。因此，`AttributeName`中仅包含大写字母和数字以及`-`，而不是小写字母。并且在`=`号前后不能包含空格。

`AttributeValue`的取值范围如下：
* 十进制无符号整数，其取值范围从`0`至`2^64-1(18446744073709551615)`。
* 十六进制整数，以`0x`或`0X`开头的十六进制数。
* 浮点数，有符号或无符号的十进制浮点数。
* 带引号字符串，在引号内不能出现`\n`、`\r`以及`"`。
* 枚举字符串，由AttributeName明确定义，不能出现`"`,`,`或`空格`。
* 由`x`分隔的整数，`width`x`height`。

### Playlist Tags
Playlist Tags以`#EXT`开头。除TAGS以外的以`#`开头的注释行都应该被忽略。

#### Base Tags

`Base Tags`在`Media Playlists`和`Master Playlists`中都能使用。

##### EXTM3U
EXTM3U标记表明该文件是一个扩展的`M3U Playlisty`文件。
Playlist文件中的第一行必须以`#EXTM3U`开头。
##### EXT-X-VERSION
`EXT-X-VERSION`表示Playlist文件的版本信息。其格式如下：
```
#EXT-X-VERSION:<version number>
```
`version number`表示当前文件的版本号整数，一个`Playlist`文件中只能出现一次，并且必须出现一次。


#### Media Playlist TAGS

一个`Media Playlist`包含一个`Media Segments`列表，播放器将按顺序播放其中的多媒体内容，当顺序播放时，能播放整个完整的流；要想播放这个`Playlist`，`client`需要首先下载它，然后播放里面的每一个`Media Segment`。

##### EXTINF

`EXTINF` 是记录标记，描述了下一行的`URI`指向的媒体文件。每一个媒体文件`URI`的上一行必须是`#EXTINF`标签。格式如下：
```
#EXTINF:<duration>,[<title>]
```
`duration`是一个整数或浮点数，指定了媒体文件的持续时间，以秒为单位。当`EXT-X-VERSION`版本小于3时，其值必须是一个4舍5入的整数。
`title`是一个可选参数，其表示的是媒体文件的标题，是该媒体分片的适宜阅读的标题信息。

##### EXT-X-BYTERANGE

`EXT-X-BYTERANGE`表示媒体段是一个媒体URI资源中的一段，只对其后的media URI有效，格式如下：
```
#EXT-X-BYTERANGE:<n>[@o]
```
其中n表示这个区间的大小，o表在URI中的offset；`EXT-X-BYTERANGE` 只在版本4及能上出现。


##### EXT-X-DISCONTINUITY
`EXT-X-DISCONTINUITY` 标明前后两个文件片段不连续。如两个文件的编码格式不一致、文件类型不一致、track的类型和数量不同、编码参数不一致，编码序号不连接、时间戳序号不连贯等。

其格式如下：
```
#EXT-X-DISCONTINUITY
```

如果编码参数或编码的序号有变化时，`EXT-X-DISCONTINUITY`必须存在。


##### EXT-X-KEY
表示怎么对`media segments`进行解密。其作用范围是下次该tag出现前的所有media URI。

其格式如下：
```
 #EXT-X-KEY:<attribute-list>
```
`attribute-list`中各项值定义如下：
* METHOD 枚举值，标明加密方法，必须存在。
该值可使用如下值：NONE, AES-128, 和 SAMPLE-AES。
    - NONE 表示`media segments`未加密，如果加密方法是NONE,则别的属性一律不许出现
    - AES-128 表示`media segments`使用AES-128-CBC及PKCS7补齐加密，URI字段必须存在，IV为可选项
    - SAMPLE-AES 表示`media segments`中的包含使用AES-128加密的`media sample`，如audio或video，这些取样的加密和封装方式与媒体文件的编码和文件片类型有关。如fMP4使用的是[COMMON_ENC]，H.264, AAC, AC-3和Enhanced AC-3则使用的[SampleEnc](`HLS Sample Encryption specification`)。

* URI 该值是一个带引号的字符串，其中包含一个URI，用于指定如何获取密钥。除非METHOD值为NONE，否则此属性是必需的存在。

* IV 该值是一个128-bit的十六进制整数初始化向量，用于AES-128加密中，在AES-128-CBC中，IV和KEY可以是相同的。

* KEYFORMAT 该值是带引号的字符串，用于指定URI中的密钥方式，可选参数。

* KEYFORMATVERSIONS 该值是包含一个或多个正数的带引号的字符串以`/`字符分隔的整数；如，`1`，`1/2`，或`1/2/5`。其表示如果特定`KEYFORMAT`有多个版本定义，此属性可用于指示哪个版本适用。可选参数。

##### EXT-X-MAP
`EXT-X-MAP`用于说明如何获取用于解析`media segments`的头部信息，比如传输流 PAT/PMT 或者WebVTT头。其作用于其后出现的所有`media segments`，直到出现下一个`EXT-X-MAP`、或文件末尾、或出现`EXT-X-DISCONTINUITY`。

该TAG应该出现在当`Playlist`中的第一个`Media Segment`中没有`Media Initialization Section`时，且`Playlist`中有`EXT-X-I-FRAMES-ONLY`时。

如果由`EXT-X-MAP`声明的`Media Initialization Section`是用`AES-128`的方法加密，`EXT-X-KEY`的`IV`需要适用于`EXT-X-MAP`。

其格式如下：
```
#EXT-X-MAP:<attribute-list>
```
`attribute-list`中各属性值如下：
* URI 该值是一个带引号的字符串，表示包含头部信息的资源的URI，必选参数。
* BYTERANGE 该值是一个带引号的字符串，表示URI资源的一定字节范围；可选参数，不填写该属性时，指代URI所指的全部资源。

##### EXT-X-PROGRAM-DATE-TIME
`EXT-X-PROGRAM-DATE-TIME`表示其后一个文件片第一个取样的日期和时间。其仅作用于其下的`Media Segment`。
其格式如下：
```
#EXT-X-PROGRAM-DATE-TIME:<date-time-msec>
```
`date-time-msec`是基于[ISO/IEC 8601:2004]表示的时间；其格式如下为`YYYY-MM-DDThh:mm:ss.SSSZ`，它包含时区信息，并精确到毫秒。
如表示时区+8时间为2010年02月19日下午14点54分23秒031毫秒：
```
#EXT-X-PROGRAM-DATE-TIME:2010-02-19T14:54:23.031+08:00
```

##### EXT-X-DATERANGE
`EXT-X-DATERANGE` 表示关联一个日期范围（即由开始日期和结束日期定义的时间），以`/`分隔的一组属性对。

其格式如下：
```
#EXT-X-DATERANGE:<attribute-list>
```
`attribute-list`中各值定义如下：
* ID 带引号的字符串，用于在PlayList中标识日期范围。必选参数。
* CLASS 客户端自定义的一组相关联的带引号的属性字符串，所有具体相同日期范围的CLASS属性都必须遵守该定义。可选参数。
* START-DATE `ISO-8601`标准的日期字符串，表示日期范围的开始日期。必选参数。
* END-DATE `ISO-8601`标准的日期字符串，表示日期范围结束日期。可选参数。
* DURATION 浮点正数，表示日期范围的持续时间，以秒为单位。

例如下一个`Media Playlist`：
```
#EXTM3U
#EXT-X-TARGETDURATION:10

#EXTINF:9.009,
http://media.example.com/first.ts
#EXTINF:9.009,
http://media.example.com/second.ts
#EXTINF:3.003,
http://media.example.com/third.ts
```





每一个 `Media Segment` 通过一个 URI 指定, 可能包含一个 byte range.
每一个 `Media Segment` 的 duration 通过 EXTINF tag 指定.
每一个 `Media Segment` 有一个唯一的整数 Media Segment Number.
有些媒体格式需要一个 format-specific sequence 来初始化一个 parser, 在 Media Segment 被 parse 之前. 这个字段叫做 Media Initialization Section, 通过 EXT-X-MAP tag 来指定. 支持的 Media Segment 格式

### Master Playlist

更加复杂的情况是，`Playlist`是一个`Master Playlist`, 包含一个`Variant Stream`集合, 通常每个`Variant Stream`里面是同一个流的多个不同版本(如: 分辨率, 码率不同)。

## HLS Media Segments 


## TS

## 参考


[http_live_streaming](https://developer.apple.com/documentation/http_live_streaming)
[解析HLS视频格式](https://www.jianshu.com/p/dbac4c041de8)
[HTTP Live Streaming](https://developer.apple.com/streaming/)
[HLS 架构简介及播放加密的HLS](http://www.rosoo.net/m/view.php?aid=17526)
[HTTP Live Streaming (HLS) - 概念](https://www.jianshu.com/p/2ce402a485ca)
[HLS协议草案（中文翻译）](https://blog.ibaoger.com/2015/11/27/http-live-streaming-draft/)
[rfc8216](https://tools.ietf.org/html/rfc8216)
[HLS 协议详解](https://www.jianshu.com/p/dc4e5d55758a)
[hls协议中m3u8文件tag总结](https://blog.csdn.net/zhoushuaiyin/article/details/38759427)

[COMMON_ENC]: http://www.iso.org/iso/catalogue_detail.htm?csnumber=68042
[SampleEnc]: https://developer.apple.com/library/ios/documentation/AudioVideo/Conceptual/HLS_Sample_Encryption
[ISO/IEC 8601:2004]: http://www.iso.org/iso/catalogue_detail?csnumber=40874