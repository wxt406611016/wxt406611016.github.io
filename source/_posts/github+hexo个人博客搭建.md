---
title: ubuntu下 github+hexo个人博客搭建
date: 2019-02-07
---

# github+hexo搭建详细过程:

我的操作系统为ubuntu16.04,在搭建的过程中参考了网上很多例子,但都没有完全适合自己的,多多少少还是会碰到一些问题,下面将详细展现github+hexo的搭建过程.  
tips:萌新一枚,有不足之处,还请指出,tks!!!

**在搭建个人博客之前需要先安装一些环境:**
## 1.安装nvm
nvm是node的包版本控制工具,nvm的好处在于你可以找到各种你想要的node版本,这里我们选择安装稳定版的node.(如果选择终端命令apt-get install nodejs方法安装node,node的版本较低,在后续的搭建过程中可能会报错),所以这里建议先安装nvm.

代码安装nvm(git过程中可能速度缓慢,如果你会翻墙可以翻墙git,个人亲测,速度提升明显)
```
git clone https://github.com/creationix/nvm.git ~/.nvm && cd ~/.nvm && git checkout `git describe --abbrev=0 --tags`
```
接下来我们编辑环境变量配置文件
 

```
cd 
vim .bashrc
```
将`source ~/.nvm/nvm.sh. `添加到.bashrc中,然后保存退出
输入命令`source .bashrc` 将新增的nvm添加到系统中.
输入命令`nvm --version`  如果显示版本,则显示安装成功.
 
## 2.安装node

安装完nvm接下来使用nvm安装node和npm,首先检查远程仓库的可选版本
```
nvm ls-remote
```
安装node的稳定版

```
nvm install stable
```
启用安装好的版本,并设置为默认版本

```
nvm use node
nvm alias default node
```


## 3.安装hexo
如果nvm安装完成且成功配置好稳定版以后，就可以安装hexo了.

```
mkdir blog       #创建一个文件夹,文件夹名自拟,本例中为blog
hexo init blog 
cd blog         
npm install 
npm update -g
```

执行没有报错的话hexo应该就可以用了,在blog目录下输入以下命令.

```
hexo g     # 生成静态界面
hexo s     # 开启本地服务
```
如果命令行结果如下则安装成功了,此时你可以crtl+鼠标点击命令行url进入该页面。
![测试安装成功](https://s2.ax1x.com/2019/02/08/kNeniR.png)

## 4.配置本机全局git环境
假设我的邮箱是tom@gmail.com,github的名字是tom

```
git config --global user.email “tom@gmail.com” 
git config --global user.name “tom”
```
## 5.生成ssh密钥
命令端输入,生成ssh密钥

```
ssh-keygen -t rsa -C tom@gmail.com
```
会提示让你输入文件夹的名字来存放ssh秘钥，并且让你确认一个验证的密码，按要求操作就好了。然后输入下面命令查看ssh密钥文件中的公钥内容.

```
less ~/.ssh/id_rsa.pub
```
## 6.创建博客工程
建一个新的仓库，假设你的用户名是tom，那么你生成的这个工程命名必须为tom@github.io。(下图中是我个人的github用户名,为wxt406611016)
![本例中用户名为wxt406611016](https://s2.ax1x.com/2019/02/08/kNeuJ1.png)
其他的按照默认配置就可以,然后点击Create repository创建.

**然后将你的ssh密钥添加到你的github中**
在你的github页面依次点击Settings->SSH and GPG keys->New SSH key,将第五步中的公钥内容复制到Key里,点击Add SSH key.(如下图)
![添加页面](https://s2.ax1x.com/2019/02/08/kNeeo9.png)
## 7.在hexo中配置git
在blog文件夹下的 config.yml文件中配置你的git,添加以下代码.
注意代码中的xxx为你的github的用户名

```
deploy:
  type: git
  repository: ssh://git@github.com/xxx/xxx.github.io.git
  branch: master
```
## 8.将本地文件上传到github
先安装hexo deploy插件,输入以下代码.

```
npm install hexo-deployer-git --save
```
最后将本地文件编译后上传到github上,在终端输入以下代码.
```
hexo g          #生成静态文件
hexo d          #上传部署网站
```


**大功告成了,github+hexo搭建的个人博客就完成了,快在浏览器中输入xxx.github.io(xxx为你github的账户名)来看看你的博客吧**

***关于如何更改主题,查看下一片文章点击***[--github+hexo更换主题](https://wxt406611016.github.io/2019/02/01/github+hexo%E6%9B%B4%E6%8D%A2%E4%B8%BB%E9%A2%98/)
