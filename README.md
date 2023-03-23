# rtsp-webrtc-demo

WEB浏览器要调看摄像头视频，通常有flash插件、HLS拉流、WEBRTC 3种方式，随着flash被禁用，若要实现低延时，需要使用到WEBRTC，很多浏览器已经支持该框架，本文旨在利用webrtcstremer搭建一个DEMO

## 安装步骤

### 步骤1. 安装服务

找一台linux服务器并安装docker

```
docker pull mpromonent/webrtc-streamer
docker run -itd -p 8000:8000 --name webrtcstremer mpromonent/webrtc-streamer
```

### 步骤2. 修改index.html中的转换单元服务地址和RTSP流地址

```
var webRtcServer      = null;
window.onload         = function() {
    webRtcServer      = new WebRtcStreamer("video","http://192.168.21.223:8000");
webRtcServer.connect("rtsp://admin:admin@192.168.21.224:554/Streaming/Channels/101?transportmode=unicast&amp;profile=Profile_1");
```

其中192.168.21.223为第一步的服务器IP，rtsp://...为摄像头RTSP流地址，可以通过ONVIF工具探测，需要保证服务器可以访问到摄像头IP

### 步骤3. 部署前端代码

可以将`adapter.min.js`、`index.html`、`webrtcstremer.js`放到WEB容器下，或者使用下面命令可以快速启动一个容器

```
python -m SimpleHTTPServer 18080

```

这里需要注意的是，一定要把上述文件放在WEB容器，如果直接打开是无法播放

### 步骤4. 浏览器调看视频（建议使用Chrome）

在浏览器上输入WEB容器地址即可播放，测试延时在1秒内，而且经过长时间测试，延时没有偏差。

```
http://localhost:18080/
```

![](https://github.com/smallmuou/rtsp-webrtc-demo/blob/main/demo.png)
