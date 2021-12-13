#### [8、new / delete 与 malloc / free的异同](https://interviewguide.cn/#/Doc/Knowledge/C++/基础语法/基础语法?id=8、new-delete-与-malloc-free的异同)

**相同点**

- 都可用于内存的动态申请和释放

**不同点**

- 前者是C++运算符，后者是C/C++语言标准库函数
- new自动计算要分配的空间大小，malloc需要手工计算
- new是类型安全的，malloc不是。例如：

```cpp
int *p = new float[2]; //编译错误
int *p = (int*)malloc(2 * sizeof(double));//编译无错误Copy to clipboardErrorCopied
```

- new调用名为**operator new**的标准库函数分配足够空间并调用相关对象的构造函数，delete对指针所指对象运行适当的析构函数；然后通过调用名为**operator delete**的标准库函数释放该对象所用内存。后者均没有相关调用
- 后者需要库文件支持，前者不用
- new是封装了malloc，直接free不会报错，但是这只是释放内存，而不会析构对象

------

#### [10、malloc和new的区别？](https://interviewguide.cn/#/Doc/Knowledge/C++/基础语法/基础语法?id=10、malloc和new的区别？)

- malloc和free是标准库函数，支持覆盖；new和delete是运算符，不重载。
- malloc仅仅分配内存空间，free仅仅回收空间，不具备调用构造函数和析构函数功能，用malloc分配空间存储类的对象存在风险；new和delete除了分配回收功能外，还会调用构造函数和析构函数。
- malloc和free返回的是void类型指针（必须进行类型转换），new和delete返回的是具体类型指针。

------

#### [11、既然有了malloc/free，C++中为什么还需要new/delete呢？直接用malloc/free不好吗？](https://interviewguide.cn/#/Doc/Knowledge/C++/基础语法/基础语法?id=11、既然有了mallocfree，c中为什么还需要newdelete呢？直接用mallocfree不好吗？)

- malloc/free和new/delete都是用来申请内存和回收内存的。
- 在对非基本数据类型的对象使用的时候，对象创建的时候还需要执行构造函数，销毁的时候要执行析构函数。而malloc/free是库函数，是已经编译的代码，所以不能把构造函数和析构函数的功能强加给malloc/free，所以new/delete是必不可少的。