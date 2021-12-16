详细思路

二分查找，如果mid是要的直接返回，考虑leftmid重叠也就是数组长度为1或者2直接返回，否则必须找到一个非递减数组，判断target在其中或者不在其中进入或者排除，

精确定义

left数组第一

right数组最后一个元素

```c
class Solution {
public:
    bool search(vector<int>& nums, int target) {
        int n=nums.size();
        int left=0,right=n-1;
        while(left<=right){
            int mid=left+(right-left)/2;
            if(nums[mid]==target)return true;
            if(left==mid){
                if(nums[left]==target||nums[right]==target)return true;
                else return false;
            }
            if(nums[left]==nums[mid])left++;
            else if(nums[right]==nums[mid])right--;
            else if(nums[left]<nums[mid]){
                if(nums[left]<=target&&target<nums[mid])right=mid-1;
                else left=mid+1;
            }
            else if(nums[mid]<nums[right]){
                if(nums[mid]<target&&target<=nums[right])left=mid+1;
                else right=mid-1;
            }
        }
        return false;
    }
};

```


踩过的坑

​      if(nums[left]==nums[mid])left++;

​      else if(nums[right]==nums[mid])right--;

不这么做，10111找0根本无法判断哪个是非递减数组

​      if(left==mid){

​        if(nums[left]==target||nums[right]==target)return true;

​        else return false;

​      }

只有两个或一个先输出

​       if(nums[left]<=target&&target<nums[mid])r

target在left处也算在非递减数组中

