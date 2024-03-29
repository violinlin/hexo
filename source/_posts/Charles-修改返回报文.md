---
title: Charles--修改返回报文
photos:
  - '/img/pictures/picture17.jpg'
date: 2016-10-23 22:49:28
tags: [Charles,Android]
---


> [Charles](https://www.charlesproxy.com/)是一款很强大的抓包工具。除了经常使用到的抓取出入口数据外还有其它很强大的功能，例如：修改请求和返回报文、模拟网络环境、给服务器做简单压测等。今天主要和大家聊聊博主经常使用的功能--修改返回报文。

<!--more-->

![](/img/charles-logo.png)

## 通过映射方式修改

> Charles中有两种修改映射的方法`Map Local`和`Map Remote`，两种方法各有不同的用处。

### 本地映射`Map Local`
> 其实这种方法很好理解，就是修改映射方式让请求去读取我们本地的文件并返回，具体操作可以参考下面。

**选中一条网络请求-->右键-->Save Response** 将返回报文保存到本地磁盘

![enter image description here](/img/saveResponse.png)

**选中相同的请求-->右键-->Map Local** 将请求映射到刚才保存的返回报文

![enter image description here](/img/mapLocal.png)

设置完映射之后每次该请求都会从设置的本地文件中读取并返回数据，我们可以随心所欲的修改保存的报文达到调试接口的目的。
另外如果在开发初期，客户端和服务器端已经定义好了接口，但是服务器端还没有部署好环境。我们也可以通过这个方法把定义好的报文保存到本地，通过修改映射的方式来进行初步开发。

### 远程映射`Map Remote`

> 修改远程映射是通过修改请求的HOST来实现的，可以将请求切换到不同的环境（例如当前的包是正式环境，你想看看测试环境的修改有没有生效，可以不用再打一个测试环境的包，通过修改远程映射快速实现。也省的改手机的HOST的了）。

选中要修改的请求-->右键-->Map Remote 在Map To 栏中填写要映射到的地址的一些参数。

![enter image description here](/img/mapRemote.png)



## 通过断点修改
> 除了修改映射的方法，通过设置断点也可以实现想要的效果。和我们在IDE上设置断点一样，在断点处会进入调试模式，请求会暂时中断，这是我们可以进行一些自定义的操作。

首先开启断点模式，在Charles的面板上方将断点的图标点亮

![enter image description here](/img/duandian.png)

设置断点，选中要修改的请求-->右键-->BreakPoints （左面出现对勾表示设置成功）。设置完成后，Charles再次抓取到该请求时会自动进入到调试模式。

进入调试模式后可以修改请求报文，或者直接`Execute`进行下一步

![enter image description here](/img/breakPoints1.png)

请求报文发送成功后，Charles会拦截服务器返回的数据，这里可以通过`Edit Response`修改返回数据，或者`Execute`进行下一步
![enter image description here](/img/breakPoints2.png)

## 总结

上面的方法可以用来修改一般的网络请求，但如果数据做了加密处理看不到明文那用处就不大了。
需要注意一点，通过断点的方式会存在一定的问题。数据被拦截后，客户端的请求超时时长是不会停止计算的，如果没在设置的超时时间内返回数据本次请求也就按失败处理了。最后提一下博主通常使用修改本地映射的方法。希望这篇文章会给大家带来帮助。