---
title: "看看写写：闲话程序员职业"
date: 2021-11-04T23:10:01+08:00
categories:
- 生活
- 随笔
tags:
- 随笔
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/diary.jpg
---
21世纪的程序员是一个经常游走在舆论场里的职业。看了一些事情，听了一些事情，自然有一些事情想要写下来。
<!--more-->
## 推荐频道
- [一些很好的软件开发分享 NDC Conference](https://www.youtube.com/@NDC)

## 关于职业

### 软件设计
1. 一些开发过程中的体会：
    1. 设计先行：
        1. 一定要先捋顺业务流程，然后按照流程，以及可能的扩展去思考设计
        1. 永远不要想着，“先完成一个版本，反正是个简单的需求”，后期需求越来越复杂，没有设计的程序早晚会被重构
        1. 也不要过度优化和设计，在模块变得足够大的时候再进行拆分也是可以的，但业务划分，模块、类之间的协作方式是要尽早设计和优化的
    1. 重视测试：
        1. 先编写测试用例，或者至少在开发进行到一定程度后，就应该开始使用测试用例
    1. 不写重复代码：
        1. 只要有大段的重复（超过5行），就说明设计不合理，重新考虑功能的封装。
    1. 时刻思考合作：
        1. 代码是要给自己看，也要给别人看的，别人看了还有可能要用，要扩展
    > 失败的教训：对于TPU上位程序的维护，最初只需要做一点简单的事情，但是随着需求复杂化，通信的状态机越来越复杂，原有的处理方式难以承受。
### 工作意识
1. 提高计划性：日常维护自己的工作计划，按照优先级顺序，主动找事做，推动相关合作同事。
1. 提高责任意识：对自己主责的业务部分，要做到全周期的负责。
1. 坚持学习：提高业务理解，整理业务笔记，提高自己相关领域的知识储备。最终要做一个能**独当一面**的有管理能力的工程师。
1. 懂行业方向：知道行业的热门领域，工业发展方向。
1. 带动同事：对于业务相关部门，要互相配合，并想办法为对方提供更好的合作环境。
1. 积极主动：多承担一些任务，多在任务中付出一些。瞄准长期回报。
### 一些分享
1. 2022年初元旦聚会分享：
    1. 定期思考各种为什么：为什么缺钱？为什么技术学不好？为什么程序写的不好？
    1. 分析问题，想办法，做各种尝试。短期规划、长期规划。
    1. 做PPT的能力始终很重要，背后代表了汇报能力和展示能力。也要时刻提醒自己：**工作是做给别人看的，程序是写给别人用的**
    1. 沟通能力是重中之重，沟通的越精细越好

### 活动策划
&emsp;&emsp;2021年9月，参加了北京市的优培计划培训班。出于未知的某种原因被选为小组长，在这个短暂的一周的学习时间里，也体验到了许久未曾体验过的管理岗的经历。上一次还是在小学做班长。

&emsp;&emsp;总的来说，因为现在面对的都是同龄人，所以相对来说不需要太多工作，主要就有以下几个意识即可。
- 提前了解组员基本信息，大致分配工作。
- 提前准备一些合适的暖场活动（谁是卧底、抢数字等），在这个过程中可以让输了、赢了的人做一下自我介绍。
- 及时通知、安排必要的议程
- 保证能联系到所有人，并保证所有人都能联系到我。
### 项目管理
&emsp;&emsp;2022年8月上旬和中旬，参加了中国电子学会主办的第七届世界机器人大会的筹备工作中的闸机研制、保障工作。在这段工作经历中，非常清晰的认识到了自己目前在项目管理方面的一些欠缺问题。虽然这是一个团队的工作，不应把问题全部归咎于自身，但是确实也显示出了自己在相关方面缺少经验的问题。

&emsp;&emsp;从这次的经历，我大概理解项目中的需要项目经理负责流程主要分为：需求整理、技术调研、编排计划、开发、测试调优、部署、保障、收尾这几个部分。

&emsp;&emsp;**前三个**阶段，最重要的事情就是需求整理，这次活动的主要问题是缺少明确清晰的需求整理，而且是边做边问需求（闸机的各类性能指标、界面要求、流程要求），极大的造成了时间上的浪费。而这些可以**先自己形成几套方案**，供领导、同事开会快速讨论，快速决定。计划方面，则需要保留充足的预留时间，建议有1/3的预留量，并尽量提前完成。

&emsp;&emsp;**开发**过程，一定要保持良好习惯，尤其是**项目配置**、**版本控制**。在开发进入中后期再调整配置、版本管理是非常困难的，

&emsp;&emsp;**测试调优**，在这个过程中，应当尽量使用工具代替人，所以也需要进行**测试技术**调研。测试和开发应当同步进行。调优则一定要放到最后开展，并搜索、使用更强力的**调优工具**。

&emsp;&emsp;**部署&保障**，这其实才是最容易出现问题的环节。一般来说作为技术人员，开发过程总是可以想办法控制的。但是部署和保障，需要和其他公司、团队、个人沟通，而这些第三方往往有不同的办事流程、办事风格，具体到个人，可能有人乐于合作，有人很难推进。这就需要充分**提前沟通**，**做好预案**，每一步都要考虑出错的可能，比如：闸机是否允许直接就地安装，电力、网络搭建是否能够快速完成，人员能否进场、天气温度湿度是否影响设备运行、备用件是否充足等等。而需要开放给公众使用的设备、软件，则要充分考虑不同人群的**困难点**，如：老弱病残孕是否都能很方便的使用、缺少流程必备材料怎么办、设备是否会造成人员生命财产安全问题、舆情如何管控。可以用换位思考的方式，**列表格模拟各类人群**，总结一些注意要点。

&emsp;&emsp;**临场能力**，作为项目负责人，应当具备和各类人群接触、各类问题接触的能力。在遇到问题的时候，保持平和的心态，尽量不要和其他人产生冲突，**克制负面情绪**，学习更好的和人沟通的办法，无论是对上级还是同事，都应当做到不卑不亢，想办法尽量达成自己的目的。

- 总结：
    1. 和其他单位接触时，最好和更有责任心的人沟通对接，如果对方不够负责，在自己应当负责的限度内做到最好，并保留相关证据
    1. 和政府部门协调工作时，可以先问清所属单位，然后根据对方需求回答关键问题。比如公安最看重的是安全问题，回答警察提出的问题时应当先从安全角度考虑，不要因为紧张而说了过多的内容，反而会导致对方听不懂而怀疑回答的正确性
    1. 各类情况都要准备应急预案，本质上都是对服务做一个冗余、容灾
    1. 项目再忙也要保重身体，留时间关爱家人
### 文件编撰
1. 在立项方向的文件编撰工作中也遇到了一些问题，和同事们沟通中学到了一些经验：
    1. 立项、研究类文件，可以着眼于三个要点：创意、创新、创效。其中创意是基础，创新是实现手段，创效是最终目标。
    1. 同时包含创意、创新、创效是最好的，但是很难。用传统方法实现一个创意，达成创效，也是非常常见的情况。
    1. 核心点一定要放到创效上，分析的内容主要包括可行性、经济性。突出工作内容的意义和价值。
    1. 技术在文件中并不需要很大篇幅来说明，技术只是手段，不是目的。
## 关于身体
## 关于生活
1. 人际关系：
    1. 打好自身条件：光靠嘴说是表达不出来个人能力等各方面的，对他人有吸引力是打造良好人际关系的第一步
    1. 和人做朋友：大多数情况下表达赞美、表达肯定，但不应单方面付出，或者不平等付出
        - 要点
            1. 不熟悉的情况下，表达自己乐于结交，表明喜欢欣赏的态度即可，想办法打消别人的芥蒂，而不是过度贴近对方，把握好度
            1. 没有必须的事情，极好的话题，**不要**没事就找人聊天
            1. 推进同事、朋友等人际关系的简单办法：找**吃饭**
            1. 真正关心的问题，如果不确认对方的反应，可以想一个开玩笑的想法去打听
            1. 嘴上说的，不代表实际要做的。无论自己还是对方，都是一样
            1. 平常心、保持自我，该做的都做了就行了，不能因为一个人、一件事，**武断**地改变自己
            1. 慢就是快，快反而慢。所谓日久见人心，做事情也不要冲动
        - 雷点
            1. 在吗？在干嘛？吃了嘛？：即使是很熟的人这类话也没啥意义。找人直接说事儿，没事儿不闲聊
            1. 不要满脑子想：**如何让对方**重视自己、欣赏自己、喜欢自己。无论双方地位、水平如何，就是以**朋友**的方式相处
            1. 不要打着做朋友的旗号，行自私自利的事情，**克制自己**的控制欲、占有欲
            1. 即使对对方有所图，也应该建立在朋友互惠互利的基础上
1. 自我管理：
    1. 不要过度沉溺于快餐式的任何工作、事情、关系
    1. 做事情不带情绪，带情绪就不要去做
    1. 不自我感动、不妄自菲薄
    1. 适当强硬
    1. 相信自我，相信未来
1. 驾驶车辆：
    1. 安全是第一要义，在确保安全的前提下，保持一个平稳的速度就可以了
    1. 进入村庄等视线狭窄，人员密集的位置，必须减速慢行，注意行人
    1. 借道超车必须保持足够距离，一旦开始，必须明确让所有相关车辆、人员知道自己要超车，给出充分的信号（闪灯、鸣笛）
        1. 72km/h时，是20m/s，此时每晚一秒制动，将多失去20m安全距离。
        1. 借对向车道超车注意：双向72km/h时，相对速度是40m/s。如果超车需要10s完成，则与对向车辆需要至少间隔800m以上，总之就是非常远才行。这还是72km/h。实际上在国道上，经常出现90km/h左右的速度。因此间隔需要进一步提升。另外注意：不建议超车时加速过多，遇到突发情况难以处理。
        1. 所以整体建议就是，没必要的话，单向只有一个车道的情况下，就不要超车了。
    1. 多摁喇叭，安全大于一切
    > 从驾驶能看出来的性格问题：优柔寡断，做事准备不充分
## 关于副业
1. 考虑一下辅导机构，做一些兼职辅导
1. 考虑一些代码相关的项目，比如做一些玩具
## 参考资料
- [在中国，也能一辈子做程序员](https://mp.weixin.qq.com/s/lb-c8vfbygUp28FcQA9FRg)