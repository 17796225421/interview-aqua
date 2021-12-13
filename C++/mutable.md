（2）**mutable**

mutable的中文意思是“可变的，易变的”，跟constant（既C++中的const）是反义词。在C++中，mutable也是为了突破const的限制而设置的。被mutable修饰的变量，将永远处于可变的状态，即使在一个const函数中。我们知道，如果类的成员函数不会改变对象的状态，那么这个成员函数一般会声明成const的。但是，有些时候，我们需要**在const函数里面修改一些跟类状态无关的数据成员，那么这个函数就应该被mutable来修饰，并且放在函数后后面关键字位置**。

样例

```cpp
class person
{
int m_A;
mutable int m_B;//特殊变量 在常函数里值也可以被修改
public:
     void add() const//在函数里不可修改this指针指向的值 常量指针
     {
        m_A=10;//错误  不可修改值，this已经被修饰为常量指针
        m_B=20;//正确
     }
}

class person
{
int m_A;
mutable int m_B;//特殊变量 在常函数里值也可以被修改
}
int main()
{
const person p;//修饰常对象 不可修改类成员的值
p.m_A=10;//错误，被修饰了指针常量
p.m_B=200;//正确，特殊变量，修饰了mutable
}
```