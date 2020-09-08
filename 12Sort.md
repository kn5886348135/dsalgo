快排思想在O(n)内查找第K大元素
===

&emsp;&emsp;冒泡排序、插入排序、选择排序这三种排序算法，它们的时间复杂度都是 O(n^2^)，比较高，适合小规模数据的排序。归并排序和快速排序的时间复杂度为 O(n$log{n}$) ，适合大规模的数据排序，比前面那三种排序算法要更常用。

&emsp;&emsp;归并排序和快速排序都用到了分治思想，非常巧妙。我们可以借鉴这个思想，来解决非排序的问题，比如：如何在 O(n) 的时间复杂度内查找一个无序数组中的第 K 大元素？ 

##### 归并排序（Merge Sort）的原理

&emsp;&emsp;归并排序的核心思想还是蛮简单的。如果要排序一个数组，我们先把数组从中间分成前后两部分，然后对前后两部分分别排序，再将排好序的两部分合并在一起，这样整个数组就都有序了。

<img src="assert/MergeSortDiagram.jpg" alt="归并排序分解图" style="zoom:50%;" />

&emsp;&emsp;归并排序使用的就是**分治思想**,可以用递归来实现。分治，顾名思义，就是分而治之，将一个大问题分解成小的子问题来解决。小的子问题解决了，大问题也就解决了。

&emsp;&emsp;分治思想跟递归思想很像，分治算法一般都是用递归来实现的。**分治是一种解决问题的处理思想，递归是一种编程技巧**。

&emsp;&emsp;写递归代码的技巧就是，分析得出递推公式，然后找到终止条件，最后将递推公式翻译成递归代码。

```C
// 递推公式：
merge_sort(p…r) = merge(merge_sort(p…q), merge_sort(q+1…r))

// 终止条件：
p >= r 不用再继续分解
```

&emsp;&emsp;merge_sort(p…r) 表示，给下标从 p 到 r 之间的数组排序。我们将这个排序问题转化为了两个子问题，merge_sort(p…q) 和 merge_sort(q+1…r)，其中下标 q 等于 p 和 r 的中间位置，也就是 (p+r)/2。当下标从 p 到 q 和从 q+1 到 r 这两个子数组都排好序之后，我们再将两个有序的子数组合并在一起，这样下标从 p 到 r 之间的数据就也排好序了。

```C
// 伪代码
// 归并排序算法, A 是数组，n 表示数组大小
merge_sort(A, n) {
  merge_sort_c(A, 0, n-1)
}

// 递归调用函数
merge_sort_c(A, p, r) {
  // 递归终止条件
  if p >= r  then return

  // 取 p 到 r 之间的中间位置 q
  q = (p+r) / 2
  // 分治递归
  merge_sort_c(A, p, q)
  merge_sort_c(A, q+1, r)
  // 将 A[p...q] 和 A[q+1...r] 合并为 A[p...r]
  merge(A[p...r], A[p...q], A[q+1...r])
}
```

&emsp;&emsp;merge(A[p…r], A[p…q], A[q+1…r]) 这个函数的作用就是，将已经有序的 A[p…q] 和 A[q+1…r] 合并成一个有序的数组，并且放入 A[p…r]。

&emsp;&emsp;我们申请一个临时数组 tmp，大小与 A[p…r] 相同。我们用两个游标 i 和 j，分别指向 A[p…q] 和 A[q+1…r] 的第一个元素。比较这两个元素 A[i] 和 A[j]，如果 A[i]<=A[j]，我们就把 A[i] 放入到临时数组 tmp，并且 i 后移一位，否则将 A[j] 放入到数组 tmp，j 后移一位。直到其中一个子数组中的所有数据都放入临时数组中，再把另一个数组中的数据依次加入到临时数组的末尾，这个时候，临时数组中存储的就是两个子数组合并之后的结果了。最后再把临时数组 tmp 中的数据拷贝到原数组 A[p…r] 中。

<img src="assert/MergeOfMergeSort.jpg" alt="归并排序的合并过程" style="zoom:50%;" />

