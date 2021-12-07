# memset签名

```
void *memset(void *dst, int val, size_t n)
```

# memset使用

```
#include <stdio.h>
#include <string.h>

int main()
{
    char str[50];

    strcpy(str, "This is string.h library function");
    puts(str);

    memset(str, '$', 7);
    puts(str);
}
```



# memset实现

```
#include <stdio.h>
#include <string.h>

void *memset(void *dst,int val,size_t n){
    void *d=dst;
    while(n--){
        *(char*)d=(char)val;
        d=(char*)d+1;
    }
    return dst;
}

int main()
{
    char str[50];

    strcpy(str, "This is string.h library function");
    puts(str);

    memset(str, '$', 7);
    puts(str);
}
```

```
This is string.h library function
$$$$$$$ string.h library function
```
