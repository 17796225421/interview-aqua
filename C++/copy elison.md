#### Copy Elision

g++ 编译器是默认开启 `copy elison` 选项的。如果要关闭这个选项，使用 `-fno-elide-constructors`。*copy elision* 主要包括以下两项内容：

**返回值优化**

即通过将返回对象所占空间的直接构造到他们本来要复制/移动到的对象中去，依次来避免拷贝/移动操作。返回值优化包括具名返回值优化 `NRVO` 与 匿名返回值优化 `URVO`，区别在于返回值是具名的局部变量（`NRVO`）还是无名的临时对象（`URVO`）

- *URVO* 与 彻底 `Copy elision`

  ```cpp
  class Copyable { 
  
    public:
    Copyable() { std::cout<<"default ctor"<<std::endl; }
  
      Copyable(const Copyable& rhs) = delete; 
      Copyable(Copyable&& rhs) = delete;
  };
  
  Copyable return_urvo_value() { 
  
      return Copyable{}; // since c++17 ok
  }
  
  int main(int argc, char const *argv[]) {
  
      auto x  = return_urvo_value();
      
      return 0;
    }
  ```

  上述代码在C++17中是可以编译通过的。在 C++17 之前，并没有明确的提出在什么情况下，可以彻底进行 `Copy Elision`（这里的彻底的指的是包括不进行检查是否有可用的 copy/move 构造函数）。在C++17中，对于匿名对象（or 临时对象）不论是传递参数，还是以返回值返回时，都不会调用拷贝/移动构造。因此上面的这段代码在C++17是可以正常编过的，而在C++14会编译出错。

  ```cpp
    $ g++ main.cc -std=c++14 -o main && ./main
    main.cc: In function ‘Copyable return_urvo_value()’:
    main.cc:29:19: error: use of deleted function ‘Copyable::Copyable(Copyable&&)’
      29 |   return Copyable{};
         |                   ^
    main.cc:24:3: note: declared here
      24 |   Copyable(Copyable&& rhs) = delete;
         |   ^~~~~~~~
    main.cc: In function ‘int main(int, const char**)’:
    main.cc:34:31: error: use of deleted function ‘Copyable::Copyable(Copyable&&)’
      34 |   auto x  = return_urvo_value();
         |                               ^
    main.cc:24:3: note: declared here
      24 |   Copyable(Copyable&& rhs) = delete;
         |   ^~~~~~~~
  ```

  自然，只要将上面代码中的如下两行注释掉，即可正常编译，并且 `Copyable`的构造函数都是只被调用一次，即`copy elision` 起作用了。 注意：`Copyable`的复制/移动构造函数必须同时可访问。

  ```cpp
    Copyable(const Copyable& rhs) = delete; 
    Copyable(Copyable&& rhs) = delete;
  ```

因此，在C++17以前，对于 `urvo` 不在乎是否返回的对象的复制/移动构造函数是否存在或者可访问，`copy elision` 都能生效。而在 `C++14` 之前，返回的对象可以没有复制/移动构造函数，但是必须可以访问。

- `nrvo`
  在 `nrvo`时，返回对象的复制/移动构造函数必须可访问。否则编译不过。

  ```cpp
    class Copyable { 
    public:
    Copyable() { std::cout<<"default ctor"<<std::endl; }
  
      Copyable(const Copyable& rhs) = delete;
      Copyable(Copyable&& rhs) = delete;
  };
  
  Copyable return_urvo_value() { 
  
      return Copyable{};
  }
  
    Copyable return_nrvo_value() { 
    Copyable local;
  
      return local;
  }
  
  int main(int argc, char const *argv[]) {
  
      auto x  = return_urvo_value();
      auto y  = return_nrvo_value();
      
      return 0;
    }
  ```

  如上代码，即使是C++17也会编译失败，必须将如下两行代码注释掉，使得 `Copyable` 对象的复制/移动构造函数可访问。`copy elision` 才能生效：`Copyable` 的默认构造函数只调用一次。

  ```cpp
    // Copyable(const Copyable& rhs) = delete;
    // Copyable(Copyable&& rhs) = delete;
  ```

**右值拷贝优化**

右值拷贝优化，当某一个类的临时对象以值传递给该类的另一个对象时，也可以直接利用该临时对象的来避免拷贝/移动操作。在上面的基础上，加上如下的代码：

```cpp
  void pass_by_value(Copyable rhs) { 

  }

  int main(int argc, char const *argv[]) {

    auto x  = return_urvo_value();
    auto y  = return_nrvo_value();

    pass_by_value(Copyable());
    
    return 0;
  }
```

最终的输出也是调用默认三次构造函数：

```cpp
  $ g++ main.cc -std=c++11 -o main && ./main
  default ctor
  default ctor
  default ctor
```

到此，`copy elision` 基本分析结束。如果想查看没有`copy elision` 作用下的输出，开启`-fno-elide-constructors`。

#### Copy Elision 作用

对于一些没有拷贝/移动构造的对象，如 `unique_ptr`、 `atomic` 等。现在我们能够定义一个工厂函数，即使没有复制或移动构造函数都可以返回一个对象。例如，以下通用工厂函数:

```cpp
  template <typename T, typename... Args>
  T make_instance(Args&& ... args)
  {
    return T{ std::forward<Args>(args)... };
  }
  
  int main()
  {
    int i   = make_instance<int>(42);
    // std::unique_ptr 实现了 移动构造函数，因此可以编译成功 
    auto up = make_instance<std::unique_ptr<int>>(new int{ 42 }); 
    // 禁止了复制构造函数，但是也没有实现移动构造函数，因此要到 C++17 才能编译过
    auto ai = make_instance<std::atomic<int>>(42);                  
  
    return 0;
  }
```