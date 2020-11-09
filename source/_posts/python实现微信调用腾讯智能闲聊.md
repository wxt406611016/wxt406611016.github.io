﻿---
title: python+微信+腾讯智能闲聊
date: 2020-02-08
---
**继上一篇调用图灵机器人的玩法后，不满于每天100条的限额，于是！！！又找到了腾讯云产品-智能闲聊，免费！无限额！
[申请智能闲聊](https://ai.qq.com/)**
## 1.构建调用api文件：
由于wxpy库没有集成腾讯智能闲聊的api，我们需要自己配置接口，所以会比调用图灵机器人显得麻烦。上python，下面是api.py的内容，主要用于向腾讯云发送请求，并获取回复内容，相当于一个调用api。如下api.py:
```
###api.py###
import hashlib
import time
import requests
import random
import string
from urllib.parse import quote
 
def curlmd5(src):
    m = hashlib.md5(src.encode('UTF-8'))
    return m.hexdigest().upper()               # 将得到的MD5值所有字符转换成大写
 
def get_params(plus_item):                    #用于返回request需要的data内容
    global params
    t = time.time()                                             #请求时间戳（秒级）,（保证签名5分钟有效）
    time_stamp=str(int(t))
    nonce_str = ''.join(random.sample(string.ascii_letters + string.digits, 10))            # 请求随机字符串，用于保证签名不可预测  
    app_id='2135462408'                              # 修改成自己的id  
    app_key='w3Lv6zsb95T89fay'             # 修改成自己的key  
    params = {'app_id':app_id,
              'question':plus_item,
              'time_stamp':time_stamp,
              'nonce_str':nonce_str,
              'session':'10000'
             }
    sign_before = ''
    for key in sorted(params):                      #要对key排序再拼接
        sign_before += '{}={}&'.format(key,quote(params[key], safe=''))    # 拼接过程需要使用quote函数形成URL编码
    sign_before += 'app_key={}'.format(app_key)                                           # 将app_key拼接到sign_before后
    sign = curlmd5(sign_before)
    params['sign'] = sign                                 # 对sign_before进行MD5运算
    return params                                              #得到request需要的data内容
 
def get_content(plus_item):
    global payload,r
    url = "https://api.ai.qq.com/fcgi-bin/nlp/nlp_textchat"         # 聊天的API地址  
    plus_item = plus_item.encode('utf-8')
    payload = get_params(plus_item)
    r = requests.post(url,data=payload)                                                 #带参请求api地址
    result=r.json()["data"]["answer"]
    return result                                                                                                #获得返回内容

```

## 2.与智能闲聊进行聊天
终于将前提基础工作完成了，调用api构建好后，调用智能闲聊也就很简单了，调用wxpy库，初始化和构建自动发送、回复函数和上一章都一样。为了简化工作，本例中只实现自动回复功能，也是最常用的功能。依然上ai.py的代码，用于聊天。ai.py如下：
```
from wxpy import *
from ai import *                                #导入上一步构建的ai.py文件
bot = Bot()
friend = bot.friends().search('AIbot')[0]

@bot.register(friend)
def auto_reply(msg):
    a=msg.text
    answer=get_content(a)
    if answer=='':                                 #防止返回内容为空
        for i in range(2):                     
            time.sleep(2)
            answer=get_content(a)
            if answer!='' and answer!="emmmm，我不是很懂你的意思":
                break
            else:
                answer="emmmm，我不是很懂你的意思"
    return '[Stephen] {} '.format(answer)

bot.join()
```
由于访问腾讯api的时候，偶尔会返回空内容，为了避免这种情况，当返回空时，隔2s再请求一次，这般重复3次，如果依然返回空，则自动将返回值设为"emmmm，我不是很懂你的意思"。

## 3.效果展示：
ps：我已将智能闲聊的名字设为了"Stephen"，上图最重要：
![和智能闲聊增进感情](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS5heDF4LmNvbS8yMDIwLzA1LzA4L1lteElScy5qcGc?x-oss-process=image/format,png)
腾讯智能闲聊的智能化程度也挺高，日常闲聊也没有太大问题，有些回答甚至比图灵机器人还要讨喜，各有千秋吧。最关键的是免费！不限额！
尽管wxpy功能很强大，但是部分用户无法登陆微信网页版，也就无法体验wxpy的服务了，可以说是很可惜了！
#### 如有错误或建议，请在评论处指出！
