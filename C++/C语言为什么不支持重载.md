# by the way

最后，顺便提下，为什么C编译器不支持函数重载？或者说，在C++环境中调用C的代码并且要求编译器按照C的编译风格来编译这部分代码时，要加上`"extern C"`？

就是因为C编译器的`name mangling`规则与C++的不同：C语言的命名规则仅依赖于函数名，和函数参数类型无关。比如：

```c
int func(int val) {return 0; }
```

经过`name mangling`后得到的标识符是`func`：

```c
$ gcc name.c -o name.o && objdump  -t name.o | grep func
0000000000001129 g     F .text  0000000000000012              func
```

这就导致了C编译器不支持函数重载。