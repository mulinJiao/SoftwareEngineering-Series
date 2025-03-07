[![返回目录](https://i.postimg.cc/WzXsh0MX/image.png)](https://parg.co/UdT)

> 本文是整体倾向于表述 GraphQL 相对于 REST 的一些优越性，但是并不代表笔者认为 GraphQL 就可以完全代替 REST。尺有所短，寸有所长，REST 的简单易用性导致一定缺陷的同时也方便很多需要快速开发的环境。笔者在自己的选择中会使用 REST 作为稳定的、长期的、简单的接口风格，选择 GraphQL 作为易变的、复杂的接口实现方式。另外本文篇幅略长，建议读者选择自己感兴趣的章节阅读。笔者正在进行[ConfigurableAPIServer](https://github.com/wx-chevalier/ConfigurableAPIServer)这个开源项目，即是允许通过纯配置方式进行可审查的动态 API 配置，可以用于数据中心或者需要快速使用的业务场景。笔者拟将自己的一些最佳实践融合进去，因此也欢迎大家提意见与开 Issue。

# WebAPI 风格变迁

逝者如斯夫，不舍日月，软件开发的技术架构日新月异，一直处于不断地变革与迭代之中，而程序猿们一直也在简单易用与复杂可维护、优化与差不多得了、用些黑科技与不断测试保证稳定之间徘徊游离。着眼于服务端应用程序开发，或者称之为 API 开发中，我们也一直在寻找合适的方案来解决不断变革的功能逻辑需求与简单可扩展高可用之间的矛盾。随着移动端的兴起与 Web 技术的革新，富客户端程序日渐占据市场主流，从一个前端开发者的角度来看这些年的变革，我们所 Consume 的 API 的风格也经过了好几代的变迁。在早期以 WebService 为主流，SOA 、 XML-RPC 流行的年代里，访问远程服务需要使用某些特定的通信协议传送大量的定义严谨的结构化数据，异常复杂。后来在通信方式上以通用 Web 技术 (HTTP) 日渐成为主流，HTTP 协议也就成为了客户端 Consumer 与服务端 Server 之间主要的通信协议。从笔者的经验与个人认知而言，在服务端应用程序开发中，可以将 API 定义或者路由定义与数据库定义看做两个自变量，将实际的代码实现看做因变量。一旦 API 与数据库定义好了，那么中间的分层系统的代码实现在现有通用的架构风格或者架构模式下都是有章可循。换言之，如果我们希望能实现一个健壮、可用、可扩展、可维护的服务端应用程序，那我们就应该进行合理的 API 设计与数据库设计。在早期的架构设计中，我们习惯使用 SQLServer、Oracle 这样的单点商业级数据库，即可能一个系统的全部业务表都存放在单个关系型数据库中。此时很多业务逻辑我们实际上是通过复杂的 SQL 连接操作实现的，不可否认，这是对于性能的保障，但就像笔者在下文 CAP 理论中讨论的那样，高性能与灵活性之间本身也存在着矛盾。在这个时代，一旦数据库表划分与接口设计定了，服务端程序更像对于 CRUD 的封装。后来随着大数据时代的到来与 NoSQL 存储的兴起，无论是对于关系型数据库的垂直划分还是譬如 Redis 这样的缓存数据库的应用，导致了原本单点存储的业务数据被分散在多个存储媒介中，这也就赋予了服务端应用程序更大的灵活性。另外随着现在微服务架构的火热，使得人们更倾向于将原本巨石型的系统拆分为多个解耦的功能模块，而每个功能模块往往也会有自己独立的数据存储。 **在这种情况下，API 风格定义，或者在某些文章中所称的路由定义，也就成了一个重要的不可避免问题。**

## Overview | 概览

谈到 API 风格，笔者最早期是习惯采用动作化、语义化定义，譬如描述对于登录操作的接口可以定义为：

```
http://api.com/doLogin?username=&password=
```

此时对于 Consumer 而言，可以通过接口的定义方便的知道接口的功能与应该提交的参数，不过这种原生的方式也存在着致命的缺陷：

- 命名的随意性：语义化命名最大的缺陷就是在于主观性太强，笔者在团队实践中发现每个人都有自己的命名习惯，很多在自己看来显而易见的命名在其他人看来简直就是妈的智障。这一点会使得整个前后台交互、代码管理存在极大的障碍，很多 Consumer 都抱怨压根不知道应该使用哪个 API。
- 接口功能的不确定性：语义化的接口，每个接口都是为了处理某个具体的业务逻辑。譬如我们如果有获取我收藏的商品与我购买的商品这两个接口需求，那么就会定义为两个接口：

```
http://api.com/getFavoriteGoods -> 获取我收藏的商品
http://api.com/getBuyedGoods -> 获取我购买的商品
```

如果从资源的角度来看，这两个接口最终返回的都是商品这一资源，但是却分割为了两个 API，会让 Consumer 觉得这两个 API 返回的是完全不相关的东西，使得无论前后台都存在着冗余重复的代码。后来，REST 出现了。在 REST 的关于统一接口原则的描述中，使用了多个约束来指导 API 的行为：

- 资源的识别(identification of resources )，在部分描述中也被称作 End Point，直观的理解就是一个 URL
- 通过表述对资源执行的操作，直观的理解就是 HTTP 动词：GET/PUT/DELETE/POST
- 自描述的消息
- 作为应用状态引擎的超媒体

在 REST 架构风格下，服务端会决定应该传输给客户端的数据。而每个 Endpoint(URL) 会返回某个特定的资源，譬如一个用户或者一篇博客，或者一组评论的集合。而如果你需要返回一个资源的不同属性，譬如有时候在列表中只需要简略信息而有时候需要详情，那么你应该创建两个不同的 Endpoints，或者使用不过的过滤参数。笔者一直认为，REST 在这里强调的对于信息抽象的核心 —— 资源，强调的是任何被命名的信息都能够作为一个资源，譬如一份文档或者一张图片都可以被称为资源。REST 并没有着重分析我们现在遇到的关于业务逻辑中处理对象的资源划分与资源关联 ( 实体映射 ) 问题，因此，REST 只是描述了一种简单的接口风格。譬如在上文提及的用户登录操作的 API，资源化方式应该是：

```
[GET] http://api.com/user?username=&password?
or
[POST] http://api.com/user_state?username=&password=
```

即登录这个操作应该看做对于某个用户资源的获取或者对于某个用户状态资源的创建。REST 这种资源化的接口风格非常美妙，十分简单易懂。不过一旦涉及到复杂的界面呈现或者接口逻辑，问题就来了。假设我们现在正在为某个 APP 提供接口支撑，每个界面都是由多个复杂组件构成的，而每个组件都会从某个特定的 API Endpoint 中获取数据，此时无论前后台的代码都是简单而优雅的。但是，问题来了，你在一个界面上需要调用 N 个不同的 API，极大的影响了效率，被 PM 骂了一顿。因此你要为这个界面所有需要的数据提供一个单独的统一接口，在一次请求中即将所有的数据响应返回。客户端也需要重构部分请求代码，然后将获得的数据依次返回给各个组件。灾难从此开始，随着你的 APP 的业务需求、用户界面的不断迭代，你写了越来越多这样的聚合型接口，REST 是没有版本控制机制的，因此每次你在重构服务端代码的时候，你都不敢动原有的，即使已经被废弃的代码，你只敢不断地创建新的接口，然后让老接口自生自灭。最后，估计你对你的 Server，就这个一团乱麻般的感觉: ![](http://7u2q25.com1.z0.glb.clouddn.com/1-5cfW5V8SyQB54UUJecY96Q.jpeg) 举个现实中的例子，如下图有一个列表，每个列表项都是一个卡片。当我对 Android 开发说让他先去请求下列表，获取所有 Card ID 再根据 Card ID 迭代 N 次获取 Card 详情的时候，我差点被打死: ![](https://coding.net/u/hoteam/p/Cache/git/raw/master/2016/6/5/379C27B1-1E67-48A2-BE9B-457E1BA324D0.png) 还有一个比较常见的问题，现在流行一个概念，叫 BFF(Backend for Forend)，正如上文所说的，Consumer 在拿到数据之前并不知道自己能获取哪些数据，传输过来的数据完全由服务端决定，客户端只能够在拿到数据之后再进行验证。笔者在实践中就经常发现，因为 Server 的疏忽大意可能把返回的某个键值写错了，或者生产环境下出现了某个异常导致返回出错，以至于前端直接崩溃。用吃货的角度来看，现有的接口是服务员给你个菜单，你下单点菜，但是你压根不能确定他会给你烧一盘啥样的菜，并且你也不能点菜单之外的菜。而以 BFF 的角度看，应该是 Consumer 把几个食材交给服务员，并且指定要做成啥样子的菜，厨房再怎么失火走水，至少也要把食材还给我。上文描述的这种快速变化的业务逻辑与界面呈现与可扩展、可维护的接口实现之间的矛盾，前端对于接口易用性的需求与后端对于代码的可维护性、模块化的需求之间的矛盾，催生了笔者对于 RARF 架构风格的讨论。不过因为能力所限，RARF 还是宣告失败，但是笔者的设计理念与设计思想的趋势上是与 GraphQL 保持一致，因此笔者最终放弃了 RARF 而使用 GraphQL 这一完整的数据查询语言与详尽的说明书式的接口风格。对于 GraphQL 与 REST 的对比可以参照下图，具体的从 REST 到 GraphQL 的变迁对比请参考下文的 GraphQL 章节: ![](https://coding.net/u/hoteam/p/Cache/git/raw/master/2016/6/5/1-RapSQHY4a5TWqDaKpxUe-A.png)

## CAP & BASE: 服务端应用程序开发中的 CAP 与 BASE 理论

CAP 理论是分布式系统构建中的基础理论之一，其中的 C(Consistency)，一致性即指所有的节点都能访问同一份最新的数据副本、A(Availability) ，可用性即指每个请求都能接收到一个响应，而 P(Partition Tolerance)，分区容忍性即指除了整个网络的故障外，其他的故障 ( 集 ) 都不能导致整个系统无法正确响应。CAP 理论的核心即是一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，最多只能同时较好的满足两个。在笔者刚才描述的服务端应用程序构建的场景中，同样可以用类似于 CAP 理论的方式进行描述。

- **Partition Tolerance**: 在上文的描述中，我们业务系统构建的一个目标是在干净整洁的代码的基础上提供灵活多变的接口，这就好像饭店一样，如果我们为 Consumer 提供了一盘做好的黄焖鸡米饭，Consumer 是很难再把它变成炒鸡杂。而如果我们提供的是原料食材，那么 Consumer 想做成什么都可以。封装程度越高的 Service/API 的可重用性与可组合性越低。笔者在上文提及，任何一个复杂的应用程序皆可以表示为简单的 CURD 操作的排列组合，而如果我们的 Service 层或者 API 层只提供最基本的，即粒度最小的操作，那么 Controller 层或者 Consumer 的自由组合的能力也就越强，整个应用系统的灵活性与可变性也就越高。总结而言，Partition Tolerance 即是衡量一个服务端应用系统可以提供对外的 API 的粒度，粒度越低代表着分割容忍性越好。最极端的情况即是 Server 只提供最基本的 CRUD 操作，就好像只是对于数据库做了简单的封装。
- **Consistency**: 直观的说，如果我们以保证灵活性、可变性为目标，势必会减少对于数据库连接操作的使用。换言之，原本一条 SQL 语句连接查询获得的内容变成了执行两次 SQL 语句，这就导致了可能在两次查询的间隙数据已经发生了变化，最终导致返回给 Consumer 的数据存在不一致性。
- **Availability**: 在 Server 中，可用性即指某个 API 的性能，或者说是响应时间。如果要保证分割容忍性，势必会将原本可以一次执行的 SQL 语句分为多次，导致查询效率急剧下降，也就导致了可用性受到影响。而如果要保证一致性，那么可以使用事务的方式，同样会导致可用性的减小。

目前绝大部分的 Server 端的实现都是以**CA**为目标，即牺牲了分割容忍性，保证强一致性与可用性，这也是 ACID 理论的表现。很多情况下我们会发生某个 API 其实只是执行了某条及其复杂的 SQL 语句，该 SQL 语句可能连接查询了十几张表并且返回数据。这也就导致了 API 的可变性极差，你要修改 API 返回的某个字段也就意味着要修改最底层被执行的那个 SQL 语句，等于重新构建了一个新的 API。如果我们的系统目标是保证**CP**，即牺牲可用性。那么我们可以引入事务或者锁，即在一个 Session 的多次查询中锁定全表，从而保证返回数据的一致性。如果我们的系统目标是保证**PA**，即牺牲一致性，那么最好的方式就是引入缓存机制，可以设置在固定时间或者事件触发的情况下再进行真实查询操作，其他情况下可以从缓存中获取数据，这样就能够较好地保证用户的响应时间。

随着分布式系统的发展，eBay 工程师提出大规模分布式系统的实践总结，在 ACM 上发表文章提出 Bash 理论是基本可用、软状态和最终一致性。不要求实时一致性，但一定要实现最后一点。笔者目前秉持的在 Server 端的开发过程中也是遵循 BASE 理论，这在下面会有详细介绍，这里还是阐述下 BASE 的内涵：

- 基本可用(Basically Available )。分布式系统在故障时允许损失可用性，保证核心业务可用。音频直播或是做活动时，当业务量非常大的时候可以降级。做游戏也是，在战斗的时候最关心数值的增长，看了多少人都无所谓，缓解核心内容的压力。
- 软状态(Soft State )。允许系统中出现的中间状态，中间状态不会耽误可用性。在写代码、编程业务的设计上，必须容忍有一定的临时数据同步，考虑到全局锁和数据多版本的对比，把各个节点的相关数据都上锁，这是一个悲观锁，一旦写任务，其他人都能改我的数据，这是比较悲观的心态。

## Backends For Frontends: 一切为了前端

> [Pattern: Backends For Frontends](http://samnewman.io/patterns/architectural/bff/)

### General-Purpose API Backend: 统一的后端接口

### Consumer-Oriented Response: 面向消费者的响应 / 前端定义接口

GraphQL 最有趣的特性之一，是它本质上是面向消费者的，响应体的结构不取决于服务端，而是完全由客户端驱动。直观的理解就是 GraphQL 规定客户端在发起请求的时候必须显示的标注所需要获得的原子资源的类型以及在返回体中的位置，这一点和 Immutable.js 中的`update`函数很类似，你都需要显示指明到叶子节点。这样确实使得整个请求需要很多额外的数据参数与编码工作，但是它就将消费者解耦了，并且强迫服务端遵守 Postel 法则 ( 对自己严格，对他人宽容 )。

## Reference

### REST

### RARF

### GraphQL

> 本部分大量文章来自于 Medium，请自备梯子

# REST: 架构风格与统一接口

> **REST 做了很多，但也没做很多** 本部分是对于 REST 做一个简短的回顾总结，了解的同学可以直接跳到下一章

笔者一直认为，就好像深度学习一样，REST 这个概念被神化了，人们赋予了很多它并不具备的功能。笔者认为，REST 在那个年代，最大的贡献就是提出了 CS 架构与无状态。REST 提出了资源化的接口风格，但是并没有解决真正实际应用中的复杂资源获取问题。后来也有很多关于 REST 的最佳实践，不过都没有形成一套统一的标准，还是各行其是。

## 从 BS 架构到 CS 架构的认知

## Best Practices: 来自 PayPal 的最佳实践

笔者在上文中一直强调 REST 的简单性，但是不可忽视的是在目前的真实应用中，我们可以通过各种各样的最佳实践来解决笔者在文中描述的 CAP 问题或者 BFF 模式。

# RARF: 基于响应式抽象资源流的服务端应用程序架构模式

> 本部分的[详细地址](https://segmentfault.com/a/1190000004600730) 本部分是笔者基于自己的工作对于 REST 做的一些封装与改进，其核心思想介于 REST 与 GraphQL 之间，可以直接跳过

## Motivation: 动因

RARF 的全称是 Reactive Abstract Resource Flow，字面意思是响应式抽象资源流，笔者把它称为基于抽象资源流的响应式处理的架构风格。一般来说，Server-Side 开发包含接口定义、具体业务逻辑处理方法、数据存储交互 (SQL+NoSQL) 这三个层次，RARF 也是面向这三个方面，提出了一些自己的设想。这边的 Reactive 表示 RFRF 同时关注了并发与异步这一方面的问题，并选择了 Reactive Programming 或者更明确一点的，FPR 来作为方法论。

### 面向用户的接口可读性与可用性 ( 可组合性 ) 的提升

#### 接口的可读性

REST 风格的一个特征就是资源化的接口描述，譬如：

```
[GET] http://api.com/books
[GET] http://api.com/book/1
[POST] http://api.com/book
```

这种资源化的接口可读性还是比较好的，通过动词与资源名的搭配很容易就能知道该接口描述的业务逻辑是啥。不过，在笔者浅薄的认知里，包括 REST 原版论文和后面看的各式各样的描述，都只针对单个资源进行了描述，即简单逻辑进行了表述。如果现在我们要添加一个购买书籍的接口，在该接口内包括了创建订单、扣费等一系列复杂逻辑操作，相信程序猿会倾向于使用：

```
http://api.com/doBookBuy
```

对于单个接口而言，可读性貌似上去了，但是，这最终又会导致接口整体命名的混乱性。确实可以通过统一的规划定义来解决，但是笔者个人认为这并没有从接口风格本身来解决这个问题，还是依赖于项目经理或者程序猿的能力。笔者在 RARF 中定义了一个概念，叫 Uniform Resource Flow Path( 统一资源流动路径 ) 来增加接口的描述性，同时 URFP 也是整个 RARF 的驱动基础。

#### 前端定制的接口 (Frontend-Defined APIs And URFP-Driven)

无论是 WebAPP 还是 iOS 还是 Android，都需要与业务服务器进行交互，我们一般称之为消费接口，也就是所谓 Consume API 的感觉。笔者一直有个形象的感觉，现在前后端交互上，就好像去饭店吃饭，厨师把菜单拿过来，消费者按照菜单定好的来点菜。而笔者希望能在 RARF 达成的效果，就是把点菜变成自助餐，也就是所谓的前端可以根据后端提供的基本的资源的四种操作来完成自己复杂的业务逻辑。

### 逻辑处理函数中的状态管理

#### 状态混乱

目前很多的 Controller 里是没有一个全局状态的，我们习惯了调用不同的 Service 然后进行判断处理在最后整合成一个结果然后返回给用户。这就导致了存在着大量的临时变量，譬如上面伪代码中的`a,b,c`。笔者认为的状态管理的结果，就是在一个逻辑处理的流程中，不需要看前 N 步代码即可以判断出当前的变量状态或者可能的已知值，特别是在存在嵌套了 N 层的条件判断之下。笔者认为的状态混乱的第一个表现即是缺失这样一种全局状态，顺便插一句，如果有学过 Redux 的朋友肯定也能感受到，在有全局状态树的情况下可以更好地追踪代码执行过程并且重现错误。另一个可能引起状态混乱的原因，即是 Service 的不可控性。借用函数式编程中的概念，我们是希望每个被调用的 Service 函数都是纯函数 ( 假设把数据库输入也作为一个抽象的参数 )，不会存在 Side Effect。但是不可否认，现行的很多代码中 Service 函数会以不可预知的方式修改变量，虽然有时候是为了节约内存空间的考虑，譬如：

```
ArrayList<Integer> array = new ArrayList();
Service(array);//在Service中向array里添加了数据
```

而这样一种 Side Effect 的表现之一就是我们在调试或者修改代码的时候，需要递归 N 层才能找到某变量到底是在哪边被修改了，这对于代码的可维护性、可变性都造成了很大的负担。

#### 抽象混乱

**因为每一个 Controller 都是面向一条完整的业务逻辑进行抽象，所以在朴素的认知下并不能很好地进行代码分割。**在符合自然认知规律的开发模式中，我们习惯先定义出接口，然后在具体的接口需要的功能时进行抽象，譬如在项目之初，我们需要一个获取所有商品列表的接口，我们可以定义为：

```
[GET]:/goodss -> 映射到getGoodsController
```

在编写这个 getGoodsController 方法的过程中，我们发现需要去 goods 表中查询商品信息，那我们会提出一个`selectGoodsAllService`的服务方法，来帮助我们进行数据查询。然后，又多了一个需求，查询所有价格小于 10 块的商品：

```
[GET]:/goods?query={"price":"lt 10"} -> 映射到getGoodsController
```

这时候我们就需要在 Controller 加个判断了，判断有没有附带查询条件，如果附带了，就要调用专门的 Service。或者也可以为这个查询写一个专门的 Controller：

```
[GET]:/getGoodsByPriceLess?price=10 -> getGoodsByPriceLessController
```

同样需要编写一个根据价格查询的 Service。然后，随着业务的发展，我们需要根据商家、剩余货物量等等进行查询，然后 Service 越来越多，慢慢的，就陷入了抽象漏洞的困境，就是我们虽然抽象了，但是根本不敢去用下面的代码，因为就怕那是个大坑，比较不知道它的抽象到底是遵循了什么的规则。

#### 依赖混乱

![](http://7xkt0f.com1.z0.glb.clouddn.com/5AB7141F-CDAA-4045-87CF-466CD3AEE73B.png)

上图描述了一个请求处理过程中的依赖传递问题，直观的感受是，当我们的业务逻辑系统变得日渐复杂之后，不依赖 Code Review 或者测试，特别是对于小团队而言，不敢去随便乱改一个现有的服务，宁可去为自己要处理的逻辑写一个新的服务，因为压根不知道现有的服务到底被多少个东西依赖着。牵一发而动全身啊。这就导致了随着时间的增长，系统中的函数方法越来越多，并且很多是同构的。举例而言，有时候业务需要根据不同的年龄来获取不同的用户，但是程序猿在初期接到的业务需求是查找出所有未成年的人，它就写了个方法`getMinorsService()`，这个服务不用传入任何参数，直接调用就好。然后某天，有个新的需求，查找所有的没到 20 岁的人，于是又有了一个服务`getPeopleYoungerThan20()`。这样系统中就多出了大量的功能交叉的服务，而这些服务之间也可能互相依赖，形成一个复杂的依赖网络。如果有一天，你要把数据库的表结构动一下 ( 譬如要把部分数据移到 NoSQL 中 )，就好像要踩 N 颗地雷一样的感觉吧。

说到依赖，相信很多人首先想到的就是 Spring 的 DI/IoC 的概念。不过笔者认为依赖注入和本文笔者纠结的依赖混乱的问题，还是有所区别的。依赖注入的原理和使用确实非常方便，但是它还是没有解决依赖划分混乱的问题，或者需要大量的劳动力去在代码之初就把所有的依赖确定好，那么这一点和 RARF 文初假设的场景也是不一致的。

最后，笔者在 RARF 中，不会解决依赖传递或者多重依赖的问题，只是说通过划分逻辑资源的方式，把所有的依赖项目明晰化了。譬如上面提到的 N 个具有交叉功能的 GET 函数，都会统一抽象成对于某个资源的抽象 GET 操作。换言之，以 URFP 的抽象原则进行统一调用。

### 挣脱数据库表结构 / 文档结构的束缚

在富客户端发展的大潮之下，服务端越来越像一个提供前端进行 CRUD 的工具。任何一个学过数据库，学过 SQL 的人，都知道，联表查询比分成几个单独的查询效率要来的好得多，这也是毋庸置疑的。但是，在本文所描述的情况下，联表查询会破坏原本资源之间的逻辑分割。这边先说一句废话：**在 MVC 中，当数据表设计和接口设计定了之后，中间的代码实现也就定了。** 举例来说，我们存在两个业务需求：获取我收藏的商品列表和我购买过的商品列表，这边涉及到三个表：goods(goods_id) ， goods_favorite(goods_id,user_id)，goods_order(goods_id,user_id) 。在 MVC 中，我们会倾向于写上三个接口，配上三个 Service，这就是典型的提高了查询效率，并且符合人们正常开发顺序的方式。不过，如果单纯从资源的逻辑分割的角度考虑的话，对于商品查询而言，应该只有一个`selectGoodsByGoodsId`这个 Service，换言之，把原来的单次查询拆分为了：先在其他表中查询出 goods_id，然后再用 goods_id 在 goods 表中查询商品详情。这样的逻辑分割是不是明晰了一点？是啊，这肯定会影响效率的，笔者也在想着能不能通过某种方法达成一个平衡。

## Solution: 解决方案

RESTful 中也有`all is resource`的概念，但是 RESTful 强调的是像超文本啊、某个音视频啊，这些都可以通过 URI 访问到，也就是可以当做一个资源然后被前端获取。这一点已经获得了广泛地认可，而在 RARF 中强调的抽象资源，就好像在电商系统中，我们都知道 goods 与 user 是两种资源，那么在描述用户收藏夹的时候，即对于 user_goods 这个表，算不算资源呢？当然算啊！换言之，描述两个资源之间关系的，无论是一对一，还是一对多，只要具备唯一标识的，就是独立的抽象资源。不过，如果哪一天逻辑设计上，把用户收藏的商品，变成了 JSON 字段然后存储在 user 表中，即成了 user 资源的一个属性，那么此时这种映射关系就不是一个资源了。因为它没有一个唯一标识。

## Pursuit: 愿景

缩了那么多，最后，我还是陈述下我在设计 RARF 一些莫名其妙的东西时候的愿景吧，其实看到现在机智的同学，应该能够感觉到，这个 RARF 和 MicroService 在很多设计理念上还是很类似的，这里先盗用下 MicroService 的 Benefits：

> Microservices do not require teams to rewrite the whole application if they want to add new features.
> Smaller codebases make maintenance easier and faster. This saves a lot of development effort and time, therefore increases overall productivity.
> The parts of an application can be scaled separately and are easier to deploy.

那么改造一下，RARF 的愿景就是：

- RARF 希望能够在修改或者增删某些功能时不需要把全部代码过一遍
- 基于 Resource 分割的代码库会更小并且更好管理，这会大大节省开发周期，提供产品能力
- 整个应用程序能够独立扩展、易于部署。就像 RARF 中，如果发现哪个 ResourceHandler 需求比较大，可以无缝扩展出去。

# GraphQL: 面向对象图网络 (Object Graphs) 的查询语言

> GraphQL is a Specification.

如果说 RESTful API 给笔者最大的诱惑，那应该是在于其的简单易用，我们可以用最简单的而且遵循一定规范的约定俗成的命名方式来定义接口风格并且实现接口。不过借用 ThoughtWorks 在 2016 年初的技术雷达指引中的描述，现实世界里的 REST 实现，经常发现它被错误地当做通过客户端服务器间频繁交互来获取对象网络的手段，而 GraphQL 本身就是一种远程接收对象图网络的协议。
