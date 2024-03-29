前K个高频元素

![img](image/1628844763669.png)

变量简洁正确完整思路

按出现次数放入最小堆，一旦超过k立刻pop，最后剩余答案

最小堆priority_queue<pair<int值,int次数>,vector<pair<int,int>,cmp>minHeap

struct cmp 比较lhs.second>rhs.second

unordered_map<int值,int次数>num2index

```c
struct cmp{
    bool operator()(const pair<int,int>&lhs,const pair<int,int>&rhs){
        return lhs.second>rhs.second;
    }
};
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        priority_queue<pair<int,int>,vector<pair<int,int>>,cmp>minHeap;
        unordered_map<int,int>num2cnt;
        for(int &num:nums)num2cnt[num]++;
        for(auto &mPair:num2cnt){
            minHeap.push(mPair);
            if(minHeap.size()>k)minHeap.pop();
        }
        vector<int>ans;
        while(!minHeap.empty()){
            ans.push_back(minHeap.top().first);
            minHeap.pop();
        }
        return ans;
    }
};
```



踩过的坑

堆用push不用insert

struct cmp{

  bool operator()(const pair<int,int>&lhs,const pair<int,int>&rhs){

​    return lhs.second>rhs.second;

  }

};

自定义方法

变量简洁正确完整思路

快速排序vector<pair<int值,int次数>>i==j，次数比[i]小于等于都在i左边，

次数比[i]大都在i右边，如果n-i大于k则答案在右边，如果n-i小于k则答案为右边+

dfs左边次数前k-右边个数，

```c
class Solution {
public:
    vector<int>ans;
    int n;
    vector<int> topKFrequent(vector<int>& nums, int k) {
        unordered_map<int,int>mp;
        for(int &num:nums)mp[num]++;
        vector<pair<int,int>>num2cnt;
        for(auto &mPair:mp){
            num2cnt.push_back(mPair);
        }
        n=num2cnt.size();
        dfs(0,n-1,k,num2cnt);
        return ans;
    }
    void dfs(int beg,int end,int k,vector<pair<int,int>>&num2cnt){
        //cout<<beg<<' '<<end<<endl;
        if(beg>end)return;
        int i=beg,j=end;
        while(i<j){
            while(num2cnt[j].second>num2cnt[beg].second&&j>i)j--;
            while(num2cnt[i].second<=num2cnt[beg].second&&i<j)i++;
            if(i<j)swap(num2cnt[i],num2cnt[j]);
        }
        swap(num2cnt[beg],num2cnt[i]);
        //cout<<beg<<' '<<end<<' '<<end-i-1<<endl;
        if(end-i==0){
            ans.push_back(num2cnt[i].first);
            dfs(beg,i-1,k-1,num2cnt);
        }
        else if(end-i==k){
            for(int ii=i+1;ii<=end;ii++){
                ans.push_back(num2cnt[ii].first);
            }
            return ;
        }else if(end-i>k){
            dfs(i+1,end,k,num2cnt);
        }else if(end-i<k){
            for(int ii=i+1;ii<=end;ii++){
                ans.push_back(num2cnt[ii].first);
            }
            dfs(beg,i,k-(end-i),num2cnt);
        }
    }
};
```



踩过的坑

先用unordered_map处理后保存到vector<pair。无法跳过unordered_map

i左边小于等于，i右边大于，分清楚，只需要num2cnt[j].second>num2cnt[beg].second

​    if(end-i==0){

​      ans.push_back(num2cnt[i].first);

​      dfs(beg,i-1,k-1,num2cnt);

​    }

右边长度为0，至少为1





 先拿10000个数建堆，然后一次添加剩余元素，如果大于堆顶的数（10000中最小的），将这个数替换堆顶，并调整结构使之仍然是一个最小堆，这样，遍历完后，堆中的10000个数就是所需的最大的10000个。建堆时间复杂度是O（mlogm），算法的时间复杂度为O（nmlogm）（n为10亿，m为10000）。

        优化的方法：可以把所有10亿个数据分组存放，比如分别放在1000个文件中。这样处理就可以分别在每个文件的10^6个数据中找出最大的10000个数，合并到一起在再找出最终的结果。
    
        以上就是面试时简单提到的内容，下面整理一下这方面的问题：

top K问题
        在大规模数据处理中，经常会遇到的一类问题：在海量数据中找出出现频率最好的前k个数，或者从海量数据中找出最大的前k个数，这类问题通常被称为top K问题。例如，在搜索引擎中，统计搜索最热门的10个查询词；在歌曲库中统计下载最高的前10首歌等。
        针对top K类问题，通常比较好的方案是分治+Trie树/hash+小顶堆（就是上面提到的最小堆），即先将数据集按照Hash方法分解成多个小数据集，然后使用Trie树活着Hash统计每个小数据集中的query词频，之后用小顶堆求出每个数据集中出现频率最高的前K个数，最后在所有top K中求出最终的top K。

