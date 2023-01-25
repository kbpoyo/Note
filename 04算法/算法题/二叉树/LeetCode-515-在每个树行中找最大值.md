#### [515. 在每个树行中找最大值](https://leetcode.cn/problems/find-largest-value-in-each-tree-row/)

##### 题目描述

给定一棵二叉树的根节点 `root` ，请找出该二叉树中每一层的最大值。

**示例1：**

![](https://assets.leetcode.com/uploads/2020/08/21/largest_e1.jpg)

>**输入:** root = [1,3,2,5,3,null,9]
**输出:** [1,3,9]

**示例2：**

>**输入:** root = [1,2,3]
**输出:** [1,3]

**提示：**

-   二叉树的节点个数的范围是 `[0,104]`
-   `-231 <= Node.val <= 231 - 1`


### 题解

#### 解一：深度优先

**思路**
- 用先序遍历将二叉树中每层最左边的一个节点值放入**res** 链表中
- 用**curDeepth** 记录当前深度
- 在后序遍历中比较当前深度节点的值与**res** 中对应**curDeepth** 位置的值的大小，更新为两者中的较大值

**代码**
```java
class Solution {

    public List<Integer> largestValues(TreeNode root) {

        List<Integer> res = new ArrayList();
        
        if (root == null) return new ArrayList();

        dfs(res, root, 0);

        return res;

    }
    
    public void dfs(List<Integer> res, TreeNode root, int curDeepth) {

        if (curDeepth == res.size()) {
            res.add(root.val);
        } else {
            res.set(curDeepth, Math.max(res.get(curDeepth), root.val));
	    }
	    
        if (root.left != null) {
            dfs(res, root.left, curDeepth + 1);
        }

        if (root.right != null) {
            dfs(res, root.right, curDeepth + 1);
        }
    }
}

```

**时间复杂度：O(n)**
其中 *n* 为二叉树节点个数。二叉树的遍历中每个节点会被访问一次且只会被访问一次。

**空间复杂度：O(height)**
其中*height* 表示二叉树的高度。递归函数需要栈空间，而栈空间取决于递归的深度，因此空间复杂度等价于二叉树的高度。


#### 解二：广度优先

**思路**
- 用队列 的方式进行层序遍历
- 对每一层的节点值单独进行比较，将最大值放入**res** 中

**代码**
```java
class Solution {

    public List<Integer> largestValues(TreeNode root) {

        if (root == null) {

            return new ArrayList();

        } 

        List<Integer> res = new ArrayList();

        Queue<TreeNode> queue = new ArrayDeque<TreeNode>();

        queue.offer(root);

  

        while (!queue.isEmpty()) {

            int len = queue.size();

            int max = Integer.MIN_VALUE;

            while (len > 0) {

                len--;

                TreeNode node = queue.poll();

                max = Math.max(max, node.val);

                if (node.left != null) {

                    queue.offer(node.left);

                }

                if (node.right != null) {

                    queue.offer(node.right);

                }

            }

            res.add(max);

        }

  

        return res;

    } 

}
```

**时间复杂度：O(n)**
其中 *n* 为二叉树节点个数，每一个节点仅会进出队列一次。

**空间复杂度：O(n)**
存储二叉树节点的空间开销。
