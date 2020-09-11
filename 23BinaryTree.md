二叉树基础（上）：什么样的二叉树适合用数组来存储？
===

&emsp;&emsp;栈、队列等都是线性表结构，树是一种非线性表结构。树这种数据结构比线性表的数据结构要复杂得多。

树、二叉树、二叉查找树、平衡二叉查找树、红黑树、递归树。

&emsp;&emsp;**二叉树有哪几种存储方式？什么样的二叉树适合用数组来存储？**



##### 树（Tree）

&emsp;&emsp;树的定义

<img src="assert/DefinitionOfTreea.jpg" alt="树的定义" style="zoom:50%;" />

&emsp;&emsp;每个元素叫作“节点”；用来连线相邻节点之间的关系，叫作“父子关系”。

&emsp;&emsp;比如下面这幅图，A 节点就是 B 节点的**父节点**，B 节点是 A 节点的**子节点**。B、C、D 这三个节点的父节点是同一个节点，所以它们之间互称为**兄弟节点**。把没有父节点的节点叫作**根节点**，也就是图中的节点 E。把没有子节点的节点叫作**叶子节点或者叶节点**，比如图中的 G、H、I、J、K、L 都是叶子节点。

<img src="assert/DefinitionOfTreeb.jpg" alt="树的定义" style="zoom:50%;" />

&emsp;&emsp;关于“树”，还有三个比较相似的概念：高度（Height）、深度（Depth）、层（Level）。它们的定义是这样的：

<img src="assert/DefinitionOfTreec.jpg" alt="树的定义" style="zoom:50%;" />

<img src="assert/DefinitionOfTreed.jpg" alt="树的定义" style="zoom:50%;" />

&emsp;&emsp;树的高度，从最底层开始计数，并且计数的起点是 0。

&emsp;&emsp;树这的深度，从根结点开始度量，并且计数起点也是 0。

&emsp;&emsp;树的层，从根节点开始计数，计数起点是 1，也就是说根节点的位于第 1 层。



##### 二叉树（Binary Tree）

&emsp;&emsp;最常用的树结构是二叉树。

&emsp;&emsp;二叉树，每个节点最多有两个子节点，分别是左子节点和右子节点。不过，二叉树并不要求每个节点都有两个子节点，有的节点只有左子节点，有的节点只有右子节点。

<img src="assert/BinaryTree.jpg" alt="二叉树" style="zoom:50%;" />

&emsp;&emsp;编号 2 的二叉树中，叶子节点全都在最底层，除了叶子节点之外，每个节点都有左右两个子节点，这种二叉树就叫作**满二叉树**。

&emsp;&emsp;编号 3 的二叉树中，叶子节点都在最底下两层，最后一层的叶子节点都靠左排列，并且除了最后一层，其他层的节点个数都要达到最大，这种二叉树叫作**完全二叉树**。

<img src="assert/CompleteBinaryTree.jpg" alt="完全二叉树" style="zoom:50%;" />

&emsp;&emsp;存储一棵二叉树，一种是基于指针或者引用的二叉链式存储法，一种是基于数组的顺序存储法。

&emsp;&emsp;简单、直观的**链式存储法**，每个节点有三个字段，其中一个存储数据，另外两个是指向左右子节点的指针。只要拎住根节点，就可以通过左右子节点的指针，把整棵树都串起来。这种存储方式比较常用。大部分二叉树代码都是通过这种结构来实现的。

<img src="assert/LinkedStorageOfCompleteBinaryTree.jpg" alt="完全二叉树的链式存储" style="zoom:50%;" />

&emsp;&emsp;基于数组的**顺序存储法**。把根节点存储在下标 i = 1 的位置，那左子节点存储在下标 2 * i = 2 的位置，右子节点存储在 2 * i + 1 = 3 的位置。以此类推，B 节点的左子节点存储在 2 * i = 2 * 2 = 4 的位置，右子节点存储在 2 * i + 1 = 2 * 2 + 1 = 5 的位置。

<img src="assert/SequentialStorageOfCompleteBinaryTree.jpg" alt="完全二叉树的顺序存储" style="zoom:50%;" />

&emsp;&emsp;如果节点 X 存储在数组中下标为 i 的位置，下标为 2 * i 的位置存储的就是左子节点，下标为 2 * i + 1 的位置存储的就是右子节点。反过来，下标为 i/2 的位置存储就是它的父节点。通过这种方式，只要知道根节点存储的位置（一般情况下，为了方便计算子节点，根节点会存储在下标为 1 的位置），这样就可以通过下标计算，把整棵树都串起来。

&emsp;&emsp;刚刚的例子是一棵完全二叉树，所以仅仅“浪费”了一个下标为 0 的存储位置。如果是非完全二叉树，其实会浪费比较多的数组存储空间。

