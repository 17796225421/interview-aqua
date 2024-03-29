## **sleep系统调用**

我是一个线程，生活在Linux帝国。一直以来辛勤工作，日子过得平平淡淡，可今天早上发生了一件事让我回想起来都后怕。

早上，我还是如往常一样执行着人类编写的代码指令，不多时走到了一个冷门的分支，一个`sleep()`函数调用摆在了我的面前。

终于可以去休息了！听老一辈的线程们说，执行了这个函数就可以休息休息了。我瞄了一眼参数，足足有5秒钟的休息时间，我简直乐坏了，没有犹豫，赶紧执行了这个调用。

![图片](image/640-163931318943856.webp)

进入`sleep()`函数后，又来到了`nano_sleep()`函数，接着看到了一个`syscall`系统调用指令，我继续执行，来到了内核空间。

进入内核空间后，我接连穿过了

- --> nano_sleep()
- --> hrtimer_nanosleep()
- --> do_nanosleep()
- --> freezable_schedule()

把我累得够呛，说好让我休息，没想到休息之前还有这么多事要做。

终于，我来到了一个叫`schedule()`的函数面前。

## **线程调度**

进入`schedule()`后，迎面走来一位发须皆已花白的长者。

![图片](image/640-163931318943857.webp)

“小伙子，这是要来休息了，我是负责线程调度的使者，让我看下你占用的CPU号码”，一边说一边查找着什么。

“哦，是2号CPU，来，跟我到这边来”，在他的指引下，我又来到了一个函数面前。

“你先去`pick_next_task()`找到一个接盘侠，哦不，找到下一个需要执行的线程，这是2号CPU的`就绪队列`，你可拿好了，等你办完回来我再带你去办理交接手续”，说完给我手里塞了一个参数`rq`，随即便离开了，留下我不知所措。

我只好按他说的照办，迈进了`pick_next_task()`函数的大门，一位美女接待映入眼帘。

“先生您好，您来此想必是要寻找接班线程吧”，见我到来，美女起身招呼。

“你猜的不错，要麻烦你帮我处理一下，多谢了”

“您别客气，把**就绪队列**给我看看吧”

我先是愣了一下，反应过来后将手里的`rq`参数给了她。

```
struct rq {
    raw_spinlock_t lock;
    ...
    unsigned int nr_running;
    ...
    struct cfs_rq cfs;
    struct rt_rq rt;
    struct dl_rq dl;
    ...
    struct task_struct *curr, *idle, *stop;
    ...
    struct mm_struct *prev_mm;
    ...
    struct list_head cfs_tasks;
};
```

美女拿着`rq`一阵端详，说到：“您运气不错哦，rq->nr_running和rq->cfs.h_nr_running相等，看来没有实时线程，全是普通线程，您直接去那边的公平调度CFS窗口`fair_sched_class`那里去办理吧。”

我顺着美女指向的方向看去，那边一共有5个窗口：

- `stop_sched_class`
- `dl_sched_class`
- `rt_sched_class`
- `fair_sched_class`
- `idle_sched_class`

“唉，美女，那要是不相等该去哪个窗口办理呢？你告诉我一下，下次来我就知道了”

“不相等的话那就说明就绪队列里除了普通线程还有其他优先级更高的线程，就得按照优先级从`stop_sched_class`窗口挨个向后询问，直到找到一个线程。不过我在这干了这么久，就实时线程所在的`rt_sched_class`窗口和普通线程所在的`fair_sched_class`最常用”，美女耐心的给我解释到。

听了她的解释，我想到了一个问题：“那要是都找不到线程需要执行怎么办，比如他们都在等待IO事件之类的？那我怎么交差啊”

“放心吧，最后那个`idle_sched_class`窗口绝对不会让你空手而归的”，美女笑着说。

原来如此，我点了点头。

![图片](image/640-163931318943858.webp)

来到`fair_sched_class`窗口的旁边，递交了我的`rq`参数，只见工作人员取出了其中的`cfs_rq`：

