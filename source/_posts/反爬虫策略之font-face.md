﻿---
title: 反爬虫策略之font-face
date: 2020-03-02
---
**在互联网快速发展的时代里，爬虫可以说是无处不在。各类热搜以及热点新闻，时时刻刻都在被爬虫获取。如果开发者并不想让内容大范围传播呢？如何应对爬虫呢？常见的方法有验证码验证，以及对频繁登陆ip进行屏蔽，除此之外，还有一种通过font-face自定义字体的方式，可以让爬取到的内容和网页显示的内容完全不一样，从而产生一种是否爬错内容了的感觉。**
## 1.font-face介绍：
谈到font-face，也许很多人并不熟悉，但是说到应用此技术的公司，大家肯定都很熟悉了，58同城，猫眼电影等，都采用了font-face策略来应对爬虫。以[58同城](https://su.58.com/zufang/pn2/?PGTID=0d300008-0000-5965-8ce1-1873463f7758&ClickID=1)为例，打开审查元素，发现页面内的3室2厅1卫 和120㎡中的数字均变成了乱码 ----"驋室麣厅龒卫","龒麣龤㎡"。所以即便爬虫找到了正确位置，也依然无法获得正确信息。
![58同城页面信息](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS5heDF4LmNvbS8yMDIwLzA1LzA3L1llWkV3Ui5wbmc?x-oss-process=image/format,png)
## 2.font-face原理：
仔细观察审查元素信息，在styles栏里发现了font-family，这就是自定义字体，可以把乱码显示成正常的数字。
![审查元素](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS5heDF4LmNvbS8yMDIwLzA1LzA3L1llbVl6OC5wbmc?x-oss-process=image/format,png)
于是我们要找到font-family字样，右键点击网页源代码，搜索font-family，找到@font-face等关键字，通过分析data:application/font-ttf，可以猜测后面长串字符串应该是对某个ttf字体进行了base64编码。
![查看网页源代码](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS5heDF4LmNvbS8yMDIwLzA1LzA3L1lld0Z3bi5wbmc?x-oss-process=image/format,png)
上python，解码生成ttf字体文件。

```
from fontTools.ttLib import TTFont
import base64
import io

key='''
AAEAAAALAIAAAwAwR1NVQiCLJXoAAAE.......AAAA    #key值为源代码中的长串字符串
'''
data = base64.b64decode(key) #base64解码
fonts = TTFont(io.BytesIO(data)) #生成二进制字节
fonts.save('font-face.ttf')
```
将生成的font-face.ttf通过[百度字体编辑器](http://fontstore.baidu.com/static/editor/index.html)打开，可以看到对所有数字进行了重新编码。
![自定义字体](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS5heDF4LmNvbS8yMDIwLzA1LzA3L1llRHJrUi5wbmc?x-oss-process=image/format,png)
是不是有点儿疑惑？怎么对数字重新编码？？？其实也可以理解为是对编码对应的图形进行了改变，比如同样是编码(0x9a4b)，在常规字体中是"驋"的模样可能在B字体中就是"2"的模样，在自己设计的字体里，可以任性的把(0x9a4b)编码对应的图形设计成"2"的形状，举个例子:
![自定义字体有不一样的形状](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS5heDF4LmNvbS8yMDIwLzA1LzA3L1llUmxpNC5qcGc?x-oss-process=image/format,png)
同样是(0x9a4b)编码，在字体A中，你认为是2，在字体B中，即便和字体A中形状不一样，你一样认为是2，但在自定义字体中，你会认为是w。同样的道理，我们可以把常规字体中的乱码，设计成数字的形状。
所以在自定义字体的基础上，我们可以变相认为对字体进行了加密，从而起到保护重要数据的作用。
## 3.实现font-face：
理解了font-face的原理后，我们可以将font-face策略应用到自己的web项目中。为了简便操作，我直接用了第二步中在58同城网站下载的字体hhh(自行设计字体也很cool，想怎么画怎么画)。在前端html文件中加入以下内容:
```
<style>
@font-face{font-family:'secret';src:url('data:application/font-ttf;
charset=utf-8;base64,AAEAAAALAIAAAwAwR1NVQiCLJXoA.......AAAAAAAAA')
format('truetype')}.strongbox{font-family:'fangchan-secret','Hiragino Sans GB',
'Microsoft yahei',Arial,sans-serif,'宋体'!important}
</style>
<style>
.font{font-family:'secret','Hiragino Sans GB','Microsoft yahei',Arial,sans-serif,'\5B8B\4F53' !important}
</style>
```
上面一段的作用为引入名叫"secret"的自定义字体,然后通过style设计.font，使所有class为"font"的标签应用此自定义字体。换言之，当你想用自定义字体时，在相应的标签中加入class="font"，如下html源码和前端显示结果:
![html源码](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS5heDF4LmNvbS8yMDIwLzA1LzA3L1llb3Q5VS5wbmc?x-oss-process=image/format,png)
![前端显示结果](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS5heDF4LmNvbS8yMDIwLzA1LzA3L1llb0poVC5wbmc?x-oss-process=image/format,png)
可见2020 04-27日期字样在源码中显示为乱码。
[访问上图中我的web项目，点击链接即可，](http://wxt123.top/news)(暂时只对页面右侧一栏中的日期做了font-face加密。)


#### 如有错误或建议，请在评论处指出！
