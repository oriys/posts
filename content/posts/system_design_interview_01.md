---
title: "系统设计::从零到一百万"
date: 2021-02-01T22:20:24+08:00
draft: false
keywords: ["系统设计", "可扩展性", "负载均衡", "数据库复制", "CDN"]
---

## 从零到一百万

设计一个支持数百万用户的系统是一个挑战，这是一个需要不断完善和无止境改进的历程。在本章中，我们将构建一个支持单个用户的系统，并逐步将其扩展到服务数百万用户。读完本章，你将掌握一手的技巧，帮助你破解系统设计的面试题。

**单服务器设置**

千里之行始于足下，构建一个复杂的系统也不例外。先从简单的东西开始，所有的东西都运行在一台服务器上。图 1-1 是单服务器设置的说明，所有的东西都在一台服务器上运行：Web 应用、数据库、缓存等。

<img src="../../../system_design_interview/index-6_1.jpg" width="50%"/>

为了理解这种设置，研究一下请求流程和流量来源是很有帮助的。我们先来看看请求流程（图 1-2）。

<img src="../../../system_design_interview/index-6_2.jpg" width="50%"/>

1. 用户通过域名访问网站，如 api.mysite.com。通常，域名系统（DNS）是由第三方提供的付费服务，而不是由我们的服务器托管。
2. 互联网协议（IP）地址返回给浏览器或移动应用。在本例中，返回的 IP 地址为 15.125.23.214。
3. 获得 IP 地址后，超文本传输协议（HTTP）[1]请求直接发送到您的网络服务器。
4. Web 服务器返回 HTML 页面或 JSON 响应进行渲染。

接下来，我们来看看流量来源。你的 Web 服务器的流量来自两个方面：Web 应用和移动应用。

- Web 应用：它使用服务器端语言（Java、Python 等）组合来处理业务逻辑、存储等，使用客户端语言（HTML 和 JavaScript）来进行展示。
- 移动应用。HTTP 协议是移动应用与 Web 服务器之间的通信协议。JavaScript 对象符号（JSON）由于其简单性，是常用的 API 响应格式来传输数据。JSON 格式的 API 响应示例如下所示。

```json
{
  "firstName": "John",
  "lastName": "Smith",
  "address": {
    "streetAddress": "21 2nd street",
    "city": "New York",
    "state": "NY",
    "postal Code": 10021
  },
  "phoneNumbers": ["212 555-1234", "646 555-4567"]
}
```

`GET /users/12 – Retrieve user object for id = 12`

**数据库**

随着用户群的增长，一台服务器是不够的，我们需要多台服务器：一台用于 web/移动流量，另一台用于数据库（图 1-3）。将 web/移动流量（web 层）和数据库（数据层）服务器分开，可以让它们独立扩展。

<img src="../../../system_design_interview/index-8_1.jpg" width="50%"/>

**使用哪种数据库?**

你可以选择传统的关系型数据库和非关系型数据库。让我们来看看它们的区别。

关系型数据库也叫关系型数据库管理系统（RDBMS）或 SQL 数据库。最流行的有 MySQL、Oracle 数据库、PostgreSQL 等。关系型数据库以表和行来表示和存储数据。你可以在不同的数据库表之间使用 SQL 进行连接操作。

非关系型数据库也叫 NoSQL 数据库。常用的有 CouchDB、Neo4j、Cassandra、HBase、Amazon DynamoDB 等。[2]. 这些数据库分为四类：键值存储、图存储、列存储和文档存储。在非关系型数据库中，一般不支持 Join 操作。

对于大多数开发人员来说，关系型数据库是最好的选择，因为关系型数据库已经存在了 40 多年，而且从历史上看，关系型数据库运行良好。然而，如果关系型数据库不适合你的特定用例，那么探索关系型数据库之外的东西是至关重要的。在以下情况下，非关系型数据库可能是正确的选择。

- 你的应用需要超低的延迟
- 你的数据是非结构化的，或者你没有任何关系型数据。
- 你只需要序列化和反序列化数据（JSON、XML、YAML 等）。
- 你需要存储大量的数据。

**垂直扩展与水平扩展**

垂直扩展，简称为 "扩容"，指的是为服务器增加更多功率（CPU、RAM 等）的过程。

