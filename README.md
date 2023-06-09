# rtsp-webrtc-demo

WEB浏览器要调看摄像头视频，通常有flash插件、HLS拉流、WEBRTC 3种方式，随着flash被禁用，若要实现低延时需要使用到WEBRTC，很多主流浏览器已经支持该框架，本文旨在利用webrtcstremer搭建一个DEMO。

## 安装步骤

### 步骤1. 安装webrtc-streamer服务

安装docker，若已安装跳过该步骤
```
yum install docker
systemctl start docker
```

安装webrtc-streamer服务
```
docker pull mpromonent/webrtc-streamer
docker run -itd -p 8000:8000 --network=host --name webrtcstremer mpromonent/webrtc-streamer
```
这里需要增加--network=host，否则会把docker容器的IP返回给WEB前端，这样就无法建立通讯

### 步骤2. 修改index.html中的服务地址和RTSP流地址

```
var webRtcServer      = null;
window.onload         = function() {
    webRtcServer      = new WebRtcStreamer("video","http://192.168.21.223:8000");
webRtcServer.connect("rtsp://admin:admin@192.168.21.224:554/Streaming/Channels/101?transportmode=unicast&amp;profile=Profile_1");
```

其中192.168.21.223为第一步的服务器IP，rtsp://...为摄像头RTSP流地址，可以通过ONVIF工具探测，需要保证服务器可以访问到摄像头IP

### 步骤3. 部署前端代码

可以将adapter.min.js、index.html、webrtcstremer.js文件放到WEB容器下，或者使用下面命令可以快速启动一个容器

```
python -m SimpleHTTPServer 18080
```

这里需要注意的是，一定要把上述文件放在WEB容器，如果直接打开是无法播放

若要部署到NGINX容器下，可以参考如下配置：

/etc/nginx/conf.d/default.conf
```
server {
  listen 443 ssl;
  server_name  xxxxxxx;
  ssl_certificate /etc/nginx/ssl/xxxxxx.pem;
  ssl_certificate_key /etc/nginx/ssl/xxxxxx.key;
  ssl_session_timeout 5m;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
  ssl_prefer_server_ciphers on;


location /rtsp2webrtc {
  root /var/www/html;
  index index.html;
}

location / {
  proxy_pass http://localhost:8000;
}

}
```
PS: xxxxxxx为你的域名及证书，部署NGINX后，index.html中new WebRtcStreamer需要修改为HTTPS地址

### 步骤4. 浏览器调看视频（建议使用Chrome）

在浏览器上输入WEB容器地址即可播放，测试延时在1秒内，而且经过几个小时长时间测试，基本没有延时，服务器CPU占用55%（单核）。

```
http://localhost:18080/
```

![](https://github.com/smallmuou/rtsp-webrtc-demo/blob/main/demo.jpg)

### 附：流程图

通过对js源码分析梳理出如下流程图：

![](https://github.com/smallmuou/rtsp-webrtc-demo/blob/main/webrtc-steamer-workflow.jpg)