```
struct cfs_rq {
 struct load_weight load;
 unsigned int nr_running, h_nr_running;
 ...
 struct rb_root tasks_timeline;
 struct rb_node *rb_leftmost;
 ...
 struct sched_entity *curr, *next, *last, *skip;
 ...
 struct rq *rq;
};
```

我这才注意到，原来这个`cfs_rq`中指向了一棵红黑树，再仔细一看，这树上的每个节点都是一个线程`task_struct`。

![图片](image/640-163931318943859.webp)

工作人员很快就取出了一个`task_struct`交给我，一个年纪轻轻的线程小T，我带着小T告别了美女接待，回到了`schedule()`。

## **context_switch**

看到我回来，长者起身言道：“小伙子，回来啦，走，带你们去`context_switch()`”

进入这个`context_switch()`之后，长者又带着我又做了一些准备工作，比如把当前的进程地址空间换成了小T的，最终我们来到了一个叫`switch_to`的地方。

“小伙子，再往前走几步就是换班的地方了，就可以休息了，我就不送你了，感谢你这段时间的辛苦工作”，长者一边说一边拍拍我的肩膀。

告别了长者，我和小T踏上了这神秘的`switch_to`，跟随着一步一步的指令，我把自己线程上下文的寄存器都保存到了我的内核栈上面，然后将栈指针指向了小T的内核栈，最后把小T保存在他内核栈的指令地址加载进指令寄存器，终于完成了交接工作。

“小T，接下来就该你工作了，我要去休息了”，我和小T握手告别，来到旁边准备眯一会儿。

![图片](image/640-163931318943960.webp)

## **神秘的唤醒**

“醒醒，醒醒”，睡梦中听到有人唤我。

我揉揉睡眼，看了下时间，这才睡了2s，时间还没到，难道现在是在做梦？

“总算把你叫醒了，快起来，换班时间到了，该你上了”，我抬起头才发现另外一个线程小H站在面前。

“我休息时间还没够啊，怎么选中了我啊，让我再睡会儿”，说罢我就要躺下。

小H一把拉住了我，“别磨叽了，就是你，快走”。

![图片](image/640-163931318943961.webp)

在小H的带领下，我们又来到了那个叫`switch_to`地方，只不过这一次我的角色变了。

小H一顿和我之前一样的操作，把执行流程交给了我。

我再次获得了执行代码的能力，随后回到了`context_switch()`，又回到了`schedule()`，看到了熟悉的长者。

我和长者再次告了别，继续返回，最后通过`sysret`虫洞，回到了用户态空间。

不过我马上意识到事情不对劲，这里并不是我最开始调用`sleep()`的地方，而是一片完全陌生的区域，难道哪里出了问题，我这是到了哪里？

![图片](image/640-163931318943962.webp)

这一定是在做梦，我还在`sleep()`呢，时间还没够，我只好这样安慰自己。

我小心翼翼的执行了这里的代码，只是简单输出了一行日志，然后来到了一个叫`__restore_rt()`的函数，又一条`syscall`指令摆在了我的面前，我没有犹豫再一次一头扎进了内核空间。

经过一番折腾，又来到了`sysret`虫洞面前，不知道这一次又将带我去到哪里。我闭上了眼睛跳了进去···

![图片](image/640-163931318943963.webp)

等我睁开眼睛，竟然奇迹般的回到了最开始调用`sleep()`的地方，这场梦终于醒了，庆幸我回到了这里。

我看了一眼`sleep()`的返回值，竟然是3。我又看了一眼日志文件，竟看到了梦里输出的那一行日志。

难道那不是梦？这究竟是怎么一回事？

**未完待续······**

## **彩蛋**

> “奇怪，这个TCP数据包的ACK和SEQ怎么和刚才那个一样，估计是重传了吧”，帝国网络部的小Q丢掉了这个重复的数据包。
>
> 不过，同样的事情接二连三的出现，经历了上次那件事的小Q不敢大意，赶紧向安全部长汇报了情况。
>
> *预知后事如何，请关注后续精彩······*