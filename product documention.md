## 概述
又拍云直播分发网络( UPYUN Live Streaming Delivery Network )，即又拍云直播 CDN，可以对推流到又拍边缘的直播流进行推流和分发加速，同时也可以对 RTMP、HTTP-flv 和 HLS 回客户源的直播进行分发加速。同时，又拍云还提供直播相关的录制、转码等附加功能，帮助用户更好地完善自身业务。

### 服务类型
又拍云直播系统支持两种服务类型：又拍云源和自主源站。

架构图片：客户端推流/编码器推流——又拍云加速中心（转码/录制）——用户观看端

又拍云源：表示流媒体内容直接推流到又拍云 CDN ，将推流与分发都进行 CDN 加速。又拍云源现仅支持 RTMP 推流。

> 管理后台：直播加速 > 创建服务 > 又拍云源

创建`又拍云源`，填写`服务名`，填写已备案的播放域名与推流域名，推流域名与播放域名不能共用。
可选择性填写接入点，如果填写了具体的接入点，则只有该接入点的 URL 可进行推流，其他接入点均推流失败。若没有填写具
体接入点，则任意接入点可进行推流（本阶段无论填什么接入点，均无限制作用，待服务版本更新再通知）。

> 当推流域名填写 push.domain.com ，播放域名填写 play.domain.com ，接入点填写 app ，如果推流 URL 为 rtmp://push.domain.com/app/stream 时，其 rtmp 播放 URL 为 rtmp://play.domain.com/app/stream
> 其中，stream 为流名，又称流密钥，app/stream 又称为频道，同一条流推拉流域名不同，但频道名一致，下同。

架构图片：客户源——又拍云加速中心（转码/录制）——用户观看端

自主源站:表示流媒体内容在客户源站，又拍云通过回客户源拉流的方式对其内容进行 CDN 分发。

> 管理后台：直播加速 > 创建服务 > 自主源站

`自主源站`需要明确回源协议是 RTMP 还是 HTTP，并填写源站 IP 或者 域名，回源 IP 现不支持多个，若回源 IP 会经常变更，建议源站填写域名形式。当回源协议是 RTMP 时，默认端口是1935，当回源协议是 HTTP 时，默认端口为 80，支持端口自定义。因自主源站是回客户源拉流，帮无推流域名，只需要设置播放域名。

接入点说明同又拍云源。

以上两种业务模式，均支持RTMP 播放及HTTP-FLV、HLS 协议转换，后续章节会详细介绍。

### 域名绑定

> 直播管理后台：服务 > 加速域名  
> 源站类型：全部

域名绑定仅支持已备案成功的自定义域名进行绑定，如果该域名未备案成功，则不能通过域名绑定审核。
审核结果会通过邮件进行通知，正常审核时间为一个工作日。
以下是关于域名绑定的一些重要说明（同文件加速）：

* 除常规形式的域名外（包括顶级域名），我们还支持泛域名绑定，比如 *.yourdomain.com；特别地，其中 *最多支持匹配 4 层：

```
[v] *.yourdomain.com => foo.yourdomain.com
[v] *.yourdomain.com => bar.yourdomain.com
[v] *.yourdomain.com => foo.bar.yourdomain.com
[v] *.yourdomain.com => foo.bar.baz.yourdomain.com
[v] *.yourdomain.com => foo.bar.baz.qux.yourdomain.com
[x] *.yourdomain.com => foo.bar.baz.qux.quxx.yourdomain.com

[v] *.bar.yourdomain.com => foo.bar.yourdomain.com
[v] *.bar.yourdomain.com => baz.foo.bar.yourdomain.com
[x] *.bar.yourdomain.com => foo.bar.baz.yourdomain.com
```

* 添加域名绑定后，需要到域名服务商的 DNS 解析管理中，将推流域名的 CNAME 解析到 `<bucket>.s1.aicdn.com`，将播放域名的 CNAME 解析到 `<bucket>.s0.aicdn.com`。

> 当业务模式为又拍云源时，用户需要将其推流域名和播放域名分别 CNAME 到对应的又拍云内部域名。 推流内部域名为`<bucket>.s1.aicdn.com`，播放内部域名为`<bucket>.s0.aicdn.com`。

> 当业务模式为自主源站时，用户只需要将其播放域名 CNAME 到对应的又拍云播放内部域名`<bucket>.s0.aicdn.com`
其中 `<bucket> `请用实际服务名称代替，下同。  

### 回源配置
> 管理后台：服务 > 基础配置 > 回源配置  
> 源站类型：客户源站

截图(回客户源回源配置)

