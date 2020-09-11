线性排序：如何根据年龄给100万用户数据排序？
===

&emsp;&emsp;三种时间复杂度是 O(n) 的排序算法：桶排序、计数排序、基数排序。因为这些排序算法的时间复杂度是线性的，所以叫作**线性排序**（Linear sort）。这三个算法是非基于比较的排序算法，都不涉及元素之间的比较操作，所以能做到线性的时间复杂度。

&emsp;&emsp;这几种排序算法理解起来都不难，时间、空间复杂度分析起来也很简单，但是对要排序的数据要求很苛刻，所以重点是掌握这些排序算法的适用场景。

&emsp;&emsp;**如何根据年龄给 100 万用户排序？** 归并、快排可以完成功能，但是时间复杂度最低也是 O(nlogn)。

##### 桶排序（Bucket sort）

&emsp;&emsp;桶排序的核心思想是将要排序的数据分到几个有序的桶里，每个桶里的数据再单独进行排序(快排)。桶内排完序之后，再把每个桶里的数据按照顺序依次取出，组成的序列就是有序的了。

<img src="assert/BucketSort.jpg" style="zoom:50%;" />

&emsp;&emsp;如果要排序的数据有 n 个，把它们均匀地划分到 m 个桶内，每个桶里就有 k=n/m 个元素。每个桶内部使用快速排序，时间复杂度为 O($k * logk$)。m 个桶排序的时间复杂度就是 O($m * k * logk$)，因为 k=n/m，所以整个桶排序的时间复杂度就是 O($n*log(n/m)$)。当桶的个数 m 接近数据个数 n 时，log(n/m) 就是一个非常小的常量，这个时候桶排序的时间复杂度接近 O(n)。

&emsp;&emsp;桶排序看起来很优秀，但是不可以替代我们之前讲的排序算法。桶排序对要排序数据的要求是非常苛刻的。

&emsp;&emsp;要排序的数据需要很容易就能划分成 m 个桶，并且，桶与桶之间有着天然的大小顺序。这样每个桶内的数据都排序完之后，桶与桶之间的数据不需要再进行排序。

&emsp;&emsp;数据在各个桶之间的分布是比较均匀的。如果数据经过桶的划分之后，有些桶里的数据非常多，有些非常少，很不平均，那桶内数据排序的时间复杂度就不是常量级了。在极端情况下，如果数据都被划分到一个桶里，那就退化为 O($nlogn$) 的排序算法了。

&emsp;&emsp;**桶排序比较适合用在外部排序中**。比如数据存储在外部磁盘中，数据量比较大，内存有限，无法将数据全部加载到内存中。

##### 计数排序（Counting sort）

&emsp;&emsp;**计数排序其实是桶排序的一种特殊情况**。当要排序的 n 个数据，所处的范围并不大的时候，比如最大值是 k，就可以把数据划分成 k 个桶。每个桶内的数据值都是相同的，省掉了桶内排序的时间。

&emsp;&emsp;假设只有 8 个考生，分数在 0 到 5 分之间。这 8 个考生的成绩我们放在一个数组 A[8] 中，它们分别是：2，5，3，0，2，3，0，3。

&emsp;&emsp;考生的成绩从 0 到 5 分，我们使用大小为 6 的数组 C[6] 表示桶，其中下标对应分数。不过，C[6] 内存储的并不是考生，而是对应的考生个数。我们只需要遍历一遍考生分数，就可以得到 C[6] 的值。

<img src="assert/CountingSorta.jpg" alt="计数排序" style="zoom:50%;" />

&emsp;&emsp;分数为 3 分的考生有 3 个，小于 3 分的考生有 4 个，所以，成绩为 3 分的考生在排序之后的有序数组 R[8] 中，会保存下标 4，5，6 的位置。

<img src="assert/CountingSortb.jpg" alt="计数排序" style="zoom:50%;" />

&emsp;&emsp;快速计算出每个分数的考生在有序数组中对应的存储位置,这个处理方法非常巧妙。对 C[6] 数组顺序求和，C[6] 存储的数据就变成了下面这样子。C[k] 里存储小于等于分数 k 的考生个数。