```Go
merge(A[p...r], A[p...q], A[q+1...r]) {
  var i := p，j := q+1，k := 0 // 初始化变量 i, j, k
  var tmp := new array[0...r-p] // 申请一个大小跟 A[p...r] 一样的临时数组
  while i<=q AND j<=r do {
    if A[i] <= A[j] {
      tmp[k++] = A[i++] // i++ 等于 i:=i+1
    } else {
      tmp[k++] = A[j++]
    }
  }
  
  // 判断哪个子数组中有剩余的数据
  var start := i，end := q
  if j<=r then start := j, end:=r
  
  // 将剩余的数据拷贝到临时数组 tmp
  while start <= end do {
    tmp[k++] = A[start++]
  }
  
  // 将 tmp 中的数组拷贝回 A[p...r]
  for i:=0 to r-p do {
    A[p+i] = tmp[i]
  }
}
```
&emsp;&emsp;merge() 合并函数如果借助哨兵，代码就会简洁很多。

###归并排序的性能分析

归并排序的三个问题。

1. 归并排序是稳定的排序算法
    归并排序稳不稳定关键要看 merge() 函数，也就是两个有序子数组合并成一个有序数组的那部分代码。在合并的过程中，如果 A[p…q] 和 A[q+1…r] 之间有值相同的元素，那我们可以像伪代码中那样，先把 A[p…q] 中的元素放入 tmp 数组。这样就保证了值相同的元素，在合并前后的先后顺序不变。所以，归并排序是一个稳定的排序算法。

2. 归并排序的时间复杂度是多少？
    如果我们定义求解问题 a 的时间是 T(a)，求解问题 b、c 的时间分别是 T(b) 和 T( c)，那我们就可以得到这样的递推关系式：
    T(a) = T(b) $+$ T(c) $+$ K
    其中 K 等于将两个子问题 b、c 的结果合并成问题 a 的结果所消耗的时间。
    **不仅递归求解的问题可以写成递推公式，递归代码的时间复杂度也可以写成递推公式。**
    我们假设对 n 个元素进行归并排序需要的时间是 T(n)，那分解成两个子数组排序的时间都是 T(n/2)。我们知道，merge() 函数合并两个有序子数组的时间复杂度是 O(n)。所以，套用前面的公式，归并排序的时间复杂度的计算公式就是：
    ```
    T(1) = C；   n=1 时，只需要常量级的执行时间，所以表示为 C。
    T(n) = 2*T(n/2) + n； n>1
    ```
    ```
    T(n) = 2*T(n/2) + n
        = 2*(2*T(n/4) + n/2) + n = 4*T(n/4) + 2*n
        = 4*(2*T(n/8) + n/4) + 2*n = 8*T(n/8) + 3*n
        = 8*(2*T(n/16) + n/8) + 3*n = 16*T(n/16) + 4*n
        ......
        = 2^k * T(n/2^k) + k * n
        ......
    ```
    通过这样一步一步分解推导，我们可以得到 T(n) = 2^k^$*$T(n/2^k^)+kn。当 T(n/2^k^)=T(1) 时，也就是 n$/$2^k^=1，我们得到 k=log2n 。我们将 k 值代入上面的公式，得到 T(n)=Cn+nlog2n 。如果我们用大 O 标记法来表示的话，T(n) 就等于 O(nlogn)。所以归并排序的时间复杂度是 O(nlogn)。

&emsp;&emsp;归并排序的执行效率与要排序的原始数组的有序程度无关，所以其时间复杂度是非常稳定的，不管是最好情况、最坏情况，还是平均情况，时间复杂度都是 O(nlogn)。

