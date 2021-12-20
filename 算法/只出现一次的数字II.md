只出现一次的数字II

![img](image/1627984612258-16399125347427.png)

变量简洁正确完整思路

哈希表numToCnt，对于每个数nums[i]保存进numToCnt，遍历numToCnt，if it->second==1 return it->first

精确定义

```c
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        unordered_map<int,int>numToCnt;
        for(int i=0;i<nums.size();i++){
            numToCnt[nums[i]]++;
        }
        for(auto mPair:numToCnt){
            if(mPair.second==1)return mPair.first;
        }
        return -1;
    }
};

```

踩过的坑

for(auto it:numToCnt){

mPair是<int,int>

变量简洁正确完整思路

所有出现3次的num[i]的同一个二进制位的和必定是3的倍数，所有nums[i]

同一位的和%3，结果是出现1次的ans的同一个二进制位，枚举32位第i位，枚举所有数第i位

的二进制为ansi+=(nums[j]>>i)&1，枚举完如果ansi%3==1，ans|=(1<<i)

精确定义

i第i位

j第j个数

ansi第i位二进制和

ans

```c
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int ans=0;
        for(int i=0;i<32;i++){
            int ansi=0;
            for(int j=0;j<nums.size();j++){
                ansi+=(nums[j]>>i)&1;
            }
            if(ansi%3==1)ans|=(1<<i);
        }
        return ans;
    }
};
```





