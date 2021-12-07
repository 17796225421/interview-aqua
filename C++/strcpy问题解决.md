# 缓冲区溢出

strcpy，目的地址字符串空间不能小于源地址字符串空间，否则按照strcpy实现，dst的'\0'会被src的字符覆盖，产生未定义行为。编译器会对此缓冲区溢出做出警告，但依旧可以通过编译。

```
#include <stdio.h>
#include <string.h>

int main()
{
    char src[10];
    char dst[12];

    memset(dst, '\0', sizeof(dst));
    strcpy(src, "This is runoob.com");
    strncpy(dst, src, 10);

    printf("最终的目标字符串： %s\n", dst);

    return (0);
}
```



## 解决方法

改用strncpy或者strcpy_s

### strncpy

strncpy不仅需要n大小，还会自动在dst结尾补上'\0'

```
char *strncpy(char *dst, const char *src, size_t n)
```



```
#include <stdio.h>
#include <string.h>

int main()
{
    char src[40];
    char dst[12];

    memset(dst, '\0', sizeof(dst));
    strcpy(src, "This is runoob.com");
    strncpy(dst, src, 10);

    printf("最终的目标字符串： %s\n", dst);

    return (0);
}
```



```
最终的目标字符串： This is ru
```

### strcpy_s

strcpy_s

```
 void strcpy_s(char *dst, const char *src);
```

当有缓冲区溢出问题，strcpy_s函数会抛出异常，而使用strcpy函数结果是未定义。简单的说，如果存在缓冲区溢出，strcpy_s 在编译期就做了检查，可以检查出错误，而 strcpy 在编译时可能检查不出来，在运行期间溢出时才可能被监测到。

# 内存重叠

strcpy和memcpy一样有着内存重叠的问题，解决方式是改用memmove



> 内存重叠问题
>
> ![image-20211207125242341](image/image-20211207125242341.png)
>
> 第一种情况，拷贝重叠区域不会出现问题，内容均可以正确被拷贝。
>
> 第二种情况，问题出现在右边两个字节，这两个字节的原先内容一开始就被覆盖了，而且没有保存。所以接下来拷贝的时候，拷贝的是已经被覆盖的内容，显然是有问题的。
>
> 
>
> 对于内存重叠第二种情况，该memcpy行为是未定义的，编译器可能会让程序崩溃，也可能只是复制结果不正确。
>
> 
>
> 但是memmove有很好的考虑到内存重叠第二种情况会发生的问题，从后往前复制来避免拷贝到被覆盖的内容。
>
> 
>
> memmove在memcpy的基础上，很好解决了内存重叠问题，memmove开头加了一个分支，不重叠的时候走memcpy一样的代码。用memmove来代替memcpy，放在现在来设计标准库，只提供一个函数才是正确的设计，显然memcpy是历史包袱。
>
> 
>
> memcpy也不是一无是处。
>
> 当memcpy、memmove优化到极致，多一次两次判断对整体的性能影响是比较大的，特别是在流水线比较长的处理器体系中。
>
> 同时，内存复制的使用时非常频繁的，几个字节都用memcpy，多一条少一条分支判断都会影响总体性能。
>
> 同时，实际情况下，两个区域是否重叠往往是可预期的，真的重叠了，那应该是程序bug，此时未定义行为如果是程序崩溃反而是好事，提醒你这里有个bug，但未定义行为如果只是复制结果不正确，那隐藏的bug难以找出来。

memmove实现

>
>
># 函数签名
>
>```
>void *memmove(void *dst, const void *src, size_t n)
>```
>
># 函数用法
>
>```
>#include <stdio.h>
>#include <string.h>
>
>int main()
>{
>    char dst[] = "oldstring";
>    const char src[] = "newstring";
>    printf("Before memmove dest = %s, src = %s\n", dst, src);
>    memmove(dst, src, 9);
>    printf("After memmove dest = %s, src = %s\n", dst, src);
>}
>```
>
>```
>Before memmove dest = oldstring, src = newstring
>After memmove dest = newstring, src = newstring
>```
>
>
>
># 函数实现
>
>```
>#include <stdio.h>
>#include <string.h>
>
>void *memmove(void *dst,const void *src,size_t n){
>    char *d=(char *)dst;
>    char *s=(char *)src;
>    if(d>=s&&d<=s+n){
>        // 内存重叠第二种情况，必须从后往前复制
>        d+=n-1;
>        s+=n-1;
>        while(n--){
>            *d--=*s--;
>        }
>    }else {
>        while(n--){
>            *d++=*s++;
>        }
>    }
>    return dst;
>}
>
>int main()
>{
>    char dst[] = "oldstring";
>    const char src[] = "newstring";
>    printf("Before memmove dest = %s, src = %s\n", dst, src);
>    memmove(dst, src, 9);
>    printf("After memmove dest = %s, src = %s\n", dst, src);
>}
>```
>
>