<img src="assert/CountingSortc.jpg" alt="计数排序" style="zoom:50%;" />

&emsp;&emsp;从后到前依次扫描数组 A。比如，当扫描到 3 时，我们可以从数组 C 中取出下标为 3 的值 7，也就是说，到目前为止，包括自己在内，分数小于等于 3 的考生有 7 个，也就是说 3 是数组 R 中的第 7 个元素（也就是数组 R 中下标为 6 的位置）。当 3 放入到数组 R 中后，小于等于 3 的元素就只剩下了 6 个了，所以相应的 C[3] 要减 1，变成 6。

&emsp;&emsp;以此类推，当我们扫描到第 2 个分数为 3 的考生的时候，就会把它放入数组 R 中的第 6 个元素的位置（也就是下标为 5 的位置）。当我们扫描完整个数组 A 后，数组 R 内的数据就是按照分数从小到大有序排列的了。

~[计数排序](assert/CountingSortd.jpg)

```C
// 计数排序，a 是数组，n 是数组大小。假设数组中存储的都是非负整数。
public void countingSort(int[] a, int n) {
  if (n <= 1) return;
 
  // 查找数组中数据的范围
  int max = a[0];
  for (int i = 1; i < n; ++i) {
    if (max < a[i]) {
      max = a[i];
    }
  }
 
  int[] c = new int[max + 1]; // 申请一个计数数组 c，下标大小 [0,max]
  for (int i = 0; i <= max; ++i) {
    c[i] = 0;
  }
 
  // 计算每个元素的个数，放入 c 中
  for (int i = 0; i < n; ++i) {
    c[a[i]]++;
  }
 
  // 依次累加
  for (int i = 1; i <= max; ++i) {
    c[i] = c[i-1] + c[i];
  }
 
  // 临时数组 r，存储排序之后的结果
  int[] r = new int[n];
  // 计算排序的关键步骤，有点难理解
  for (int i = n - 1; i >= 0; --i) {
    int index = c[a[i]]-1;
    r[index] = a[i];
    c[a[i]]--;
  }
 
  // 将结果拷贝给 a 数组
  for (int i = 0; i < n; ++i) {
    a[i] = r[i];
  }
}
```

&emsp;&emsp;利用另外一个数组来计数，这也是这种排序算法叫计数排序的原因。

##### &emsp;&emsp;计数排序只能用在数据范围不大的场景中，如果数据范围 k 比要排序的数据 n 大很多，就不适合用计数排序了。而且，计数排序只能给非负整数排序，如果要排序的数据是其他类型的，要将其在不改变相对大小的情况下，转化为非负整数。



##### 基数排序（Radix sort）

&emsp;&emsp;假设有 10 万个手机号码，希望将这 10 万个手机号码从小到大排序，有什么比较快速的排序方法呢？

&emsp;&emsp;快排的时间复杂度可以做到 O(nlogn)。手机号码有 11 位，范围太大，显然不适合用桶排序、计数排序这两种排序算法。针对这个排序问题，基数排序时间复杂度是 O(n) 的算法。

&emsp;&emsp;借助稳定排序算法，先按照最后一位来排序手机号码，然后，再按照倒数第二位重新排序，以此类推，最后按照第一位重新排序。经过 11 次排序之后，手机号码就都有序了。

<img src="assert/RadixSortDiagram.jpg" alt="计数排序分解图" style="zoom:50%;" />

&emsp;&emsp;这里按照每位来排序的排序算法要是稳定的，如果是非稳定排序算法，那最后一次排序只会考虑最高位的大小顺序，完全不管其他位的大小关系，那么低位的排序就完全没有意义了。

&emsp;&emsp;根据每一位来排序，可以用桶排序或者计数排序，它们的时间复杂度可以做到 O(n)。如果要排序的数据有 k 位，那我们就需要 k 次桶排序或者计数排序，总的时间复杂度是 O(k$*$n)。当 k 不大的时候，比如手机号码排序的例子，k 最大就是 11，所以基数排序的时间复杂度就近似于 O(n)。

