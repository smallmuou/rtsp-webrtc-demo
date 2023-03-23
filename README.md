# rtsp-webrtc-demo

WEB浏览器要调看摄像头视频，通常有flash插件、HLS拉流、WEBRTC 3种方式，随着flash被禁用，若要实现低延时，需要使用到WEBRTC，很多浏览器已经支持该框架，本文旨在利用webrtcstremer搭建一个DEMO

## 安装步骤

##### 1. 安装服务

```
docker pull mpromonent/webrtc-streamer
docker run -itd -p 8000:8000 --name webrtcstremer mpromonent/webrtc-streamer
```
若服务器没有安装docker，先使用yum或者apt安装

##### 2. 修改index.html中的转换单元服务地址和RTSP流地址

```
var webRtcServer      = null;
window.onload         = function() {
    webRtcServer      = new WebRtcStreamer("video","http://192.168.21.223:8000");
webRtcServer.connect("rtsp://admin:admin@192.168.21.224:554/Streaming/Channels/101?transportmode=unicast&amp;profile=Profile_1");
```

其中192.168.21.223为服务器IP，rtsp://...为摄像头RTSP流地址，可以通过ONVIF工具探测，需要保证服务器可以访问到摄像头IP

##### 3. 部署前端代码

可以将`adapter.min.js`、`index.html`、`webrtcstremer.js`放到WEB容器下，或者使用下面命令可以快速启动一个容器

```
python -m SimpleHTTPServer 18080

```

##### 4. 浏览器调看视频

在浏览器上输入WEB容器地址即可播放，测试下延时在1秒内。

```
http://localhost:18080/
```