3. 归并排序的空间复杂度是多少？
    归并排序的时间复杂度任何情况下都是 O(nlogn)，看起来非常优秀。（即便是快速排序，最坏情况下，时间复杂度也是 O(n^2^)。）但是，归并排序并没有像快排那样，应用广泛，这是为什么呢？因为它有一个致命的“弱点”，那就是归并排序不是原地排序算法。
    这是因为归并排序的合并函数，在合并两个有序数组为一个有序数组时，需要借助额外的存储空间。
    递归代码的空间复杂度并不能像时间复杂度那样累加。刚刚我们忘记了最重要的一点，那就是，尽管每次合并操作都需要申请额外的内存空间，但在合并完成之后，临时开辟的内存空间就被释放掉了。在任意时刻，CPU 只会有一个函数在执行，也就只会有一个临时的内存空间在使用。临时内存空间最大也不会超过 n 个数据的大小，所以空间复杂度是 O(n)。

###快速排序的原理（QuickSort）

&emsp;&emsp;我们习惯性把它简称为“快排”。快排利用的也是分治思想。它有点像归并排序，但是思路其实完全不一样。如果要排序数组中下标从 p 到 r 之间的一组数据，我们选择 p 到 r 之间的任意一个数据作为 pivot（分区点）。

&emsp;&emsp;我们遍历 p 到 r 之间的数据，将小于 pivot 的放到左边，将大于 pivot 的放到右边，将 pivot 放到中间。经过这一步骤之后，数组 p 到 r 之间的数据就被分成了三个部分，前面 p 到 q-1 之间都是小于 pivot 的，中间是 pivot，后面的 q+1 到 r 之间是大于 pivot 的。

![快速排序思想](assert/QuickSort.jpg)

&emsp;&emsp;根据分治、递归的处理思想，我们可以用递归排序下标从 p 到 q-1 之间的数据和下标从 q+1 到 r 之间的数据，直到区间缩小为 1，就说明所有的数据都有序了。

```
递推公式：
quick_sort(p…r) = quick_sort(p…q-1) + quick_sort(q+1, r)

终止条件：
p >= r
```
```C
// 快速排序，A 是数组，n 表示数组的大小
quick_sort(A, n) {
  quick_sort_c(A, 0, n-1)
}
// 快速排序递归函数，p,r 为下标
quick_sort_c(A, p, r) {
  if p >= r then return

  q = partition(A, p, r) // 获取分区点
  quick_sort_c(A, p, q-1)
  quick_sort_c(A, q+1, r)
}
```
&emsp;&emsp;归并排序中有一个 merge() 合并函数，我们这里有一个 partition() 分区函数。partition() 分区函数实际上我们前面已经讲过了，就是随机选择一个元素作为 pivot（一般情况下，可以选择 p 到 r 区间的最后一个元素），然后对 A[p…r] 分区，函数返回 pivot 的下标。

&emsp;&emsp;如果我们不考虑空间消耗的话，partition() 分区函数可以写得非常简单。我们申请两个临时数组 X 和 Y，遍历 A[p…r]，将小于 pivot 的元素都拷贝到临时数组 X，将大于 pivot 的元素都拷贝到临时数组 Y，最后再将数组 X 和数组 Y 中数据顺序拷贝到 A[p…r]。

![快排分区](assert/PartitionOfQuickSort.jpg)

&emsp;&emsp;但是，如果按照这种思路实现的话，partition() 函数就需要很多额外的内存空间，所以快排就不是原地排序算法了。如果我们希望快排是原地排序算法，那它的空间复杂度得是 O(1)，那 partition() 分区函数就不能占用太多额外的内存空间，我们就需要在 A[p…r] 的原地完成分区操作。
```Go
partition(A, p, r) {
  pivot := A[r]
  i := p
  for j := p to r-1 do {
    if A[j] < pivot {
      swap A[i] with A[j]
      i := i+1
    }
  }
  swap A[i] with A[r]
  return i
```
&emsp;&emsp;这里的处理有点类似选择排序。我们通过游标 i 把 A[p…r-1] 分成两部分。A[p…i-1] 的元素都是小于 pivot 的，我们暂且叫它“已处理区间”，A[i…r-1] 是“未处理区间”。我们每次都从未处理的区间 A[i…r-1] 中取一个元素 A[j]，与 pivot 对比，如果小于 pivot，则将其加入到已处理区间的尾部，也就是 A[i] 的位置。

