N个长度不一的有序数组的TOPK问题
假设K个长度不一的有序数组，求解TOPK最小元素 问题的C++语言实现

总结：
可以查阅左程云算法书第八章TOPK问题（java），本文章属于自写，细节可能写的不多，但是基本思想和其是一致的。

基本上在熟悉堆排序源代码的基础之上，只要把以前只存在数值的堆节点变成结构体堆节点，其他逻辑都是一样的。
结构体节点：当前节点的数值大小，当前节点数值在二维数组中的行号，当前节点元素的所在行的下一个元素的列下标。
 提取K个数组中每一个数组的第一个元素，建立K大小的最小堆（从最后一个非叶子节点开始调整）。

 弹出堆顶元素并存放在新数组中，并将堆顶元素所在二维数组对应行，下一列元素放入堆顶，针对堆顶进行最小堆调整。

 执行TOPK次，就是TOPK个最小元素

tip：通过利用vector的size（）函数可以有效控制判断数组是否到达行数组的末尾。

设计到结构体的这种，最好写一个swap函数进行交换。

时间复杂性：建立堆O(K)+重建堆O(NKlogK)=O(NKlogK)

空间复杂性：O(1)

```c
#include <iostream>
#include <vector>
using namespace std;

#define K 3
#define MaxValue SHRT_MAX

//堆节点定义
struct Node
{
	short value;//数组的当前元素对应值大小
	short arrnum;//数组索引
	short index;//数组的当前元素下一个元素的索引
};

//最小堆数组
Node MinHeap[K];

//交换元素
void Swap(Node &X, Node &Y)
{
	Node temp = X;
	X = Y;
	Y = temp;
}

//针对K个元素，以i节点的向下过滤函数
void PerDown(Node A[], const char i, const char N)
{
	Node temp = A[i];

	char Parent, Child;
	for (Parent = i; Parent * 2 + 1 < N; Parent = Child)
	{
		Child = Parent * 2 + 1;
		if (Child+1 < N && A[Child+1].value < A[Child].value)
		{
			Child++;
		}
	
		if (temp.value <= A[Child].value)
		{
			break;
		}
		else
		{
			swap(A[Parent], A[Child]);
		}
	}
	A[Parent].arrnum = temp.arrnum;
	A[Parent].index = temp.index;
	A[Parent].value = temp.value;

}

//创建最小堆
void BuildMinHeap()
{
	for (char i = (K - 1) / 2; i >= 0; i--)
	{
		PerDown(MinHeap, i, K);
	}
}


void TopK(const vector<vector<short>> &InArray, vector<short> &OutArray, const short &TOPK)
{
	//创建K大小的数组
	for (char i = 0; i < K; i++)
	{
		MinHeap[i].value = InArray[i][0];
		MinHeap[i].arrnum = i;
		MinHeap[i].index = 1;
	}

	//针对数组做最小堆处理
	BuildMinHeap();
	
	for (char i = 0; i < TOPK; i++)
	{
		//先取节点出来
		Node temp = MinHeap[0];
		//把节点的值取出来
		OutArray.push_back(temp.value);
	
		//数组索引无需修改，数组元素索引+1
		if (temp.index < InArray[temp.arrnum].size())
		{
			MinHeap[0].value = InArray[temp.arrnum][temp.index];
			MinHeap[0].index++;
		}
		else//溢出处理
		{
			MinHeap[0].value = MaxValue;
		}
		
		PerDown(MinHeap, 0, K);
	}

}

void main()
{
	short TOPK = 15;
	//最终存放TOPK个最小元素的容器
	vector <short> OutArray;
	//K个原始数组
	short arr1[3] = { 56,78,981 };
	short arr2[4] = { 24,89,901,1003 };
	short arr3[6] = { 35,47,79,103,256,345 };
	//转为一维向量
	vector<short> InArray1(arr1, arr1 + sizeof(arr1) / sizeof(short));
	vector<short> InArray2(arr2, arr2 + sizeof(arr2) / sizeof(short));
	vector<short> InArray3(arr3, arr3 + sizeof(arr3) / sizeof(short));
	//总元素个数以及TOPK的确定
	short total = InArray1.size() + InArray2.size() + InArray3.size();
	TOPK = TOPK > total ? total : TOPK;
	//转为二维向量
	vector<vector<short>> InArray;
	InArray.push_back(InArray1);
	InArray.push_back(InArray2);
	InArray.push_back(InArray3);

	//求解TOPK
	TopK(InArray, OutArray, TOPK);
	//打印输出
	for (char i = 0; i < TOPK; i++)
	{
		cout << OutArray[i] << endl;
	}
	
	system("pause");

}

```

