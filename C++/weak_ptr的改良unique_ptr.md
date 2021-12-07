> 原文章出处https://zhuanlan.zhihu.com/p/378406523

从本期，就开始智能指针源码分析之路，从源码中了解他们的设计。

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

编译指令如下：

```cpp
$ g++ -std=c++03  main.cc -o main && ./main
```

下面只贴出最初的错误信息：

```cpp
/c++/9.0/ext/new_allocator.h:146:9: error: no matching function for call to ‘std::auto_ptr<int>::auto_ptr(const std::auto_ptr<int>&)’
       { ::new((void *)__p) _Tp(__val); }
         ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

先给出导致错误的结论：是由于类`std::auto_ptr` 没有提供`const std::auto_ptr<T>&`类型的复制构造函数。

那为啥会没有呢？

因为`std::auto_ptr`的设计者，想使`std::auto_ptr`的复制构造函数具备移动构造函数的属性（如果不懂右值、移动等内容，可以看看 [右值引用的正确用法](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s%3F__biz%3DMzkyMjIxMzIxNA%3D%3D%26mid%3D2247483868%26idx%3D1%26sn%3D3271bc5f1694818a493fd54185b341dd%26chksm%3Dc1f68fedf68106fb21fedd83136aa39a1ed01761ad68018d9bedd1591d9241cb8838432d634b%26token%3D1599630750%26lang%3Dzh_CN%23rd)），这就使得`std::auto_ptr`复制构造函数的输入参数`__a`不能由 const 修饰，否则`__a`指向的资源就无法移动到新创建的对象中了。

```cpp
auto_ptr(auto_ptr& __a) throw() : _M_ptr(__a.release()) {}
```

这就导致`std::auto_ptr`中，所有和赋值有关的操作，都不能有`const`修饰。

`std::auto_ptr`的核心代码如下。

```cpp
template <typename _Tp>
class auto_ptr
{
private:
  _Tp *_M_ptr;
public:
  typedef _Tp element_type;

  explicit auto_ptr(element_type *__p = 0) throw() : _M_ptr(__p) { }

  auto_ptr(auto_ptr& __a) throw() : _M_ptr(__a.release()) {} 

  template <typename _Tp1>
  auto_ptr(auto_ptr<_Tp1>& __a) throw() : _M_ptr(__a.release()) {}

  auto_ptr &operator=(auto_ptr& __a) throw() {
    reset(__a.release());
    return *this;
  }

  ~auto_ptr() { delete _M_ptr; }

  element_type* release() throw() {
    element_type *__tmp = _M_ptr;
    _M_ptr = 0;
    return __tmp;
  }

  void reset(element_type *__p = 0) throw() {
    if (__p != _M_ptr) {
      delete _M_ptr;
      _M_ptr = __p;
    }
  }
  //...
};
```

# by the way

对于最上面的demo，我们再啰嗦几点：为什么会在 `construct`函数中报错？

1. 因为`push_back`函数中，输入参数`__x`是`const std::auto_ptr&`类型，能接受`iptr`：

```
cpp template<typename _Tp, typename _Alloc> void std::vector<_Tp, _Alloc>::push_back(const value_type& __x);
```

1. 在`push_back`函数内部会调用 `_Alloc_traits::construct` 函数来构造一个新的`std::auto_ptr`对象`obj`，然后将这个`obj`放到`integer_vec`中。

```
cpp _Alloc_traits::construct(this->_M_impl, this->_M_impl._M_finish, __x);
```

1. 因为要构造`obj`，那么必要会调用`std::auto_ptr`的复制构造函数，且输入参数是`__x`；
2. 但由于`__x` 是 `const std::auto_ptr&` 类型，而`std::auto_ptr`的复制构造函数输入类型是`std::auto_ptr&`，接受不了`__x`作为输入，因此会导致`construct`函数执行失败。出现上述的错误。

# std::unique_ptr

C++11引入移动语义，提出了`std::unique_ptr`，才真正地完成了`std::auto_ptr`的设计意图，而原本的`std::auto_ptr`也被标记为`deprecated`。

由于 `std::unique_ptr` 对象管理的资源，不可共享，只能在 `std::unique_ptr` 对象之间转移，因此类`std::unique_ptr` 就禁止了复制构造函数、赋值表达式，仅实现了[移动构造函数](https://www.zhihu.com/search?q=移动构造函数&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A378406523})等。

此外，`std::unique_ptr` 有两个版本：

1. 管理单个对象（例如以 `new` 分配）
2. 管理动态分配的对象数组（例如以 `new[]` 分配）

因此， `std::unique_ptr` 的类模板有如下两个。

```cpp
/// 适合 new 分配的内存
  template <typename _Tp, typename _Dp = default_delete<_Tp>>
  class unique_ptr { /****/ };

  /// 针对 new[] 特化
  template <typename _Tp, typename _Dp>
  class unique_ptr<_Tp[], _Dp> { /****/ };
