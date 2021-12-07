# 函数签名

```
char *strcpy(char *dst, const char *src)
```

# 函数使用

```
#include <stdio.h>
#include <string.h>
 
int main()
{
   char src[40];
   char dest[100];
  
   memset(dest, '\0', sizeof(dest));
   strcpy(src, "This is runoob.com");
   strcpy(dest, src);
 
   printf("最终的目标字符串： %s\n", dest);
   
   return(0);
}
```

```
最终的目标字符串： This is runoob.com
```

```
#include <stdio.h>
#include <string.h>

int main()
{
    char str1[] = "Sample string";
    char str2[40];
    char str3[40];
    strcpy(str2, str1);
    strcpy(str3, "copy successful");
    printf("str1: %s\nstr2: %s\nstr3: %s\n", str1, str2, str3);
    return 0;
}
```

```
str1: Sample string
str2: Sample string
str3: copy successful
```

# strcpy实现

```
#include <stdio.h>
#include <assert.h>
#include <string.h>

char *strcpy(char *dst,const char *src){
    assert(dst!=NULL&&src!=NULL);
    char *d=dst;
    while((*d++=*src++)!='\0');
    return dst;
}

int main()
{
    char str1[] = "Sample string";
    char str2[40];
    char str3[40];
    strcpy(str2, str1);
    strcpy(str3, "copy successful");
    printf("str1: %s\nstr2: %s\nstr3: %s\n", str1, str2, str3);
    return 0;
}
```

```
str1: Sample string
str2: Sample string
str3: copy successful
```