&emsp;&emsp;有时候要排序的数据并不都是等长的，比如我们排序牛津字典中的 20 万个英文单词，最短的只有 1 个字母，最长的有 45 个字母。对于这种不等长的数据，**可以把所有的单词补齐到相同长度，位数不够的可以在后面补“0”**，因为根据ASCII 值，所有字母都大于“0”，所以补“0”不会影响到原有的大小顺序。这样就可以继续用基数排序了。

&emsp;&emsp;**基数排序对要排序的数据是有要求的，需要可以分割出独立的“位”来比较，而且位之间有递进的关系，如果 a 数据的高位比 b 数据大，那剩下的低位就不用比较了。除此之外，每一位的数据范围不能太大，要可以用线性排序算法来排序，否则，基数排序的时间复杂度就无法做到 O(n) 了**。



##### 解答开篇

&emsp;&emsp;根据年龄给 100 万用户排序，就类似按照成绩给 50 万考生排序。假设年龄的范围最小 1 岁，最大不超过 120 岁。可以遍历这 100 万用户，根据年龄将其划分到这 120 个桶里，然后依次顺序遍历这 120 个桶中的元素。这样就得到了按照年龄排序的 100 万用户数据。



##### 内容小结

&emsp;&emsp; 3 种线性时间复杂度的排序算法，桶排序、计数排序、基数排序。它们对要排序的数据都有比较苛刻的要求，应用不是非常广泛。但是如果数据特征比较符合这些排序算法的要求，应用这些算法，会非常高效，线性时间复杂度可以达到 O(n)。

&emsp;&emsp;桶排序和计数排序的排序思想是非常相似的，都是针对范围不大的数据，将数据划分成不同的桶来实现排序。基数排序要求数据可以划分成高低位，位之间有递进关系。比较两个数，我们只需要比较高位，高位相同的再比较低位。而且每一位的数据范围不能太大，因为基数排序算法需要借助桶排序或者计数排序来完成每一个位的排序工作。



##### 课后思考

&emsp;&emsp;实际上有很多看似是排序但又不需要使用排序算法就能处理的排序问题。

&emsp;&emsp;假设我们现在需要对 D，a，F，B，c，A，z 这个字符串进行排序，要求将其中所有小写字母都排在大写字母的前面，但小写字母内部和大写字母内部不要求有序。比如经过排序之后为 a，c，z，D，F，B，A，这个如何来实现呢？如果字符串中存储的不仅有大小写字母，还有数字。要将小写字母的放到前面，大写字母放在最后，数字放在中间，不用排序算法，又该怎么解决呢？




用两个指针a、b：a指针从头开始往后遍历，遇到大写字母就停下，b从后往前遍历，遇到小写字母就停下，交换a、b指针对应的元素；重复如上过程，直到a、b指针相交。
对于小写字母放前面，数字放中间，大写字母放后面，可以先将数据分为小写字母和非小写字母两大类，进行如上交换后再在非小写字母区间内分为数字和大写字母做同样处理

利用桶排序思想，弄小写，大写，数字三个桶，遍历一遍，都放进去，然后再从桶中取出来就行了。相当于遍历了两遍，复杂度O(n)


计数排序中，从数组A中取数，也是可以从头开始取,只不过就不是稳定排序算法了


稳定性，若单个桶内用归并排序，则可保证桶排序的稳定性；若使用快排则无法保证稳定性。
空间复杂度，桶都是额外的存储空间，只有确定了单个桶的大小才能确定空间复杂度；n个元素假设分为m个桶，每个桶分配n/m个元素的大小？个人觉得单个桶的大小不好确定，但是范围应该在n/m~n之间
桶的大小设置的原则 权衡空间 时间复杂度 在你能接受的执行时间和内存占用下完成就可以 并没有一个标准答案
桶大小的设置，让桶的能动扩容 要么用链表 要么用动态扩容的数组

计数排序中可以从头向后取数据但就不是稳定排序算法了

包含数字的话。其实就是一个荷兰国旗问题 思路与第一题类似 一个指针控制左边界 一个指针控制右边界 左边界控制小写字母 右边界控制大写字母 另外一个指针扫描 遇到小写字母跳过 遇到大写字母则将右边界-1的元素交换过来 此时q指针应保持原位置不动 因为右边还未扫描过 交换过来的元素无法保证就是小写字母