```

在下面的情况中，`std::unique_ptr`对象管理的资源，会调用传入的析构器`_Dp`来释放放资源：

- 当前`std::unique_ptr` 对象被销毁，生命周期结束；
- 重新给当前`std::unique_ptr`对象赋值，比如调用 `operator=`、`reset()` 等操作。

下面就先讲解下默认的析构器`std::default_delete`。

# std::default_delete

由于`std::unique_ptr` 有两个版本，因此默认的析构器也存在两个版本，即对`new[]` 进行特化。

此外，这就导致后文的`make_unique` 函数也需要对`new[]`进行特化。

```cpp
template <typename _Tp>
  struct default_delete { /****/ };

  template <typename _Tp>
  struct default_delete<_Tp[]> { /****/ };
```

由于默认采用`new`、`new[]`来分配内存的，而`sd::default_delete` 实际上是个仿函数，内部也是基于`delete`、`delete[]`来释放内存资源的。

# delete

类`std::default_delete` 实际上是个仿函数，并且是个空类，因此他的默认构造函数直接设置为`default`了。

```cpp
template <typename _Tp>
  struct default_delete
  {
    /// @brief 默认构造函数
    constexpr default_delete() noexcept = default;

    /// @brief 复制构造函数
    ///        _Up* 必须能转为 _Tp*，否则无法编译通过
    template <typename _Up, 
              typename = typename enable_if<is_convertible<_Up*, _Tp*>::value>::type>
    default_delete(const default_delete<_Up>& ) noexcept { }

    /// @brief 释放内存
    void operator()(_Tp *__ptr) const;
  };
```

此外，`std::default_delete` 还有个复制构造函数，这里传入`_Up*`参数 必须能转换为 `_Tp*` 类型，否则在编译期会报错。所谓`_Up*` 能转换为 `_Tp*`，即`is_convertible<_Up*, _Tp*>::value` 为 true，这个值在编译期就能确定，如果为false，就相当于不存在这个复制构造函数。

不信，可以把下面的代码复制到IDE中，会有红线提示，编译会出错。

```cpp
std::is_convertible<float*, double*>::value;                  // false: float* 不能直接转换为 double*
 std::default_delete<double> de(std::default_delete<float>{}); // compile error
```

在`std::default_delete`的内部，实现的`operator()` 函数会调用`delete`来 析构传入的指针`__ptr`，对于`__ptr`需要满足两点：

- `__ptr`不能是个`void`类型；
- 大小也不能是0。

否则无法提供完整的信息去析构`__ptr`指向的内存。

```cpp
void operator()(_Tp *__ptr) const
  {
    static_assert(!is_void<_Tp>::value, "can't delete pointer to incomplete type");
    static_assert(sizeof(_Tp) > 0, "can't delete pointer to incomplete type");
    delete __ptr;
  }
```

那么，你就可以这样来使用`std::default_delete`：

```cpp
int* iptr = new int;
  //...
  std::default_delete<int>()(iptr); // delete iptr;
```

使用`valgrind`检测也不存在内存泄露。

# delete[]

当输入类型是`_Tp[]`时，会进入此版本。实现和上面的版本差不多。

```cpp
template <typename _Tp>
  struct default_delete<_Tp[]>
  {
  public:
    /// @brief 默认构造函数
    constexpr default_delete() noexcept = default;

    /// @brief 复制构造函数
    template <typename _Up, 
              typename = typename enable_if<is_convertible<_Up (*)[], _Tp (*)[]>::value>::type>
    default_delete(const default_delete<_Up[]> &) noexcept {}

    /// @brief 释放内存
    template <typename _Up>
              typename enable_if<is_convertible<_Up (*)[], _Tp (*)[]>::value>::type
    operator()(_Up *__ptr) const
    {
      static_assert(sizeof(_Tp) > 0, "can't delete pointer to incomplete type");
      delete[] __ptr;
    }
  };
```

因此，这样就可以使用

```cpp
int* iptr = new int[3];
  //...
  std::default_delete<int[]>()(iptr);
