---
title: 利用docker将web项目部署到服务器
date: 2019-03-31
---
**完成了一个简单的web项目，想部署到云服务器，利用docker是一个很好的办法**

## 创建docker容器
在当前web项目目录下创建Dockerfile和requirements.txt两个文件。文件填写内容根据你的web项目来定，具体参考官网教程。[～～docker官网～～](https://docs.docker.com/get-started/part2/#define-a-container-with-dockerfile)
![我的目录结构](https://s2.ax1x.com/2019/03/30/ABTpIs.png)

接下来创建docker镜像，打开终端，输入
`docker image build -t name .`
name为你创建的镜像名字，自定义，注意不要漏了name后的那个“.”
然后输入`docker image ls`
![在这里插入图片描述](https://s2.ax1x.com/2019/03/30/ABTdJI.png)
显示已存在的docker镜像，cumt是我的web项目，下面我要将cumt这个镜像部署到我的阿里云服务器。

## 生成静态文件
将docker镜像导出为静态文件
` docker save cumt:latest > cumt.tar`

cumt:latest为镜像名, cumt.tar为新生成的静态文件名。

## 部署到云服务器
使用scp将静态文件上传到服务器端
`scp ./cumt.tar root@47.106.130.6:/cumt.tar`

使用ssh登陆你的云服务器
`ssh root@47.106.130.6`

输入ls查看是否上传成功
`ls`

如果ls下没有找到你上传的静态文件，使用find命令查找它。
`find / -name cumt.tar`

## 还原静态文件
在静态文件所在位置目录下运行docker load命令还原。
`docker load < cumt.tar`
输入`docker image ls`发现，已还原为镜像文件。
![cumt镜像已还原](https://s2.ax1x.com/2019/03/30/AB7mX8.png)

大功告成，可以直接运行了，如果不需要连接数据库的话直接输入
`docker run -p 4000:5000 cumt`
如果需要连接数据库，则需要用link方法挂载数据库（前提是你已经有数据库镜像）
`docker run --link your-mysql -p 4000:5000 cumt`
访问服务器公网的4000端口就可以访问你的web项目了。注意要在控制台打开服务器的4000端口（映射端口号可以自定义）

## 小结:
**利用静态文件进行容器的迁移, 是一件非常简单的事情, 你可以像发布一个软件包一样将自己的docker容器生成的静态文件分发到各类操作系统, docker才是真正的跨平台呀!**



