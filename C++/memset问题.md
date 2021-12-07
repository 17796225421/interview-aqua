# 按字节填充

```
#include <stdio.h>
#include <string.h>
int main()
{
    int a[] = {1, 2, 3, 4, 5};
    memset(a, 1, sizeof(a));
    for (int i = 0; i < 5; i++)
    {
        printf("%d\n", a[i]);
    }
    return 0;
}

```

```
16843009
16843009
16843009
16843009
16843009
```

memset初始化的时候是按字节一个一个填充的，int有四个字节，于是填充成0000 0001 0000 0001 0000 00001 0000 0001，这样得出数组中的每个元素的值就是16843009了。

当初始化一个字节单位的数组时，可以用memset把每个数组单元初始化成任何你想要的值，比如，

1. char data[10];
2. memset(data, 1, sizeof(data)); *// right*
3. memset(data, 0, sizeof(data)); *// right*

而在初始化其他基础类型时，则需要注意，比如

1.int data[10];

2.memset(data, 0, sizeof(data)); // right

3.memset(data, -1, sizeof(data)); // right

4.memset(data, 1, sizeof(data)); // wrong, data[x] would be 0x0101 instead of 1





# 结构体指针



```
#include <iostream>
#include <string.h>
using namespace std;

struct A{
    int x;
    int *p_x;
};

int main()
{
    A a;
    a.p_x=new int[10];
    memset(&a,0,sizeof(a));
}
```

当memset初始化时，并不会初始化p_x指向的int数组单元的值，而会把已经分配过内存的p_x指针本身设置为0，造成内存泄漏。同理，对std::vector等数据类型，显而易见也是不应该使用memset来初始化的。



# 虚函数

```
#include <iostream>
#include <string.h>
using namespace std;

class BaseParameters

{

public:
    virtual void reset() {}
};

class MyParameters : public BaseParameters

{

public:
    int data[3];

    int buf[3];
};

int main()
{
    MyParameters my_pars;

    memset(&my_pars, 0, sizeof(my_pars));

    BaseParameters *pars = &my_pars;
    
    MyParameters* my = dynamic_cast<MyParameters*>(pars);
}

```



程序运行到dynamic_cast时发生异常。原因其实也很容易发现，我们的目的是为了初始化[数据结构](https://www.zhihu.com/search?q=数据结构&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A48288010})MyParameters里的data和buf，正常来说需要初始化的内存空间是sizeof(int) * 3 * 2 = 24字节，但是使用memset直接初始化MyParameters类型的数据结构时，sizeof(my_pars)却是28字节，因为为了实现[多态机制](https://www.zhihu.com/search?q=多态机制&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A48288010})，C++对有虚函数的对象会包含一个指向虚函数表(V-Table)的指针，当使用memset时，会把该虚函数表的指针也初始化为0，而dynamic_cast也使用RTTI技术，运行时会使用到V-Table，可此时由于与V-Table的链接已经被破坏，导致程序发生异常。