```

使用`valgrind`检查也没有任何内存泄露。

因此，`std::default_delete` 对两种常用的内存释放方式进行重载，提供了同一个统一接口。

# std::__uniq_ptr_impl

类`std::unique_ptr` 内部只有一个成员变量，其类型是 `std::__uniq_ptr_impl`。

类`std::__uniq_ptr_impl`实际上就一个`<pointer, Deleter>`的一个wrapper，即简单封装了指向资源的指针，以及对应的析构器 `Deleter`。

先大致看下`std::__uniq_ptr_impl`的部分实现。

```cpp
template <typename _Tp, typename _Dp>
  class __uniq_ptr_impl
  {
    //...
  public:
    using _DeleterConstraint = enable_if<__and_<__not_<is_pointer<_Dp>>, 
                                                is_default_constructible<_Dp>>::value>;

    __uniq_ptr_impl() = default;
    __uniq_ptr_impl(pointer __p) : _M_t() { _M_ptr() = __p; }

    template <typename _Del>
    __uniq_ptr_impl(pointer __p, _Del&& __d)
    : _M_t(__p, std::forward<_Del>(__d)) { }

    pointer&   _M_ptr()           { return std::get<0>(_M_t); }
    pointer    _M_ptr()     const { return std::get<0>(_M_t); }
    _Dp&       _M_deleter()       { return std::get<1>(_M_t); }
    const _Dp& _M_deleter() const { return std::get<1>(_M_t); }

    void swap(__uniq_ptr_impl& __rhs) noexcept
    {
      std::swap(this->_M_ptr(), __rhs._M_ptr());
      std::swap(this->_M_deleter(), __rhs._M_deleter());
    }

  private:
    tuple<pointer, _Dp> _M_t; // pointer 的定义后文讲解
  };
```

类`std::__uniq_ptr_impl` 的成员变量`_M_t`是由`tuple`类型：

- 第一个成员：是指针，指向资源；
- 第二个成员：是析构器，用于释放指针指向的资源。

由于在X86-84位系统上指针大小是8，而`_Dp`在默认情况下（即`std::default_delete`）是个[空类](https://www.zhihu.com/search?q=空类&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A378406523})，得益于[空基类优化](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s%3F__biz%3DMzkyMjIxMzIxNA%3D%3D%26mid%3D2247485249%26idx%3D1%26sn%3Df395397f1621cf8a4d897d0213ca1788%26chksm%3Dc1f68970f68100666cbb65313780797bc65490703dac02a9603e0b9733fa01db270ca445f0e6%26token%3D1599630750%26lang%3Dzh_CN%23rd)，因此`_M_t` 的大小是8。

> **NOTICE**：如果自定义了一个`Deleter`，且不是空类，则`std::unique_ptr`的大小会增加。

下面，来分析下`std::__uniq_ptr_impl` 中的 `pointer`类型，他的完整实现及注释如下。

因此，当`_Dp`是默认的析构器`std::default_delete`时，`pointer` 即 `_Tp*`。

```cpp
template <typename _Tp, typename _Dp>
  class __uniq_ptr_impl
  {    
    /// 原型
    /// @brief 特化版本决议失败，则会进入此版本
    /// @type  此时 type 就是 _up*
    template <typename _Up, typename _Ep, typename = void>
    struct _Ptr
    {
      using type = _Up*;
    };

    /// 特化版本
    /// @brief 如果 类 _Ep 中有 pointer 的定义，则会进入此特化版本
    /// @type  此时 type 是 _Ep 中的 pointer 
    template <typename _Up, typename _Ep>
    struct _Ptr<_Up, _Ep, __void_t<typename remove_reference<_Ep>::type::pointer>>
    {
      using type = typename remove_reference<_Ep>::type::pointer;
    };
    public:
    using pointer = typename _Ptr<_Tp, _Dp>::type;
    //...
  }；
