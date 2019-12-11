---
title: 关于NebulaSolarDash
date: 2017-05-10 22:06:10
lastmod:
tags: [Python, 资源监控, 工具]
categories:
 - 造轮子
typora-root-url: ..
typora-copy-images-to: ../images
---

### 2017年的5月1日，三天假期，闭门造了个轮子

写这个工具的目的是为了解决工作问题。 个人工作生产环境无法连接互联网，也没有自建的yum源等，手头又有很多服务器需要进行监控，使用现有的开源方案安装部署是个问题， 各种依赖组件包需要挨个安装，很麻烦，所以想找一款依赖较少部署简单的分布式服务器资源监控工具，找来找去没找到，索性自己动手写一个。 我的本职工作是测试，所以就用最熟悉的Python来写吧，第一次写web应用，先做出来再边学边优化吧。

工具分为客户端和服务端两部分： 服务端使用了bottle来作为web框架，echarts来渲染生成图表； 客户端使用Python原生类库采集服务器资源，客户端采集数据部分代码参考了pyDash

##### 效果如下
![](/images/NebulaSolarDash2.0.gif) 


##### 项目链接链接
[NebulaSolarDash](https://github.com/toddlerya/NebulaSolarDash)

-----

2017年05月10日 于 南京  
[Email](toddlerya@qq.com)  
[GitHub](https://github.com/toddlerya)