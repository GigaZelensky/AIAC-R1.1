# AI-AC:半开源时频变换+深度学习云反作弊(云服务端也公开)

### QQ群 764360489

## **源代码**

```
https://github.com/huzpsb/AIAClient
```

## **简介**

人工智能领导的反作弊市场已经被闭源付费垄断很久了。我要打破它。

本项目只是POC，并没有完善人工智能以外的其他部分。即本项目的唯一功能是：在假设所有攻击全部是能够被完成的前提下，判断玩家是否使用了第三方瞄准工具。（推荐配合Grim食用）

## **原理**

在明确了本工具只是一个技术测试项目的前提下，我们可以来聊一聊原理而不是急着说该怎么用。



------------------------------------

首先，是采样。采样部分全部开源。

采样分为2步。

第一步，是计算SHB(SignedHitBox)

什么是SHB?简单而言，SHB大小等于攻击一个实体，实体所需的最小正方形HitBox。

没听懂？没事，有图：

![image](https://user-images.githubusercontent.com/41772578/175223257-36d1891e-bdb7-4eeb-8175-429dc9b9820e.png)



SHB的符号取决于箭头相对于连线是顺时针还是逆时针。

第二步，是进行FFT。

什么事FFT?唉，好累，不想码字。得了，[传送门](https://zhuanlan.zhihu.com/p/347091298)

有的人可能会问，~~诶你在说什么我听不懂~~诶SHB不是实数吗，怎么变成复数了?实数就是复数的一部分呢~你品,你细品。



------------------------------------



然后，是通讯。通讯部分客户端开源。

通讯其实 不就是普普通通的C/S架构嘛。

采样时，我们会得到一个流式的数据。这对FFT与特征提取带来了很大的障碍。怎么办呢？

切分。我在这里选择将数据22改一组，切割后发送给服务端。

服务端计算返回分类。

对了，差点忘了一个很重要的问题：明明只是一个反作弊插件，为什么要搞什么C/S架构？

~~因为Matrix、Reflex、AAC都是这么搞的，所以我直接借鉴了，正好蹭一下云概念的热度~~

因为体积。众所周知，Java作为编程语言，最大的缺点是慢。太慢了。C++快。都但是C++它不好写，更无法直接与MC交互。怎么办呢？

有一个叫JavaCPP的桥梁。但是JavaCPP心宽体胖（？），所以导致打包后直接150MB就去了。

你不可能一个插件150MB，对吧？

所以只能采用C/S架构。服务端是多线程的，不用担心卡顿，并且我还进行了基准性能测试，每秒，我的笔记本，可以处理150次请求。相当于在起床战争中的一万名玩家（假设玩家三秒一次有效攻击）的数据量。



------------------------------------



最后，是训练与计算。本部分不开源。但是也不混淆。你们，看着办。

通讯写好了，最难的还是得扛。来吧！

首先，我构造了一个5层的神经网络，节点数为22,30,30,30,3。

![image](https://user-images.githubusercontent.com/41772578/175223295-cd1115aa-a402-45a6-8a75-d3069fd63cf5.png)



刚开始，我以为我所有努力白费了。因为模型死活不收敛。

后来，因为一次意外，我训练次数多打了亿个0，然后就去睡觉了

![image](https://user-images.githubusercontent.com/41772578/175223325-fe147988-2790-4aa7-8358-dbb3ee9c24e7.png)

第二天起来，我发现这个训练了13万次的神经网络后验准确率已经有了99%。

这个模型就这我发布的演示版内。

计算没什么新鲜的，得了，略去（doge）



------------------------------------



## **使用方法**

~~终于入题了是吧~~

**1,** 运行AI-AC服务端。

![image](https://user-images.githubusercontent.com/41772578/175223344-6ce7ae22-fe92-4540-8c4f-d060a5f946b2.png)

应该，都会吧？

**2,** 为自己创建账号。

set <用户名> <有效期(天)> <ip> <调用次数限制>

这个用户名只是标识符。与客户端（mc服务端）无关。

事实上，mc插件里面就没有对应登录的配置。

可能有人问，你服务端这些圈钱功能都写好了，我可以拿它圈钱吗？

嗯，是这样的，我写这些功能的初衷是希望群组等共享反作弊的计算资源，并且易于计量各方投入，而不是拿它圈钱。当然如果你能一边遵守AGPL一边圈钱，我也拦不住你。

**3,** 配置MC服务端

服务端配置文件如下：

``` yaml
#是否显示调试信息，如序列异常，vl等
debug: false
#AI服务器地址
server: "127.0.0.1"
#AI端口
port: 1451
#惩罚指令
cmd: "ban %p"
```

其中server、port与刚才的服务端一致即可。

设置好惩罚命令，好了没了。

对了，目前这个插件不适合生产使用。但是测试，应该，问题不大，吧？



## **捐献数据**

**不需要懂Java，不影响游戏，您们的数据对我很重要！**

非常简单，方法如下

1，输入/type <你的数据名称>

2，继续游戏

3，在结束时向我提交您的服务端下的AI-AC文件夹

请不要捐献垃圾数据（例如：故意疯狂摇晃鼠标、攻击没有击退或大小与人不一致的实体、在无其他AC时使用百米等）既是为了社区，也是为了你↓

本项目为公益项目，无法为捐献者提供其他回报；但是如果需要，可以提供基础模型在您的自定义数据上额外训练100轮的定制版模型（你需要捐献不少于1000组数据，因为这是1000*100=100000的最低训练量）

“没有人比你自己更懂自己的需要。” 定制版模型将给您带来惊喜！ 
  
 *为了节省算力，默认不提供本赠品。如有需要，请加群联系。

（隐私政策：这个定制版模型不会公开；但是您捐献的数据将在匿名化后用于训练社区基础模型；我们不保留原始数据，所以请单次提交不少于1000组数据。请勿重复提交相同数据，否则可能会因为训练指标异常而被认为提交垃圾数据。）

数据名称请包含：

昵称，是手打还是G，如果是G是KA的Single,Multi还是AimBot(Trigger/AutoClicker不算G)

除此之外，数据名称请尽可能不要更换。万分感谢！



## **下载地址**

https://huzpsb.lanzouw.com/b09heaish

密码:74i4

说明这个下载包里面虽然什么都有，但是我可能在Release中发布对其中某些组分的更新。

即：所有release都是相当于这个包的差量更新。什么？听不懂？那你还是去群里面下吧 =.=


## **完结撒花**

捐献数据: 邮件到huzpsb@qq.com 标题注明[AI-AC捐献] 或者加群

捐献资金：https://afdian.net/@huzpsb

感谢来自华中科技大学数学系的刘老师，光学与电子信息学院的郑老师，The **[Eclipse Deeplearning4J](https://deeplearning4j.konduit.ai/)** (DL4J) ecosystem

