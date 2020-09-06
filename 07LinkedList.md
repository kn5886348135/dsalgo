如何轻松写出正确的链表代码
===

&emsp;&emsp;链表反转、有序链表合并等是书写链表代码的难点

#### 链表代码技巧

* **理解指针或引用的含义**
&emsp;&emsp;将某个变量赋值给指针，实际上就是将这个变量的地址赋值给指针，或者反过来说，指针中存储了这个变量的内存地址，指向了这个变量，通过指针就能找到这个变量。

* **警惕指针丢失和内存泄漏**

<img src="assert/LinkedListAdd.jpg" alt="插入节点" style="zoom:50%;" />

&emsp;&emsp;插入结点时，一定要注意操作的顺序，要先将结点 x 的 next 指针指向结点 b，再把结点 a 的 next 指针指向结点 x，这样才不会丢失指针，导致内存泄漏。删除链表结点时，也一定要记得手动释放内存空间。
```C
x->next = p->next;
// 将 x 的结点的 next 指针指向 b 结点；
p->next = x;
// 将 p 的 next 指针指向 x 结点；
```

* ##### 利用哨兵简化实现难度

  &emsp;&emsp;针对链表的插入、删除操作，需要对插入第一个结点和删除最后一个结点的情况进行特殊处理。
```C
// 空链表中插入节点
if (head == null) {
  head = new_node;
}
```
```C
// 删除p的后继节点
p->next = p->next-next;
```
```C
// 删除链表的最后一个节点
if (head->next == null) {
   head = null;
}
```
&emsp;&emsp;引入哨兵结点，在任何时候，不管链表是不是空，head 指针都会一直指向这个哨兵结点。把这种有哨兵结点的链表叫带头链表,没有哨兵结点的链表就叫作不带头链表。哨兵结点是不存储数据的。因为哨兵结点一直存在，所以插入第一个结点和插入其他结点，删除最后一个结点和删除其他结点，都可以统一为相同的代码实现逻辑了。

<img src="assert/LinkedListWithFirstNode.jpg" alt="带头链表" style="zoom:50%;" />

&emsp;&emsp;实际上，这种利用哨兵简化编程难度的技巧，在很多代码实现中都有用到，比如插入排序、归并排序、动态规划等。例如
```C
// 在数组 a 中，查找 key，返回 key 所在的位置
// 其中，n 表示数组 a 的长度
int find(char* a, int n, char key) {
  // 边界条件处理，如果 a 为空，或者 n<=0，说明数组中没有数据，就不用 while 循环比较了
  if(a == null || n <= 0) {
    return -1;
  }
  
  int i = 0;
  // 这里有两个比较操作：i<n 和 a[i]==key.
  while (i < n) {
    if (a[i] == key) {
      return i;
    }
    ++i;
  }
  
  return -1;
}
```
```C
// 在数组 a 中，查找 key，返回 key 所在的位置
// 其中，n 表示数组 a 的长度
// 我举 2 个例子，你可以拿例子走一下代码
// a = {4, 2, 3, 5, 9, 6}  n=6 key = 7
// a = {4, 2, 3, 5, 9, 6}  n=6 key = 6
int find(char* a, int n, char key) {
  if(a == null || n <= 0) {
    return -1;
  }
  
  // 这里因为要将 a[n-1] 的值替换成 key，所以要特殊处理这个值
  if (a[n-1] == key) {
    return n-1;
  }
  
  // 把 a[n-1] 的值临时保存在变量 tmp 中，以便之后恢复。tmp=6。
  // 之所以这样做的目的是：希望 find() 代码不要改变 a 数组中的内容
  char tmp = a[n-1];
  // 把 key 的值放到 a[n-1] 中，此时 a = {4, 2, 3, 5, 9, 7}
  a[n-1] = key;
  
  int i = 0;
  // while 循环比起代码一，少了 i<n 这个比较操作
  while (a[i] != key) {
    ++i;
  }
  
  // 恢复 a[n-1] 原来的值, 此时 a= {4, 2, 3, 5, 9, 6}
  a[n-1] = tmp;
  
  if (i == n-1) {
    // 如果 i == n-1 说明，在 0...n-2 之间都没有 key，所以返回 -1
    return -1;
  } else {
    // 否则，返回 i，就是等于 key 值的元素的下标
    return i;
  }
}
```
&emsp;&emsp;第二段代码中，我们通过一个哨兵 a[n-1] = key，成功省掉了一个比较语句 i &lt; n。当累积执行万次、几十万次时，累积的时间就很明显了。因为a[n-1] == key，所以并不会发生数组越界的场景。

* **重点留意边界条件处理**

&emsp;&emsp;软件开发中，代码在一些边界或者异常情况下，最容易产生 Bug。链表代码也不例外。要实现没有 Bug 的链表代码，一定要在编写的过程中以及编写完成之后，检查边界条件是否考虑全面，以及代码在边界条件下是否能正确运行。检查链表代码是否正确的边界条件有这样几个：


        * 如果链表为空时，代码是否能正常工作？
        * 如果链表只包含一个结点时，代码是否能正常工作？
        * 如果链表只包含两个结点时，代码是否能正常工作？
        * 代码逻辑在处理头结点和尾结点的时候，是否能正常工作？

&emsp;&emsp;针对不同的场景，可能还有特定的边界条件。写任何代码时，都要考虑业务正常情况下的功能和边界情况或者异常情况。

* **举例画图，辅助思考**
&emsp;&emsp;对于稍微复杂的链表操作，可以举例法和画图法。

