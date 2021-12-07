# 函数签名

```
void *memmove(void *dst, const void *src, size_t n)
```

# 函数用法

```
#include <stdio.h>
#include <string.h>

int main()
{
    char dst[] = "oldstring";
    const char src[] = "newstring";
    printf("Before memmove dest = %s, src = %s\n", dst, src);
    memmove(dst, src, 9);
    printf("After memmove dest = %s, src = %s\n", dst, src);
}
```

```
Before memmove dest = oldstring, src = newstring
After memmove dest = newstring, src = newstring
```



# 函数实现

```
#include <stdio.h>
#include <string.h>

void *memmove(void *dst,const void *src,size_t n){
    char *d=(char *)dst;
    char *s=(char *)src;
    if(d>=s&&d<=s+n){
        // 内存重叠第二种情况，必须从后往前复制
        d+=n-1;
        s+=n-1;
        while(n--){
            *d--=*s--;
        }
    }else {
        while(n--){
            *d++=*s++;
        }
    }
    return dst;
}

int main()
{
    char dst[] = "oldstring";
    const char src[] = "newstring";
    printf("Before memmove dest = %s, src = %s\n", dst, src);
    memmove(dst, src, 9);
    printf("After memmove dest = %s, src = %s\n", dst, src);
}
```

