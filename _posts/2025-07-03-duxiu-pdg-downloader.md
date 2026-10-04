---
title: 史总家的包库图书下载
categories: 技术 读秀
---

自从秒传大规模流行开来，好像有几年没有亲自下载过包库图书了。去年年底秒传彻底失效，还是得自己动手下载。当年下载神器棒棒糖（CxCandy）已不再支持 Windows 11 的虚拟化环境，自己写代码下载吧，也算是了解一下背后的原理。

这些年听说过很多神乎其神的说法，「棒棒糖截取了加密前的 PDG」、「封面只能手动下载大图」、「下载不到书签」，实际上一番实践下来根本没有什么门槛，整个下载工具大约 100 行 Python 就能搞定。

## 包库基本概念

前文叙述过了读秀的一些基本情况。包库是指图书馆已采购，用户可以阅读全文的电子书，通常是扫描的图片，近年来也有 PDF 格式（被称为「文字版」PDG），也有极个别的 ePub。

包库图书能下载全文的前提是，可以在阅读器（SSReader 4 或 5 均可）里浏览。

整本图书是 `.pdz` 格式，它需要配合 `HDDINFO` 解密，这个我未曾涉及。后面讨论的是下载单页 PDG 的方法。

## 获取报文

网上所有资料都是「能看就能下」，我们模拟的是阅读器的正常浏览行为，进而下载到全部未加密的图书页面。

5 代阅读器的数据包不能用 Fiddler、HTTP Analyzer 或 HTTP Debugger 抓到。本以为是 HTTPS 导致的，所以这个下载项目搁置了几个月，直到最近用到了 WireShark，它很顺利地抓到了 5 代阅读器的数据包，一切得以正常推进。

我们在 5 代阅读器内浏览图书，此时在 WireShark 里可以看到已经抓到了相关 HTTP 包。我们先筛选 `http`，再找 `certnet` 就能看到请求了。这些请求通常称作「**报文**」。操作界面见下图。

!\[](./assets/v2-80039cbc73eb5d1e274fca51aa2af885\_r.jpg)

左下角的窗口能看到报文，这就是我们需要的数据。在「Hypertext Transfer Protocol」上右键选择 `copy` → `...as printable text` 即可获得完整的报文。格式大概是下面这样，其中的 `...` 是被隐去的。

```text
GET /certnet.dll?/.../000002.pdg HTTP/1.1
Host: bfts.5read.com
Accept: \\\*/\\\*
User-Agent: SSREADER/5.0.0.0001 unRegister
SSUA: SSREADER/5.0.0.0001 unRegister
SSKEY: 65BB86D3842A2E...
SSCT: F38E64DE72E8F2C0FFD95B47E507FC025...
SSAuth: 5c5c5c5c...
SSRANDOM: 33372501
BookAuth: EFF648CE9EA1957...
ServerAuth: 
HDDINFO: -1000000000
```

## 需要下载的内容

读秀图书的组成部分有如下三类，后面我们要分别下载。

1. 图书信息 `bookcontents.dat`   它是一个压缩包，但是文件头有改动，技术细节可参考，也可以用马健的软件 PdgCntEditor 来直接编辑它。
2. 附属页   包括封面、目录等正文之外的页面，它包括六种类型，分别是 `cov` 封面封底、`bok` 内封、`leg` 版权、`fow` 前言、`!` 目录、`att` 插页。
3. 正文

所有的 PDG 文件名都是 6 位，不足的用 0 填充。

## 编程实践

我们有了请求头、文件名之后，就可以写 Python 来下载了。

需要注意的是，报文里没有标题、页码、SS 号，这三样都需要手动指定。顺带说一句，从 SS 只能获得封面和书签，得不到书名以及更多信息。

为了下载更快，可以用多线程并行下载。一个合理的并行数量是 5，毕竟棒棒糖是这么用的。如果采用并行，最好是提前告知程序总页码，否则多线程的实现极其复杂。

具体代码就不放出来了，满打满算 100 行即可解决。

实际上有一个关键参数 `SSAuth` 的生成方法没提到，如果 `SSAuth` 不变的话应该是无法下载整本的。它的生成方法已经有人公布过了，我也是参考的这里。可能的一个坑是，传入该算法的 url 是不带域名的，直接从 `/certnet.dll` 开始。因原帖设置了权限，我就不在这里放出完整函数了。

## 结语

知道了包库图书的下载内容、流程、方法之后，相信你一定可以写一个属于自己的下载器，甚至带有漂亮的 UI。

## 参考

1. 揭开PDG和PDF书签制作的小秘密（上） [https://www.readfree.net/bbs/thread-4633567-1-1.html](https://www.readfree.net/bbs/thread-4633567-1-1.html)
2. 目录编辑器 [https://www.cnblogs.com/stronghorse/p/11519730.html](https://www.cnblogs.com/stronghorse/p/11519730.html)
3. coolspace：CX ssauth算法 [https://www.readfree.net/bbs/forum.php?mod=viewthread\&tid=4839741\&highlight=SSAuth](https://www.readfree.net/bbs/forum.php?mod=viewthread&tid=4839741&highlight=SSAuth)



