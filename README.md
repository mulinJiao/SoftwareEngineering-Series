![](https://i.postimg.cc/dtdTv30G/image.png)

# Software Engineering Series | 软件工程、算法与架构

软件开发就是把一个复杂的问题分解为一系列简单的问题，再把一系列简单的解决方案组合成一个复杂的解决方案。而软件开发中最大的挑战，就是即能够快速高效地针对需求、环境的变化做出改变，也能够持续提供稳定、高可用的服务。

在笔者的系列文章中，分门别类地讨论了 Web、服务端、基础架构等某些具体的领域，而本系列关注形而上的软件工程相关的思考方式、通用能力与设计原则。为了更好地阐述，本系列默认使用 Java 为实现语言，不过你也可以在各篇的『编程实现』章节中查阅 TS、Go、Python、PHP、Rust 等其他版本的实现。

# Preface | 前言，从编码到软件系统

一般来说，在软件工程中，我们往往会面临三类问题：性能问题（The Performant System Problem）、系统的嵌入式问题（the Embedded System Problem）以及复杂领域建模问题（the Complex Domain Problem）。

![](https://i.postimg.cc/wBjMt2jj/image.png)

## 性能

以著名的 Twitter 为例，你注册，你发推文，你喜欢别人的推文，这就是它。如果 Twitter 很简单，为什么其他人不能这样做呢？很明显，Twitter 的真正挑战实际上不是“它做什么”，而是它“如何做它能做的事情”。

Twitter 面临着每天为大约 5 亿用户提供服务的独特挑战。Twitter 解决的难题实际上是一个性能问题。当挑战是表现时，我们是否使用严格类型的语言并不重要。

## 系统嵌入

嵌入式系统是计算机硬件和软件的组合，其目的是实现对系统的机械或电气方面的控制。我们今天使用的大多数系统都是建立在一个非常复杂的代码层上，如果不是最初编写的，通常会编译成 C 或 C ++。

用这些语言编码不适合胆小的人。在 C 中，没有物体这样的东西; 我们作为人类喜欢物体，因为我们可以很容易地理解它们。 C 是程序性的，这使我们用这种语言编写的代码更加难以保持清洁。 这些问题还需要了解较低级别的细节。

C ++确实让生活变得更好，因为它具有面向对象，但挑战仍然是与较低级别的硬件细节进行有趣的交互。因为我们对这些问题使用的语言并没有那么多的选择，所以在这里讨论 TypeScript 是无关紧要的。

## 编程

## 架构与复杂性控制

对于某些问题，这个挑战不是关于处理更多请求的扩展，而是根据代码库的大小进行扩展。企业公司有复杂的现实问题需要解决。在这些公司中，最大的工程挑战通常是：

- 能够逻辑地（通过域）将整块的各个部分分成更小的应用程序。然后，物理上（通过有界上下文的微服务）将它们分开，以便可以分配团队来维护它们

- 处理这些应用程序之间的集成和同步

- 对域概念进行建模并实际解决域的问题

- 创建由开发人员和领域专家共享的无处不在（包罗万象）的语言

- 不会迷失在大量编写的代码中，并且在不破坏现有代码的情况下无法添加新功能

我基本上描述了 Domain Driven Design 解决的问题类型。对于这些类型的项目，您甚至不会考虑不使用像 TypeScript 这样的严格类型的语言。

## 强/弱类型语言

对于设计到复杂领域建模的问题，如果您不选择 TypeScript 而是选择 JavaScript，则需要一些额外的努力才能成功。您不仅需要更加熟悉 vanilla JavaScript 中的对象建模功能，而且还必须知道如何利用面向对象编程的 4 个原则（封装，抽象，继承和多态）。

这可能很难做到。 JavaScript 自然不具备接口和抽象类的概念。使用 vanilla JavaScript 无法轻松实现 SOLID 设计原则中的“接口隔离”。单独使用 JavaScript 也需要一定程度的纪律作为开发人员才能保持代码清洁，而且一旦代码库足够大，这一点至关重要。您还需要确保您的团队在如何在 JavaScript 中实现通用设计模式方面拥有相同的学科，经验和知识水平。如果没有，你需要引导他们。

在像这样的领域驱动项目中，使用严格类型语言的强大好处不是表达可以做什么，而是更多关于使用封装和信息隐藏来通过限制实际允许的域对象来减少错误的表面区域。代码大小通常与复杂域问题相关联，其中大型代码库意味着复杂的域，但情况并非总是如此。当项目的代码量达到一定大小时，跟踪存在的所有内容变得更加困难，并且更容易最终重新实现已编码的内容。

复制是精心设计和稳定的软件的敌人。当新开发人员开始在已经很大的代码库上编码时，这一点尤为明显。Visual Studio Code 的自动完成和 Intellisense 有助于浏览大型项目。它适用于 TypeScript，但它在 JavaScript 方面有限。对于我知道将保持简单和小型的项目，或者如果我知道它最终将被丢弃，我将不太愿意推荐 TypeScript 作为必需品。

## 生产级别软件与 Demo

生产软件是您关心的代码，或者如果它不起作用您将遇到麻烦的代码。 这也是你为其编写测试的代码。 一般的经验法则是，如果您关心代码，则需要对其进行单元测试。如果您不在乎，请不要进行测试。

宠物项目是不言自明的。做你喜欢的事。 您没有专业承诺维护任何标准的工艺。

## 性能与高可用

## 研发效能

# About

![default](https://i.postimg.cc/y1QXgJ6f/image.png)

## Copyright | 版权

![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg) ![](https://parg.co/bDm)

笔者所有文章遵循 [知识共享 署名-非商业性使用-禁止演绎 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh)，欢迎转载，尊重版权。如果觉得本系列对你有所帮助，欢迎给我家布丁买点狗粮(支付宝扫码)~

![](https://i.postimg.cc/y1QXgJ6f/image.png?raw=true)

## Home & More | 延伸阅读

![](https://i.postimg.cc/CMDmg2SQ/image.png)

您可以通过以下导航来在 Gitbook 中阅读笔者的系列文章，涵盖了技术资料归纳、编程语言与理论、Web 与大前端、服务端开发与基础架构、云计算与大数据、数据科学与人工智能、产品设计等多个领域：

- 知识体系：《[Awesome Lists | CS 资料集锦](https://ngte-al.gitbook.io/i/)》、《[Awesome CheatSheets | 速学速查手册](https://ngte-ac.gitbook.io/i/)》、《[Awesome Interviews | 求职面试必备](https://github.com/wx-chevalier/Awesome-Interviews)》、《[Awesome RoadMaps | 程序员进阶指南](https://github.com/wx-chevalier/Awesome-RoadMaps)》、《[Awesome MindMaps | 知识脉络思维脑图](https://github.com/wx-chevalier/Awesome-MindMaps)》、《[Awesome-CS-Books | 开源书籍（.pdf）汇总](https://github.com/wx-chevalier/Awesome-CS-Books)》

- 编程语言：《[编程语言理论](https://ngte-pl.gitbook.io/i/)》、《[Java 实战](https://ngte-pl.gitbook.io/i/java/java)》、《[JavaScript 实战](https://ngte-pl.gitbook.io/i/javascript/javascript)》、《[Go 实战](https://ngte-pl.gitbook.io/i/go/go)》、《[Python 实战](https://ngte-pl.gitbook.io/i/python/python)》、《[Rust 实战](https://ngte-pl.gitbook.io/i/rust/rust)》

- 软件工程、模式与架构：《[编程范式与设计模式](https://ngte-se.gitbook.io/i/)》、《[数据结构与算法](https://ngte-se.gitbook.io/i/)》、《[软件架构设计](https://ngte-se.gitbook.io/i/)》、《[整洁与重构](https://ngte-se.gitbook.io/i/)》、《[研发方式与工具](https://ngte-se.gitbook.io/i/)》

* Web 与大前端：《[现代 Web 开发基础与工程实践](https://ngte-web.gitbook.io/i/)》、《[数据可视化](https://ngte-fe.gitbook.io/i/)》、《[iOS](https://ngte-fe.gitbook.io/i/)》、《[Android](https://ngte-fe.gitbook.io/i/)》、《[混合开发与跨端应用](https://ngte-fe.gitbook.io/i/)》

* 服务端开发实践与工程架构：《[服务端基础](https://ngte-be.gitbook.io/i/)》、《[微服务与云原生](https://ngte-be.gitbook.io/i/)》、《[测试与高可用保障](https://ngte-be.gitbook.io/i/)》、《[DevOps](https://ngte-be.gitbook.io/i/)》、《[Node](https://ngte-be.gitbook.io/i/)》、《[Spring](https://github.com/wx-chevalier/Spring-Series)》、《[信息安全与渗透测试](https://ngte-be.gitbook.io/i/)》

* 分布式基础架构：《[分布式系统](https://ngte-infras.gitbook.io/i/)》、《[分布式计算](https://ngte-infras.gitbook.io/i/)》、《[数据库](https://ngte-infras.gitbook.io/i/)》、《[网络](https://ngte-infras.gitbook.io/i/)》、《[虚拟化与编排](https://ngte-infras.gitbook.io/i/)》、《[云计算与大数据](https://ngte-infras.gitbook.io/i/)》、《[Linux 与操作系统](https://ngte-infras.gitbook.io/i/)》

* 数据科学，人工智能与深度学习：《[数理统计](https://ngte-aidl.gitbook.io/i/)》、《[数据分析](https://ngte-aidl.gitbook.io/i/)》、《[机器学习](https://ngte-aidl.gitbook.io/i/)》、《[深度学习](https://ngte-aidl.gitbook.io/i/)》、《[自然语言处理](https://ngte-aidl.gitbook.io/i/)》、《[工具与工程化](https://ngte-aidl.gitbook.io/i/)》、《[行业应用](https://ngte-aidl.gitbook.io/i/)》

* 产品设计与用户体验：《[产品设计](https://ngte-pd.gitbook.io/i/)》、《[交互体验](https://ngte-pd.gitbook.io/i/)》、《[项目管理](https://ngte-pd.gitbook.io/i/)》

* 行业应用：《[行业迷思](https://github.com/wx-chevalier/Business-Series)》、《[功能域](https://github.com/wx-chevalier/Business-Series)》、《[电子商务](https://github.com/wx-chevalier/Business-Series)》、《[智能制造](https://github.com/wx-chevalier/Business-Series)》

此外，你还可前往 [xCompass](https://wx-chevalier.github.io/home/#/search) 交互式地检索、查找需要的文章/链接/书籍/课程；或者在 [MATRIX 文章与代码索引矩阵](https://github.com/wx-chevalier/Developer-Zero-To-Mastery)中查看文章与项目源代码等更详细的目录导航信息。最后，你也可以关注微信公众号：『**某熊的技术之路**』以获取最新资讯。
