---
title: Hexo 生成永久文章链接
categories: Hexo
toc: true
tags:
  - Hexo
abbrlink: '29174125'
date: 2020-04-21 22:03:59
---

Hexo 默认文章链接生成规则是按照年、月、日、标题来生成的。一旦文章标题或者发布时间被修改，URL 就会发生变化，之前文章地址也会变成 404，而且 URL 层级很深，不利于分享和搜索引擎收录。

<!-- more -->

如果文章标题中有中文，URL 被转码后会很长，比如：`https://lujiahao0708.github.io/2020/04/11/%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%9B%B8%E5%85%B3/GitHub%20Actions%20%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2%20Hexo/`。接下来介绍一个插件 `hexo-abbrlink`，该插件会为每篇生成一个唯一字符串，并不受文章标题和发布时间的影响，比如：`https://lujiahao0708.github.io/p/df27ccfb.html`。

# 1.安装
点击即可访问插件源码地址 [hexo-abbrlink](https://github.com/rozbo/hexo-abbrlink) 。
```
npm install hexo-abbrlink --save
```
> 可能会出现依赖，依据提示安装即可。

# 2.配置
修改博客根目录配置文件 `_config.yml` 的 `permalink`：
```
# permalink: :year/:month/:day/:title/
permalink: p/:abbrlink.html  # p 是自定义的前缀
abbrlink:
    alg: crc32   #算法： crc16(default) and crc32
    rep: hex     #进制： dec(default) and hex
```

不同算法和进制生成不同的格式：
```
crc16 & hex
https://post.zz173.com/posts/66c8.html
crc16 & dec
https://post.zz173.com/posts/65535.html

crc32 & hex
https://post.zz173.com/posts/8ddf18fb.html
crc32 & dec
https://post.zz173.com/posts/1690090958.html
```

# 3.验证
先清理下本地的文件 `hexo clean`，然后重新生成 `hexo g`，启动博客 `hexo s`。该插件会在每篇文章的开头增加内容：
```
abbrlink: df27ccfb
```

这个字符串就是这篇文章的唯一标识，无论修改标题还是发布文章都不会改变。浏览器打开 `http://localhost:4000/` 查看成果吧！

---

> 欢迎访问我的博客：https://lujiahao0708.github.io/

## Tips
欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 

欢迎关注我：后端小哥，专注后端开发，希望和你一起进步！

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)
