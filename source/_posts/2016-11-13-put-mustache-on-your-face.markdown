---
layout: post
title: "mustache——实时给人脸贴胡子"
date: 2016-11-13 10:45:35 +0800
comments: true
categories: [Machine Learning]
tags: [opencv,websocket,https]
---

大约去年暑假时候，大把的时间精力又无事可做，就全放到倒腾这些百无一用的事情上来了。开始是看到一个项目，作者用php做了一个玩具，给用户上传的头像贴上性感的小胡子。当时就想，我可以做一个能够支持用户实时视频的，最后花了一周左右，也捣鼓出一个人模狗样的东西出来，就放在实验室的内网里，让师弟师妹门耍耍，后来也就慢慢忘了，十月底的时候，腾讯云的同事要征集一些小项目，把项目迁移到腾讯云，可以免费使用一年的云主机:). 哈哈，所以就有了此文。
<!--more-->
##  概述
给人脸贴上胡子，首先这个要有在背景中检测出人脸，并定位到口鼻位置，这部分工作要依赖目标检测算法，人脸检测的论文汗牛充栋，本文也不深究细节，可以利用现有的模型帮助快速实现功能。其次是实时图像的采集，利用了html5的getUserMedia来获取视频，采集到的视频回传到后端server后，server执行检测算法将检测到的贴图区域发送给浏览器，浏览器根据用户选好的胡子样式贴图。

本文将从人脸检测、实时传图以及后续的https改造这三个部分来分别介绍。

## 人脸检测
该模块的作用是在背景图中定位出适合贴胡子的区域。显而易见，该区域正好位于人的口鼻之间位置，检测到了口鼻就基本上可以推算出适合贴胡子的位置，为了保证结果的鲁棒性，可以先行检测出背景中的人脸位置，由于口鼻是位于脸部区域之内的，以此做一次粗筛，将false positive区域剔除。

OpenCV提供了一系列的检测算子与训练模型，我们还是奉行拿来主义，使用提供的级连分类器api：CascadeClassifier，训练好的haar特征在OpenCV的安装目录即可找到，我使用了fronal_face,mcs_mouth,mcs_nose三个特征数据，分别对应脸、口、鼻区域检测。筛选时候会判断口鼻位置是否在面部，然后根据口鼻位置来决定胡子的位置。

而最终选定的贴胡子位置区域，横轴(x)以鼻子为中心左侧一半宽度处，纵轴(y)口鼻正中间往下又1/4处；贴片高度固定为10，宽度约为嘴巴1.8倍(经验值)。

	if(len(self.mouth) > 0 and len(self.nose) > 0):
	    width = self.mouth[0][2]*1.8
	    height = 10
	    x = self.nose[0][0] + self.nose[0][2]*0.5 - 0.5*width
	    t = 0.5*(self.nose[0][1] + self.mouth[0][1])+0.25*(self.nose[0][3] + self.mouth[0][3])
	    y = t - 0.5*height
	    return int(x),int(y),int(width),int(height)

## 实时图像采集与传送
图像的采集采用了基于浏览器的方案，html5提供了getUserMedia()方法，可以方便地进行视频抓取，在获取了摄像头图片后，我们要实时往服务端发送，并获取贴图位置。这也是一种应答模式，使用http不间断地传送图像也可以实现，不过，对于这种客户端频繁更新数据请求的模式，websocket是更好的选择(
[When to use a HTTP call instead of a WebSocket (or HTTP 2.0)](https://blogs.windows.com/buildingapps/2016/03/14/when-to-use-a-http-call-instead-of-a-websocket-or-http-2-0/))。

一般来讲，webserver是不具备往client主动发数据的能力的(应答模式的短链接，回复即断掉当前链接)。websocket主要用于client－server需要维持长链接的情况——比如向client推送消息。不过，我们使用websocket的即时发送能力，将采集到的图像源源不断地发送给后端,而后端收到图图像后，使用上文中的检测算法计算出将要贴上胡子的区域，并回传给websocket client(浏览器)，浏览器根据收到的胡子位置，把胡子样式图片贴在图像的相应位置。

websocket默认支持的二进制数据是blob的，我们需要把html canvas元素转化为blob对象，所以使用了[JavaScript-Canvas-to-Blob](https://github.com/blueimp/JavaScript-Canvas-to-Blob)来做类型的转换。

	//即时传图
	update = function() {
		// 视频数据复制到画布中
	    ctx.drawImage(video, 0, 0, 320, 240);
	    // 转为blob 并发送给server
	    if(canvas.toBlob){
	        canvas.toBlob(function(blob) {
	            ws.send(blob);
	        }, 'image/jpeg');
	    }

	    // 贴胡子。openCvCoords为回传到贴图区域，mustache为选定的胡子图片
		if(typeof(openCvCoords) != "undefined")
	    {
	        if(openCvCoords[0] != -1)
	        {
	            ctx.drawImage(mustache,openCvCoords[0], openCvCoords[1], openCvCoords[2], openCvCoords[3]);
	        }
	    }    
	}


## https改造
不过，完成所有代码编写并部署后，你会发现只能够在本地运行，因为通过http远程访问getUserMedia已经在较新的chrome版本中被[废弃](https://sites.google.com/a/chromium.org/dev/Home/chromium-security/deprecating-powerful-features-on-insecure-origins)了，只有在https方式下才能继续使用这个特性。所以我们需要对站点进行适当改造，使其能够处理https请求。

首先是全站资源的https化，对用使用的外部js，辛好现在cdn厂商提供了http与https与自适应三种方式，可以使用后两者替换掉http url即可。同时，实时数据传输，采用了websocket的解决方案，websocket提供了ws与wss两种协议，使用wss告诉websocket client side 使用https方式连接 server side。

另外，还要证书的生成与签名(认证)。未认证的证书会被主流浏览器拦截，就想12306网站那样，提示用户不是私密连接。不过[Let’s Encrypt](https://letsencrypt.org/)提供了一种廉价的认证方式，用户将证书上传认证后可获得三个月的免费认证，三月后可继续续约。我使用了[acme-tiny](https://github.com/diafygi/acme-tiny)帮助生成认证证书

最后是web 框架的https支持。基本上主流框架都提供了https支持，tornado通过开启ssl_options指定签名证书与生成证书的私钥。

	http_server = tornado.httpserver.HTTPServer(Mustache(),ssl_options={
	    "certfile": os.path.join(os.path.abspath("cert/"), "chained.pem"),
	    "keyfile": os.path.join(os.path.abspath("cert/"), "domain.key"),
	    })

当使用了nginx作为反代的情况，配置nginx的ssl选项，client到nginx使用https，而nginx使用http访问上游webserver。

## 其他
附带[项目路径](https://github.com/tianyaqu/mustache.git)，欢迎继续发挥。。。

{% img /images/mustache_snapshot.png 性感小胡子 %}