水平扩展，称为 "scale-out"，允许您通过向资源池中添加更多的服务器来扩展。

当流量较低时，垂直扩展是一个很好的选择，垂直扩展的简单性是其主要优势。不幸的是，它有严重的局限性。

- 垂直扩展有一个硬件限制。不可能在一台服务器上增加无限的 CPU 和内存。
- 垂直扩展没有故障转移和冗余。如果一台服务器出现故障，网站/应用也会随之完全瘫痪。

由于垂直扩展的局限性，水平扩展对于大规模的应用更为理想。
在之前的设计中，用户是直接连接到网站服务器的。如果 Web 服务器离线，用户将无法访问网站。在另一种情况下，如果很多用户同时访问 Web 服务器，达到了 Web 服务器的负载极限，用户一般会出现响应速度较慢或无法连接到服务器的情况。负载均衡是解决这些问题的最佳技术。

**负载均衡**

负载均衡将传入的流量均匀地分配给定义在负载均衡集群中的 Web 服务器。图 1-4 显示了负载均衡器的工作原理。

<img src="../../../system_design_interview/index-10_1.jpg" width="50%"/>

如图 1-4 所示，用户直接连接到负载均衡器的公网 IP。通过这种设置，Web 服务器已经无法被客户端直接访问了。为了提高安全性，服务器之间的通信采用私有 IP。私有 IP 是指只有同一网络中的服务器之间才能到达的 IP 地址，但是，通过互联网是无法到达的。负载均衡器通过私有 IP 与 Web 服务器进行通信。

在图 1-4 中，增加了一个负载均衡器和第二台 Web 服务器后，我们成功解决了无故障切换问题，提高了 Web 层的可用性。下面将详细说明。

- 如果服务器 1 离线，所有的流量将被路由到服务器 2。这样可以防止网站离线。我们也会在服务器池中增加一个新的健康网站服务器来平衡负载。
- 如果网站流量快速增长，两台服务器不足以处理流量，负载均衡器可以优雅地处理这个问题。你只需要向 Web 服务器池添加更多的服务器，负载平衡器就会自动开始向它们发送请求。

现在 web 层看起来不错，那数据层呢？目前的设计只有一个数据库，所以它不支持故障转移和冗余。数据库复制是解决这些问题的一个常用技术。让我们来看看。

**数据库复制**

引自维基百科。"数据库复制可用于许多数据库管理系统，通常原始数据库(主数据库)和副本(从数据库)之间存在主/从关系"[3]。

主数据库一般只支持写操作。从数据库从主数据库获取数据的副本，只支持读操作。所有的插入、删除、更新等数据修改命令都必须发送到主数据库。大多数应用对读与写的比例要求更高，因此，系统中从数据库的数量通常大于主数据库的数量。图 1-5 显示了一个主数据库与多个从数据库的情况。

<img src="../../../system_design_interview/index-12_1.jpg" width="50%"/>

**数据库复制的优势**

- 性能更好。在主从模式中，所有写入和更新都发生在主节点上；而读操作则分布在从节点上。这种模式可以提高性能，因为它允许并行处理更多的查询。
- 可靠性。如果你的一个数据库服务器被自然灾害摧毁，如台风或地震，数据仍然会被保存下来。您不必担心数据丢失，因为数据是在多个地点复制的。
- 高可用性。通过在不同地点复制数据，即使数据库离线，您的网站仍然可以运行，因为您可以访问存储在另一个数据库服务器的数据。

在上一节中，我们讨论了负载均衡器如何帮助提高系统的可用性。我们在这里提出同样的问题：如果其中一个数据库离线了怎么办？图 1-5 中讨论的架构设计可以处理这种情况。

- 如果只有一个从数据库可用，而它又脱机了，读操作将被暂时导向主数据库。一旦发现问题，新的从数据库将取代旧的数据库。如果有多个从数据库可用，读取操作将被重定向到其他健康的从数据库。新的数据库服务器将取代旧的数据库。
- 如果主数据库下线，一个从数据库将被提升为新的主数据库。所有的数据库操作将暂时在新的主数据库上执行。新的从数据库将立即取代旧的数据库进行数据复制。在生产系统中，推广新的主数据库比较复杂，因为从数据库中的数据可能不是最新的。缺少的数据需要通过运行数据恢复脚本来更新。虽然其他一些复制方法，如多主站和循环复制可以帮助我们，但这些设置比较复杂；而且它们的讨论也超出了本书的范围。有兴趣的读者可以参考列出的参考资料[4][5]。

