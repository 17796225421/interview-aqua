x的平方根![img](image/1627548030375.png)详细思路

二分，找到中间，比较mid*mid和x，如果mid*mid<=x并且mid+1*mid+1>x答案，退出

精确定义

left第一个数

right最后一个数

mid需要判断

```c

class Solution {
public:
    int mySqrt(int x) {
        int left=1,right=x,ans=0;
        while(left<=right){
            int mid=left+(right-left)/2;
            if(mid<=x/mid&&(mid+1)>x/(mid+1)){
                ans=mid;
                break;
            }
            else if(mid<x/mid)left=mid+1;
            else right=mid-1;
        }
        return ans;
    }
};
```



踩过的坑，要么longlong要么乘法变成除法

x/mid

```c
class Solution {
public:
    int mySqrt(int x) {
        int left=1,right=x,ans=0;
        while(left<=right){
            int mid=left+(right-left)/2;
            if((long long)mid*mid<=x&&(long long)(mid+1)*(mid+1)>x){
                ans=mid;
                break;
            }
            else if((long long)mid*mid<x)left=mid+1;
            else right=mid-1;
        }
        return ans;
    }
};

```

fabs求浮点绝对值，abs求整数绝对值

牛顿迭代法是二次收敛的

画图

![img](image/1627561281640.png)

详细定义

xi为当前需要判断，x0为上一个接近的点，通过切线算出xi，如果x0-xi<1e-7（一个极小的非负数），就可以认为x0的整数部分就是答案

```c
class Solution {
public:
    int mySqrt(int x) {
        if(x==0)return 0;
        double x0=x,c=x;
        while(1){
            double xi=(c/x0+x0)/2.0;
            if(x0-xi<1e-7)break;
            x0=xi;
        }
        return (int)x0;
    }
};
```

浮点数求平方根只需要return (int)x0;去掉强制转换。

打印保留几位有效数字

1.printf("%3.0f",floatNum)：不保留小数

说明：%3.0f表明待打印的浮点数（floatNum）至少占3个字符宽，且不带小数点和小数部分，整数部分至少占3个位宽；

2.printf("%6.2f".floatNum)：保留两位小数

说明：%6.2f 表明待打印的数（floatNum）至少占6个字符宽度（包括两位小数和一个小数点），且小数点后面有2位小数，小数点占一位，所以整数部分至少占3位。

3.printf("%.0f\n", 100.00);表示对字符宽度没有限制，但是保留0位小数。

四舍五入
