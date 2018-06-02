>原文链接：[https://mp.weixin.qq.com/s/4EXgR4GkriTnAzVxluJxmg](https://mp.weixin.qq.com/s/4EXgR4GkriTnAzVxluJxmg)

「itchat」一个开源的微信个人接口，今天我们就用itchat爬取微信好友信息，无图言虚空  
三张图分别是「微信好友头像拼接图」、「性别统计图」、「个性签名统计图」  

「微信好友头像拼接图」  
![](https://user-gold-cdn.xitu.io/2018/6/2/163be26b40e33f0b?w=640&h=640&f=png&s=888348)  

「性别统计图」  
![](https://user-gold-cdn.xitu.io/2018/6/2/163be28718f4e9a0?w=640&h=480&f=png&s=9706)

「个性签名统计图」  
![](https://user-gold-cdn.xitu.io/2018/6/2/163be2baaf9ea3ab?w=536&h=539&f=png&s=192442)

#### 安装
```
pip3 install itchat
```
主要用到的方法：  
`itchat.login()` 微信扫描二维码登录  
`itchat.get_friends()` 返回完整的好友列表，每个好友为一个字典, 其中第一项为本人的账号信息，传入`update=True`, 将更新好友列表并返回, `get_friends(update=True)`  
`itchat.get_head_img(userName="")` 根据`userName`获取好友头像

#### 微信好友头像拼接图
获取好友信息，`get_head_img`拿到每个好友的头像，保存文件，将头像缩小拼接至一张大图。  
先获取好友头像：
```
def headImg():
    itchat.login()
    friends = itchat.get_friends(update=True)
    # itchat.get_head_img() 获取到头像二进制，并写入文件，保存每张头像
    for count, f in enumerate(friends):
        # 根据userName获取头像
        img = itchat.get_head_img(userName=f["UserName"])
        imgFile = open("img/" + str(count) + ".jpg", "wb")
        imgFile.write(img)
        imgFile.close()
```
这里需要提前在同目录下新建了文件夹`img`，否则会报`No such file or directory`错误，`img`用于保存头像图片，遍历好友列表，根据下标`count`命名头像，到这里可以看到文件夹里已经保存了所有好友的头像。  

接下来就是对头像进行拼接 

遍历文件夹的图片，`random.shuffle(imgs)`将图片顺序打乱

用640*640的大图来平均分每一张头像，计算出每张正方形小图的长宽，压缩头像，拼接图片，一行排满，换行拼接，好友头像多的话，可以适当增加大图的面积，具体代码如下：
```
def createImg():
    x = 0
    y = 0
    imgs = os.listdir("img")
    random.shuffle(imgs)
    # 创建640*640的图片用于填充各小图片
    newImg = Image.new('RGBA', (640, 640))
    # 以640*640来拼接图片，math.sqrt()开平方根计算每张小图片的宽高，
    width = int(math.sqrt(640 * 640 / len(imgs)))
    # 每行图片数
    numLine = int(640 / width)

    for i in imgs:
        img = Image.open("img/" + i)
        # 缩小图片
        img = img.resize((width, width), Image.ANTIALIAS)
        # 拼接图片，一行排满，换行拼接
        newImg.paste(img, (x * width, y * width))
        x += 1
        if x >= numLine:
            x = 0
            y += 1

    newImg.save("all.png")

```
好友头像图成型，头像是随机打乱拼接的  

![](https://user-gold-cdn.xitu.io/2018/6/2/163be26b40e33f0b?w=640&h=640&f=png&s=888348) 

#### 性别统计图
同样`itchat.login()`登录获取好友信息，根据`Sex`字段判断性别，1 代表男性（man），2 代表女性（women），3 未知（unknown）
```
def getSex():
    itchat.login()
    friends = itchat.get_friends(update=True)
    sex = dict()
    for f in friends:
        if f["Sex"] == 1: #男
            sex["man"] = sex.get("man", 0) + 1
        elif f["Sex"] == 2: #女
            sex["women"] = sex.get("women", 0) + 1
        else: #未知
            sex["unknown"] = sex.get("unknown", 0) + 1
    # 柱状图展示
    for i, key in enumerate(sex):
        plt.bar(key, sex[key])
    plt.show()
```
性别统计柱状图
![](https://user-gold-cdn.xitu.io/2018/6/2/163be28718f4e9a0?w=640&h=480&f=png&s=9706)
#### 个性签名统计图
获取好友信息，`Signature`字段是好友的签名，将个性签名保存到.txt文件，部分签名里有表情之类的会变成emoji 类的词，将这些还有特殊符号的替换掉。
```
def getSignature():
    itchat.login()
    friends = itchat.get_friends(update=True)
    file = open('sign.txt', 'a', encoding='utf-8')
    for f in friends:
        signature = f["Signature"].strip().replace("emoji", "").replace("span", "").replace("class", "")
        # 正则匹配
        rec = re.compile("1f\d+\w*|[<>/=]")
        signature = rec.sub("", signature)
        file.write(signature + "\n")

```

`sign.txt`文件里写入了所有好友的个性签名，使用wordcloud包生成词云图，`pip install wordcloud`  
同样可以采用`jieba`分词生成词图，不使用分词的话就是句子展示，使用`jieba`分词的话可以适当把`max_font_size`属性调大，比如100。   
需要注意的是运行不要在虚拟环境下，`deactivate` 退出虚拟环境再跑，详细代码如下：  
```

# 生成词云图
def create_word_cloud(filename):
    # 读取文件内容
    text = open("{}.txt".format(filename), encoding='utf-8').read()

    # 注释部分采用结巴分词
    # wordlist = jieba.cut(text, cut_all=True)
    # wl = " ".join(wordlist)

    # 设置词云
    wc = WordCloud(
        # 设置背景颜色
        background_color="white",
        # 设置最大显示的词云数
        max_words=2000,
        # 这种字体都在电脑字体中，window在C:\Windows\Fonts\下，mac下可选/System/Library/Fonts/PingFang.ttc 字体
        font_path='C:\\Windows\\Fonts\\simfang.ttf',
        height=500,
        width=500,
        # 设置字体最大值
        max_font_size=60,
        # 设置有多少种随机生成状态，即有多少种配色方案
        random_state=30,
    )

    myword = wc.generate(text)  # 生成词云 如果用结巴分词的话，使用wl 取代 text， 生成词云图
    # 展示词云图
    plt.imshow(myword)
    plt.axis("off")
    plt.show()
    wc.to_file('signature.png')  # 把词云保存下

```
句子图

![](https://user-gold-cdn.xitu.io/2018/6/2/163be2baaf9ea3ab?w=536&h=539&f=png&s=192442)

使用`jieba`分词产生的词云图  
![](https://user-gold-cdn.xitu.io/2018/6/2/163be324e8dc099e?w=491&h=497&f=png&s=195209)  
看来，「努力」 「生活」 还是很重要的  

itchat 除了以上的信息，还有省市区等等信息都可以抓取，另外还可以实现机器人自动聊天等功能，这里就不一一概述了。  

最后附上github地址：[https://github.com/taixiang/itchat_wechat](https://github.com/taixiang/itchat_wechat)  

欢迎关注我的博客：[https://blog.manjiexiang.cn/](http://blog.manjiexiang.cn/)  
欢迎关注微信号：春风十里不如认识你  
![image.png](https://upload-images.jianshu.io/upload_images/7569533-cfeb1f55473a2143.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