图 1-6 是增加负载均衡器和数据库复制后的系统设计。

<img src="../../../system_design_interview/index-14_1.jpg" width="50%"/>

我们来看一下设计。

- 用户从 DNS 获取负载均衡器的 IP 地址。
- 用户用这个 IP 地址连接负载均衡器。
- HTTP 请求被路由到服务器 1 或服务器 2。
- Web 服务器从从属数据库读取用户数据。
- Web 服务器将任何数据修改操作路由到主数据库。这包括写入、更新和删除操作。

现在，你已经对网络和数据层有了坚实的了解，是时候提高加载/响应时间了。这可以通过添加缓存层和将静态内容（JavaScript/CSS/图片/视频文件）转移到内容传输网络（CDN）来实现。

**缓存**

缓存是一个临时的存储区域，它将昂贵的响应结果或频繁访问的数据存储在内存中，以便后续的请求能够更快地得到服务。如图 1-6 所示，每次加载新的网页时，都会执行一次或多次数据库调用来获取数据。由于反复调用数据库，应用性能受到很大影响。缓存可以缓解这个问题。

**缓存层**

缓存层是一个临时的数据存储层，比数据库快得多。单独设置缓存层的好处包括更好的系统性能，能够减少数据库的工作负载，以及能够独立地扩展缓存层。图 1-7 显示了一个缓存服务器的可能设置。

<img src="../../../system_design_interview/index-15_1.jpg" />

在收到请求后，Web 服务器首先检查缓存是否有可用的响应。如果有，它就把数据发回给客户端。如果没有，它就查询数据库，将响应存储在缓存中，然后再发回给客户端。这种缓存策略称为读通式缓存。根据数据类型、大小和访问模式，还有其他缓存策略可供选择。之前的一项研究解释了不同缓存策略的工作原理[6]。

与缓存服务器的交互很简单，因为大多数缓存服务器都提供了通用编程语言的 API。下面的代码片段展示了典型的 Memcached API。

```js
SECONDS = 1;
cache.set("myKey", "hi there", 3600 * SECONDS);
cache.get("myKey");
```

**使用缓存的注意事项**

以下是使用缓存系统的几个注意事项。

- 决定何时使用缓存。当数据经常被读取但不经常被修改时，考虑使用缓存。由于缓存数据存储在易失性内存中，因此缓存服务器并不是持久化数据的理想选择。例如，如果缓存服务器重新启动，内存中的所有数据都会丢失。因此，重要的数据应该保存在持久化数据存储中。
- 过期策略。实施过期策略是一个很好的做法。一旦缓存数据过期，它就会从缓存中删除。当没有过期策略时，缓存数据将永久保存在内存中。建议不要把过期日期定得太短，否则会导致系统过于频繁地从数据库中重新加载数据。同时，建议不要把有效期做得太长，因为数据会变得陈旧。
- 一致性。这涉及到保持数据存储和缓存的同步。由于对数据存储和缓存的数据修改操作不在一个事务中，所以会发生不一致的情况。当跨多个区域扩展时，保持数据存储和缓存之间的一致性是一个挑战。更多细节，请参考 Facebook 发布的题为 "Scaling Memcache at Facebook "的论文[7]。
- 缓解故障。单个缓存服务器代表了一个潜在的单点故障（SPOF），在维基百科中的定义如下。"单点故障(SPOF)是指系统的一部分，如果它发生故障，将使整个系统停止工作"[8]。因此，建议在不同的数据中心设置多台缓存服务器，以避免 SPOF 的发生。另一种推荐的方法是按一定的百分比超额提供所需的内存。这样可以在内存使用量增加时提供一个缓冲区。
- 驱逐政策。一旦缓存满了，任何向缓存添加项目的请求都可能导致现有项目被删除。这就是所谓的缓存驱逐。最少最近使用（LRU）是最流行的缓存驱逐策略。其他的驱逐策略，如最不常用(LFU)或先进先出(FIFO)，可以满足不同的用例。

