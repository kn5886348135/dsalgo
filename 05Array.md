### 05 | 数组：为什么很多编程语言中数组都从0开始编号？

为什么数组要从 0 开始编号，而不是从 1 开始呢？ 

利用数组随机访问内存的时候，从0开始可以减少一次CPU的计算

历史原因，编程语言沿用C语言的习惯。



#### 随机访问？

数组（Array）是一种线性表数据结构。它用一组连续的内存空间，来存储一组具有相同类型的数据。



##### 线性表（Linear List）

顾名思义，线性表就是数据排成像一条线一样的结构。每个线性表上的数据最多只有前和后两个方向。其实除了数组，链表、队列、栈等也是线性表结构。

![线性表](assert/LinearList.jpg)

而与它相对立的概念是非线性表，比如二叉树、堆、图等。之所以叫非线性，是因为，在非线性表中，数据之间并不是简单的前后关系。

![非线性表](assert/NonLinearList.jpg)

##### 连续的内存空间和相同类型的数据。

正是因为这两个限制，它才有了一个堪称“杀手锏”的特性：“随机访问”。这两个限制也让数组的很多操作变得非常低效，比如要想在数组中删除、插入一个数据，为了保证连续性，就需要做大量的数据搬移工作。



以一个长度为 10 的 int 类型的数组 int[] a = new int[10] 来举例。计算机给数组 a[10]，分配了一块连续内存空间 1000～1039，其中，内存块的首地址为 base_address = 1000。

![数组随机访问](assert/RandomAccessOfArray.jpg)

计算机会给每个内存单元分配一个地址，计算机通过地址来访问内存中的数据。
```Java
a[i]_address = base_address + i * data_type_size
```
其中 data_type_size 表示数组中每个元素的大小。数组中存储的是 int 类型数据，所以 data_type_size 就为 4 个字节。



数组是适合查找操作，但是查找的时间复杂度并不为 O(1)。即便是排好序的数组，你用二分查找，时间复杂度也是 O(logn)。

数组支持随机访问，根据下标随机访问的时间复杂度为 O(1)。



数组为了保持内存数据的连续性，会导致插入、删除这两个操作比较低效。

##### 插入操作

假设数组的长度为 n，将一个数据插入到数组中的第 k 个位置。最好时间复杂度为 O(1)，最坏时间复杂度是 O(n)， 平均情况时间复杂度为 (1+2+…n)/n=O(n)。

如果数组中的数据是有序的，在某个位置插入一个新的元素时，就必须按照刚才的方法搬移 k 之后的数据。如果数组中存储的数据并没有任何规律，数组只是被当作一个存储数据的集合。在这种情况下，如果要将某个数组插入到第 k 个位置，为了避免大规模的数据搬移，直接将第 k 位的数据搬移到数组元素的最后，把新的元素直接放入第 k 个位置。

##### 删除操作

如果要删除第 k 个位置的数据，为了内存的连续性，也需要搬移数据，不然中间就会出现空洞，内存就不连续了。

最好情况时间复杂度为 O(1)，最坏情况时间复杂度为 O(n)，平均情况时间复杂度也为 O(n)。

在某些特殊场景下，并不一定非得追求数组中数据的连续性。如果将多次删除操作集中在一起执行，删除的效率会提高很多。

先记录下已经删除的数据。每次的删除操作并不是真正地搬移数据，只是记录数据已经被删除。当数组没有更多空间存储数据时，再触发执行一次真正的删除操作，这样就大大减少了删除操作导致的数据搬移。

这就是 JVM 标记清除垃圾回收算法的核心思想吗。



##### 数组的访问越界问题


```Java
int main(int argc, char* argv[]){
    int i = 0;
    int arr[3] = {0};
    for(; i<=3; i++){
        arr[i] = 0;
        printf("hello world\n");
    }
    return 0;
}
```
这段代码的运行结果并非是打印三行“hello word”，而是会无限打印“hello world”.

数组大小为 3，a[0]，a[1]，a[2]，而我们的代码因为书写错误，导致 for 循环的结束条件错写为了 i<=3 而非 i<3，所以当 i=3 时，数组 a[3] 访问越界。

在 C 语言中，只要不是访问受限的内存，所有的内存空间都是可以自由访问的。a[3] 也会被定位到某块不属于数组的内存地址上，而这个地址正好是存储变量 i 的内存地址，那么 a[3]=0 就相当于 i=0，所以就会导致代码无限循环。

数组越界在 C 语言中是一种未决行为，并没有规定数组访问越界时编译器应该如何处理。访问数组的本质就是访问一段连续内存，只要数组通过偏移计算得到的内存地址是可用的，那么程序就可能不会报任何错误。

