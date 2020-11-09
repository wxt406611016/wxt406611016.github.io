﻿---
title: python实现微信自动回复+图灵机器人
date: 2020-01-12
---
**前段时间无意间看到wxpy，彷佛捡到了宝，立马开始探索用python玩微信，可以将很多操作自动化，既方便又有趣。
wxpy是 在 itchat 的基础上，通过大量接口优化提升了模块的易用性，并进行丰富的功能扩展，其中就包括平时常用的自动回复，主动发消息，甚至还能引入智能聊天机器人。话不多说，上python！
[更多玩法关注wxpy官方文档](https://wxpy.readthedocs.io/zh/latest/)**
## 1.初始化：
```
from wxpy import *           # 导入模块
bot = Bot()                             # 初始化机器人，扫码登陆 
friend = bot.friends().search('AIbot')[0]       #搜索昵称为"AIbot"的好友
group = bot.groups().search('test')[0]          #搜索名称为"test"的群聊
```

## 2.自动发送、回复指定消息：
自动发送消息：
```
friend.send('for test')             #自动向"AIbot"好友发送消息
group.send('for test')             #自动向"test"群组发送消息
```
自动回复消息：
```
@bot.register(friend)         # 回复 friend 的消息
def reply_my_friend(msg):
    return 'received: {})'.format(msg.text)        # 回复消息内容

@bot.register([friend,Group])          #自动回复所有好友和群聊
def auto_reply(msg):
    if isinstance(msg.chat, Group) and not msg.is_at:
        return                      #如果是群聊，但没有被 @，则不回复
    else:
        return '收到消息: {}'.format(msg.text)        # 回复消息内容
```

## 3.引入图灵机器人：
引入智能聊天机器人，才是python玩微信的精髓hhh。wxpy库目前提供了两种自动聊天机器人接口，图灵机器人和小i机器人，本文中选择受众度更高的图灵机器人。
调用图灵机器人需要申请api-key，[申请点击链接即可！](http://www.turingapi.com/)

下面给出图灵机器人自动回复好友"AIbot"消息的全代码：

```
from wxpy import *
bot = Bot()
friend = bot.friends().search('AIbot')[0]
tuling = Tuling(api_key='4e2xxxxxxxa8')    #api_key为你申请的api_key

@bot.register(friend)
def reply_my_friend(msg):
    tuling.do_reply(msg)     #图灵机器人自动回复

bot.join()              #保持登陆状态
```
话不多说，上图，日常基本聊天还是可以的：
![和图灵机器人的聊天](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS5heDF4LmNvbS8yMDIwLzA1LzA3L1ltVDlmSy5qcGc?x-oss-process=image/format,png)
图灵机器人的智能化程度还是很高的，日常聊天都很ok。
但是虽然是免费的，但是有限制次数，认证后一天100条。总的来说日常简短的聊天还是不错的选择。
#### 如有错误或建议，请在评论处指出！
