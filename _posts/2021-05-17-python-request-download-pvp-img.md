---
layout: post
title: "30行python代码下载王者荣耀官网英雄海报"
subtitle: '使用python自动下载所有王者荣耀英雄皮肤图'
author: "chaoxiaodi"
header-style: text
tags:
  - python
  - requests
  - 爬虫
  - 正则
  - 王者荣耀
---

### 前言

本篇文章是记录学习和使用python中的 `requests` 与 `re` 两个库

`requests` 库是python用来进行网页交互最常用的库

网上也有相当多的爬虫的教程

本文只用最简单的最朴素的方法进行一些页面的下载

不涉及自动翻页、自动查找元素

而是通过正则表达式，获取到所有想要的信息

### 实现过程

首先打开王者荣耀官网的英雄资料页面

[王者荣耀英雄资料列表页](https://pvp.qq.com/web201605/herolist.shtml)

通过浏览器检查发现英雄列表页规则如下：

    <li><a href="herodetail/139.shtml" target="_blank"><img
每个英雄对应的那个数字是不一样的

要根据正则所有的行都找出来

获取到所有的英雄列表后

然后处理英雄对应的详情页面

    比如：刘备
    https://pvp.qq.com/web201605/herodetail/170.shtml
可以看到上面获取到的id就对应了每个英雄的详情页面

同样使用检查功能可以获取到英雄名称

    <h2 class="cover-name">刘备</h2>
    
#### 更新
使用检查功能可以获取到皮肤的图片地址，皮肤名称

    <img src="//game.gtimg.cn/images/yxzj/img201606/heroimg/170/170-smallskin-2.jpg" alt="" data-imgname="//game.gtimg.cn/images/yxzj/img201606/skin/hero-info/170/170-bigskin-2.jpg" data-title="万事如意" data-icon="16">

图片地址要用那个 `date-imgname` 是大图

在我最初实现的时候没有获取英雄皮肤的名称

用了简单粗暴的方法

现在最多皮肤没有超过10个，所以我只把连接循环10次

如果可以访问就保存文件

最后就是循环访问各个图片连接，然后保存文件

### 代码

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-
    '''
    fun：下载王者荣耀官网英雄海报
    '''
    import re
    import requests
    
    IMG_DIR = 'WZRYIMG/' 
    HEROLIST_URL = 'https://pvp.qq.com/web201605/herolist.shtml'
    BIGSKIN_URL = 'https://game.gtimg.cn/images/yxzj/img201606/skin/hero-info/'
    HERODETIAL_URL = 'https://pvp.qq.com/web201605/herodetail/'
    RE_HEROID = re.compile('.*<li><a href="herodetail/\d{3}.shtml" target="_blank"><img.*')
    RE_HERONAME = re.compile('.*<h2 class="cover-name">.*</h2>.*')
    
    
    def get_url_text(url):
        res = requests.get(url)
        res.encoding = 'gb2312'
        return res.text
    
    
    if __name__ == "__main__":
        hero_ids = re.findall(RE_HEROID, get_url_text(HEROLIST_URL))
        for hero_id_info in hero_ids:
            hero_id = hero_id_info.replace(' ', '')[23:26]
            herodetial_url = HERODETIAL_URL + str(hero_id) + '.shtml'
            heroname = re.findall(RE_HERONAME, get_url_text(herodetial_url))[0].replace(' ', '')[22:-5]
            for i in range(1, 10):
                img_url = BIGSKIN_URL + hero_id + '/' + hero_id + '-bigskin-' + str(i) + '.jpg'
                imgres = requests.get(img_url)
                imgres.encoding = 'gb2312'
                if imgres.status_code == 200:
                    file_name = heroname + '_' + str(i) + '.jpg'
                    file_path = IMG_DIR + file_name
                    with open(file_path, 'wb') as fp:
                        fp.write(imgres.content)


### 后记

代码最后把对应的图片按照英雄保存到本地文件夹

可以对代码进行简单的改动后

按照英雄的职业进行分类保存

ps：不要使用代码进行大批量访问网站，否则后果自负


Q：594934249

---我是超小弟·一名不务专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏

![](/img/erweima.jpg)