<img src="../../../system_design_interview/index-16_1.jpg" width="50%"/>

**内容传输网络(CDN)**

CDN 是一个由地理上分散的服务器组成的网络，用于传输静态内容。CDN 服务器缓存静态内容，如图片、视频、CSS、JavaScript 文件等。

动态内容缓存是一个比较新的概念，超出了本书的范围。它可以实现基于请求路径、查询字符串、Cookie 和请求头的 HTML 页面的缓存。关于这方面的内容，请参考参考资料[9]中提到的文章。本书主要介绍如何使用 CDN 来缓存静态内容。

下面是 CDN 的顶层工作原理：当用户访问一个网站时，离用户最近的 CDN 服务器将提供静态内容。直观地说，用户离 CDN 服务器越远，网站的加载速度越慢。例如，如果 CDN 服务器在旧金山，那么洛杉矶的用户将比欧洲的用户更快地获得内容。图 1-9 是一个很好的例子，它显示了 CDN 如何改善加载时间。

<img src="../../../system_design_interview/index-17_1.jpg" width="50%"/>

图 1-10 展示了 CDN 的工作流程。

<img src="../../../system_design_interview/index-17_2.jpg" width="75%"/>

1. 用户 A 试图通过图片 URL 获取 image.png。该 URL 的域名由 CDN 提供商提供。以下两个图片 URL 是用来演示 Amazon 和 Akamai CDN 上的图片 URL 的示例。
   - https://mysite.cloudfront.net/logo.jpg
   - https://mysite.akamai.com/image-manager/img/logo.jpg
2. 如果 CDN 服务器的缓存中没有 image.png，CDN 服务器就会向原点请求该文件，这个原点可以是 Web 服务器，也可以是 Amazon S3 等在线存储。
3. 原点将 image.png 返回给 CDN 服务器，其中包括可选的 HTTP 头 Time-to-Live（TTL），它描述了图像被缓存的时间。
4. CDN 缓存图像并将其返回给用户 A，图像一直在 CDN 中缓存，直到 TTL 过期。
5. 用户 B 发送一个请求来获取相同的图像。
6. 只要 TTL 没有过期，图像就会从缓存中返回。

**使用 CDN 的注意事项**

- 费用。CDN 由第三方供应商运营，您需要为进出 CDN 的数据传输付费。缓存不经常使用的资产并不能提供显著的好处，所以你应该考虑将它们移出 CDN。
- 设置一个合适的缓存到期时间。对于时间敏感的内容，设置一个缓存到期时间很重要。缓存到期时间既不能太长也不能太短。如果太长，内容可能过期。如果太短，可能会导致从源服务器到 CDN 的内容重复重载。
- CDN 回源。你应该考虑你的网站/应用如何应对 CDN 故障。如果 CDN 出现临时中断，客户端应该能够检测到问题，并从源服务器请求资源。
- 使文件无效。您可以通过执行以下操作之一，在文件过期前从 CDN 中删除文件。
  - 使用 CDN 供应商提供的 API 使 CDN 对象无效。
  - 使用对象版本化来服务对象的不同版本。要对对象进行版本管理，可以在 URL 中添加一个参数，例如版本号。例如，在查询字符串中添加版本号 2：image.png?v=2。

图 1-11 是添加 CDN 和缓存后的设计。

<img src="../../../system_design_interview/index-19_1.jpg" width="75%"/>

1. 静态资产（JS、CSS、图片等）不再由 Web 服务器提供服务。它们从 CDN 获取，以获得更好的性能。
2. 通过缓存数据，减轻了数据库的负载。

**无状态 Web 层**

现在是时候考虑横向扩展 Web 层了。为此，我们需要将状态（例如用户会话数据）移出 web 层。一个好的做法是将会话数据存储在持久性存储中，如关系型数据库或 NoSQL。集群中的每个 Web 服务器都可以从数据库中访问状态数据。这就是所谓的无状态 Web 层。

**有状态架构**

有状态服务器和无状态服务器有一些关键的区别。有状态的服务器会记住客户的数据（状态），从一个请求到下一个请求。无状态服务器不保留状态信息。

图 1-12 显示了一个有状态架构的例子。

