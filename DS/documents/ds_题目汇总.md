> Leetcode 题源：hot100，剑指offer，代码随想录，Acwing

## 1. 树

参考：[代码随想录 - 二叉树](https://programmercarl.com/%E4%BA%8C%E5%8F%89%E6%A0%91%E7%90%86%E8%AE%BA%E5%9F%BA%E7%A1%80.html#%E7%AE%97%E6%B3%95%E5%85%AC%E5%BC%80%E8%AF%BE)（剩余25/29/30/31/33）

### 1.1 遍历二叉树

**前序遍历 | 中序遍历 | 后序遍历 | 层序遍历（BFS） | DFS**

[94. 二叉树的中序遍历](https://leetcode.cn/problems/binary-tree-inorder-traversal/description/?envType=study-plan-v2&envId=top-100-liked)

[101. 对称二叉树](https://leetcode.cn/problems/symmetric-tree/)

[102. 二叉树层序遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal/submissions/525365479/)

[104. 二叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/)

[107. 二叉树层序遍历2](https://leetcode.cn/problems/binary-tree-level-order-traversal-ii/submissions/525367340/)

[110. 平衡二叉树](https://leetcode.cn/problems/balanced-binary-tree/)

[111. 二叉树的最小深度](https://leetcode.cn/problems/minimum-depth-of-binary-tree/)

[112. 路径总和](https://leetcode.cn/problems/path-sum/)

[116. 填充每个节点的下一个右侧节点指针](https://leetcode.cn/problems/populating-next-right-pointers-in-each-node/)

[199. 二叉树的右视图](https://leetcode.cn/problems/binary-tree-right-side-view/description/)

[222. 完全二叉树的节点个数](https://leetcode.cn/problems/count-complete-tree-nodes/)

[226. 翻转二叉树](https://leetcode.cn/problems/invert-binary-tree/)

[257. 二叉树的所有路径](https://leetcode.cn/problems/binary-tree-paths/)【+】

[404. 左叶子之和](https://leetcode.cn/problems/sum-of-left-leaves/)

[429. N叉树的层序遍历](https://leetcode.cn/problems/n-ary-tree-level-order-traversal/description/)

[513. 找树左下角的值](https://leetcode.cn/problems/find-bottom-left-tree-value/)

[515. 在每个树行中找最大值](https://leetcode.cn/problems/find-largest-value-in-each-tree-row/)

[559. N 叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-n-ary-tree/)

[617. 合并二叉树](https://leetcode.cn/problems/merge-two-binary-trees/)

[637. 二叉树的层序平均值](https://leetcode.cn/problems/average-of-levels-in-binary-tree/submissions/525371272/)

### 1.2 构造二叉树

> 中序 + 后序、中序 + 先序可以构造唯一的二叉树

[思路参考：代码随想录](https://programmercarl.com/0106.%E4%BB%8E%E4%B8%AD%E5%BA%8F%E4%B8%8E%E5%90%8E%E5%BA%8F%E9%81%8D%E5%8E%86%E5%BA%8F%E5%88%97%E6%9E%84%E9%80%A0%E4%BA%8C%E5%8F%89%E6%A0%91.html#%E7%AE%97%E6%B3%95%E5%85%AC%E5%BC%80%E8%AF%BE)

[105. 从前序与中序遍历序列构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)【+】

[106. 从中序与后序遍历序列构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)【+】

[654. 最大二叉树](https://leetcode.cn/problems/maximum-binary-tree/)

### 1.3 二叉搜索树 BST

> 中序遍历下，输出的二叉搜索树节点的数值是有序序列

[98. 验证二叉搜索树](https://leetcode.cn/problems/validate-binary-search-tree/)【+】

[108. 将有序数组转换为二叉搜索树](https://leetcode.cn/problems/convert-sorted-array-to-binary-search-tree/)

[114. 二叉树展开为链表](https://leetcode.cn/problems/flatten-binary-tree-to-linked-list/)【++】

[1382. 将二叉搜索树变平衡](https://leetcode.cn/problems/balance-a-binary-search-tree/)【+】

[230. 二叉搜索树中第K小的元素](https://leetcode.cn/problems/kth-smallest-element-in-a-bst/)

[530. 二叉搜索树的最小绝对差](https://leetcode.cn/problems/minimum-absolute-difference-in-bst/)【+】

[700. 二叉搜索树中的搜索](https://leetcode.cn/problems/search-in-a-binary-search-tree/)

### 1.4 路径问题

[437. 路径总和 III](https://leetcode.cn/problems/path-sum-iii/)【+++】

[235. 二叉搜索树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-search-tree/)【+++】

[236. 二叉树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/)【+++】

[124. 二叉树中的最大路径和](https://leetcode.cn/problems/binary-tree-maximum-path-sum/)【++++】

<br>

## 2. 回溯（暴搜）

参考：[代码随想录：理论基础](https://programmercarl.com/%E5%9B%9E%E6%BA%AF%E7%AE%97%E6%B3%95%E7%90%86%E8%AE%BA%E5%9F%BA%E7%A1%80.html#%E7%90%86%E8%AE%BA%E5%9F%BA%E7%A1%80)

**回溯三部曲**：

1. 确定回溯函数参数及返回值
2. 确定回溯终止条件
3. 回溯遍历过程

```cpp
void trace() {
    if(终止条件) {
        // 处理
	}
    
    // 剪枝处理
    for(遍历){
        // 1. 记录
        // 2. 递归
        // 3. 恢复现场
    }
}
```

### 2.1 组合问题

[77. 组合](https://leetcode.cn/problems/combinations/)【+++】（留意Go的写法）

[17. 电话号码的字母组合](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/)【+++】

[39. 组合总和](https://leetcode.cn/problems/combination-sum/)【+++】

[40. 组合总和 II](https://leetcode.cn/problems/combination-sum-ii/)【+++】

[216. 组合总和 III](https://leetcode.cn/problems/combination-sum-iii/)【+++】

[22. 括号生成](https://leetcode.cn/problems/generate-parentheses/)【+++】

### 2.2 搜索问题

[79. 单词搜索](https://leetcode.cn/problems/word-search/)【++++】（思考一下怎么继续优化）

[131. 分割回文串](https://leetcode.cn/problems/palindrome-partitioning/)【+++】

### 2.3 排列问题

[46. 全排列](https://leetcode.cn/problems/permutations/)

[51. N 皇后](https://leetcode.cn/problems/n-queens/)【+++】

### 2.4 集合划分问题

[参考讲解](https://lfool.github.io/LFool-Notes/algorithm/%E7%BB%8F%E5%85%B8%E5%9B%9E%E6%BA%AF%E7%AE%97%E6%B3%95%EF%BC%9A%E9%9B%86%E5%90%88%E5%88%92%E5%88%86%E9%97%AE%E9%A2%98.html)

[698. 划分为k个相等的子集](https://leetcode-cn.com/problems/partition-to-k-equal-sum-subsets/)【++++】（多方向考虑剪枝）

[473. 火柴拼正方形](https://leetcode.cn/problems/matchsticks-to-square/)【+++】

### 2.5 岛屿问题

[参考讲解（岛屿问题模板）](https://lfool.github.io/LFool-Notes/algorithm/%E7%A7%92%E6%9D%80%E6%89%80%E6%9C%89%E5%B2%9B%E5%B1%BF%E9%A2%98%E7%9B%AE(DFS).html)

[200. 岛屿数量](https://leetcode.cn/problems/number-of-islands/)【++】

[1254. 统计封闭岛屿的数目](https://leetcode.cn/problems/number-of-closed-islands/)【+++】

[695. 岛屿的最大面积](https://leetcode.cn/problems/max-area-of-island/)【++】

[1905. 统计子岛屿](https://leetcode.cn/problems/count-sub-islands/)【+++】



<br>

## 3. 动态规划

### 3.1 线性DP

[509. 斐波那契数](https://leetcode.cn/problems/fibonacci-number/)

[70. 爬楼梯](https://leetcode.cn/problems/climbing-stairs/)

[746. 使用最小花费爬楼梯](https://leetcode.cn/problems/min-cost-climbing-stairs/)【+】

[62. 不同路径](https://leetcode.cn/problems/unique-paths/)

[63. 不同路径 II](https://leetcode.cn/problems/unique-paths-ii/)【+】

### 3.2 计数DP

[343. 整数拆分](https://leetcode.cn/problems/integer-break/)【+++】

### 3.3 树形DP

[96. 不同的二叉搜索树](https://leetcode.cn/problems/unique-binary-search-trees/)【+++】

### 3.4 背包问题

[416. 分割等和子集](https://leetcode.cn/problems/partition-equal-subset-sum/)【+++】

[1049. 最后一块石头的重量 II](https://leetcode.cn/problems/last-stone-weight-ii/)【+++】

### 3.5 数组划分

<br>

## 4. 子串/子序列

### 4.1 滑动窗口

[76. 最小覆盖子串](https://leetcode.cn/problems/minimum-window-substring/)【+++】

[567. 字符串的排列](https://leetcode.cn/problems/permutation-in-string/)【+++】

[438. 找到字符串中所有字母异位词](https://leetcode.cn/problems/find-all-anagrams-in-a-string/)【+++】

[3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)【+++】







