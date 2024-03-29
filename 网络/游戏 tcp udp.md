在编写网络游戏的时候，到底使用UDP还是TCP的问题迟早都要面对。

　　一般来说你会听到人们这样说：“除非你正在写一个动作类游戏，否则你就用TCP吧” 或者是 “你能够在MMO游戏中用TCP,因为魔兽世界就用的TCP！”

　　遗憾的是，这些观点都没有反映这个问题的复杂性。

**一、背景**

　　首先，说明一下，我之前主要是用TCP进行网络编程。我曾为一个流行的在线纸牌游戏编写服务器了好几年，在高峰期我们的每台服务器能够承受4000到10000个连接（同一台物理机器上有多个服务器进程在跑）都没有问题。在我来看，TCP是一种安全而且常见的选择。

　　尽管如此，我们最新的项目却是使用UDP协议，而且我们的项目无法通过任何方式在TCP下工作。事实上，项目一开始使用的TCP，但是后来发现我们使用TCP无法达到我们需求的连接数量时，我们只能换成UDP了。

**二、在使用中TCP表现怎么样呢**

　　从原理上，TCP的优势有：

　　简单直接的长连接

　　可靠的信息传输

　　数据包的大小没有限制

　　任何一个和TCP打过交道的人都知道，要实现一个稳定的TCP网络连接，需要处理各种隐藏的坑，比如断线检测、慢速客户端响应阻塞数据包，对开放连接的各种dos攻击，阻塞和非阻塞IO模型等等。

　　除了上面列出的这些问题外，一个好的TCP模块确实不好编码实现。

　　但是，TCP最糟糕的特性是它对阻塞的控制。一般来说，TCP假定丢包是由于网络带宽不够造成的，所以发生这种情况的时候，TCP就会减少发包速度。

在3G或WiFi下，一个数据包丢失了，你希望的是立马重发这个数据包，然而TCP的阻塞机制却完全是采用相反的方式来处理！

　　而且没有任何办法能够绕过这个机制，因为这是TCP协议构建的基础。这就是为什么在3G或者WiFi环境下，ping值能够上升到1000多毫秒的原因。