以前了解桶排序，以为计数排序就是桶排序的优化，很简单，没想到里面用"顺序求和"快速得出值对应的原排序对象位置(有多个相同值的时候是这个值在排序后的数组中的最后位置，用一次以后减少1)，这样就可以用对象属性来将对象进行排序了

基数排序用了排序算法的稳定性，排多次

如果分为三个桶（大写、小写、数字），那么时间复杂度应该不会达到O(n)，因为O(nlog(n/m))中的m只有3，时间复杂度会退化到O(nlogn)。如果要达到O(n)的复杂度，我认为应该使用计数排序，将A-Za-z0-9作为62个桶，这样遍历一次就可以完成排序。

sort(array,comparator)//对比第11位
sort(array,comparator)//对比第10位
sort(array,comparator)//对比第9位
sort(array,comparator)//对比第8位
sort(array,comparator)//对比第7位
sort(array,comparator)//对比第6位
sort(array,comparator)//对比第5位
sort(array,comparator)//对比第4位
sort(array,comparator)//对比第3位
sort(array,comparator)//对比第2位
sort(array,comparator)//对比第1位
每一位排序用的是O（n）时间复杂度的桶排序或者计数排序

快排划分的思想分成两半，分成三份(双轴)，只不过固定选取一个合适的pivot就ok。

根据acill码的天然顺序，分三个桶就可以把

在计数排序中，第一次得到数组int[] c = new int[]{2,0,2,3,0,1}之后，
能不能直接遍历数组c，
int j=0;
for(int i=0; i<c.length; i++){
    int count = c[i];
    for(int k=0;k<count;k++){
        a[j++] = i;
    }
}
这样不是也对所有的分数进行排序了吗？这个不知道可以不？
作者回复: 如果你排序的不是单纯的数字 而是一个对象呢



1、自问的三个问题只有了时间复杂度分析，少了是否为稳定排序算法和空间复杂度两个问题。
1.1）稳定性，若单个桶内用归并排序，则可保证桶排序的稳定性；若使用快排则无法保证稳定性。
1.2）空间复杂度，桶都是额外的存储空间，只有确定了单个桶的大小才能确定空间复杂度；n个元素假设分为m个桶，每个桶分配n/m个元素的大小？个人觉得单个桶的大小不好确定，但是范围应该在n/m~n之间
2、没有伪代码实现，自己在代码实现中遇到了一些问题
2.1）桶个数的设置依据什么原则？ 权衡空间 时间复杂度 在你能接受的执行时间和内存占用下完成就可以 并没有一个标准答案
2.2）桶大小的设置，让桶的能动扩容？ 要么用链表 要么用动态扩容的数组



包含数字的话。其实就是一个荷兰国旗问题 思路与第一题类似 一个指针控制左边界 一个指针控制右边界 左边界控制小写字母 右边界控制大写字母 另外一个指针扫描 遇到小写字母跳过 遇到大写字母则将右边界-1的元素交换过来 此时q指针应保持原位置不动 因为右边还未扫描过 交换过来的元素无法保证就是小写字母



以前了解桶排序，以为计数排序就是桶排序的优化，很简单，没想到里面用"顺序求和"快速得出值对应的原排序对象位置(有多个相同值的时候是这个值在排序后的数组中的最后位置，用一次以后减少1)，这样就可以用对象属性来将对象进行排序了，这波操作666

基数排序用了排序算法的稳定性，排多次


如果分为三个桶（大写、小写、数字），那么时间复杂度应该不会达到O(n)，因为O(nlog(n/m))中的m只有3，时间复杂度会退化到O(nlogn)。如果要达到O(n)的复杂度，我认为应该使用计数排序，将A-Za-z0-9作为62个桶，这样遍历一次就可以完成排序。（如果上述理解有偏差，请作者务必指出，多谢！）



思考题，快排划分的思想分成两半，分成三份(双轴)，只不过固定选取一个合适的pivot就ok。