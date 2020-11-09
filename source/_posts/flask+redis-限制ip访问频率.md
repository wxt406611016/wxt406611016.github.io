﻿---
title: flask+redis限制ip访问速率
date: 2020-04-11
---
**上篇介绍了利用font-face进行反爬虫，这篇介绍另一种常见方法--验证码校验，对频繁访问的ip进行限制，强制要求验证码验证，验证成功方可继续访问。**
## 1.限制逻辑/策略：
1. 利用request.remote_addr获取客户端ip；
2. 客户端访问时，服务器判断redis中是否有客户端ip；
3. 若不存在，将客户端ip存入redis中，对应值设为1；若存在，客户端ip对应值+1；
4. 若redis中客户端ip对应值在5分钟内>5，即5分钟内访问页面超过5次，则视为恶意ip，跳转至验证码校验页面；
5. 验证成功后，向后端请求过渡函数B，进行redis操作，B函数的功能为将验证成功的ip从redis中删除，删除后再重定向至开始页面；

## 2.代码实现限制逻辑：
后端代码如下，index()方法用于判断redis中是否存在客户端ip，以及ip对应值是否大于5：
```
@app.route('/', methods=['GET'])
def index():
    ip=request.remote_addr
    if(rs.exists(ip)):                                         #rs可操作redis
        rs.setrange(ip,0,int(rs.get(ip))+1)
    else:
        rs.set(ip,1,ex=300)                               #5分钟5次
    if (int(rs.get(ip))<6):
        return render_template('index.html')           #正常返回
    else:
        return render_template("antibot.html",ip=ip,secret=secret)    #返回验证码校验页面
```
如果跳转至antibot.html(验证码校验页面)后，前端通过js对参数a进行base64加密，并带参数访问/mid页面，之所以要带参数，是为了在后端验证参数是否有效(即判断是否是通过验证码校验后跳转而来)：
```
<script>
......
onSuccess: function() {
      document.getElementById('msg').innerHTML = '登录成功！'
      var a="{{ip}}"+"{{secret}}"                   //secret是从后端传来的随机密钥
      a=window.btoa(a)                                 //对变量a进行base64加密
      window.location.href="/mid?id="+a
    },
......
</script>
```
mid()方法用于通过验证码校验后，将ip从redis中删除，并重定向至开始页面：
```
@app.route('/mid', methods=['GET'])
def mid():
    cmp=request.args.get('id','0')
    ip=request.remote_addr
    sec=ip+secret                                                 #secret作为随机密钥进行加密
    result=base64.b64encode(sec.encode())
    result=str(result)[2:-1]
    if(cmp==result):                                            #判断前端js传来的参数是否有效
        rs.delete(ip)
    response =  redirect(url_for('index'))
    return response    
```

## 3.效果展示：
打开flask项目，可以正常访问：
![正常访问](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS5heDF4LmNvbS8yMDIwLzA1LzA3L1ltMGtMUS5wbmc?x-oss-process=image/format,png)
刷新5次后，触发了限制ip策略，返回验证码校验界面：
![返回验证码校验](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS5heDF4LmNvbS8yMDIwLzA1LzA3L1ltMEZzZy5wbmc?x-oss-process=image/format,png)
验证成功后，成功返回开始页面。从而实现了flask上用redis进行限制ip访问频率。

## PS:若设置反向代理需注意！！！
如果设置了反向代理的话，request.remote_addr则不会返回客户端的真实ip。以nginx反向代理为例，客户想要访问服务器，实际请求路线是client->nginx->server，而request.remote_addr表示的是上一级ip，而server的上一级ip是nginx服务器的ip.

解决办法：以nginx为例，在配置文件中使用X-Forwarded-For，然后后端获取ip方式由 `ip=request.remote_addr`更改为`ip = request.headers.getlist("X-Forwarded-For")[0]` 即可。

[访问上图中我的web项目，点击链接即可，](http://wxt123.top)

#### 如有错误或建议，请在评论处指出！