<img src="assert/SequentialStorageOfCompleteBinaryTree.jpg" alt="非完全二叉树的顺序存储" style="zoom:50%;" />

&emsp;&emsp;如果某棵二叉树是一棵完全二叉树，那用数组存储无疑是最节省内存的一种方式。因为数组的存储方式并不需要像链式存储法那样，要存储额外的左右子节点的指针。这也是为什么完全二叉树要求最后一层的子节点都靠左的原因。堆其实就是一种完全二叉树，最常用的存储方式就是数组。



##### 二叉树的遍历

&emsp;&emsp;二叉树中非常重要的操作，二叉树的遍历。经典的方法有三种，前序遍历、中序遍历和后序遍历。其中，前、中、后序，表示的是节点与它的左右子树节点遍历打印的先后顺序。

* 前序遍历是指，对于树中的任意节点来说，先打印这个节点，然后再打印它的左子树，最后打印它的右子树。
* 中序遍历是指，对于树中的任意节点来说，先打印它的左子树，然后再打印它本身，最后打印它的右子树。
* 后序遍历是指，对于树中的任意节点来说，先打印它的左子树，然后再打印它的右子树，最后打印这个节点本身。

<img src="assert/TraversalOfABinaryTree.jpg" alt="二叉树的遍历" style="zoom:50%;" />

&emsp;&emsp;**二叉树的前、中、后序遍历就是一个递归的过程**。比如，前序遍历，其实就是先打印根节点，然后再递归地打印左子树，最后递归地打印右子树。
```C
// 前序遍历的递推公式：
preOrder(r) = print r->preOrder(r->left)->preOrder(r->right)
 
// 中序遍历的递推公式：
inOrder(r) = inOrder(r->left)->print r->inOrder(r->right)
 
// 后序遍历的递推公式：
postOrder(r) = postOrder(r->left)->postOrder(r->right)->print r
```
```C
void preOrder(Node* root) {
  if (root == null) return;
  print root // 此处为伪代码，表示打印 root 节点
  preOrder(root->left);
  preOrder(root->right);
}
 
void inOrder(Node* root) {
  if (root == null) return;
  inOrder(root->left);
  print root // 此处为伪代码，表示打印 root 节点
  inOrder(root->right);
}
 
void postOrder(Node* root) {
  if (root == null) return;
  postOrder(root->left);
  postOrder(root->right);
  print root // 此处为伪代码，表示打印 root 节点
}
```
&emsp;&emsp;二叉树的前、中、后序遍历，每个节点最多会被访问两次，所以遍历操作的时间复杂度，跟节点的个数 n 成正比，也就是说二叉树遍历的时间复杂度是 O(n)。



##### 解答开篇 & 内容小结

&emsp;&emsp;树是一种非线性表数据结构，有几个比较常用的概念，根节点、叶子节点、父节点、子节点、兄弟节点，还有节点的高度、深度、层数，以及树的高度。

&emsp;&emsp;最常用的树就是二叉树。二叉树的每个节点最多有两个子节点，分别是左子节点和右子节点。二叉树中，有两种比较特殊的树，分别是满二叉树和完全二叉树。满二叉树又是完全二叉树的一种特殊情况。

&emsp;&emsp;二叉树既可以用链式存储，也可以用数组顺序存储。数组顺序存储的方式比较适合完全二叉树，其他类型的二叉树用数组存储会比较浪费存储空间。除此之外，二叉树里非常重要的操作就是前、中、后序遍历操作，遍历的时间复杂度是 O(n)，需要理解并能用递归代码来实现。



##### 课后思考

&emsp;&emsp;给定一组数据，比如 1，3，5，6，9，10。你来算算，可以构建出多少种不同的二叉树？

&emsp;&emsp;讲了三种二叉树的遍历方式，前、中、后序。实际上，还有另外一种遍历方式，也就是按层遍历，你知道如何实现吗？


1.是卡特兰数，是C[n,2n] / (n+1)种形状，c是组合数，节点的不同又是一个全排列，一共就是n!$*$C[n,2n] $/$ (n+1)个二叉树。可以通过数学归纳法推导得出。
2.层次遍历需要借助队列这样一个辅助数据结构。（其实也可以不用，这样就要自己手动去处理节点的关系，代码不太好理解，好处就是空间复杂度是o(1)。不过用队列比较好理解，缺点就是空间复杂度是o(n)）。根节点先入队列，然后队列不空，取出对头元素，如果左孩子存在就入列队，否则什么也不做，右孩子同理。直到队列为空，则表示树层次遍历结束。树的层次遍历，其实也是一个广度优先的遍历算法。



