#### [剑指 Offer 54. 二叉搜索树的第k大节点](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/)

难度简单224

给定一棵二叉搜索树，请找出其中第k大的节点。

 

**示例 1:**

```
输入: root = [3,1,4,null,2], k = 1
   3
  / \
 1   4
  \
   2
输出: 4
```

**示例 2:**

```
输入: root = [5,3,6,2,4,null,null,1], k = 3
       5
      / \
     3   6
    / \
   2   4
  /
 1
输出: 4
```

解题思路：
本文解法基于此性质：二叉搜索树的中序遍历为 递增序列 。

根据以上性质，易得二叉搜索树的 中序遍历倒序 为 递减序列 。
因此，求 “二叉搜索树第 kk 大的节点” 可转化为求 “此树的中序遍历倒序的第 kk 个节点”。


中序遍历 为 “左、根、右” 顺序，递归法代码如下：

PythonJava

# 打印中序遍历
def dfs(root):
    if not root: return
    dfs(root.left)  # 左
    print(root.val) # 根
    dfs(root.right) # 右
中序遍历的倒序 为 “右、根、左” 顺序，递归法代码如下：

PythonJava

# 打印中序遍历倒序
def dfs(root):
    if not root: return
    dfs(root.right) # 右
    print(root.val) # 根
    dfs(root.left)  # 左
为求第 kk 个节点，需要实现以下 三项工作 ：
递归遍历时计数，统计当前节点的序号；
递归到第 kk 个节点时，应记录结果 resres ；
记录结果后，后续的遍历即失去意义，应提前终止（即返回）。
递归解析：
终止条件： 当节点 rootroot 为空（越过叶节点），则直接返回；
递归右子树： 即 dfs(root.right)dfs(root.right) ；
三项工作：
提前返回： 若 k = 0k=0 ，代表已找到目标节点，无需继续遍历，因此直接返回；
统计序号： 执行 k = k - 1k=k−1 （即从 kk 减至 00 ）；
记录结果： 若 k = 0k=0 ，代表当前节点为第 kk 大的节点，因此记录 res = root.valres=root.val ；
递归左子树： 即 dfs(root.left)dfs(root.left) ；

1 / 7

复杂度分析：
时间复杂度 O(N)O(N) ： 当树退化为链表时（全部为右子节点），无论 kk 的值大小，递归深度都为 NN ，占用 O(N)O(N) 时间。
空间复杂度 O(N)O(N) ： 当树退化为链表时（全部为右子节点），系统使用 O(N)O(N) 大小的栈空间。
代码：
题目指出：1 \leq k \leq N1≤k≤N （二叉搜索树节点个数）；因此无需考虑 k > Nk>N 的情况。
若考虑，可以在中序遍历完成后判断 k > 0k>0 是否成立，若成立则说明 k > Nk>N 。

PythonJava

class Solution:
    def kthLargest(self, root: TreeNode, k: int) -> int:
        def dfs(root):
            if not root: return
            dfs(root.right)
            if self.k == 0: return
            self.k -= 1
            if self.k == 0: self.res = root.val
            dfs(root.left)

        self.k = k
        dfs(root)
        return self.res

