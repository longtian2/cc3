# Commit message 日志规范 #
 *如果引用此文请提供原文出处。*

在1982年3月，詹姆士·威尔逊（James Q. Wilson）及乔治·凯林（George L. Kelling）在《Broken Windows》一文中提出破窗理论，该理论是依据美国斯坦福大学心理学家菲利普·辛巴杜（Philip Zimbardo）于1969年进行的一项关于两辆汽车停放的对比实验而来，理论认为：如果有人打坏了一幢建筑物的窗户玻璃，而这扇窗户又得不到及时的维修，别人就可能受到某些示范性的纵容去打烂更多的窗户，久而久之，这些破窗户就给人造成一种无序的感觉，结果在这种公众麻木不仁的氛围中，犯罪就会滋生、繁荣。

我在耳闻该理论还得益于阿里的一位架构师，后来在阅读《代码整洁之道》一书时，发现书中也提到该理论，此时引起了我的共鸣。现实中也不乏这样的事情发生，我们会发现：如果大家都在遵守交通规则等红绿灯时，此时如有人带头闯红灯，遵守规矩的人也会按耐不住去效仿他，造成交通秩序的混乱。在软件项目的质量管理上也是如此，如果我们不制定规范，疏于管理，久而久之，那么你将得到如废墟一般的项目，因为它将处于混乱不堪，晦涩难懂，业务扩展困难的窘境。此时再来谈规范、管理，已经难上加难了，为何不在“闯红灯”时加以制止，因为那将是最容易的。

在确定了软件项目的分支流程管理模型后，我们应该确定Commit message 日志规范。

无论是通过SVN还是Git来管理项目源代码文件，都应该树立Commit message日志规范。在业界被广泛认可的当属AngularJS所提倡的规范，我们一起来领略一下它的风采。[也可以去AngularJS GitHub上查看。](https://github.com/angular/angular.js/blob/f3377da6a748007c11fde090890ee58fae4cefa5/CONTRIBUTING.md#commit "也可以去AngularJS GitHub上查看。")

**消息格式**：**提交的每条消息都是由标题，正文和页脚组成**。而标题是由类型、影响范围（选填）和主题构成，是必不可少的；正文和页脚是可选填的；整个消息体内容应控制在100字以内。

    <type>(<scope>): <subject>
    //空一行
    <body>
    //空一行
    <footer>

1、Type做如下区分：

	feature或者feat：一个新功能。虽然AngularJS是使用的feat，而我更倾向于使用feature，这恰好于feature分支呼应起来。
	fix：一个错误修复
	docs：仅文档更改
	style：不影响代码含义的更改（空格，格式化，缺少分号等）
	refactor：代码更改既不修复也不增加功能
	perf：改进性能的代码更改
	test：添加缺少的测试
	chore：对构建过程或辅助工具和库（如文档生成）的更改
	revert：代码回退

2、scope 用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同

3、subject 是 commit 目的的简短描述，不超过50个字符

4、body 是更为详细的描述

5、可以包含 Breaking Changes 和 Closes 信息

以上就是AngularJS 的Git commit message 规范，是否也能够引起你的共鸣勒？这毕竟是按照pull-request的风格设计的，是为了规范社区开发者贡献代码所提出的规范，而我们要将该规范放在公司项目上，为了能更好的体现公司项目迭代开发的风格，可以对Type部分做优化。分享一下目前项目中采用的方式，是在Type段上加上迭代版本号和需求号或者修复的Bug号，譬如“***1.0.0-feature[11]:XXXX***”。在确定规范后必须严格执行才行，如有不法分子，项目负责人要在第一时间找到开发同学并且revert所提交的代码。在这里抛砖引玉了，如有更好的方式请一起讨论。

参考文献：

https://github.com/angular/angular.js/blob/f3377da6a748007c11fde090890ee58fae4cefa5/CONTRIBUTING.md#commit
http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html

联系方式：

https://github.com/longtian2

**如有用请不吝打赏**

![](https://github.com/longtian2/cc3/blob/master/images/wechat_pay.png)