* **多写多练，没有捷径**
&emsp;&emsp;可以多练习常见的链表操作(LeetCode对应编号：206，141，21，19，876)
    * 单链表反转
    * 链表中环的检测
    * 两个有序的链表合并
    * 删除链表倒数第 n 个结点
    * 求链表的中间结点

环的检测









 





可不可以用链表将数组链接起来？也就是说链表里每个node存储了数组指针，这样每增加一个节点就可以多存放很多元素。如果可以的话，与直接使用动态数组或者直接使用链表比有没有什么优缺点，为何在网上搜索几乎找不到人这样用？

内存池










/**
public class Node {
  public char c;
  public Node next;

  public Node(char c) {
    this.c = c;
  }
}
**/

public static Node reverse(Node head) {
    if(head == null || head.next == null) {
      return head;
    }
    
```
Node prev = null;
Node cur = head;
Node next = head.next;

while(next != null) {
  cur.next = prev;
  prev = cur;
  cur = next;
  next = cur.next;
}
cur.next = prev;
return cur;
```
  }


  public static boolean existsCircle(Node head) {    
    Node slow = head;
    Node fast = head;  
    while(fast != null && fast.next != null) {
      slow = slow.next;
      fast = fast.next.next;    
      if(slow == fast) {
        return true;
      }
    }    
    return false;
  }

```
public static Node merge(Node head1, Node head2) {

Node guard = new Node('/');
Node cur = guard;

while(head1 != null && head2 != null) {
  if(head1.c <= head2.c) {
    while(head1 != null && head1.c <= head2.c) {
      cur.next = head1;
      cur = cur.next;
      head1 = head1.next;
      
    }
  } else {
    while(head2 != null && head1.c > head2.c) {
      cur.next = head2;
      cur = cur.next;
      head2 = head2.next;
      
    }
  }
}

if(head1 != null) {
  cur.next = head1;
} 
if(head2 != null) {
  cur.next = head2;
}

return guard.next;
```

  }

  public static Node deleteLastN(Node head, int n) {
    if(n < 1 || head == null) {
      return head;
    }
    Node guard = new Node('/');
    guard.next = head;
    
```
Node slow = guard;
Node fast = guard;

for(int i = 0; i < n; i++) {
  if(fast != null) {
    fast = fast.next;
  }
}
while(fast != null && fast.next != null) {
  slow = slow.next;
  fast = fast.next;
}
slow.next = slow.next.next;
return guard.next;
```
  }

  public static Node getMiddle(Node head, int n) {
    Node slow = head;
    Node fast = head;
    
```
while(fast.next != null && fast.next.next != null) {
  slow = slow.next;
  fast = fast.next.next;
}

return slow;
```
  }





1.三个节点p.pre，p，p.next，将p的next指针指向p.pre，然后p.pre=p，p=p.next，p.next=p.next.next移动指针，就可以实现单链表反转。
2.最简单就是一个节点在头，一个节点一直遍历，地址相等就是环，不过好像还有一种简单的办法，快慢前进，一次就能搞定。这个老师能不能说下自己的思路，我有点想不明白。
3.建立第三个链表，每次比较a链表当前节点和b链表当前节点的大小。如果a比b小，则c的next指针指向a当前节点，c=c.next，然后a指针后移。接着继续比较a.b当前节点大小，反之则把a换成b就行了。
4.一个p节点，然后找到距离p有n个next节点的点，一起往后遍历，到pn.next为空的时候，p就是我们要求的那个地址。
5.快慢指针，一个每次前进2个节点一个每次前进1节点。前进两个节点到表尾的时候，前进一个的就是中间点。





C语言，二级指针可以绕过不带头结点链表删除操作的边界检查。









1、 函数中需要移动链表时，最好新建一个指针来移动，以免更改原始指针位置。

2、 单链表有带头节点和不带头结点的链表之分，一般做题默认头结点是有值的。

3、 链表的内存时不连续的，一个节点占一块内存，每块内存中有一块位置（next）存放下一节点的地址（这是单链表为例）。

3、 链表中找环的思想：创建两个指针一个快指针一次走两步一个慢指针一次走一步，若相遇则有环，若先指向nullptr则无环。

4、 链表找倒数第k个节点思想：创建两个指针，第一个先走k-1步然后两个在一同走。第一个走到最后时则第二个指针指向倒数第k位置。

5、 反向链表思想：从前往后将每个节点的指针反向，即.next内的地址换成前一个节点的，但为了防止后面链表的丢失，在每次换之前需要先创建个指针指向下一个节点。

6、 两个有序链表合并思想：这里用到递归思想。先判断是否有一个链表是空链表，是则返回两一个链表，免得指针指向不知名区域引发程序崩溃。然后每次比较两个链表的头结点，小的值做新链表的头结点，此节点的next指针指向本函数（递归开始，参数是较小值所在链表.next和另一个链表）。



算法设计思路应该是
// 用来找出给定key在数组中的下标，找不到则返回-1

a是被遍历的数组
n是数组长度
key是要寻找的值

1， 判断尾节点是不是要寻找的值，是的话返回n-1，因为数组下标从0开始所以要长度-1才是下标

2， 使用哨兵变量保存尾节点

3， 把key放到尾节点，让key成为数组中最后一个值，这样做是为了下一步的遍历

4， 开始从头开始遍历数组，i为数组下标，如果找到与key相等的元素则退出遍历，否则遍历整个数组

5， 如果i是尾节点下标，说明没有找到key，如果不是则i为寻找的节点下标，返回i

6， 把哨兵变量还原赋值到数组尾节点，也就是还原数组

也就是说平时用的临时变量就是哨兵变量