```

# std::unique_ptr

分析的进度终于到了类`std::unique_ptr`，它的设计主要如下四点： 1. 禁止复制构造函数、复制赋值的重载，即设置为`=delete`； 2. 实现各种移动构造函数； 3. 实现移动赋值重载，即`operator=`，需要先释放本身的资源，再将对方的资源移动过来； 4. 如果资源没有释放过，则会在[析构函数](https://www.zhihu.com/search?q=析构函数&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A378406523})中释放。

为便于理解，先看下`std::unique_ptr`的部分源码及其注释，另一个特化版本差不多。

```cpp
template <typename _Tp, typename _Dp = default_delete<_Tp>>
  class unique_ptr
  {
    template <typename _Up>
    using _DeleterConstraint = typename __uniq_ptr_impl<_Tp, _Up>::_DeleterConstraint::type;

    __uniq_ptr_impl<_Tp, _Dp> _M_t; // 成员变量

  public:
    using pointer = typename __uniq_ptr_impl<_Tp, _Dp>::pointer;
    using element_type = _Tp;
    using deleter_type = _Dp;

  private:
    /// 用于检测从另一个 std::unique_ptr 对象转换过来是否安全
    template <typename _Up, typename _Ep>
    using __safe_conversion_up = __and_<is_convertible<typename unique_ptr<_Up, _Ep>::pointer, pointer>,
                                        __not_<is_array<_Up>>>;
  public:
    /*** Move constructors. ***/

    /// @brief 移动构造函数
    unique_ptr(unique_ptr&& __u) noexcept
    : _M_t(__u.release(), std::forward<deleter_type>(__u.get_deleter())) { }

    /// @brief 移动赋值
    unique_ptr& operator=(unique_ptr&& __u) noexcept
    {
      reset(__u.release()); 
      get_deleter() = std::forward<deleter_type>(__u.get_deleter());
      return *this;
    }

    /// @brief 设置 unique_ptr 对象为初始化状态
    unique_ptr& operator=(nullptr_t) noexcept
    {
      reset();
      return *this;
    }

    /// @brief 禁止复制
    unique_ptr(const unique_ptr &) = delete;
    unique_ptr &operator=(const unique_ptr &) = delete;

    ~unique_ptr() noexcept
    {
      static_assert(__is_invocable<deleter_type&, pointer>::value,
                    "unique_ptr's deleter must be invocable with a pointer");
      auto& __ptr = _M_t._M_ptr();
      if (__ptr != nullptr)
         get_deleter()(std::move(__ptr)); // 析构
      __ptr = pointer();                  // 设置为初始化状态
    }

    /// @brief 返回指针
    pointer get() const noexcept
    { return _M_t._M_ptr(); }

     /// @brief 返回一个指向内部的deleter的引用
    deleter_type& get_deleter() noexcept
    {
      return _M_t._M_deleter();
    }

    /// Return @c true if the stored pointer is not null.
    explicit operator bool() const noexcept
    {
      return get() == pointer() ? false : true;
    }

    /// Release ownership of any stored pointer.
    pointer release() noexcept
    {
      pointer __p = get();
      _M_t._M_ptr() = pointer();
      return __p;
    }

    void reset(pointer __p = pointer()) noexcept
    {
      static_assert(__is_invocable<deleter_type &, pointer>::value,
                    "unique_ptr's deleter must be invocable with a pointer");
      std::swap(_M_t._M_ptr(), __p);
      if (__p != pointer())
        get_deleter()(std::move(__p));
    }

    void swap(unique_ptr &__u) noexcept
    {
      static_assert(__is_swappable<_Dp>::value, "deleter must be swappable");
      _M_t.swap(__u._M_t);
    }
    //...
  };
```

`std::unique_ptr` 的设计总体比较清晰。

到此，`std::unique_ptr`的设计分析差不多就结束了，下面对源码中（上面未列出）几个模板稍微分析下。

# _DeleterConstraint

`_DeleterConstraint` 定义于`std::__uniq_ptr_impl`之中，用于限制传入的析构器`Deleter`必须满足以下两点：

1. 传入的`Deleter` 不能是指针；
2. 必须具备默认[构造函数](https://www.zhihu.com/search?q=构造函数&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A378406523})。

否则，`std::unique_ptr` 的构造函数会失败。

```cpp
using _DeleterConstraint = enable_if<__and_<__not_<is_pointer<_Dp>>, 
                                            is_default_constructible<_Dp>>::value>;

    /// 构造函数
    template <typename _Del = _Dp, typename = _DeleterConstraint<_Del>>
    constexpr unique_ptr() noexcept
    : _M_t()
    { }

    template <typename _Del = _Dp, typename = _DeleterConstraint<_Del>>
    explicit unique_ptr(pointer __p) noexcept
    : _M_t(__p)
    { }
```

# _Require

其原型如下：

```cpp
template<typename... _Cond>
 using _Require = __enable_if_t<__and_<_Cond...>::value>;