关于问题1，如果是完全二叉树，老师说过可以放在数组里面，那么问题是否 可以简化为数组内的元素有多少种组合方式，这样的话，就是 n!，不知是否可以这样理解 ？




树，总共包含4节内容。具体如下：
1.树、二叉树
2.二叉查找树
3.平衡二叉树、红黑树
4.递归树

一、树
1.树的常用概念
根节点、叶子节点、父节点、子节点、兄弟节点，还有节点的高度、深度以及层数，树的高度。
2.概念解释
节点：树中的每个元素称为节点
父子关系：相邻两节点的连线，称为父子关系
根节点：没有父节点的节点
叶子节点：没有子节点的节点
父节点：指向子节点的节点
子节点：被父节点指向的节点
兄弟节点：具有相同父节点的多个节点称为兄弟节点关系
节点的高度：节点到叶子节点的最长路径所包含的边数
节点的深度：根节点到节点的路径所包含的边数
节点的层数：节点的深度+1（根节点的层数是1）
树的高度：等于根节点的高度
二、二叉树
1.概念
①什么是二叉树？
每个节点最多只有2个子节点的树，这两个节点分别是左子节点和右子节点。
②什么是满二叉树？
有一种二叉树，除了叶子节点外，每个节点都有左右两个子节点，这种二叉树叫做满二叉树。
③什么是完全二叉树？
有一种二叉树，叶子节点都在最底下两层，最后一层叶子节都靠左排列，并且除了最后一层，其他层的节点个数都要达到最大，这种二叉树叫做完全二叉树。
2.完全二叉树的存储
①链式存储
每个节点由3个字段，其中一个存储数据，另外两个是指向左右子节点的指针。我们只要拎住根节点，就可以通过左右子节点的指针，把整棵树都串起来。这种存储方式比较常用，大部分二叉树代码都是通过这种方式实现的。
②顺序存储
用数组来存储，对于完全二叉树，如果节点X存储在数组中的下标为i，那么它的左子节点的存储下标为2*i，右子节点的下标为2*i+1，反过来，下标i/2位置存储的就是该节点的父节点。注意，根节点存储在下标为1的位置。完全二叉树用数组来存储时最省内存的方式。
3.二叉树的遍历
①前序遍历：对于树中的任意节点来说，先打印这个节点，然后再打印它的左子树，最后打印它的右子树。
②中序遍历：对于树中的任意节点来说，先打印它的左子树，然后再打印它的本身，最后打印它的右子树。
③后序遍历：对于树中的任意节点来说，先打印它的左子树，然后再打印它的右子树，最后打印它本身。
前序遍历的递推公式：
preOrder(r) = print r->preOrder(r->left)->preOrder(r->right)
中序遍历的递推公式：
inOrder(r) = inOrder(r->left)->print r->inOrder(r->right)
后序遍历的递推公式：
postOrder(r) = postOrder(r->left)->postOrder(r->right)->print r
时间复杂度：3种遍历方式中，每个节点最多会被访问2次，所以时间复杂度是O(n)。
三、思考
1.二叉树有哪几种存储方式？什么样的二叉树适合用数组来存储？
2.给定一组数据，比如1，3，5，6，9，10.你来算算，可以构建出多少种不同的二叉树？
3.我们讲了三种二叉树的遍历方式，前、中、后序。实际上，还有另一种遍历方式，也就是按层遍历，你知道如何实现吗？
4.如何用循环实现二叉树的遍历？



1、既然是数组了，说明是完全二叉树，应该有n的阶乘个组合。
2、二叉树按层遍历，可以看作以根结点为起点，图的广度优先遍历的问题。




确定两点：
1）n个数，即n个节点，能构造出多少种不同形态的树？
2）n个数，有多少种不同的排列？
当确定以上两点，将【1)的结果】乘以 【2)的结果】，即为最终的结果。
但是有一个注意的点： 如果n中有相等的数，产生的总排列数就不是n！了哟

通过这一题，我学到了【卡塔兰数】：https://en.wikipedia.org/wiki/Catalan_number

层序遍历，借用队列辅助即可，根节点先入队列，然后循环从队列中pop节点，将pop出来的节点的左子节点先入队列，右节点后入队列，依次循环，直到队列为空，遍历结束。



leetcode上有个类似的题目，链接为：https://leetcode.com/problems/binary-tree-level-order-traversal/

Java代码如下：

