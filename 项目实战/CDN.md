DNS

ISP(Internet Service Provider)：互联网服务提供商，又称**因特网服务提供者**、**互联网服务提供商**、**网络服务供应商**，即指提供[互联网](https://zh.wikipedia.org/wiki/互聯網)[访问](https://zh.wikipedia.org/wiki/網路存取)服务的公司。通常大型的[电信公司](https://zh.wikipedia.org/wiki/電訊公司)都会兼任互联网服务供应商，一些ISP则独立于电信公司之外。

ISP route：ISP路由可用于出站流量和链路负载均衡流量。出站流量的路由是根据以下优先级选择的：链路本地路由-自流量使用链路本地路由。LLB链路策略路由配置的策略路由优先于默认路由。



CName：**真实名称记录**（英语：Canonical Name Record），即**CNAME记录**，是域名系统（[DNS](https://zh.wikipedia.org/wiki/DNS)）的一种记录。CNAME记录用于将一个[域名](https://zh.wikipedia.org/wiki/域名)（同名）映射到另一个域名（真实名称），[域名解析服务器](https://zh.wikipedia.org/wiki/域名解析服务器)遇到CNAME记录会以映射到的目标重新开始查询。[[1\]](https://zh.wikipedia.org/wiki/CNAME记录#cite_note-1)

这对于需要在同一个IP地址上运行多个服务的情况来说非常方便。若要同时运行[文件传输](https://zh.wikipedia.org/wiki/文件传输协议)服务和[Web服务](https://zh.wikipedia.org/wiki/Web服务)，则可以把*ftp.example.com*和*www.example.com*都指向DNS记录*example.com*，而后者则有一个指向IP地址的A记录。如此一来，若服务器IP地址改变，则只需修改*example.com*的A记录即可。

CNAME记录必须指向另一个域名，而不能是IP地址。



[BGP](https://zh.wikipedia.org/wiki/%E8%BE%B9%E7%95%8C%E7%BD%91%E5%85%B3%E5%8D%8F%E8%AE%AE)

NS record：NS 记录（或名称服务器记录）是**包含域或 DNS 区域中权威名称服务器名称的 DNS 记录**。当客户端查询 IP 地址时，它可以通过一个 NS 记录找到其预期目的地的 IP 地址DNS查询.



https://aws.amazon.com/cn/what-is/cdn/

[内容分发网络](https://www.zhihu.com/search?q=内容分发网络&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A1604554133})（Content Delivery Network，简称CDN）是建立并覆盖在承载网之上，由分布在不同区域的边缘节点服务器群组成的分布式网络。



https://www.zhihu.com/question/36514327





