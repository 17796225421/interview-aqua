# memcpy的函数原型

```
void *memcpy(void *str1, const void *str2, size_t n)
```

# memcpy的使用

```
// 将字符串复制到数组 dest 中
#include <stdio.h>
#include <string.h>
 
int main ()
{
   const char src[50] = "http://www.runoob.com";
   char dest[50];
 
   memcpy(dest, src, strlen(src)+1);
   printf("dest = %s\n", dest);
   
   return(0);
}
```

```
dest = http://www.runoob.com
```

```
#include <stdio.h>
#include<string.h>
 
int main()
 
{
  char *s="http://www.runoob.com";
  char d[20];
  memcpy(d, s+11, 6);// 从第 11 个字符(r)开始复制，连续复制 6 个字符(runoob)
  // 或者 memcpy(d, s+11*sizeof(char), 6*sizeof(char));
  d[6]='\0';
  printf("%s", d);
  return 0;
}

```

```
runoob
```

```
#include<stdio.h>
#include<string.h>
 
int main(void)
{
  char src[] = "***";
  char dest[] = "abcdefg";
  printf("使用 memcpy 前: %s\n", dest);
  memcpy(dest, src, strlen(src));
  printf("使用 memcpy 后: %s\n", dest);
  return 0;
}
```

```
使用 memcpy 前: abcdefg
使用 memcpy 后: ***defg
```

# 实现memcpy

```
#include <stdio.h>
#include <string.h>

void *memcpy(void *dest,const void*src,size_t n){
    char *d=dest;
    const char *s=src;
    while(n--){
        *d++=*s++;
    }
    return dest;
}

int main()
{
    const char src[50] = "http://www.runoob.com";
    char dest[50];

    memcpy(dest, src, strlen(src) + 1);
    printf("dest = %s\n", dest);

    return (0);
}
```

# 四字节拷贝

memcpy()充分利用了机器字长为32位的特性(32位系统，一次内存读/写可操作4字节的数据， 对于64位系统，则一次可操作8字节数据)。先按4字节一组(int * 指针++)拷贝，共复制n/4次；对剩下的零头再逐字节(char * 指针++)拷贝。