&emsp;&emsp;在数组某个位置插入元素，需要搬移数据，非常耗时。当时我们也讲了一种处理技巧，就是交换，在 O(1) 的时间复杂度内完成插入操作。这里我们也借助这个思想，只需要将 A[i] 与 A[j] 交换，就可以在 O(1) 时间复杂度内将 A[j] 放到下标为 i 的位置。

![快排中的交换](assert/SwapInQuickSort.jpg)

&emsp;&emsp;因为分区的过程涉及交换操作，如果数组中有两个相同的元素，比如序列 6，8，7，6，3，5，9，4，在经过第一次分区操作之后，两个 6 的相对先后顺序就会改变。所以，快速排序并不是一个稳定的排序算法。

&emsp;&emsp;快排和归并用的都是分治思想，递推公式和递归代码也非常相似，那它们的区别在哪里呢？

![快排和归并排序的区别](assert/DifferenceBetweenMergeSortAndQuickSort.jpg)

&emsp;&emsp;归并排序的处理过程是**由下到上**的，先处理子问题，然后再合并。而快排正好相反，它的处理过程是**由上到下**的，先分区，然后再处理子问题。归并排序虽然是稳定的、时间复杂度为 O(nlogn) 的排序算法，但是它是非原地排序算法。归并之所以是非原地排序算法，主要原因是合并函数无法在原地执行。快速排序通过设计巧妙的原地分区函数，可以实现原地排序，解决了归并排序占用太多内存的问题。

###快速排序的性能分析

&emsp;&emsp;快排是一种原地、不稳定的排序算法。现在，我们集中精力来看快排的时间复杂度。

&emsp;&emsp;快排也是用递归来实现的。对于递归代码的时间复杂度，我前面总结的公式，这里也还是适用的。如果每次分区操作，都能正好把数组分成大小接近相等的两个小区间，那快排的时间复杂度递推求解公式跟归并是相同的。所以，快排的时间复杂度也是 O(nlogn)。

```
T(1) = C；   n=1 时，只需要常量级的执行时间，所以表示为 C。
T(n) = 2*T(n/2) + n； n>1
```

&emsp;&emsp;但是，公式成立的前提是每次分区操作，我们选择的 pivot 都很合适，正好能将大区间对等地一分为二。但实际上这种情况是很难实现的。

&emsp;&emsp;举一个比较极端的例子。如果数组中的数据原来已经是有序的了，比如 1，3，5，6，8。如果我们每次选择最后一个元素作为 pivot，那每次分区得到的两个区间都是不均等的。我们需要进行大约 n 次分区操作，才能完成快排的整个过程。每次分区我们平均要扫描大约 n/2 个元素，这种情况下，快排的时间复杂度就从 O(nlogn) 退化成了 O(n2)。

&emsp;&emsp;我们刚刚讲了两个极端情况下的时间复杂度，一个是分区极其均衡，一个是分区极其不均衡。它们分别对应快排的最好情况时间复杂度和最坏情况时间复杂度。那快排的平均情况时间复杂度是多少呢？

&emsp;&emsp;我们假设每次分区操作都将区间分成大小为 9:1 的两个小区间。我们继续套用递归时间复杂度的递推公式，就会变成这样：
```
T(1) = C；   n=1 时，只需要常量级的执行时间，所以表示为 C。

T(n) = T(n/10) + T(9*n/10) + n； n>1
```
&emsp;&emsp;这个公式的递推求解的过程非常复杂，虽然可以求解，但我不推荐用这种方法。实际上，递归的时间复杂度的求解方法除了递推公式之外，还有递归树。我这里直接给你结论：T(n) 在大部分情况下的时间复杂度都可以做到 O(nlogn)，只有在极端情况下，才会退化到 O(n2)。而且，我们也有很多方法将这个概率降到很低。

###解答开篇

&emsp;&emsp;快排核心思想就是**分治**和**分区**，我们可以利用分区的思想，来解答开篇的问题：O(n) 时间复杂度内求无序数组中的第 K 大元素。比如，4， 2， 5， 12， 3 这样一组数据，第 3 大元素就是 4。

