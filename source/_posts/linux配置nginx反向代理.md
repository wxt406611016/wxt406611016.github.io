﻿---
title: linux配置nginx反向代理
date: 2019-12-26
---
**将web项目部署到服务器之后，通过nginx反向代理可以帮助隐藏web项目所在的端口号，避免了端口暴露在外的风险，本文主要讲述反向代理的概念和在linux环境下如何配置nginx反向代理。**
## 1.什么是反向代理？
反向代理（Reverse Proxy）方式:
是指以代理服务器来接受Internet上的连接请求，然后将请求转发给内部网络上的服务器；并将从服务器上得到的结果返回给Internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。通常的代理服务器，只用于代理内部网络对Internet的连接请求，客户机必须指定代理服务器,并将本来要直接发送到Web服务器上的http请求发送到代理服务器中。当一个代理服务器能够代理外部网络上的主机，访问内部网络时，这种代理服务的方式称为反向代理服务。
简单来讲，正向代理是作为客户端的代理服务器，反向代理则是作为服务器端的代理服务器。
![nginx反向代理](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS5heDF4LmNvbS8yMDIwLzA1LzA3L1laalg2Sy5qcGc?x-oss-process=image/format,png)
## 2.安装nginx服务器
安装步骤非常简单，打开终端，输入：
`sudo apt-get install nginx`

在终端输入以下内容，打开nginx服务器：
`service nginx start`

## 3.配置nginx服务器
apt-get安装下的nginx配置文件的位置在 /etc/nginx/sites-available/default，打开default文件，并将proxy_pass处的值，改为你想要代理的服务器端口地址。(下图中代理的为5000端口号）
```
server {
        listen       80;
        server_name  localhost;
    
        location / { 
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header REMOTE-HOST $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass "http://127.0.0.1:5000/";
        }      
}
```
接下来要将配置信息重新载入，终端输入：
`/etc/init.d/nginx reload`
## 4.测试结果
打开一个本地web项目，开放在5000端口
![打开一个flask项目，并开放在5000端口](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS5heDF4LmNvbS8yMDIwLzA1LzA3L1llQzJFOC5wbmc?x-oss-process=image/format,png)
在浏览器中输入url：0.0.0.0或0.0.0.0:80（因为nginx默认开放80端口，并且可以省略端口号）
![访问nginx端口号，并成功返回flask项目内容](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS5heDF4LmNvbS8yMDIwLzA1LzA3L1llQ1JVUy5wbmc?x-oss-process=image/format,png)
可以看到，浏览器显示的内容正是开放在5000端口的flask项目。

#### 如有错误或建议，请在评论处指出！