这种情况下，一般都会出现莫名其妙的逻辑错误，debug 的难度非常的大。很多计算机病毒也正是利用到了代码中的数组越界可以访问非法地址的漏洞，来攻击系统。



##### 容器完全替代数组

针对数组类型，很多语言都提供了容器类，比如 Java 中的 ArrayList、C++ STL 中的 vector。



ArrayList 最大的优势就是可以将很多数组操作的细节封装起来，支持动态扩容。

扩容操作涉及内存申请和数据搬移，是比较耗时的。如果事先能确定需要存储的数据大小，最好在创建 ArrayList 的时候事先指定数据大小。



1.Java ArrayList 无法存储基本类型，比如 int、long，需要封装为 Integer、Long 类，而 Autoboxing、Unboxing 则有一定的性能消耗，所以如果特别关注性能，或者希望使用基本类型，就可以选用数组。

2.如果数据大小事先已知，并且对数据的操作非常简单，用不到 ArrayList 提供的大部分方法，也可以直接使用数组。

3.表示多维数组时，用数组往往会更加直观。
对于业务开发，直接使用容器就足够了，省时省力。毕竟损耗一丢丢性能，完全不会影响到系统整体的性能。但如果你是做一些非常底层的开发，比如开发网络框架，性能的优化需要做到极致，这个时候数组就会优于容器，成为首选。



##### 解答开篇

从 1 开始编号，每次随机访问数组元素都多了一次减法运算，对于 CPU 来说，就是多了一次减法指令。

数组作为非常基础的数据结构，通过下标随机访问数组元素又是其非常基础的编程操作，效率的优化就要尽可能做到极致。所以为了减少一次减法操作，数组选择了从 0 开始编号，而不是从 1 开始。

最主要的原因可能是历史原因。

C 语言设计者用 0 开始计数数组下标，之后的 Java、JavaScript 等高级语言都效仿了 C 语言，或者说，为了在一定程度上减少 C 语言程序员学习 Java 的学习成本，因此继续沿用了从 0 开始计数的习惯。实际上，很多语言中数组也并不是从 0 开始计数的，比如 Matlab。甚至还有一些语言支持负数下标，比如 Python。



##### 内容小结

数组是线性表，用一块连续的内存空间，来存储相同类型的一组数据，最大的特点就是支持随机访问，但插入、删除操作也因此变得比较低效，平均情况时间复杂度为 O(n)。在平时的业务开发中，我们可以直接使用编程语言提供的容器类，但是，如果是特别底层的开发，直接使用数组可能会更合适。



##### 课后思考

标记清除垃圾回收算法。

JVM通过可达性分析确定某个对象是否存活，在标记阶段，会遍历所有 GC ROOTS，将所有 GC ROOTS 可达的对象标记为存活。只有当标记工作完成后，触发GC，清理工作才会开始，这些没有被标记存活的对象从内存中删除。

不足：1.效率问题。标记和清理效率都不高，但是当知道只有少量垃圾产生时会很高效。2.空间问题。会产生不连续的内存空间碎片。



二维数组的内存寻址公式是怎样的呢？

a$[i][j]$，a$[m][n]$_addr = base_address+$(m*j+n)*$data_type_size

对于数组访问越界造成无限循环，对于不同的编译器，在内存分配时，会按照内存地址递增或递减的方式进行分配。如果是内存地址递减的方式，就会造成无限循环。

对文中示例的无限循环有疑问的同学，建议去查函数调用的栈桢结构细节（操作系统或计算机体系结构的教材应该会讲到）。

函数体内的局部变量存在栈上，且是连续压栈。在Linux进程的内存布局中，栈区在高地址空间，从高向低增长。变量i和arr在相邻地址，且i比arr的地址大，所以arr越界正好访问到i。当然，前提是i和arr元素同类型，否则那段代码仍是未决行为。

例子中死循环的问题跟编译器分配内存和字节对齐有关 数组3个元素 加上一个变量a 。4个整数刚好能满足8字节对齐 所以i的地址恰好跟着a2后面 导致死循环。。如果数组本身有4个元素 则这里不会出现死循环。。因为编译器64位操作系统下 默认会进行8字节对齐 变量i的地址就不紧跟着数组后面了。
gcc有一个编译选项（-fno-stack-protector）用于关闭堆栈保护功能。默认情况下启动了堆栈保护，不管i声明在前还是在后，i都会在数组之后压栈，只会循环4次；如果关闭堆栈保护功能，则会出现死循环。

https://www.ibm.com/developerworks/cn/linux/l-cn-gccstack/index.html
举例的内存越界的循环应该限制在x86架构的小端模式，在别的架构平台上的大端模式不是这样的！


深究 JavaScript 数组 —— 演进&性能