<img src="../../../system_design_interview/index-20_1.jpg" width="75%"/>

在图 1-12 中，用户 A 的会话数据和配置文件图像存储在服务器 1 中。要对用户 A 进行身份验证，HTTP 请求必须路由到服务器 1。如果向服务器 2 等其他服务器发送请求，认证将失败，因为服务器 2 不包含用户 A 的会话数据。同样，所有来自用户 B 的 HTTP 请求必须路由到服务器 2；所有来自用户 C 的请求必须发送到服务器 3。

问题是来自同一客户端的每个请求都必须路由到同一个服务器。这可以通过大多数负载均衡器中的粘性会话来实现[10]；然而，这增加了开销。使用这种方法增加或删除服务器要困难得多。处理服务器故障也是一个挑战。

**无状态结构**

图 1-13 为无状态架构。

<img src="../../../system_design_interview/index-21_1.jpg" width="66%"/>

在这种无状态架构中，用户的 HTTP 请求可以发送到任何 Web 服务器上，服务器从共享数据存储中获取状态数据。状态数据存储在共享数据存储中，不受 Web 服务器的影响。无状态系统更简单、更健壮、可扩展。

图 1-14 为无状态网络层的更新设计。

<img src="../../../system_design_interview/index-22_1.jpg" width="75%"/>

在图 1-14 中，我们将会话数据从 Web 层移出，并将其存储在持久化数据存储中。共享数据存储可以是关系型数据库、Memcached/Redis、NoSQL 等。选择 NoSQL 数据存储是因为它易于扩展。自动伸缩是指根据流量负载自动增加或删除 Web 服务器。当状态数据从 web 服务器中取出后，根据流量负载增加或删除服务器，就可以轻松实现 web 层的自动伸缩。

您的网站发展迅速，在国际上吸引了大量的用户。为了提高可用性，并在更广泛的地域提供更好的用户体验，支持多个数据中心至关重要。

**数据中心**

图 1-15 是一个有两个数据中心的设置实例。在正常运行中，用户会被 geoDNS-routed，也就是地理路由，到最近的数据中心，美东地区的流量分成 x%，美西地区的流量分成(100 - x)%，geoDNS 是一种 DNS 服务，可以根据用户的位置将域名解析到 IP 地址。

<img src="../../../system_design_interview/index-23_1.jpg" width="75%"/>

在任何重大的数据中心中断的情况下，我们将所有的流量引导到一个健康的数据中心。在图 1-16 中，数据中心 2（US-West）处于离线状态，100%的流量被引导到数据中心 1（US-East）。

<img src="../../../system_design_interview/index-24_1.jpg" width="75%"/>

要实现多数据中心的设置，必须解决几个技术难题。

- 流量重定向。需要有效的工具来引导流量到正确的数据中心。GeoDNS 可以根据用户所在的位置，将流量引导到最近的数据中心。
- 数据同步。来自不同地区的用户可能使用不同的本地数据库或缓存。在故障转移情况下，流量可能会被路由到数据不可用的数据中心。一个常见的策略是在多个数据中心之间复制数据。之前的一项研究展示了 Netflix 如何实现异步多数据中心复制[11]。
- 测试和部署。对于多数据中心的设置，在不同的位置测试你的网站/应用是很重要的。自动部署工具对于在所有数据中心保持服务的一致性至关重要[11]。

为了进一步扩展我们的系统，我们需要对系统的不同组件进行解耦，以便它们可以独立地进行扩展。消息队列是许多现实世界的分布式系统采用的一个关键策略，以解决这个问题。

**消息队列**

消息队列是一个持久的组件，存储在内存中，支持异步通信。它作为一个缓冲区，分发异步请求。消息队列的基本架构很简单。输入服务，称为生产者/发布者，创建消息，并将它们发布到消息队列中。其他服务或服务器，称为消费者/订阅者，连接到队列，并执行由消息定义的操作。该模型如图 1-17 所示。

<img src="../../../system_design_interview/index-25_1.jpg" width="66%"/>

解耦使得消息队列成为构建可扩展和可靠应用的首选架构。通过消息队列，当消费者无法处理消息时，生产者可以将消息发布到队列中。即使在生产者不可用时，消费者也可以从队列中读取消息。

