[智能指针](https://www.zhihu.com/search?q=智能指针&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A378406523})，可以分为两类：

- 独占型：如`std::unique_ptr`，一份资源，仅能由一个`std::unique_ptr`对象管理；
- 共享型：如`std::shared_ptr`，一份资源，可以由多个`std::shared_ptr`对象共同管理，当没有`std::shared_ptr`对象指向这份的资源，资源才会被释放，即基于引用技术原理。

本期，先来讲解`std::unique_ptr`，之后会分几期来讲解`std::shared_ptr`的设计。

不过，在讲解`std::unique_ptr`之前，先讲解下C++03中的失败品：`std::auto_ptr`。

# std::auto_ptr

原本也是想要将 `std::auto_ptr` 设计成资源独占型的指针，即像现在的`std::unique_ptr`，但由于移动语义直到C++11中才出现，使得`std::auto_ptr`终究成了失败品。

不明白，没关系，go on。

在C++03标准下，有如下demo中的一个场景。

```cpp
int main(int argc, char const *argv[]) {
    std::auto_ptr<int> iptr(new int(1));
    std::vector<std::auto_ptr<int> > integer_vec;

    integer_vec.push_back(iptr);
    return 0;
}
```

由于在C++03标准中还没有引入移动语义，只能以`push_back`函数向`vector`中添加元素。

如果你没接触过`std::auto_ptr`，应该会认为上面的demo是能编译通过的，但实际上是无法编译通过的。

先给出导致错误的结论：是由于类`std::auto_ptr` 没有提供`const std::auto_ptr<T>&`类型的复制构造函数。

那为啥会没有呢？

因为`std::auto_ptr`的设计者，想使`std::auto_ptr`的复制构造函数具备移动构造函数的属性，这就使得`std::auto_ptr`复制构造函数的输入参数`__a`不能由 const 修饰，否则`__a`指向的资源就无法移动到新创建的对象中了。

```cpp
auto_ptr(auto_ptr& __a) throw() : _M_ptr(__a.release()) {}
```

这就导致`std::auto_ptr`中，所有和赋值有关的操作，都不能有`const`修饰。

以上原因当然还不至于让编译器报错，但是，vector的`push_back`函数中，输入参数`__x`是`const std::auto_ptr&`类型

```cpp
cpp template<typename _Tp, typename _Alloc> void std::vector<_Tp, _Alloc>::push_back(const value_type& __x);
```

1. 因为要构造`obj`，那么必要会调用`std::auto_ptr`的复制构造函数，且输入参数是`__x`；
2. 但由于`__x` 是 `const std::auto_ptr&` 类型，二`std::auto_ptr`的复制构造函数输入类型是`std::auto_ptr&`，接受不了`__x`作为输入，因此会导致`construct`函数执行失败。出现上述的错误。

简而言之，auto_ptr的所有和赋值有关的操作，都不能有const修饰。但同时，vector push_back的形参是const的。互相矛盾，编译器





