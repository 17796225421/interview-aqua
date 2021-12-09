## 问题

于是，好奇心的疑问就来了，经典的四连问。

- 如果一个socket创建后并与80端口绑定后，是否就意味着该socket占用了80端口呢？
- 如果是这样的，那么当其accept一个请求后，生成的新的socket到底使用的是什么端口呢（系统会默认给其分配一个空闲的端口号）？
- 如果是一个空闲的端口，那一定不是80端口了，于是以后的TCP[数据包](https://www.zhihu.com/search?q=数据包&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A69552919})的目标端口就不是80了--防火墙一定会阻止其通过的！实际上，我们可以看到，防火墙并没有阻止这样的连接，而且这是最常见的连接请求和处理方式。然而不解就是，为什么防火墙没有阻止这样的连接？
- 它是如何判定那条连接是因为connect 80端口而生成的？是不是TCP数据包里有什么特别的标志？或者防火墙记住了什么东西？



## 分析 带着这些问题，我们再来了解TCP/IP协议栈的原理。

TCP和UDP同属于[传输层](https://www.zhihu.com/search?q=传输层&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A69552919})，共同架设在IP层（[网络层](https://www.zhihu.com/search?q=网络层&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A69552919})）之上。而IP层主要负责的是在节点之间（End to End）的数据包传送，这里的节点是一台网络设备，比如计算机。因为IP层只负责把数据送到节点，而不能区分上面的不同应用，所以[TCP和UDP协议](https://www.zhihu.com/search?q=TCP和UDP协议&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A69552919})在其基础上加入了端口的信息，端口于是标识的是一个节点上的一个应用。



除了增加端口信息，[UDP协议](https://www.zhihu.com/search?q=UDP协议&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A69552919})基本就没有对IP层的数据进行任何的处理了。而TCP协议还加入了更加复杂的传输控制，比如滑动的数据发送窗口（Slice Window），以及接收确认和重发机制，以达到数据的可靠传送。不管应用层看到的是怎样一个稳定的TCP数据流，下面传送的都是一个个的IP数据包，需要由[TCP协议](https://www.zhihu.com/search?q=TCP协议&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A69552919})来进行数据重组。

所以，有理由怀疑，防火墙并没有足够的信息判断TCP数据包的更多信息，除了IP地址和端口号。而且，我们也看到，所谓的端口，是为了区分不同的应用的，以在不同的IP包来到的时候能够正确转发。

TCP/IP只是一个协议栈，就像操作系统的运行机制一样，必须要具体实现，同时还要提供对外的操作接口。就像操作系统会提供标准的编程接口，比如Win32编程接口一样，TCP/IP也必须对外提供编程接口，这就是Socket编程接口。

在Socket编程接口里，设计者提出了一个很重要的概念，那就是socket。这个socket跟文件句柄很相似，实际上在BSD系统里就是跟文件句柄一样存放在一样的进程句柄表里。这个socket其实是一个序号，表示其在句柄表中的位置。这一点，我们已经见过很多了，比如[文件句柄](https://www.zhihu.com/search?q=文件句柄&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A69552919})，窗口句柄等等。这些句柄，其实是代表了系统中的某些特定的对象，用于在各种函数中作为参数传入，以对特定的对象进行操作--这其实是C语言的问题，在C++语言里，这个句柄其实就是this指针，实际就是对象指针啦。

现在我们知道，socket跟TCP/IP并没有必然的联系。Socket编程接口在设计的时候，就希望也能适应其他的网络协议。所以，socket的出现只是可以更方便的使用TCP/IP协议栈而已，其对TCP/IP进行了抽象，形成了几个最基本的函数接口。比如create，listen，accept，connect，read和write等等。

现在我们明白，如果一个程序创建了一个socket，并让其监听80端口，其实是向TCP/IP协议栈声明了其对80端口的占有。以后，所有目标是80端口的TCP数据包都会转发给该程序（这里的程序，因为使用的是Socket编程接口，所以首先由Socket层来处理）。

所谓[accept函数](https://www.zhihu.com/search?q=accept函数&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A69552919})，其实抽象的是TCP的连接建立过程。accept函数返回的新socket其实指代的是本次创建的连接，而一个连接是包括两部分信息的，一个是源IP和源端口，另一个是宿IP和宿端口。

所以，accept可以产生多个不同的socket，而这些socket里包含的宿IP和宿端口是不变的，变化的只是源IP和源端口。这样的话，这些socket宿端口就可以都是80，而Socket层还是能根据源/宿对来准确地分辨出IP包和socket的归属关系，从而完成对TCP/IP协议的操作封装！而同时，放火墙的对IP包的处理规则也是清晰明了，不存在前面设想的种种复杂的情形。

##   总结

明白socket只是对TCP/IP协议栈操作的抽象，而不是简单的映射关系，这很重要！