考虑以下用例：你的应用程序支持照片定制，包括裁剪、锐化、模糊等。这些定制任务需要时间来完成。在图 1-18 中，Web 服务器将照片处理作业发布到消息队列中。照片处理工作者从消息队列中接取作业，并异步执行照片定制任务。生产者和消费者可以独立伸缩。当队列的规模变大时，会增加更多的工人以减少处理时间。但是，如果队列大部分时间是空的，可以减少工人的数量。

<img src="../../../system_design_interview/index-25_2.jpg" width="66%"/>

**记录、指标、自动化**

当与一个在少数服务器上运行的小型网站合作时，日志，指标和自动化支持是良好的实践，但不是必需品。然而，现在你的网站已经成长为服务于一个大型企业，投资于这些工具是必不可少的。

日志记录。监控错误日志很重要，因为它有助于识别系统中的错误和问题。您可以在每个服务器级别监控错误日志，或者使用工具将它们汇总到一个集中的服务，以便于搜索和查看。

指标。收集不同类型的指标有助于我们获得业务洞察力，了解系统的健康状况。以下一些指标是有用的。

- 主机级指标： CPU、内存、磁盘 I/O 等。
- 汇总级指标：例如，整个数据库层、缓存层的性能等。
- 关键业务指标：日活跃用户、留存率、收入等。

自动化。当一个系统变得庞大而复杂时，我们需要建立或利用自动化工具来提高生产力。持续集成是一个很好的实践，通过自动化来验证每一个代码的签入，让团队及早发现问题。此外，将构建、测试、部署等过程自动化，可以显著提高开发人员的生产力。

**添加消息队列和不同的工具**

图 1-19 为更新后的设计。由于篇幅所限，图中只显示了一个数据中心。

1. 设计中加入了消息队列，这有助于使系统更加松散耦合和故障恢复能力。
2. 包含了日志、监控、指标和自动化工具。

<img src="../../../system_design_interview/index-27_1.jpg" width="66%"/>

随着数据每天的增长，你的数据库会越来越过载。是时候扩大数据层的规模了。

**数据库扩展**

数据库的扩展有两大方法：垂直扩展和水平扩展。

**垂直扩展**

垂直扩展，也叫扩大规模，就是通过给现有的机器增加更多的功率（CPU、RAM、DISK 等）来进行扩展。有一些强大的数据库服务器。根据 Amazon Relational Database Service(RDS)[12]，你可以得到一个 24TB 内存的数据库服务器。这种强大的数据库服务器可以存储和处理大量的数据。例如，stackoverflow.com 在 2013 年有超过 1000 万的月度独立访客，但它只有 1 个主数据库[13]。然而，垂直扩展也有一些严重的缺点。

- 你可以在数据库服务器上增加更多的 CPU、RAM 等，但有硬件限制。如果你有大量的用户群，单台服务器是不够的。
- 单点故障的风险较大。
- 垂直扩展的整体成本很高。强大的服务器要贵得多。

**水平扩展**

水平扩展，也称为 sharding，是增加更多服务器的做法。图 1- 20 比较了垂直扩展和水平扩展。

<img src="../../../system_design_interview/index-28_1.jpg" width="66%"/>

Sharding 将大型数据库分离成更小、更容易管理的部分，称为 shard。每个分片共享相同的模式，尽管每个分片上的实际数据对该分片来说是独一无二的。

图 1-21 显示了一个分片数据库的例子。用户数据是根据用户 ID 分配到数据库服务器上的。任何时候访问数据时，都会使用一个哈希函数来找到相应的分片。在我们的例子中，user_id % 4 被用作哈希函数。如果结果等于 0，则 0 号分片被用来存储和获取数据。如果结果等于 1，则使用分片 1。同样的逻辑也适用于其他分片。

<img src="../../../system_design_interview/index-29_1.jpg" width="50%"/>

图 1-22 是分片数据库中的用户表。

<img src="../../../system_design_interview/index-29_2.jpg" width="50%"/>

在实施分区策略时，需要考虑的最重要因素是分区密钥的选择。分区键（称为分区键）由一列或多列组成，决定数据的分布方式。如图 1-22 所示，"user_id "就是 sharding 键。通过 sharding 键，可以将数据库查询路由到正确的数据库，从而有效地检索和修改数据。在选择 sharding 键时，最重要的一个标准是选择一个能够均匀分布数据的键。

