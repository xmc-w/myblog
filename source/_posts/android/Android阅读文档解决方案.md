title: Android查看office文档解决方案
date: 2018/1/11 14:18:32
categories: android
tags:
- office办公文档
- webView
---

>  在 iOS 平台上，实现移动端 Office 文档的在线阅读非常简单，只需要使用 WebView 加载网络文件的 Url 地址即可显示。
而在 Android 平台上，由于“高墙耸立”，Google 在国内的访问限制，导致这一简单的操作变得复杂起来，
开发人员不得不寻求其他解决方案，这里列举一些。

![mark](http://p2fqmxqeh.bkt.clouddn.com/blog/180112/gDhKcI0BE6.jpg?imageslim)

<!--more-->

## WebView 网页显示
  1. 借助WebView控件加载远程文档的 Url 地址即可:
     第三方的解析服务有：Google Doc , Office Web 365 ,永中云转换 等；
     部署解析服务端，开源方案有：PDF.js , pdf2htmlEX 等;
  2. 加载本地文档可以借助腾讯的浏览服务TBS实现；
  
### Google Doc
  类似 iOS ，Google 也提供了一种在线文档解析的功能，只需要按照固定的格式将远程文档的 Url 地址传给 Google 服务器，
即可利用 WebView 控件加载新的 Url 地址，显示即可。WebView 加载的 Url 地址格式如下：

    https://docs.google.com/gview?embedded=true&url=文档地址
    
  文档地址设置为服务器文档地址就可以了，无需服务器和客户端的额外工作，但是国内访问需要翻墙。
  
### Office Web362
  第三方公司提供的一种 Office 文档在线预览的功能，能够实现 Microsoft、Adobe、WPS 文档的移动端和PC端在线网页访问。
使用简单，类似 Google Doc 访问方式，一个固定格式的链接，轻松实现：

    http://ow365.cn/?i=您的网站ID&furl=文档地址
   
  功能强大，使用简单，但是**付费**使用(可免费受限使用，如访问次数，广告显示等)，具体内容可以通过[OfficeWeb365](http://www.officeweb365.com)
查看。

### 永中云转换

1.支持doc/docx、xls/xlsx、ppt/pptx和pdf、压缩文件等20多种常用Office文档转换为高度兼容的高清网页，实现快捷查阅文档内容，提升工作效率。
2.支持移动阅读，针对移动设备特别优化，支持内容自适应屏幕大小和方向；支持文档缩略图，可将文档首页内容直观呈现。
3.转换效率高，可同时支持多个文档同时转换，支持多核多线程并发访问（需服务器和网络环境支持）。
4.开发部署简单高效，只需要传入待转换的文档和相关参数，系统自动返回转换后文件URL。
5.支持本地化部署和互联网调用，支持Linux和Windows等多种操作系统。
6.基于自主可控Office核心技术，支持特定需求的定制开发。
7.支持在线编辑，实现文档编辑功能，使用简单，便捷（**付费**）。

  可以使用部署在公有云上的服务，客户端注册后获取相关服务信息，在互联网上直接远程调用永中DCS转换接口，获取文档预览效果（通过webView加载即可）或目标格式的转换文档。
同时支持在线编辑等定制化业务，详细内容可查看[永中DCS](http://www.yozodcs.com/index.html)

### 腾讯TBS浏览服务

  依托 X5 内核强大的能力，TBS 致力于提供优化移动端浏览体验的整套解决方案。 TBS 虽然核心在于提供一套 SDK 解决传统 WebView 的诸多使用问题。
但是，利用其增强浏览能力，我们还能够使用这套 SDK 实现应用内的文件浏览功能、视频播放功能等。
  使用简单集成TBS提供的SDK，详细集成操作请参考[TBS腾讯浏览服务](https://x5.tencent.com/tbs/index.html)。
  对于浏览文件这方面官方没有提供完善使用文档和Demo，可以借鉴[这篇文章](https://www.jianshu.com/p/3f57d640b24d),集成TBS的SDK后打开本地文件的代码很简单：
  
     Bundle bundle = new Bundle();
      bundle.putString("filePath", getLocalFile().getPath());
      bundle.putString("tempPath", Environment.getExternalStorageDirectory().getPath());
      boolean result = mTbsReaderView.preOpen(parseFormat(mFileName), false);
      if (result) {
        mTbsReaderView.openFile(bundle);
      }

  缺点是只能打开本地文件，需要先下载。
  
## 集成开源或者第三方的本地解析SDK

### 免费开源SDK

* [pdfium](https://android.googlesource.com/platform/external/pdfium) Google 的开源项目，也是 Chrome 浏览器的PDF渲染引擎，初始代码来自国内知名PDF技术公司「福昕」。
* [AndroidPdfViewer](https://github.com/barteksc/AndroidPdfViewer)
* [PdfiumAndroid](https://github.com/barteksc/PdfiumAndroid) 基于 pdfium 的 Android 平台实现方式，支持 PDF 文档的应用内预览，支持动画、缩放、手势和双击操作。
* [MuPDF](https://mupdf.com)

### 付费SDK

* [Foxit PDF SDK](https://www.foxitsoftware.cn/products/sdk/PDFsdk/android) 福昕出品，性能稳定，功能强大
* [plugPDF](https://plugpdf.com) 来自国外的一个付费 SDK，使用简单，只需三步即可集成到自己的应用中并使用。