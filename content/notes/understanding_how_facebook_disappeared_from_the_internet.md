---
title: "翻译::了解Facebook是如何从互联网上消失的"
date: 2021-10-05T15:54:34+08:00
draft: false
---

原文: [Understanding How Facebook Disappeared from the Internet](https://blog.cloudflare.com/october-2021-facebook-outage/)

翻译: [DeepL 翻译](https://www.deepl.com/translator) 和我

<img src="https://blog.cloudflare.com/content/images/2021/10/Understanding-how-Facebook-disappeared-from-the-Internet-header-on-blog--1-.png" width="77%">

"Facebook 不可能瘫痪，对吗？"，让我们思考一秒钟。

今天 16:51UTC，我们开了一个内部事件，题为 "Facebook DNS 查询返回 SERVFAIL"，因为我们担心我们的 DNS 解析器 1.1.1.1 出了问题。但当我们准备在我们的公共状态页面上发布时，我们意识到发生了其他更严重的事情。

社交媒体迅速爆发，报道了我们的工程师也迅速确认的情况。事实上，Facebook 及其附属服务 WhatsApp 和 Instagram 都已瘫痪。他们的 DNS 名称停止解析，他们的基础设施 IP 也无法访问。这就像有人一下子从他们的数据中心 "拔掉了电缆"，并将他们与互联网断开连接。

这怎么可能呢？

### 来自 Facebook 的更新

Facebook 现在发表了一篇[博文](https://engineering.fb.com/2021/10/04/networking-traffic/outage/)，介绍了内部发生的一些细节。在外部，我们看到了这篇文章中概述的 BGP 和 DNS 问题，但问题实际上始于一个影响整个内部骨干网的配置变化。这导致了 Facebook 和其他属性的消失，而 Facebook 内部的员工也很难再获得服务。

现在来看看我们从外面看到的情况。

### 认识 BGP

[BGP](https://www.cloudflare.com/learning/security/glossary/what-is-bgp/) 是边界网关协议的缩写。它是一种在互联网上的自治系统（AS）之间交换路由信息的机制。使互联网运行的大型路由器拥有庞大的、不断更新的可能路由列表，这些路由可用于将每个网络数据包传送到其最终目的地。没有 BGP，互联网路由器就不知道该怎么做，互联网也就无法运行。

互联网实际上是一个网络的网络，它是由 BGP 捆绑在一起的。BGP 允许一个网络（例如 Facebook）向构成互联网的其他网络宣传其存在。当我们写到 Facebook 没有宣传它的存在时，ISP 和其他网络就找不到 Facebook 的网络，因此它就不可用。

各个网络都有一个 ASN：一个自治系统号码。一个自治系统（AS）是一个具有统一的内部路由策略的单独网络。一个 AS 可以发起前缀（说他们控制着一组 IP 地址），以及过境前缀（说他们知道如何到达特定的 IP 地址组）。

Cloudflare 的 ASN 是 AS13335。每个 [ASN](https://www.peeringdb.com/asn/13335) 都需要使用 BGP 向互联网公布其前缀路由；否则，没有人会知道如何连接以及在哪里找到我们。

我们的[学习中心](https://www.cloudflare.com/learning/)对什么是 [BGP](https://www.cloudflare.com/learning/security/glossary/what-is-bgp/) 和 [ASN](https://www.cloudflare.com/learning/network-layer/what-is-an-autonomous-system/) 以及它们如何工作有一个很好的概述。

在这个简化图中，你可以看到互联网上的六个自治系统，以及一个数据包从起点到终点可能使用的两条路线。AS1→AS2→AS3 是最快的，AS1→AS6→AS5→AS4→AS3 是最慢的，但如果第一条失败，也可以使用。

<img src="https://blog.cloudflare.com/content/images/2021/10/image5-9.png" width="50%">

在 16 时 58 分，我们注意到，Facebook 已经停止公布其 DNS 前缀的路由。这意味着，至少，Facebook 的 DNS 服务器是不可用的。正因为如此，Cloudflare 的 1.1.1.1 DNS 解析器无法再响应查询 facebook.com 或 instagram.com 的 IP 地址。

```bash
route-views>show ip bgp 185.89.218.0/23
% Network not in table
route-views>

route-views>show ip bgp 129.134.30.0/23
% Network not in table
route-views>
```

同时，其他的 Facebook IP 地址仍在路由中，但并不特别有用，因为没有 DNS，Facebook 和相关服务实际上是不可用的。

```txt
route-views>show ip bgp 129.134.30.0
BGP routing table entry for 129.134.0.0/17, version 1025798334
Paths: (24 available, best #14, table default)
  Not advertised to any peer
  Refresh Epoch 2
  3303 6453 32934
    217.192.89.50 from 217.192.89.50 (138.187.128.158)
      Origin IGP, localpref 100, valid, external
      Community: 3303:1004 3303:1006 3303:3075 6453:3000 6453:3400 6453:3402
      path 7FE1408ED9C8 RPKI State not found
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
route-views>
```

我们跟踪我们在全球网络中看到的所有 BGP 更新和公告。在我们的规模下，我们收集的数据让我们看到了互联网是如何连接的，以及流量要从哪里流向地球上的各个地方。

BGP UPDATE 消息通知路由器你对前缀广告所做的任何改变，或者完全撤销前缀。在检查我们的时间序列 BGP 数据库时，我们可以清楚地看到从 Facebook 收到的更新数量。通常情况下，这个图表是相当安静的：Facebook 不会每分钟对其网络进行大量的改变。

但在 UTC 时间 15:40 左右，我们看到 Facebook 的路由变化达到了一个高峰。这就是麻烦开始的时候。

<img src="https://blog.cloudflare.com/content/images/2021/10/image4-10.png" width="100%">

如果我们把这个观点按照路由的公布和撤销来划分，我们就能更好地了解发生了什么。路由被撤销，Facebook 的 DNS 服务器下线，问题发生一分钟后，Cloudflare 的工程师在一个房间里想，为什么 1.1.1.1 不能回答 facebook.com，并担心这是我们系统的某种故障。

<img src="https://blog.cloudflare.com/content/images/2021/10/image3--2-.png" width="100%">

随着这些退出，Facebook 及其网站实际上已经与互联网断开了联系。

### DNS 受到影响

作为这一事件的直接后果，世界各地的 DNS 解析器停止了对其域名的解析。

```bash
➜  ~ dig @1.1.1.1 facebook.com
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 31322
;facebook.com.			IN	A
➜  ~ dig @1.1.1.1 whatsapp.com
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 31322
;whatsapp.com.			IN	A
➜  ~ dig @8.8.8.8 facebook.com
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 31322
;facebook.com.			IN	A
➜  ~ dig @8.8.8.8 whatsapp.com
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 31322
;whatsapp.com.			IN	A
```

他的发生是因为 DNS，像互联网上的许多其他系统一样，也有其路由机制。当有人在浏览器中键入https://facebook.com URL 时，负责将域名翻译成实际 IP 地址进行连接的 DNS 解析器，首先检查它的缓存中是否有东西并使用它。如果没有，它就试图从域名服务器中获取答案，通常由拥有该域名的实体托管。

如果域名服务器无法到达，或者由于其他原因而无法响应，那么就会返回一个 SERVFAIL，浏览器就会向用户发出一个错误。

同样，我们的学习中心对 DNS 的工作原理提供了一个很好的[解释](https://www.cloudflare.com/learning/dns/what-is-dns/)。

<img src="https://blog.cloudflare.com/content/images/2021/10/image8-7.png">

由于 Facebook 停止通过 BGP 宣布他们的 DNS 前缀路由，我们和其他人的 DNS 解析器没有办法连接到他们的名字服务器。因此，1.1.1.1、8.8.8.8 和其他主要的公共 DNS 解析器开始发出（和缓存）SERVFAIL 响应。

但这还不是全部。现在，人类行为和应用逻辑开始发挥作用，导致另一个指数效应。一场额外的 DNS 流量的海啸随之而来。

发生这种情况的部分原因是，应用程序不接受一个错误的答案，并开始重试，有时是积极的，部分原因是最终用户也不接受一个错误的答案，并开始重新加载页面，或杀死并重新启动他们的应用程序，有时也是积极的。

这就是我们在 1.1.1.1 上看到的流量增加（请求数）。

<img src="https://blog.cloudflare.com/content/images/2021/10/image6-8.png">

因此，现在，由于 Facebook 和他们的网站是如此之大，我们的 DNS 解析器在全球范围内处理比平时多 30 倍的查询，并可能对其他平台造成延迟和超时问题。

幸运的是，1.1.1.1 是建立在免费、私有、快速（正如独立的 DNS 监测器 DNSPerf 可以证明的那样）和可扩展的基础上的，我们能够继续为我们的用户提供服务，而且影响最小。

我们绝大部分的 DNS 请求都能在 10 毫秒内得到解决。同时，p95 和 p99 百分位数的最小部分的响应时间增加了，可能是由于过期的 TTL 不得不求助于 Facebook 的名字服务器和超时。10 秒的 DNS 超时限制在工程师中是众所周知的。

<img src="https://blog.cloudflare.com/content/images/2021/10/image2--2-.png">

### 对其他服务的影响

人们寻找替代方案，希望了解更多信息或讨论发生的事情。当 Facebook 变得无法联系时，我们开始看到对 Twitter、Signal 和其他消息和社交媒体平台的 DNS 查询增加。

<img src="https://blog.cloudflare.com/content/images/2021/10/image1-12.png">

我们还可以从 Facebook 受影响的 ASN 32934 的 WARP 流量中看到这种不可达性的另一个副作用。这张图显示了每个国家从 15:45 UTC 到 16:45 UTC 的流量与三小时前相比的变化。在世界各地，进出 Facebook 网络的 WARP 流量完全消失了。

 <img src="https://blog.cloudflare.com/content/images/2021/10/image7-6.png">

### 互联网

今天的事件温和地提醒我们，互联网是一个非常复杂和相互依赖的系统，由数百万个系统和协议共同工作。信任、标准化和实体间的合作是使其为全球近 50 亿活跃用户工作的核心。

### 更新

在 21:00 UTC 左右，我们看到来自 Facebook 网络的 BGP 活动再次出现，在 21:17 UTC 达到峰值。

<img src="https://blog.cloudflare.com/content/images/2021/10/unnamed-3-2.png">

该图表显示了 DNS 名称 "facebook.com "在 Cloudflare 的 DNS 解析器 1.1.1.1 上的可用性。它在 15:50 UTC 左右停止可用，并在 21:20 UTC 返回。

<img src="https://blog.cloudflare.com/content/images/2021/10/unnamed-2.png">

毫无疑问，Facebook、WhatsApp 和 Instagram 的服务将需要进一步的时间才能上线，但截至 UTC 时间 21:28，Facebook 似乎已经重新连接到全球互联网，DNS 也重新工作。