```Java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 * int val;
 * TreeNode left;
 * TreeNode right;
 * TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        if (root == null) return new ArrayList<>(0);
        
        List<List<Integer>> result = new ArrayList<>();
        
        Queue<TreeNode> queue = new LinkedList<TreeNode>();
        queue.offer(root); 
        
        Queue<TreeNode> curLevelNodes = new LinkedList<TreeNode>();
        
        while (!queue.isEmpty()) {
            TreeNode node = queue.poll();
            curLevelNodes.offer(node);
            
            if (queue.isEmpty()) {
                List<Integer> list = new ArrayList<>(curLevelNodes.size());
                while (!curLevelNodes.isEmpty()) {
                    TreeNode curNode = curLevelNodes.poll();
                    list.add(curNode.val);
                    
                    if (curNode.left != null) {
                        queue.offer(curNode.left); 
                    }
                    
                    if (curNode.right != null) {
                        queue.offer(curNode.right);
                    }
                    
                }
                result.add(list);
            }
        }
        
        
        return result;
    }
    
}

```




```Java
/**
     * 层次遍历二叉树
     * 
     * @param root
     */
    public static void levelOrder(Node root) {
        if (root == null) {
            return;
        }
        LinkedList<Node> queue = new LinkedList<Node>();
        queue.add(root);
        while (!queue.isEmpty()) {
            Node currentNode = queue.poll();
            System.out.print(currentNode.getValue() + " ");
            if (currentNode.getLeft() != null) {
                queue.add(currentNode.getLeft());
            }
            if (currentNode.getRight() != null) {
                queue.add(currentNode.getRight());
            }
        }
    }
```


我看很多人计算第一题都按照完全二叉树计算的，实际上并没有说完全二叉树，所以n阶乘肯定不对吧，只要是二叉树按照文中规则肯定可以按照数组存储，六个数字，前面五个数字最多浪费四个位置加上本身存储五个就是九个位置，然后六可以浪费一个，那就是一共十个位置，六个数字，有多少种放法就有多少种二叉树。





1 递归地理解一下：按住根节点，如果有k个左节点，则有n-k-1个右节点，分步乘法，f(n) = f(k) * f(n - k - 1) ，k可能性从0 到 n - 1 ,分步加法： f(n) = f(0)f(n-1) + ... + f(n-1)f(0) ，怎么计算该递推公式呢？参考Catalon数






老师 你在计算便利二叉树时间复杂度的时候说，“从我前面画的前、中、后序遍历的顺序图，可以看出来，每个节点最多会被访问两次”， 我想知道都是哪两次呢？ 可否帮忙解惑，从图中没看出来
作者回复: 第一次遍历到的时候算一次。递归返回的时候再一次。不过，这些说法都很笼统，你只要知道每个节点都被访问了一次，并且被访问的次数是常数次就可以了。



前、中、后序遍历，主要是针对父节点的打印顺序。父左右为前序，左父右为中序，左右父为后序





老师是否可以在您专栏的github上传一下二叉树这几节的相关代码，还有除了递归遍历二叉树，循环遍历是否也可以讲一下，或者在github上上传一下相关代码，自行研究学习。

作者回复: 非递归遍历比较复杂 不建议非得给自己制造学习难度 除非是为了面试。其他的二叉树的代码我会放到github上




后序遍历节点不是最多被访问三次嘛， 还有那个深度学的深度和层次是一样的哇
作者回复: 1 从图上看是两次
2 从生活中的理解来说 应该没有第0层之说 但是有深度为0的说法



按照层便利使用队列，广度优先搜索




我看评论有人误解 文章所说的 完全二叉树--“最后一层的叶子节点都靠左排列。”然而图例中 I 节点明明是右节点，怎么就被称作完全二叉树？其实刚开始我也理解错了。这里说的 “最后一层的叶子节点都靠左排列”不是最后一层的子节点是左节点，而是指最后一层的子节点，从 左数到右是连续，中间没有断开，缺少节点（如图例H、I、J是连续的）。结合下文所说的基于数组的顺序存储法，可以知道完全二叉树是不会浪费内存的。其实简单理解，完全是为了省内存而提出这样的概念



恕我愚钝。完全二叉树最后一层叶节点都靠左。可是图上节点9是靠右的，是不是我理解有什么问题，请教老师





思考题：
\1. 一组数能构建多少个二叉树？
第一时间想到只要排列位置有改变，那么就应该是新的二叉树。组合排列的公式有点忘记了。。。那么用笨方法：
当只有1个数的时候，能构建1个二叉树；2个数时是2个二叉树；3个数有6个二叉树；再看下4个数，原来是24个；最后得出n!
\2. 层序遍历二叉树：
数组和链表的方式都一样。先打印本身的数据，然后将左右节点塞到一个队列中；从队列里取第一个节点打印数据，并将其左右节点再塞到队列，以此类推。



刚刚思考了完全二叉树的定义 叶子结点必须要在最后两层 如果不在最后两层的话通过数组顺序存储也会浪费空间吧

作者回复: 是的



老师 我想问一下，数组存储的时候跟节点为啥是在下标为1的位置而不是0