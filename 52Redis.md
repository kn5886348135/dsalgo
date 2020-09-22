算法实战（一）：剖析Redis常用数据类型对应的数据结构
===

&emsp;&emsp;通过一些开源项目、经典系统，真枪实弹地教你，如何将数据结构和算法应用到项目中。

&emsp;&emsp;要多举一反三地思考，自己接触过的开源项目、基础框架、中间件中，都用过哪些数据结构和算法。你也可以想一想，在自己做的项目中，有哪些可以用学过的数据结构和算法进一步优化。这样的学习效果才会更好。

#### Redis 数据库介绍

&emsp;&emsp;Redis 是一种键值（Key-Value）数据库。相对于关系型数据库（比如 MySQL），Redis 也被叫**作非关系型数据库。**

&emsp;&emsp;像 MySQL 这样的关系型数据库，表的结构比较复杂，会包含很多字段，可以通过 SQL 语句，来实现非常复杂的查询需求。而 Redis 中只包含“键”和“值”两部分，只能通过“键”来查询“值”。正是因为这样简单的存储结构，也让 Redis 的读写效率非常高。

&emsp;&emsp;Redis 主要是作为内存数据库来使用，数据是存储在内存中的。但是，它也支持将数据存储在硬盘中。

&emsp;&emsp;Redis 中，键的数据类型是字符串，但是为了丰富数据存储的方式，方便开发者使用，值的数据类型有很多，常用的数据类型有这样几种，它们分别是字符串、列表、字典、集合、有序集合。

&emsp;&emsp;“字符串（string）”这种数据类型非常简单，对应到数据结构里，就是字符串。

#### 列表（list）

&emsp;&emsp;列表这种数据类型支持存储一组数据。这种数据类型对应两种实现方法，**压缩列表**（ziplist）和**双向循环链表**。

&emsp;&emsp;当列表中存储的数据量比较小的时候，列表就可以采用压缩列表的方式实现。具体需要同时满足下面两个条件：

* 列表中保存的单个数据（有可能是字符串类型的）小于 64 字节；

* 列表中数据个数少于 512 个。

&emsp;&emsp;压缩列表，并不是基础数据结构，而是 Redis 自己设计的一种数据存储结构。它有点儿类似数组，通过一片连续的内存空间，来存储数据。不过，它跟数组不同的一点是，它允许存储的数据大小不同。具体的存储结构也非常简单，你可以看我下面画的这幅图。

<img src="assert/ZipListA.jpg" alt="压缩列表" style="zoom:50%;" />

&emsp;&emsp;听到“压缩”两个字，直观的反应就是节省内存。之所以说这种存储结构节省内存，是相较于数组的存储思路而言的。数组要求每个元素的大小相同，如果要存储不同长度的字符串，那就需要用最大长度的字符串大小作为元素的大小（假设是 20 个字节）。那当存储小于 20 个字节长度的字符串的时候，便会浪费部分存储空间。听起来有点儿拗口，我画个图解释一下。

<img src="assert/ZipListB.jpg" alt="压缩列表" style="zoom:50%;" />

&emsp;&emsp;压缩列表这种存储结构，一方面比较节省内存，另一方面可以支持不同类型数据的存储。而且，因为数据存储在一片连续的内存空间，通过键来获取值为列表类型的数据，读取的效率也非常高。

&emsp;&emsp;当列表中存储的数据量比较大的时候，也就是不能同时满足刚刚讲的两个条件的时候，列表就要通过双向循环链表来实现了。

&emsp;&emsp;Redis 的这种双向链表的实现方式，非常值得借鉴。它额外定义一个 list 结构体，来组织链表的首、尾指针，还有长度等信息。这样，在使用的时候就会非常方便。

```C
/ 以下是 C 语言代码，因为 Redis 是用 C 语言实现的。
typedef struct listnode {
  struct listNode *prev;
  struct listNode *next;
  void *value;
} listNode;


typedef struct list {
  listNode *head;
  listNode *tail;
  unsigned long len;
  // .... 省略其他定义
} list;
```

