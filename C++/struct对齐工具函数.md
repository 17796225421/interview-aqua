# 工具函数alignof、offsetof、alignas

C++11引入的关键字`alignof`，可直接获取类型`T`的内存对齐要求。`alignof`的返回值类型是`size_t`，用法类似于`sizeof`。

`offsetof`函数，来获取成员变量距离类起始地址的偏移量

```cpp
#include <iostream>
#include <string.h>
using namespace std;

struct Foo{
    char c;
    int i1;
    int i2;
    long l;
};

int main()
{
    cout<<sizeof(Foo)<<' '<<alignof(Foo)<<endl;
    cout<<offsetof(Foo,c)<<endl;
    cout<<offsetof(Foo,i1)<<endl;
    cout<<offsetof(Foo,i2)<<endl;
    cout<<offsetof(Foo,l)<<endl;
}

```

```cpp
24 8
0
4
8
16
```

上面讲述的内存对齐要求都是默认情况下的，有时候考虑到cacheline、以及向量化操作，可能会需要改变一个类的`alignof`值。

怎么办？

在C++11之前，需要依赖靠编译器的扩展指令，C++11之后可以借助`alignas`关键字。

> 比如，在C++11之前，gcc实现 `alignas(alignment)` 效果的方式为  `__attribute__((__aligned__((alignment)))`

仍然以上述的`Foo`为例子，不过此时你希望`Foo`对象的起始地址总是32的倍数，C++11之后借助`alignas`关键字，可以如下操作：

```cpp
#include <iostream>
#include <string.h>
using namespace std;

struct alignas(32) Foo{
    char c;
    int i1;
    int i2;
    long l;
};

int main()
{
    cout<<sizeof(Foo)<<' '<<alignof(Foo)<<endl;
    cout<<offsetof(Foo,c)<<endl;
    cout<<offsetof(Foo,i1)<<endl;
    cout<<offsetof(Foo,i2)<<endl;
    cout<<offsetof(Foo,l)<<endl;
}

```

```cpp
32 32
0
4
8
16
```

仍然以`Foo`为例，在没有`alignas`修饰时，默认的Foo的内存对齐要求`alignof(Foo)`为8，现在尝试使用`alignas`让`Foo`的对齐要求为4，操作如下：

```c
struct alignas(4) Foo { 
  char c;
  int i1;
  int i2;
  long l; 
};
```

```c
24 8
0
4
8
16
```

可以看出，此时的`alignas`是失效的，在其他编译器下也许直接编译失败。

`alignas` 指定的大小`alignment`必须是2的正数幂（`N>0`），否则也是失效，在有些编译器下也许直接编译失败。