&emsp;&emsp;我们选择数组区间 A[0…n-1] 的最后一个元素 A[n-1] 作为 pivot，对数组 A[0…n-1] 原地分区，这样数组就分成了三部分，A[0…p-1]、A[p]、A[p+1…n-1]。

&emsp;&emsp;如果 p+1=K，那 A[p] 就是要求解的元素；如果 K$>$p+1, 说明第 K 大元素出现在 A[p+1…n-1] 区间，我们再按照上面的思路递归地在 A[p+1…n-1] 这个区间内查找。同理，如果 K$<$p+1，那我们就在 A[0…p-1] 区间查找。

![找到第K大的元素](assert/FindKth.jpg)

&emsp;&emsp;第一次分区查找，我们需要对大小为 n 的数组执行分区操作，需要遍历 n 个元素。第二次分区查找，我们只需要对大小为 n/2 的数组执行分区操作，需要遍历 n/2 个元素。依次类推，分区遍历元素的个数分别为、n/2、n/4、n/8、n/16.……直到区间缩小为 1。

&emsp;&emsp;如果我们把每次分区遍历的元素个数加起来，就是：n+n/2+n/4+n/8+…+1。这是一个等比数列求和，最后的和等于 2n-1。所以，上述解决思路的时间复杂度就为 O(n)。

&emsp;&emsp;你可能会说，我有个很笨的办法，每次取数组中的最小值，将其移动到数组的最前面，然后在剩下的数组中继续找最小值，以此类推，执行 K 次，找到的数据不就是第 K 大元素了吗？

&emsp;&emsp;不过，时间复杂度就并不是 O(n) 了，而是 O(K * n)。你可能会说，时间复杂度前面的系数不是可以忽略吗？O(K * n) 不就等于 O(n) 吗？

&emsp;&emsp;这个可不能这么简单地划等号。当 K 是比较小的常量时，比如 1、2，那最好时间复杂度确实是 O(n)；但当 K 等于 n/2 或者 n 时，这种最坏情况下的时间复杂度就是 O(n2) 了。

###小结

&emsp;&emsp;归并排序和快速排序是两种稍微复杂的排序算法，它们用的都是分治的思想，代码都通过递归来实现，过程非常相似。理解归并排序的重点是理解递推公式和 merge() 合并函数。同理，理解快排的重点也是理解递推公式，还有 partition() 分区函数。

&emsp;&emsp;归并排序算法是一种在任何情况下时间复杂度都比较稳定的排序算法，这也使它存在致命的缺点，即归并排序不是原地排序算法，空间复杂度比较高，是 O(n)。正因为此，它也没有快排应用广泛。

&emsp;&emsp;快速排序算法虽然最坏情况下的时间复杂度是 O(n^2^)，但是平均情况下时间复杂度都是 O(nlogn)。不仅如此，快速排序算法时间复杂度退化到 O(n2) 的概率非常小，我们可以通过合理地选择 pivot 来避免这种情况。



##### 课后思考

&emsp;&emsp;现在你有 10 个接口访问日志文件，每个日志文件大小约 300MB，每个文件里的日志都是按照时间戳从小到大排序的。你希望将这 10 个较小的日志文件，合并为 1 个日志文件，合并之后的日志仍然按照时间戳从小到大排列。如果处理上述排序任务的机器内存只有 1GB，你有什么好的解决思路，能“快速”地将这 10 个日志文件合并吗？

//  每次从各个文件中取一条数据，在内存中根据数据时间戳构建一个最小堆，然后每次把最小值给写入新文件，同时将最小值来自的那个文件再出来一个数据，加入到最小堆中。这个空间复杂度为常数，但没能很好利用1g内存，而且磁盘单个读取比较慢，所以考虑每次读取一批数据，没了再从磁盘中取，时间复杂度还//是一样O(n)。

