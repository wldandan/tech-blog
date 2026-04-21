# MindSpore易用性SIG 2022年上半年回顾总结

原文链接：https://zhuanlan.zhihu.com/p/544735380

---

​

目录

>  _2022年3月17日，[昇思MindSpore](https://zhida.zhihu.com/search?content_id=209185388&content_type=Article&match_order=1&q=%E6%98%87%E6%80%9DMindSpore&zhida_source=entity)社区技术委员会(TSC)决议通过昇思MindSpore易用性SIG的申请，正式成立易用性SIG，帮助开发者打通使用昇思MindSpore过程的“最后一公里”。_

时光似箭，转眼已经进入2022年下半年了。在过去的2022上半年中，MindSpore易用性SIG与开发者一起，围绕MindSpore易用性提升，开展了一系列技术活动，如易用性特性开发、易用性技术分享、知乎/CSDN等平台上的AI工程和MindSpore[易点通](https://zhida.zhihu.com/search?content_id=209185388&content_type=Article&match_order=1&q=%E6%98%93%E7%82%B9%E9%80%9A&zhida_source=entity)专栏建设等，帮助开发者打通使用MindSpore过程的“最后一公里”。非常感谢大家一直以来的支持与鼓励。打开2022上半年的记忆，让我们一起回顾过往，砥砺前行！

  * 2022年上半年，MindSpore易用性SIG：（1）共发展近**500** 名AI开发者加入交流群。（2）先后邀请**12位嘉宾** 举办**3期** 易用性技术分享活动，分享使用MindSpore的经验和案例，B站观看直播总人数**500+** 。（3）开展了手把手视频体验活动和1.7版本自动安装体验活动，各位开发者贡献了个人经验技术干货**40+** 篇。
  * 2022年4月，在知乎平台创建了AI工程和MindSpore易点通专栏，共发布AI工程系列和MindSpore易点通系列文章**23** 篇；易点通专栏中收录了各位开发者们贡献的从MindSpore安装、[数据处理](https://zhida.zhihu.com/search?content_id=209185388&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86&zhida_source=entity)、模型开发到应用案例等经验总结和案例**58** 篇，吸引关注者**860+** ，总阅读/播放量达**11万+** 次。
  * 在易用性特性开发方面，MindSpore易用性SIG先后纳入**5** 个大颗粒特性到开源活动中，累计已有**12** 名开发者报名参与并提交了**4k+** 行代码。



## 易用性SIG技术分享活动回顾

MindSpore易用性SIG分别于4月16日、5月28日、7月7日，先后线上举办了《聊聊AI框架的易用性》、《从零开始，“易”起上手》、《MindSpore在CV领域的应用案例》等三期技术分享活动，邀请高校师生/企业/个人开发者等**12** 位嘉宾分享他们使用MindSpore的经验和案例。三期技术分享共吸纳约**240+** 位开发者加入易用性SIG交流群。同时，为贴近开发者的实际诉求，向大家征集了后续的技术分享话题。调查结果主要来自个人开发者和高校师生。根据问卷调查结果显示，大家对**CV、AI[工程化](https://zhida.zhihu.com/search?content_id=209185388&content_type=Article&match_order=1&q=%E5%B7%A5%E7%A8%8B%E5%8C%96&zhida_source=entity)、NLP、强化学习**四个领域的话题比较感兴趣。我们后续在这些领域会组织更多技术话题分享。

除后续话题征集外，问卷就已经举办的前2期技术分享进行了满意度调查，平均满意度**4.9** 分（5分制）。其中，有不少受访者表示愿意作为分享嘉宾，分享自己基于MindSpore的应用案例或者[前沿技术](https://zhida.zhihu.com/search?content_id=209185388&content_type=Article&match_order=1&q=%E5%89%8D%E6%B2%BF%E6%8A%80%E6%9C%AF&zhida_source=entity)。

### **易用性SIG技术分享**

**第一期：聊聊AI框架的易用性**

      * 易用性SIG发起人**想飞就飞** 和易用性SIG Lead **Tong** ：MindSpore易用性SIG愿景、目标与规划
      * Apache软件基金会董事、华为开源能力中心专家**[姜宁](https://zhida.zhihu.com/search?content_id=209185388&content_type=Article&match_order=1&q=%E5%A7%9C%E5%AE%81&zhida_source=entity)** ：提升项目易用性，降低开发者摩擦力
      * MindSpore资深开发者**丁一超** ：我的昇思学习之路



视频回放：[https://www.bilibili.com/video/BV1Wu411y7u1?p=1](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV1Wu411y7u1%3Fp%3D1)

**第二期：从零开始，“易”起上手**

      * MindSpore资深开发者**[张辉](https://zhida.zhihu.com/search?content_id=209185388&content_type=Article&match_order=1&q=%E5%BC%A0%E8%BE%89&zhida_source=entity)** ：MindSpore[漫游世界](https://zhida.zhihu.com/search?content_id=209185388&content_type=Article&match_order=1&q=%E6%BC%AB%E6%B8%B8%E4%B8%96%E7%95%8C&zhida_source=entity)
      * MindSpore Dev Toolkit技术负责人**Chissica** ：智能辅助编程
      * MindSpore DFx专家**Miao** 与**Louie** ：AI框架功能调试的思考与MindSpore实践
      * MindSpore易用性专家**CQU弟中弟** ：MindSpore[模型迁移](https://zhida.zhihu.com/search?content_id=209185388&content_type=Article&match_order=1&q=%E6%A8%A1%E5%9E%8B%E8%BF%81%E7%A7%BB&zhida_source=entity)经验分享



视频回放：[https://www.bilibili.com/video/BV1Wu411y7u1?p=3](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV1Wu411y7u1%3Fp%3D3)

**第三期：MindSpore在CV领域的应用案例**

      * 复旦大学[计算机科学](https://zhida.zhihu.com/search?content_id=209185388&content_type=Article&match_order=1&q=%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6&zhida_source=entity)与技术学院博士生**[彭博](https://zhida.zhihu.com/search?content_id=209185388&content_type=Article&match_order=1&q=%E5%BD%AD%E5%8D%9A&zhida_source=entity)** ：基于MindSpore的深度学习模型在多媒体领域的实践
      * [南京理工大学](https://zhida.zhihu.com/search?content_id=209185388&content_type=Article&match_order=1&q=%E5%8D%97%E4%BA%AC%E7%90%86%E5%B7%A5%E5%A4%A7%E5%AD%A6&zhida_source=entity)沈阳博士生**沈阳** ：基于MindSpore的[细粒度](https://zhida.zhihu.com/search?content_id=209185388&content_type=Article&match_order=1&q=%E7%BB%86%E7%B2%92%E5%BA%A6&zhida_source=entity)哈希学习
      * MindSpore爱好者**甜粽** ：MindSpore1.7在Windows和Linux环境下的安装



视频回放：[https://www.bilibili.com/video/BV1Wu411y7u1?p=4](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV1Wu411y7u1%3Fp%3D4)

## 易点通专栏建设

2022年4月，在知乎平台创建了AI工程和MindSpore易点通专栏。知乎平台共发布AI工程系列文章和MindSpore易点通系列文章**23** 篇，包含**AI工程、AI设计模式、AI论文解读、MindSpore易点通机器人实践** 等。同时，易点通专栏中收录了各位开发者们贡献的从MindSpore安装、数据处理、模型开发到应用案例等经验总结和案例**58** 篇，共吸引关注者**860+** ，总阅读/播放量达**11万+** 次。

## 易用性特性开发任务回顾

基于前期的开发者访谈和问卷调查结果，我们梳理了开发者呼声最高的易用性提升方向，主要集中在**多平台支持、功能调试、[开发环境](https://zhida.zhihu.com/search?content_id=209185388&content_type=Article&match_order=1&q=%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83&zhida_source=entity)、模型迁移、资料查找**等方面。因此，我们在开源实习活动和开源活动中推出了一系列易用性特性开发任务，邀请各位开发者一起参与开发，并获得了各位开发者的踊跃参与。

累计已有**12** 名开发者报名参与易用性特性开发，提交了**4k+** 行代码。这些特性分别是：

  * MindSpore易点通知识问答机器人开发
  * MindSpore Dev Toolkit（IDE插件）中的知识问答机器人集成
  * MindSpore在Windows+GPU下的原生版本编译支持
  * MindSpore功能调试助手（TroubleShooter）开发
  * MindSpore易用性评测



## 易用性体验改进系列活动回顾

同时，MindSpore易用性SIG于4月-6月，策划发布手把手视频体验活动，共吸引**60+** 人参与活动，参与者们主动贡献个人经验技术干货**40+** 篇，并提出了不少优化建议，为内容优化提供基础。同时，在5月-6月，策划发布1.7版本自动安装体验活动，开发者反馈通过自动化脚本一键安装MindSpore，明显地提升了安装效率。

## 总结

2022上半年，是耕耘，也是收获。MindSpore易用性SIG各类活动的顺利举办、MindSpore易用性的提升离不开广大开发者们的支持与参与，在此向各位开发者伙伴们表示诚挚的感谢！

2022下半年，MindSpore易用性SIG将会再接再厉，和广大开发者们一起，为打造易用、好用的MindSpore持续努力，帮助开发者们打通使用MindSpore过程的“最后一公里”。
