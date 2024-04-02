### 红黑树 Red Black Tree

[定义红黑树](https://blog.csdn.net/weixin_37598682/article/details/115370538?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_utm_term~default-1-115370538-blog-17019209.235^v43^pc_blog_bottom_relevance_base6&spm=1001.2101.3001.4242.2&utm_relevant_index=4)

[红黑树插入](https://blog.csdn.net/spch2008/article/details/9338919)

[红黑树删除](https://blog.csdn.net/spch2008/article/details/9338923)

是一种特化的 AVL 树，通过插入和删除时通过特定操作保持 AVL 树的平衡

查找、插入、删除时间复杂度为`O(logN)`

#### 红黑树的性质

1. 每个节点**要么黑、要么红**
2. **根节点是黑色**
3. **叶子节点是黑色**，指向为 NULL 或叶节点是 NULL
4. **如果一个节点是红色，其子节点必须是黑色**
5. **从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点**