#### 先构建十条io流，分别指向十个文件，每条io流读取对应文件的第一条数据，然后比较时间戳，选择出时间戳最小的那条数据，将其写入一个新的文件，然后指向该时间戳的io流读取下一行数据，然后继续刚才的操作，比较选出最小的时间戳数据，写入新文件，io流读取下一行数据，以此类推，完成文件的合并， 这种处理方式，日志文件有n个数据就要比较n次，每次比较选出一条数据来写入，时间复杂度是O（n），空间复杂度是O（1）,几乎不占用内存





先取得十个文件时间戳的最小值数组的最小值a，和最大值数组的最大值b。然后取mid=(a+b)/2，然后把每个文件按照mid分割，取所有前面部分之和，如果小于1g就可以读入内存快排生成中间文件，否则继续取时间戳的中间值分割文件，直到区间内文件之和小于1g。同理对所有区间都做同样处理。最终把生成的中间文件按照分割的时间区间的次序直接连起来即可。

1.申请10个40M的数组和一个400M的数组。
2.每个文件都读40M，取各数组中最大时间戳中的最小值。
3.然后利用二分查找，在其他数组中快速定位到小于/等于该时间戳的位置，并做标记。
4.再把各数组中标记位置之前的数据全部放在申请的400M内存中，
5.在原来的40M数组中清除已参加排序的数据。[可优化成不挪动数据，只是用两个索引标记有效数据的起始和截止位置]
6.对400M内存中的有效数据[没装满]做快排。
将排好序的直接写文件。
7.再把每个数组尽量填充满。从第2步开始继续，知道各个文件都读区完毕。
这么做的好处有：
1.每个文件的内容只读区一次，且是批量读区。比每次只取一条快得多。
2.充分利用了读区到内存中的数据。曹源 同学在文件中查找那个中间数是会比较困难的。
3.每个拷贝到400M大数组中参加快排的数据都被写到了文件中，这样每个数只参加了一次快排。


合并函数借助哨兵简化方法
传入的后两个数组各在尾部多放一个和原有最后值相同的值。
循环改为：
while i<=q or j<=r do{
    if A[i] <= A[j] and i<=q {
        tmp[k++] = A[i++]
    }
    else{
        tmp[k++] = A[j++]
    }
}
可以在while循环里完成两个数组的清空，不需要专用部分完成。


王道考研书上看到的快排算法，利用哨兵减少了交换两个元素的复杂步骤，效果更好一些
private static void quickSort(int[] a, int head, int tail) {

        int low = head;
        int high = tail;
        int pivot = a[low];
        if (low < high) {
    
            while (low<high) {
                while (low < high && pivot <= a[high]) high--;
                a[low] = a[high];
                while (low < high && pivot >= a[low]) low++;
                a[high]=a[low];
            }
            a[low] = pivot;
    
            if(low>head+1) quickSort(a,head,low-1);
            if(high<tail-1) quickSort(a,high+1,tail);
        }
    
    }










可以为每个文件分配一个40M的数组，再另外分配一个400M的数组储存归并结果，每个文件每次读取40M，对十个数组做归并排序直到其中某个数组的数据被处理完，这时将归并结果写入磁盘，处理完的数组继续读入40M继续参与归并，以此类推，直到所有文件都处理完


'
partition(A, p, r) {
 pivot := A[r]
 i := p
 for j := p to r-1 do {
  if A[j] < pivot {
   swap A[i] with A[j]
   i := i+1
  }
 }
 swap A[i] with A[r]
 return i
'

' for j := p to r-1 do '

中的 r-1 应该是 r





以前写快排的时候总是喜欢用第一个元素作为k值，partion喜欢用找到左右不符合规则的元素再对调，第k大总是纠结于，下次递归应该是第几大，😂这些其实只要把最后一个元素作为key就可以简化代码了。



我用python实现了归并排序和快速排序，代码如下：
归并排序：https://github.com/lipeng1991/testdemo/blob/master/45_merge_sort.py
快速排序： https://github.com/lipeng1991/testdemo/blob/master/23_quick_sort.py