**三、为什么不用UDP**　　UDP相对TCP来说既简单又困难。　　举个例子来说，UDP是基于数据包构建，这意味着在某些方面需要你完全颠覆在TCP下的观念。UDP只使用一个socket进行通信，不像TCP需要为每一个客户端建立一个socket连接。这些都是UDP非常不错的地方。　　但是，大多数情况下你需要的仅仅是一些连接的概念罢了，一些基本的包序功能，以及所谓的连接可靠性。可惜的是，这些功能UDP都没有办法简单的提供给你，而你使用TCP却都可以免费得到。　　这也是人们为什么经常推荐TCP的原因。在用TCP的时候你可以不考虑这些问题，直到你需要同步连接的数量级达到500以上的时候。　　所以，是的，UDP没有提供所有的解决方法，但是就像你看到的那样，这也正是UDP好用的地方。在某种意义上来说，TCP对UDP就好比是Hibernate和手写SQL的区别。
**四、使用TCP失败的地方**　　人们经常给你建议，让你去使用TCP，比如“TCP跟UDP一样快”或者“游戏X用TCP如此成功，所以TCP当然是首选”，然而，他们完全没有理解为什么在那个特定的游戏中TCP是有效的，为什么UDP不按照顺序发送数据包呢？　　那么为什么魔兽世界采用TCP呢？首先我们需要解释这个问题。这个问题其实是“为什么魔兽世界有的时候1000毫秒以上的延迟还能够运行？”这是TCP的性质决定的，在发生丢包的时候，会产生巨大的延迟，因为TCP首先会去检测哪些包发生了丢失，然后重发所有丢失的包，直到他们都被接收到。　　可靠的UDP也是有延迟的，但是由于它是在UDP的基础之上建立的通信协议，所以可以通过多种方式来减少延迟，不像TCP，所有的东西都要依赖于TCP协议本身而无法被更改。就这一点来讲，一些人要开始提到Nagle算法了，实际上它是你在实现任意一个对延迟敏感的TCP模型时首先需要禁止使用的。　　那么魔兽世界以及其他的一些游戏是怎么处理延迟问题的呢？　　方法也很简单，他们能够隐藏掉延迟带来的影响。　　在魔兽世界中，玩家和玩家是无法碰撞的：因为这类碰撞是无法通过一些预测来处理的，但是玩家和环境之间的碰撞却是可以通过预测来处理的，所以这里使用TCP是没有问题的。　　我们来看一下魔兽世界的战斗就会发现，玩家的攻击指令发送给服务器的操作是放在比如“attack_entity(entity_id)”或者”cast_spell(entity_id, spell_id)“的接口中来做的，换句话说，瞄准操作是独立于进行的。如此一来，一些类似发起攻击动作和释放技能特效就能够在没有收到服务器确认的情况下就直接执行，比如展现冰冻技能的效果就可以在服务器没有返回数据前在客户端就做出来。　　客户端直接开始进行计算而不等待服务端确认是一种典型的隐藏延迟的技术。　　几年前，我为一个叫“Five Card Jazz”的纸牌游戏编写过客户端。它使用的是http协议，它比直接的TCP协议连接的延迟更加严重。　　我们用简单的纸牌绘制和抽牌的动画来掩盖延迟的问题，所以延迟的问题只在非常糟糕的连接下才会被看出来。这种方法也非常的典型：发送请求的同时开始播放牌桌的动画，一直播放翻动最后一张牌直到接收到了服务端传回来的数据为止。魔兽世界的战斗特效就是使用类似的原理。　　这也意味着，我们到底是使用TCP还是UDP取决于我们能否隐藏延迟。
**五、TCP在什么时候失效**　　一个采用TCP的游戏必须能够处理好突发的延迟问题（纸牌客户端就很典型，对突发性的一秒的延迟，玩家也不会产生什么抱怨）或者是拥有缓解延迟问题的好方法。　　但是如果你运行的是一个无法使用任何减缓延迟措施的游戏呢？玩家对玩家的动作类游戏通常就属于这个范畴，但是这也不仅仅限于动作类游戏。　　举个例子：　　我目前正在写一个多人游戏（War Arcana）。　　一种常见的操作是，你快速的移动你的角色通过一张充满战争迷雾的世界地图，但是一旦你探索过，迷雾就会被打开。　　为了确保游戏的规则，防止玩家作弊，服务器只能显示玩家当前位置附近的信息。这意味着不像魔兽世界，玩家无法在没有得到服务器响应的情况下，做出完整的动作。和Five Card Jazz相比，我们即使允许500毫秒的延迟，也已经非常困难了。　　在实现了游戏的原型后，在局域网内一切都进行的非常顺利，但当我们在WiFi环境下测试时，操作会间歇性的卡起来或者延迟高起来。写了一些测试程序之后发现，WiFi环境下偶尔会发生丢包行为，每当发生丢包的时候，服务器的响应速度就从100-150毫秒上升到1000-2000毫秒。　　没有任何办法可以绕过TCP的这个设置来避开这个问题。　　我们替换了TCP的代码，用了自定义的可靠的UDP来实现，把大量的丢包产生的延迟降到了仅仅只有50毫秒，甚至比以前TCP不丢包的情况一个来回的延迟还要小。当然，这只可能建立在UDP之上，这样我们才对可靠性拥有完全的掌控力。
**六、困惑：可靠的UDP只是TCP的一种简单的实现？**　　你有没有听过这种说法：“可靠的UDP就像TCP一样，所以还是用TCP吧”。　　问题是这种说法是错误的。可靠的UDP一点也不像TCP，要去实现一个特殊的阻塞控制。事实上，这也是你使用可靠UDP代替TCP的最大的原因，避免TCP的阻塞控制。　　另一个重点是可靠的UDP的可靠性是如何保证的。这里有很多种方法去实现。我非常喜欢Quake3网络库代码里的一些想法，它们也激发了我在War Arcana中使用UDP协议。　　你也可以使用许多支持可靠通信的UDP库，当然，这样在可靠性方面，相比自己手动实现全部的代码而言，可能会更加通用而失去了一些性能优势。
**七、底线**　　那么到底是用UDP还是TCP呢？　　如果是由客户端间歇性的发起无状态的查询，并且偶尔发生延迟是可以容忍，那么使用HTTP/HTTPS吧。　　如果客户端和服务器都可以独立发包，但是偶尔发生延迟可以容忍（比如：在线的纸牌游戏，许多MMO类的游戏），那么使用TCP长连接吧。　　如果客户端和服务器都可以独立发包，而且无法忍受延迟（比如：大多数的多人动作类游戏，一些MMO类游戏），那么使用UDP吧。　　这些也应该考虑在内：你的MMO客户端也许首先使用HTTP去获取上一次的更新内容，然后使用UDP跟游戏服务器进行连接。　　永远不要害怕去使用最佳的工具来解决问题。