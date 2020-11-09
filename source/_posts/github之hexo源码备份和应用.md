---
title: github之hexo源码备份和应用
date: 2020-11-09
---
# 1.使用git分支保存hexo博客源码
具体操作请[跳转到页面。](https://zhuanlan.zhihu.com/p/133860750)
# 2.为新电脑添加SSH keys
当你需要在一台新设备上继续写博客上，可以通过github的源码来快速搭建原环境，为了能够使用git clone、push等操作，需要在新电脑上设置github环境，并将SSH keys存储到github中，具体操作请跳转到[我之前的一篇博客](https://wxt406611016.github.io/2019/02/07/github+hexo%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA/)中4，5，6节部分。
# 3.将hexo源码clone到本地
新设备空空如也，需要将hexo源码克隆到本地，具体操作也在第一步的文档中，[跳转！](https://zhuanlan.zhihu.com/p/133860750)
# 4.配置hexo所需环境
将源码clone到本地还不够，还需要配置hexo所需环境，如nodejs，npm。。。这里我们使用nvm来安装，具体操作也在第二步的文档中，[跳转！](https://zhuxiaoxia.blog.csdn.net/article/details/88855256?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-5.compare&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-5.compare)中从第2节开始看。
# 5.Deploy和Push
到这一步基本都配置好了，需要测试使得
1. hexo d操作能部署博客到github博客仓库的显示分支，_config.yml中默认将hexo d部署到master分支，所以不用修改，直接hexo g，hexo d一顿操作就完事儿了，如果想在本地先调试，就在hexo g后跟一个hexo s。
2. git push操作能将博客源码更新到github博客仓库的源码分支，
   1. (可选)如果你是从github上clone下来的源代码，则不需要本步骤。否则的话先本地关联远程仓库：

       `git remote add origin git@github.com:yourblog/yourblog.github.io.git`
   1. 接着git三部曲就完事儿了:
       ```
       git add .    #添加所有文件到暂存区
       git commit -m "date"     #提交，标记我此处用了date字符串
       git push origin source     #推送到源码分支source
       ```