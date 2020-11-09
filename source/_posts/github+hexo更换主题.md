---
title: github+hexo更换主题:indigo
date: 2019-02-08
---

# github+hexo更换主题:

上一篇写了关于github+hexo搭建个人博客的过程,搭建成功后页面主题是默认主题,但其实有很多主题可供选择,这篇文章就主要学习一下如何更换主题,以及更换主题后如何丰富优化功能,如评论功能,站内搜索功能.

## 1.查看并选择主题
主题当然不用自己开发,这涉及到很多前端知识很复杂,而在hexo官网上,就有themes栏,有已经开发完成的主题可供选择.[(点击选择主题to themes)!](https://hexo.io/themes/)
![hexo主题页面](https://s2.ax1x.com/2019/02/08/kNmp0e.png)
 可以看到有很多主题可以选择,点击图片可以进入预览模式,选择完成后,返回主题页面,点击主题名(如上图中,点击"一个男孩"字样),即可进入开发者的github页面.本例中,我们以indigo为例,页面如下图所示,也可点击图片下方"进入indigo预览"进入预览模式.
 ![indigo页面](https://s2.ax1x.com/2019/02/08/kNmSmD.png)
 [(进入indigo预览)](https://www.imys.net/)
 然后我们进入对应开发者的github页面.[(进入开发者github)](https://github.com/yscoder/hexo-theme-indigo)

## 2.从github clone indigo主题
进入开发者github页面后,点击"clone or download",选择use ssh,会出现git命令,复制该命令到终端并运行.

```
git clone git@github.com:yscoder/hexo-theme-indigo.git
```
(速度可能较慢,还记得上篇提到的翻墙git吗,速度快很多)
将clone下来的文件夹放入hexo下themes文件夹下.
clone后,需要安装主题需要的依赖.
```
npm install hexo-renderer-less --save    #安装less,作为css的预处理工具 
npm install hexo-generator-feed --save    #安装rss的feed生成工具 
npm install hexo-generator-json-content --save    #用于生成静态站点数据，用于站内搜索的数据源 
npm install hexo-helper-qrcode --save    #用于生成微信分享二维码
```

## 3.修改_config.yml,使用indigo主题
打开_config.yml,将主题更改,如以下代码:
```
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: hexo-theme-indigo
```

## 4.调整修改个人信息
 _config.yml的author改成自己,title改为自己想展示的内容， themes/indigo/_config.yml里的email改成自己的邮箱,title_change改成自己想展示的,头像avater可以自行调整图片路径,menu可以自行修改调整.下面是我个人调整的代码,根据需要调整修改.
 

```
menu:
  home:
    url: /
  archives:
    url: /archives
  tags:
     text: CUMT
     url: http://cs.cumt.edu.cn/
     target: _blank
  github:
    url: https://github.com/xxxxx
    target: _blank
  weibo:
    url: https://weibo.com/xxxxx
    target: _blank
# 你的头像url
avatar: /img/host.jpg
# 动态定义title
title_change:
  normal: Welcome My Blog
  leave: Welcome
```

## 5.增加评论功能:valine
有很多评论系统,本例中我们选择valine,这个评论系统是基于LeanCloud的，大家应该对这个很熟悉，官网网址如下，[LeanCloud官网](https://leancloud.cn/),需要注册一个账户。
注册完成后登录,依次点击设置->应用Key,得到App ID和App Key,
打开themes/indigo/_config.yml,因为该主题集成了valine方法,所以直接修改valine对应的参数就可以了:

```
valine:
  enable: true       # 如果你想使用valine，请将值设置为 true
  appId: xxxxxxxxxxxxxxxxxxxxxxx               #在应用Key里得到的App ID
  appKey: xxxxxxxxxxxxxxxxxxxxxx             #在应用Key里得到的App Key
  notify: false      # Mail notify
  verify: false      # Verify code
  avatar: mm      # Gravatar style : mm/identicon/monsterid/wavatar/retro/hide
  placeholder: 说点什么吧!(别忘了先填写上面的昵称哦)        # Comment Box placeholder
  guest_info: nick       # Comment header info
  pageSize: 10       # comment list page size
```
登录你的LeanCloud页面,依次点击设置->安全中心,在Web安全域名下添加你的github博客网址,如下图
![本图中github账户名为wxt406611016](https://s2.ax1x.com/2019/02/08/kNmFfI.png)
## 6.将本地文件重新上传到github

最后将本地文件编译后上传到github上,在终端输入以下代码.
```
hexo clean   #清除缓存文件
hexo g          #生成静态文件
hexo d          #上传部署网站
```


**ok,主题更换完成,评论功能也添加,继续打造属于你自己的博客吧!**