eg：有1亿个浮点数，如果找出期中最大的10000个？
        最容易想到的方法是将数据全部排序，然后在排序后的集合中进行查找，最快的排序算法的时间复杂度一般为O（nlogn），如快速排序。但是在32位的机器上，每个float类型占4个字节，1亿个浮点数就要占用400MB的存储空间，对于一些可用内存小于400M的计算机而言，很显然是不能一次将全部数据读入内存进行排序的。其实即使内存能够满足要求（我机器内存都是8GB），该方法也并不高效，因为题目的目的是寻找出最大的10000个数即可，而排序却是将所有的元素都排序了，做了很多的无用功。

        第二种方法为局部淘汰法，该方法与排序方法类似，用一个容器保存前10000个数，然后将剩余的所有数字——与容器内的最小数字相比，如果所有后续的元素都比容器内的10000个数还小，那么容器内这个10000个数就是最大10000个数。如果某一后续元素比容器内最小数字大，则删掉容器内最小元素，并将该元素插入容器，最后遍历完这1亿个数，得到的结果容器中保存的数即为最终结果了。此时的时间复杂度为O（n+m^2），其中m为容器的大小，即10000。
    
        第三种方法是分治法，将1亿个数据分成100份，每份100万个数据，找到每份数据中最大的10000个，最后在剩下的100*10000个数据里面找出最大的10000个。如果100万数据选择足够理想，那么可以过滤掉1亿数据里面99%的数据。100万个数据里面查找最大的10000个数据的方法如下：用快速排序的方法，将数据分为2堆，如果大的那堆个数N大于10000个，继续对大堆快速排序一次分成2堆，如果大的那堆个数N大于10000个，继续对大堆快速排序一次分成2堆，如果大堆个数N小于10000个，就在小的那堆里面快速排序一次，找第10000-n大的数字；递归以上过程，就可以找到第1w大的数。参考上面的找出第1w大数字，就可以类似的方法找到前10000大数字了。此种方法需要每次的内存空间为10^6*4=4MB，一共需要101次这样的比较。
    
        第四种方法是Hash法。如果这1亿个书里面有很多重复的数，先通过Hash法，把这1亿个数字去重复，这样如果重复率很高的话，会减少很大的内存用量，从而缩小运算空间，然后通过分治法或最小堆法查找最大的10000个数。
    
        第五种方法采用最小堆。首先读入前10000个数来创建大小为10000的最小堆，建堆的时间复杂度为O（mlogm）（m为数组的大小即为10000），然后遍历后续的数字，并于堆顶（最小）数字进行比较。如果比最小的数小，则继续读取后续数字；如果比堆顶数字大，则替换堆顶元素并重新调整堆为最小堆。整个过程直至1亿个数全部遍历完为止。然后按照中序遍历的方式输出当前堆中的所有10000个数字。该算法的时间复杂度为O（nmlogm），空间复杂度是10000（常数）。

实际运行：
        实际上，最优的解决方案应该是最符合实际设计需求的方案，在时间应用中，可能有足够大的内存，那么直接将数据扔到内存中一次性处理即可，也可能机器有多个核，这样可以采用多线程处理整个数据集。

       下面针对不容的应用场景，分析了适合相应应用场景的解决方案。

（1）单机+单核+足够大内存
        如果需要查找10亿个查询次（每个占8B）中出现频率最高的10个，考虑到每个查询词占8B，则10亿个查询次所需的内存大约是10^9 * 8B=8GB内存。如果有这么大内存，直接在内存中对查询次进行排序，顺序遍历找出10个出现频率最大的即可。这种方法简单快速，使用。然后，也可以先用HashMap求出每个词出现的频率，然后求出频率最大的10个词。

（2）单机+多核+足够大内存
        这时可以直接在内存总使用Hash方法将数据划分成n个partition，每个partition交给一个线程处理，线程的处理逻辑同（1）类似，最后一个线程将结果归并。

        该方法存在一个瓶颈会明显影响效率，即数据倾斜。每个线程的处理速度可能不同，快的线程需要等待慢的线程，最终的处理速度取决于慢的线程。而针对此问题，解决的方法是，将数据划分成c×n个partition（c>1），每个线程处理完当前partition后主动取下一个partition继续处理，知道所有数据处理完毕，最后由一个线程进行归并。

（3）单机+单核+受限内存
        这种情况下，需要将原数据文件切割成一个一个小文件，如次啊用hash(x)%M，将原文件中的数据切割成M小文件，如果小文件仍大于内存大小，继续采用Hash的方法对数据文件进行分割，知道每个小文件小于内存大小，这样每个文件可放到内存中处理。采用（1）的方法依次处理每个小文件。

（4）多机+受限内存
        这种情况，为了合理利用多台机器的资源，可将数据分发到多台机器上，每台机器采用（3）中的策略解决本地的数据。可采用hash+socket方法进行数据分发。


        从实际应用的角度考虑，（1）（2）（3）（4）方案并不可行，因为在大规模数据处理环境下，作业效率并不是首要考虑的问题，算法的扩展性和容错性才是首要考虑的。算法应该具有良好的扩展性，以便数据量进一步加大（随着业务的发展，数据量加大是必然的）时，在不修改算法框架的前提下，可达到近似的线性比；算法应该具有容错性，即当前某个文件处理失败后，能自动将其交给另外一个线程继续处理，而不是从头开始处理。
    
        top K问题很适合采用MapReduce框架解决，用户只需编写一个Map函数和两个Reduce 函数，然后提交到Hadoop（采用Mapchain和Reducechain）上即可解决该问题。具体而言，就是首先根据数据值或者把数据hash(MD5)后的值按照范围划分到不同的机器上，最好可以让数据划分后一次读入内存，这样不同的机器负责处理不同的数值范围，实际上就是Map。得到结果后，各个机器只需拿出各自出现次数最多的前N个数据，然后汇总，选出所有的数据中出现次数最多的前N个数据，这实际上就是Reduce过程。对于Map函数，采用Hash算法，将Hash值相同的数据交给同一个Reduce task；对于第一个Reduce函数，采用HashMap统计出每个词出现的频率，对于第二个Reduce 函数，统计所有Reduce task，输出数据中的top K即可。
    
        直接将数据均分到不同的机器上进行处理是无法得到正确的结果的。因为一个数据可能被均分到不同的机器上，而另一个则可能完全聚集到一个机器上，同时还可能存在具有相同数目的数据。
