#### strcpy、strncpy和mmemcpy的区别

```cpp
char* strcpy(char*  dest, const char* src);
char* strncpy(char* dest, const char* src, size_t n);
void* memcpy (void* dest, const void* src, size_t n);
```

前面两个函数是以字符为单位，而*memcpy*是以字节为单位。

`strcpy`和`memcpy`主要有以下3方面的区别。

- 复制的内容不同。`strcpy` 只能复制字符串，而`memcpy`可以复制任意内容，例如字符数组、整型、结构体、类等。
- 复制的方法不同。`strcpy` 不需要指定长度，它遇到被复制字符的串结束符`'\0'`才结束，所以容易溢出。`memcpy` 则是根据其第3个参数决定复制的长度，而且如果字符串数据中包含`'\0'`，只能用`memcpy`。
- 用途不同。通常在复制字符串时用strcpy，而需要复制其他类型数据时则一般用memcpy

`strncpy` ：在复制字符串时，`memcpy`更加类似于`strncpy`。
`strncpy`和`memcpy`很相似，只不过它在一个终止的空字符处停止。当 `n > strlen(src)` 时，`dst[strlen(len)] ~ dst[n-1]`都是`\0`；当 `n<=strlen(src)`时，复制前`src`的n个字符。这里隐藏了一个事实，就是`dst`指向的内存一定会被写n个字符。

`memcpy`需要注意的是：

- `dest` 指针要分配足够的空间，也即大于等于 `n` 字节的空间。如果没有分配空间，会出现断错误。
- `dest` 和 `src` 所指的内存空间不能重叠（如果发生了重叠，使用 `memmove()` 会更加安全）。