#### 字典（hash）

&emsp;&emsp;字典类型用来存储一组数据对。每个数据对又包含键值两部分。字典类型也有两种实现方式，**压缩列表**和**散列表**。

&emsp;&emsp;只有当存储的数据量比较小的情况下，Redis 才使用压缩列表来实现字典类型。具体需要满足两个条件：

* 字典中保存的键和值的大小都要小于 64 字节；

* 字典中键值对的个数要小于 512 个。

&emsp;&emsp;当不能同时满足上面两个条件的时候，Redis 就使用散列表来实现字典类型。Redis 使用[MurmurHash2]([https://zh.wikipedia.org/wiki/Murmur%E5%93%88%E5%B8%8C](https://zh.wikipedia.org/wiki/Murmur哈希))这种运行速度快、随机性好的哈希算法作为哈希函数。对于哈希冲突问题，Redis 使用链表法来解决。除此之外，Redis 还支持散列表的动态扩容、缩容。

&emsp;&emsp;当数据动态增加之后，散列表的装载因子会不停地变大。为了避免散列表性能的下降，当装载因子大于 1 的时候，Redis 会触发扩容，将散列表扩大为原来大小的 2 倍左右（具体值需要计算才能得到，如果感兴趣，你可以去阅读[源码]: https://github.com/antirez/redis/blob/unstable/src/dict.c。
&emsp;&emsp;当数据动态减少之后，为了节省内存，当装载因子小于 0.1 的时候，Redis 就会触发缩容，缩小为字典中数据个数的大约 2 倍大小（这个值也是计算得到的，如果感兴趣，你也可以去阅读[源码]: https://github.com/antirez/redis/blob/unstable/src/dict.c。

&emsp;&emsp;扩容缩容要做大量的数据搬移和哈希值的重新计算，所以比较耗时。针对这个问题，Redis 使用渐进式扩容缩容策略，将数据的搬移分批进行，避免了大量数据一次性搬移导致的服务停顿。

#### 集合（set）

&emsp;&emsp;集合这种数据类型用来存储一组不重复的数据。这种数据类型也有两种实现方法，有序数组和散列表。

&emsp;&emsp;当要存储的数据，同时满足下面这样两个条件的时候，Redis 就采用有序数组，来实现集合这种数据类型。

* 存储的数据都是整数；

* 存储的数据元素个数不超过 512 个。

&emsp;&emsp;当不能同时满足这两个条件的时候，Redis 就使用散列表来存储集合中的数据。

#### 有序集合（sortedset）

&emsp;&emsp;有序集合用来存储一组数据，并且每个数据会附带一个得分。通过得分的大小，将数据组织成跳表这样的数据结构，以支持快速地按照得分值、得分区间获取数据。

&emsp;&emsp;跟 Redis 的其他数据类型一样，有序集合也并不仅仅只有跳表这一种实现方式。当数据量比较小的时候，Redis 会用压缩列表来实现有序集合。具体点说就是，使用压缩列表来实现有序集合的前提，有这样两个：

* 所有数据的大小都要小于 64 字节；

* 元素个数要小于 128 个。

#### 数据结构持久化

&emsp;&emsp;尽管 Redis 经常会被用作内存数据库，但是，它也支持数据落盘，也就是将内存中的数据存储到硬盘中。这样，当机器断电的时候，存储在 Redis 中的数据也不会丢失。在机器重新启动之后，Redis 只需要再将存储在硬盘中的数据，重新读取到内存，就可以继续工作了。

&emsp;&emsp;Redis 的数据格式由“键”和“值”两部分组成。而“值”又支持很多数据类型，比如字符串、列表、字典、集合、有序集合。像字典、集合等类型，底层用到了散列表，散列表中有指针的概念，而指针指向的是内存中的存储地址。 那 Redis 是如何将这样一个跟具体内存地址有关的数据结构存储到磁盘中的呢？

&emsp;&emsp;Redis 遇到的这个问题并不特殊，很多场景中都会遇到。叫作数据结构的持久化问题，或者对象的持久化问题。这里的“持久化”，可以笼统地可以理解为“存储到磁盘”。

&emsp;&emsp;如何将数据结构持久化到硬盘？主要有两种解决思路。

1. 清除原有的存储结构，只将数据存储到磁盘中。当需要从磁盘还原数据到内存的时候，再重新将数据组织成原来的数据结构。实际上，Redis 采用的就是这种持久化思路。不过，这种方式也有一定的弊端。那就是数据从硬盘还原到内存的过程，会耗用比较多的时间。比如，现在要将散列表中的数据存储到磁盘。当从磁盘中，取出数据重新构建散列表的时候，需要重新计算每个数据的哈希值。如果磁盘中存储的是几 GB 的数据，那重构数据结构的耗时就不可忽视了。

2. 保留原来的存储格式，将数据按照原有的格式存储在磁盘中。拿散列表这样的数据结构来举例。可以将散列表的大小、每个数据被散列到的槽的编号等信息，都保存在磁盘中。有了这些信息，从磁盘中将数据还原到内存中的时候，就可以避免重新计算哈希值。

#### 总结引申

&emsp;&emsp; Redis 中常用数据类型底层依赖的数据结构，有这五种：压缩列表（可以看作一种特殊的数组）、有序数组、链表、散列表、跳表。Redis 就是这些常用数据结构的封装。

&emsp;&emsp;有了数据结构和算法的基础之后，再去阅读 Redis 的源码，理解起来就容易多了？很多原来觉得很深奥的设计思想，是不是就都会觉得顺理成章了呢？

&emsp;&emsp;还是那句话，夯实基础很重要。同样是看源码，有些人只能看个热闹，了解一些皮毛，无法形成自己的知识结构，不能化为己用，过不几天就忘了。而有些人基础很好，不但能知其然，还能知其所以然，从而真正理解作者设计的动机。这样不但能有助于理解所用的开源软件，还能为自己创新添砖加瓦。

#### 课后思考

&emsp;&emsp;你有没有发现，在数据量比较小的情况下，Redis 中的很多数据类型，比如字典、有序集合等，都是通过多种数据结构来实现的，为什么会这样设计呢？用一种固定的数据结构来实现，不是更加简单吗？

&emsp;&emsp;讲到数据结构持久化有两种方法。对于二叉查找树这种数据结构，如何将它持久化到磁盘中呢？





思考题1：redis的数据结构由多种数据结构来实现，主要是出于时间和空间的考虑，当数据量小的时候通过数组下标访问最快、占用内存最小，而压缩列表只是数组的升级版；
因为数组需要占用连续的内存空间，所以当数据量大的时候，就需要使用链表了，同时为了保证速度又需要和数组结合，也就有了散列表。
对于数据的大小和多少采用哪种数据结构，相信redis团队一定是根据大多数的开发场景而定的。

思考题2：二叉查找树的存储，我倾向于存储方式一，通过填充叶子节点形成完全二叉树，然后以数组的形式存储到硬盘，数据还原过程也是非常高效的。如果用存储方式二就比较复杂了。








看完前边的课程，当知识点连成线和面之后，才发现原来是这么回事，再来看redis确认是豁然开朗。知识成体系之后记忆会更深刻，不过也带来了更多的思考和发现-----知识边界扩大了






老师 关于redis的压缩列表有个地方不太明白
虽然压缩列表看起来想数组 但他能像数组一样支持按照下标进行直接随机访问吗？比如我要访问下标为n的数据我启不是需要知道之前从0到n-1的所有数据的长度才能找到n，那这跟链表的时间复杂读没啥区别啊，而且还占用了连续的内存空间？ 还是说压缩列表中的每个元素的长度都记录在它的头部可以一次性的获取？
作者回复: 哈哈，你说的没错。压缩列表不支持随机访问。有点类似链表。但是比较省存储空间啊。Redis一般都是通过key获取整个value的值，也就是整个压缩列表的数据，并不需要随机访问。








数据量小时采用连续内存的数据结构是为了CPU缓存读取连续内存来提高命中率，而限制数据数量和数据大小应该是考虑到CPU缓存的大小






问题1: 压缩列表优点：访问存取快速，节省内存。但是受到操作系统空闲内存限制。越大的连续内存空间越不容易申请到。所以用了其他数据结构比如链表替代。





老师，如果字典保存的键和值的大小都小于 64 字节，并且键值对的个数小于 512 个，Redis 用压缩列表实现。从 [源码](https://github.com/antirez/redis/blob/unstable/src/ziplist.c) （ziplist.c ziplistFind） 来看压缩列表根据键查找值的方式，就是一个个遍历。如果有几百个键值对，这么查找比散列表快吗？






老师您好，有这样一个场景，A关注了B,这样的操作会同时写两个链表一个是A的关注列表，另一个是B的粉丝列表，比如用redis 的sortset来存储。现在要检查所有不一致的情况(比如，A的关注列表有B，但是B的粉丝列表没有A，或者A的关注列表没有B，但是B的粉丝列表有A)。这种情况有什么好的方法吗?






有个疑问，比如对于有序集合，这个数据量可能会逐步增加，那么数据量达到阈值时就会切换成跳表吗？是数据全部移动到跳表，然后删除列表吗？






老师说的压缩列表，是整个数据 hash 或者set 实现是 压缩列表实现的，还是指 hash 或list 里面的具体一个元素是压缩列表实现的？ 压缩列表的结构不太清楚
作者回复: 一整个都是用压缩列表实现的。







1. 这个是因为每种数据结构都有适合自己的场景，比如压缩列表（特殊的有序数组）比较适合查询操作，删除新增的时间复杂度较高为O（n）,数据量小的时候可以使用，因为结构简单，数据量大的时候删除新增的效率非常低；所以量大的时候要考虑增删改查都比较快的数据结构，比如散列表、跳表、二叉树、红黑树等等数据结构了
2. 二叉树可以通过前序+中序写入磁盘，之后通过前序+中序还原；或者类似于将堆的时候，将数据按层遍历存入数组中，从下标1开始存储，下标i的左右子树存储在(2*i,2*i+1)下标中，然后顺序写入磁盘，这个的缺点是会产生空洞，因为不一定是满二叉树





压缩列表每个元素所占用的空间大小是不一定的，所以当想要随机访问某个元素的时候还是要像列表那样从头开始遍历，所以不能太大。理解对吗？
作者回复: 是的 没错




"字典中保存的键和值的大小都要小于 64 字节"
老师，请问这句话的意思是 size(键+值)<64 还是 size(键) <64 && size(值)<64
作者回复: 后者






1.肯定对于不同的存储类型，需采用不同的数据结构，这是与自身的业务特性相关联的。
2.对于二叉树的持久化可以参考leetcode接口，采用数组即可。




老师，Redis字典数据类型中散列表的装载因子最大不就是1么，大于1是什么情况？
作者回复: 就是拉了很长的链表




压缩列表是不是用变长数组实现的呐？
char str[0[]
作者回复: 有可能。






有一个问题，关于压缩列表的描述中有句话有个疑问，“而且，因为数据存储在一片连续的内存空间，通过键来获取值为列表类型的数据，读取的效率也非常高”，数据存储在一片连续的内存空间中只是压缩列表内部的存储结构，这应该不是通过键获取值时读取效率非常高的的原因吧？






我猜压缩列表的好处是能利用l2缓存?
作者回复: 是的，有这么个好处。越小越有利于CPU缓存。






王老师，既然可以用压缩列表，为啥数据超过512个的时候不用单链表