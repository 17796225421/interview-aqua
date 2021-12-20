抽象图一二ij

最小堆proiority_queue<pair<string,int>根据int和string排序，防止重复某个单词，用unordered_map<string,int>预处理去重

```c
class Solution {
public:
    vector<string> topKFrequent(vector<string>& words, int k) {
        unordered_map<string,int>str2cnt;
        for(auto&s:words){
            str2cnt[s]++;
        }
        auto cmp=[&](pair<string,int>&a,pair<string,int>&b){
            return a.second>b.second||a.second==b.second&&a.first<b.first;
        };
        priority_queue<pair<string,int>,vector<pair<string,int>>,decltype(cmp)>pq(cmp);
        for(auto mPair:str2cnt){
            pq.push(mPair);
            if(pq.size()>k)pq.pop();
        }
        vector<string>ans;
        while(!pq.empty()){
            ans.push_back(pq.top().first);
            pq.pop();
        }
        reverse(ans.begin(),ans.end());
        return ans;

    }
};
```