Sharding 是一种很好的扩展数据库的技术，但它远不是一个完美的解决方案。它给系统带来了复杂性和新的挑战。

**重置数据**

在以下情况下需要重新 sharding 数据：

1. 由于快速增长，单个分片无法再容纳更多的数据。
2. 由于数据分布不均，某些分片可能比其他分片更快地出现分片耗尽。当分片耗尽时，需要更新 sharding 函数，并移动数据。

第 5 章将讨论的一致性哈希是解决这个问题的常用技术。

**名人问题**

这也被称为热点问题。对特定分片的过度访问可能导致服务器过载。想象一下，Katy Perry、Justin Bieber 和 Lady Gaga 的数据最终都会出现在同一个分片上。对于社交应用来说，该分片将因读取操作而不堪重负。为了解决这个问题，我们可能需要为每个名人分配一个 shard。每个分片甚至可能需要进一步分区。

**使用和去范式化**

一旦一个数据库被分片到多个服务器上 就很难在不同的数据库分片之间进行连接操作了一个常见的变通方法是对数据库进行去范式化，这样就可以在一张表中进行查询。

在图 1-23 中，我们对数据库进行分片，以支持快速增加的数据流量。同时，将一些非关系型功能转移到 NoSQL 数据存储中，以减少数据库负载。这里有一篇文章，涵盖了 NoSQL 的很多用例[14]。

<img src="../../../system_design_interview/index-31_1.jpg" width="66%"/>

**数百万用户及以上**

扩展系统是一个迭代的过程。迭代我们在本章所学到的知识可以让我们走得更远。要想扩展到数百万用户以上，还需要更多的微调和新的策略。例如，你可能需要优化你的系统，并将系统解耦到更小的服务。本章所学到的所有技术都应该为应对新的挑战打下良好的基础。在本章的最后，我们将对我们如何扩展系统以支持数百万用户进行总结。

- 保持网络层无状态
- 在每一层建立冗余
- 尽可能多地缓存数据
- 支持多个数据中心
- 在 CDN 中托管静态资产
- 通过分区来扩展您的数据层
- 将层级划分为个别服务
- 监控您的系统并使用自动化工具

恭喜你走到这一步！现在给自己拍拍背。做得好！

**参考资料**

- [[1] Hypertext Transfer Protocol](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)
- [[2] Should you go Beyond Relational Databases?](https://blog.teamtreehouse.com/should-you-go-beyond-relational-databases)
- [[3] Replication](<https://en.wikipedia.org/wiki/Replication_(computing)>)
- [[4] Multi-master replication](https://en.wikipedia.org/wiki/Multi-master_replication)
- [[5] NDB Cluster Replication: Multi-Master and Circular Replication](https://dev.mysql.com/doc/refman/5.7/en/mysql-cluster-replication-multi-master.html)
- [[6] Caching Strategies and How to Choose the Right One](https://codeahoy.com/2017/08/11/caching-strategies-and-how-to-choose-the-right-one/)
- [[7] Scaling Memcache at Facebook ](https://www.usenix.org/system/files/conference/nsdi13/nsdi13-final170_update.pdf)
- [[8] Single point of failure](https://en.wikipedia.org/wiki/Single_point_of_failure)
- [[9] Amazon CloudFront Dynamic Content Delivery](https://aws.amazon.com/cloudfront/dynamic-content/)
- [[10] Configure Sticky Sessions for Your Classic Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-sticky-sessions.html)
- [[11] Active-Active for Multi-Regional Resiliency](https://netflixtechblog.com/active-active-for-multi-regional-resiliency-c47719f6685b)
- [[12] Amazon EC2 High Memory Instances](https://aws.amazon.com/ec2/instance-types/high-memory/)
- [[13] What it takes to run Stack Overflow](http://nickcraver.com/blog/2013/11/22/what-it-takes-to-run-stack-overflow)
- [[14] What The Heck Are You Actually Using NoSQL For](http://highscalability.com/blog/2010/12/6/what-the-heck-are-you-actually-using-nosql-for.html)