```

`_Require`模板是要求所有传入的条件`Cond`都为true，则`_Require`修饰的函数才会存在。

```cpp
/// @brief 如果传入的 _Del 没有复制构造函数，则unique_ptr此版本构造函数就不存在
    template <typename _Del = deleter_type,
              typename = _Require<is_copy_constructible<_Del>>>
    unique_ptr(pointer __p, const deleter_type& __d) noexcept
    : _M_t(__p, __d) {}

    /// @brief 如果传入的 _Del 没有移动构造函数，则 unique_ptr此版本构造函数就不存在
    template <typename _Del = deleter_type,
              typename = _Require<is_move_constructible<_Del>>>
    unique_ptr(pointer __p, __enable_if_t<!is_lvalue_reference<_Del>::value, _Del&&> __d) noexcept
    : _M_t(__p, std::move(__d))
    { }
```

# __safe_conversion_up

在构造函数中，有时候需要通过其他`std::unique_ptr`对象来构造当前`std::unique_ptr`对象，但是`pointer`类型可能不同，模板`__safe_conversion_up`可以在编译期确定是否可以转换。

```cpp
template <typename _Up, typename _Ep>
using __safe_conversion_up = __and_<is_convertible<typename unique_ptr<_Up, _Ep>::pointer, pointer>,
                                    __not_<is_array<_Up>>>;
```

此外还有个条件模板`conditional`，实现编译期的条件表达式：

```cpp
// _Cond ? _Iftrue ： _Iffalse;

  // 原型
  template<bool _Cond, typename _Iftrue, typename _Iffalse>
  struct conditional
  { typedef _Iftrue type; };

  // 特化
  template<typename _Iftrue, typename _Iffalse>
  struct conditional<false, _Iftrue, _Iffalse>
  { typedef _Iffalse type; };
```

最终，可以用于构造函数。

```cpp
/// @brief 由其他std::unique_ptr对象 __u 构造this
 template <typename _Up, 
           typename _Ep, 
           typename = _Require<__safe_conversion_up<_Up, _Ep>,   // 先判断指针，必须可转换
           typename conditional<is_reference<_Dp>::value,        // 再判断析构器
                                is_same<_Ep, _Dp>,               
                                is_convertible<_Ep, _Dp>>::type>>
  unique_ptr(unique_ptr<_Up, _Ep>&& __u) noexcept
  : _M_t(__u.release(), std::forward<_Ep>(__u.get_deleter()))
  { }
```

# std::make_unique

最后，再来看下`std::make_unique` 函数，它在C++14中引入，这在之前的 [编译器优化之copy elision](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s%3F__biz%3DMzkyMjIxMzIxNA%3D%3D%26mid%3D2247484605%26idx%3D1%26sn%3D52205144a7a9819e4a8f181eccec7914%26chksm%3Dc1f68a8cf681039a0d5c454dd1b0d1899ba15d4fd99ddc0d1885a7b93cb3b5b4b6f53c9d5ca3%26token%3D1599630750%26lang%3Dzh_CN%23rd)一期中也讲解过怎么设计它。

现在，我们再来看看C++14中 `std::make_unique()` 的实现，如下。

```cpp
/*** 返回类型 ***/

  /// 原型
  template <typename _Tp>
  struct _MakeUniq
  { typedef unique_ptr<_Tp> __single_object; };

  /// @brief 为数组类型特化
  template <typename _Tp>
  struct _MakeUniq<_Tp[]>
  { typedef unique_ptr<_Tp[]> __array; };

  /// @brief 无效
  template <typename _Tp, size_t _Bound>
  struct _MakeUniq<_Tp[_Bound]>
  {
    struct __invalid_type { };
  };

  /*** 函数实现 ***/

  /// std::make_unique for single objects
  template <typename _Tp, typename... _Args>
  inline 
  typename _MakeUniq<_Tp>::__single_object make_unique(_Args&& ...__args)
  { return unique_ptr<_Tp>(new _Tp(std::forward<_Args>(__args)...)); }

  /// std::make_unique for arrays of unknown bound
  template <typename _Tp>
  inline 
  typename _MakeUniq<_Tp>::__array make_unique(size_t __num)
  { return unique_ptr<_Tp>(new remove_extent_t<_Tp>[__num]()); }

  /// Disable std::make_unique for arrays of known bound
  template <typename _Tp, typename... _Args>
  inline 
  typename _MakeUniq<_Tp>::__invalid_type make_unique(_Args&& ...) = delete;
```

结束，再回顾下`std::unique_ptr` 和 `std::auto_ptr`。

可以发现`std::auto_ptr`的失败在于CXX03中并不支持移动语义，而`std::auto_ptr` 却试图用复制构造函数来实现移动构造函数的功能，结果导致其无法与`vector` 等容器兼容，论为失败品。

`std::unique_ptr` 的分析到此为止。