#### 回源协议
回源协议支持 HTTP 和 RTMP，当选择 HTTP 回源时，默认端口号为80，当选择RTMP回源时，默认端口号为1935，具体端口号可以自定义，通过后台设置。
#### 源站
该配置项为可访问的网络地址，可以直接填 IP 地址也可以填写域名地址，现不支持多IP 。如果是域名地址，那么 CDN 在回源时会对该域名地址进行 DNS 解析，然后通过解析出来的 IP 地址再进行访问，因此若解析失败也会导致无法正常回源。

### HTTP-FLV 输出
> 管理后台：服务 > 基础配置 > HTTP-FLV 输出  
> 源站类型：全部

通过管理后台，可以配置任一播放域名作为 HTTP-FLV 的输出域名。

当推流 url 为 rtmp://push.domain.com/app/stream 时，对域名播放域名开启 HTTP-FLV 输出配置后， 可以通过 http://play.domain.com/app/stream.flv 进行播放。
### HLS+ 输出
> 管理后台：服务 > 基础配置 > HLS 输出  
> 源站类型：全部

通过管理后台，可以配置任一播放域名作为 HLS 的输出域名。

开启该配置后，可通过 http://play.domain.com/app/stream.m3u8 对 rtmp://push.domain.com/app/stream 的推流进行播放。

### 录播
> 客户需提供的配置信息：需要录制的拉流 URL  
> 录制方式：触发 or 定时录制

录播的主要作用是将推流内容录制成文件，最终用于点播。

现默认录制成 mp4 格式文件，上传到又拍云存储，客户可以自定义上传到哪个存储服务（暂不支持，接口未开放，现在是默认云存储为live-recorder）。

又拍录制系统会自动将录制下来的内容上传到又拍云存储后，可以根据云存储获取目录文件列表 来获取相关录制文件列表。
如果对录制文件格式有其他要求，可在又拍云处理中心对其进行相关处理，比如格式处理，视频拼接，详细请见[云处理文档](http://docs.upyun.com/cloud/)。

录制文件具体路径为：  
live-recorder.b0.upaiyun.com/play.domain.com/live/stream/recorder20160604163702.mp4  
其中 live-recorder.b0.upaiyun.com 为又拍云默认域名，又拍会将录制文件默认保存在相应空间以这条流 URL 为路径的目录下，即 live-recorder.b0.upaiyun.com/play.domain.com/live/stream/，  
recorder20160604163702.mp4 为具体的录制文件，recorder 为固定标识字符，20160604163702 为录制完成时间，mp4为文件类型。   
如果用又拍 cdn，直接可通过 http://用户加速域名/play.domain.com/live/stream/recorder20160604163702.mp4 来访问。

录制支持触发录制与定时录制两种方式。

#### 触发录制
指定配置某一条流为触发录制的流，则在这条流推流到又拍 CDN 的时候，又拍录制系统就对其开始录制，当这推流断开后停止录制，再推流后又继续开始录，以此循环。推流断开时，录制文件自动停止录制，下次再推流后，录制的文件名将与前一文件名不一样，这样同一条流会生成多个不同文件，并以文件名最后的时间表示其先后顺序。

#### 定时录制
指定配置某一条流从几点开始录制，并到几点结束，不管在这时间段内推流断开几次，录制系统将在相应时间段内对其推流的所有流内容进行录制在同一文件名下，定时任务只生成一个文件。

#### 回调
>客户需提供接收回调的地址（建议为URL）

录制文件所在的路径以 post 请求返回给客户，具体的 json 格式为

{"timestamp": "2016-06-04 16:38:45",  
 "path": ["http://live-recorder.b0.upaiyun.com/play.domain.com/live/stream/recorder20160604163702.mp4"]}

其中，timestamp 为发送 json 回调任务时间，path 为录制文件具体路径。

### 实时转码
支持音视频流实时转码处理，通过转码模版可配置编码标准、分辨率、码率、输出流类型等流处理参数。
默认支持使用又拍云直播服务的 RTMP，HTTP-FLV，HLS 协议的流转码 支持 12 种转码模板和客户自定义转码配置，详细模板信息：http://docs.upyun.com/cloud/attachment/ 支持自定义转码后缀，分隔符支持中划线（-）、下划线（_）和感叹号（!）。

支持触发式转码，需提前配置需要转码的流地址以及转码的触发后缀，如 需要转码的原始流为：http://play.domain.com/live/stream 触发转码的后缀匹配为 -small，对应的转码模板为 540p(16:9) 当有用户请求 http://play.domain.com/live/stream-small 时触发转码，当最后一个请求该转码流的用户断开连接后，停止转码。

> 转码和录播至少提前两天给出配置需求。
