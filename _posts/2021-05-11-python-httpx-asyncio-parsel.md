---
layout: post
title: Python爬虫工具测评：下一代请求库httpx和解析工具parsel
description: "httpx和requests到底谁更快? 异步协程爬虫和多进程和多线程爬虫相比呢?"
author: 大江狗
category: Python
---
Python网络爬虫领域两个最新的比较火的工具莫过于httpx和parsel了。httpx号称下一代的新一代的网络请求库，不仅支持requests库的所有操作，还能发送异步请求。parsel最初集成在著名Python爬虫框架Scrapy中，后独立出来成立一个单独的模块，支持XPath选择器, CSS选择器和正则表达式等多种解析提取方式, 据说相比于BeautifulSoup解析效率更高。另外httpx异步协程与多进程和多线程爬虫相比到底谁更快呢? 本文会给出公允测评。
{: .fs-6 .fw-300 }

[阅读全文](https://pythondjango.cn/python/advanced/3-httpx-parsel-requests-comparision/)