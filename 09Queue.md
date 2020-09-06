队列在线程池等有限资源池中的应用
===

&emsp;&emsp;CPU 资源是有限的，任务的处理速度与线程个数并不是线性正相关。相反，过多的线程反而会导致 CPU 频繁切换，处理性能下降。所以，线程池的大小一般都是综合考虑要处理任务的特点和硬件环境，事先设置的。
&emsp;&emsp;当我们向固定大小的线程池中请求一个线程时，如果线程池中没有空闲资源了，这个时候线程池可能拒绝请求、排队请求，有多种拒绝策略。拒绝请求、排队请求和拒绝策略这些问题的底层数据结构就是队列（queue）。

##### 理解“队列”

&emsp;&emsp;先进者先出，后进后出，这就是典型的“队列”。栈只支持两个基本操作：入栈 push()和出栈 pop()。队列跟栈非常相似，支持的操作也很有限，最基本的操作也是两个：入队 enqueue()，放一个数据到队列尾部；出队 dequeue()，从队列头部取一个元素。

<img src="assert/QueueAndStack.jpg" alt="栈和队列的比较" style="zoom:50%;" />

&emsp;&emsp;队列跟栈一样，也是一种操作受限的线性表数据结构。
&emsp;&emsp;作为一种非常基础的数据结构，队列的应用也非常广泛，特别是一些具有某些额外特性的队列，比如循环队列、阻塞队列、并发队列。它们在很多偏底层系统、框架、中间件的开发中，起着关键性的作用。比如高性能队列 Disruptor、Linux 环形缓存，都用到了循环并发队列；Java concurrent 并发包利用 ArrayBlockingQueue 来实现公平锁等。



##### 顺序队列和链式队列

&emsp;&emsp;队列跟栈一样，也是一种抽象的数据结构。它具有先进先出的特性，支持在队尾插入元素，在队头删除元素。队列可以用数组来实现，也可以用链表来实现。用数组实现的栈叫作顺序栈，用链表实现的栈叫作链式栈。同样，用数组实现的队列叫作顺序队列，用链表实现的队列叫作链式队列。

####基于数组用 Java 语言实现的顺序队列。
```Java
// 用数组实现的队列
public class ArrayQueue {
  // 数组：items，数组大小：n
  private String[] items;
  private int n = 0;
  // head 表示队头下标，tail 表示队尾下标
  private int head = 0;
  private int tail = 0;
 
  // 申请一个大小为 capacity 的数组
  public ArrayQueue(int capacity) {
    items = new String[capacity];
    n = capacity;
  }
 
  // 入队
  public boolean enqueue(String item) {
    // 如果 tail == n 表示队列已经满了
    if (tail == n) return false;
    items[tail] = item;
    ++tail;
    return true;
  }
 
  // 出队
  public String dequeue() {
    // 如果 head == tail 表示队列为空
    if (head == tail) return null;
    // 为了让其他语言的同学看的更加明确，把 -- 操作放到单独一行来写了
    String ret = items[head];
    // 出队列并且删除队头元素
    // items[head] = null;
    ++head;
    return ret;
  }
}
```

对于栈来说，我们只需要一个**栈顶指针**就可以了。但是队列需要两个指针：一个是 head 指针，指向队头；一个是 tail 指针，指向队尾。

你可以结合下面这幅图来理解。当 a、b、c、d 依次入队之后，队列中的 head 指针指向下标为 0 的位置，tail 指针指向下标为 4 的位置。

<img src="assert/ArrayBasedQueueOperationa.jpg" alt="顺序队列的操作" style="zoom:50%;" />

当我们调用两次出队操作之后，队列中 head 指针指向下标为 2 的位置，tail 指针仍然指向下标为 4 的位置。

<img src="assert/ArrayBasedQueueOperationb.jpg" alt="顺序队列的操作" style="zoom:50%;" />

&emsp;&emsp;随着不停地进行入队、出队操作，head 和 tail 都会持续往后移动。当 tail 移动到最右边，即使数组中还有空闲空间，也无法继续往队列中添加数据了。用数据搬移可以解决这个问题。但是，每次进行出队操作都相当于删除数组下标为 0 的数据，要搬移整个队列中的数据，这样出队操作的时间复杂度就会从原来的 O(1) 变为 O(n)。需要考虑优化方案。
&emsp;&emsp;我们在出队时可以不用搬移数据。如果没有空闲空间了，我们只需要在入队时，再集中触发一次数据的搬移操作。借助这个思想，出队函数 dequeue() 保持不变，我们稍加改造一下入队函数 enqueue() 的实现，就可以轻松解决刚才的问题了。下面是具体的代码：

```Java
 // 入队操作，将 item 放入队尾
  public boolean enqueue(String item) {
    // tail == n 表示队列末尾没有空间了
    if (tail == n) {
      // tail ==n && head==0，表示整个队列都占满了
      if (head == 0) return false;
      // 数据搬移
      for (int i = head; i < tail; ++i) {
        items[i-head] = items[i];
      }
      // 搬移完之后重新更新 head 和 tail
      tail -= head;
      head = 0;
    }
    
    items[tail] = item;
    ++tail;
    return true;
  }
```
&emsp;&emsp;从代码中我们看到，当队列的 tail 指针移动到数组的最右边后，如果有新的数据入队，我们可以将 head 到 tail 之间的数据，整体搬移到数组中 0 到 tail-head 的位置。

<img src="assert/ArrayBasedQueueOperationc.jpg" alt="顺序队列的操作" style="zoom:50%;" />

&emsp;&emsp;这种实现思路中，出队操作的时间复杂度仍然是 O(1)，入队操作的最好时间复杂度是 O(1)，均摊时间复杂度是O(1)。

##### 基于链表的队列实现方法

&emsp;&emsp;基于链表的实现，我们同样需要两个指针：head 指针和 tail 指针。它们分别指向链表的第一个结点和最后一个结点。如图所示，入队时，tail->next= new_node, tail = tail->next；出队时，head = head->next。

<img src="assert/LinkedListBasedQueue.jpg" alt="链式队列" style="zoom:50%;" />

##### 循环队列

&emsp;&emsp;用数组来实现队列的时候，在 tail==n 时，会有数据搬移操作，这样入队操作性能就会受到影响。循环队列可以解决这个问题。

&emsp;&emsp;循环队列，顾名思义，它长得像一个环。原本数组是有头有尾的，是一条直线。现在我们把首尾相连，连成了一个环。

<img src="assert/CircularQueuea.jpg" alt="循环队列" style="zoom:50%;" />

&emsp;&emsp;我们可以看到，图中这个队列的大小为 8，当前 head=4，tail=7。当有一个新的元素 a 入队时，我们放入下标为 7 的位置。但这个时候，我们并不把 tail 更新为 8，而是将其在环中后移一位，到下标为 0 的位置。当再有一个元素 b 入队时，我们将 b 放入下标为 0 的位置，然后 tail 加 1 更新为 1。所以，在 a，b 依次入队之后，循环队列中的元素就变成了下面的样子：

<img src="assert/CircularQueueb.jpg" alt="循环队列" style="zoom:50%;" />

&emsp;&emsp;通过这样的方法，我们成功避免了数据搬移操作。看起来不难理解，但是循环队列的代码实现难度要比前面讲的非循环队列难多了。要想写出没有 bug 的循环队列的实现代码，我个人觉得，最关键的是，确定好队空和队满的判定条件。

在用数组实现的非循环队列中，队满的判断条件是 tail == n，队空的判断条件是 head == tail。那针对循环队列，队列为空的判断条件仍然是 head == tail。但队列满的判断条件就稍微有点复杂了。

<img src="assert/CircularQueuec.jpg" alt="循环队列" style="zoom:50%;" />

&emsp;&emsp;如图中画的队满的情况，tail=3，head=4，n=8，所以总结一下规律就是：(3+1)%8=4。多画几张队满的图，你就会发现，当队满时，(tail+1)%n=head。

&emsp;&emsp;当队列满时，图中的 tail 指向的位置实际上是没有存储数据的。所以，循环队列会浪费一个数组的存储空间。

```Java
public class CircularQueue {
  // 数组：items，数组大小：n
  private String[] items;
  private int n = 0;
  // head 表示队头下标，tail 表示队尾下标
  private int head = 0;
  private int tail = 0;
 
  // 申请一个大小为 capacity 的数组
  public CircularQueue(int capacity) {
    items = new String[capacity];
    n = capacity;
  }
 
  // 入队
  public boolean enqueue(String item) {
    // 队列满了
    if ((tail + 1) % n == head) return false;
    items[tail] = item;
    tail = (tail + 1) % n;
    return true;
  }
 
  // 出队
  public String dequeue() {
    // 如果 head == tail 表示队列为空
    if (head == tail) return null;
    String ret = items[head];
    head = (head + 1) % n;
    return ret;
  }
}
```



##### 阻塞队列和并发队列

&emsp;&emsp;队列这种数据结构很基础，平时的业务开发不大可能从零实现一个队列，甚至都不会直接用到。而一些具有特殊特性的队列应用却比较广泛，比如阻塞队列和并发队列。

&emsp;&emsp;**阻塞队列**其实就是在队列基础上增加了阻塞操作。简单来说，就是在队列为空的时候，从队头取数据会被阻塞。因为此时还没有数据可取，直到队列中有了数据才能返回；如果队列已经满了，那么插入数据的操作就会被阻塞，直到队列中有空闲位置后再插入数据，然后再返回。

<img src="assert/BlockingQueuea.jpg" alt="阻塞队列" style="zoom:50%;" />

&emsp;&emsp;上面的定义就是一个“生产者 - 消费者模型”！我们可以使用阻塞队列，轻松实现一个“生产者 - 消费者模型”！

&emsp;&emsp;这种基于阻塞队列实现的“生产者 - 消费者模型”，可以有效地协调生产和消费的速度。当“生产者”生产数据的速度过快，“消费者”来不及消费时，存储数据的队列很快就会满了。这个时候，生产者就阻塞等待，直到“消费者”消费了数据，“生产者”才会被唤醒继续“生产”。

&emsp;&emsp;基于阻塞队列，我们还可以通过协调“生产者”和“消费者”的个数，来提高数据的处理效率。我们可以多配置几个“消费者”，来应对一个“生产者”。

<img src="assert/MultiConsumerBlockingQueue.jpg" alt="多个消费者的阻塞队列" style="zoom:50%;" />

&emsp;&emsp;前面我们讲了阻塞队列，在多线程情况下，会有多个线程同时操作队列，这个时候就会存在线程安全问题。

&emsp;&emsp;线程安全的队列我们叫作并发队列。最简单直接的实现方式是直接在 enqueue()、dequeue() 方法上加锁，但是锁粒度大并发度会比较低，同一时刻仅允许一个存或者取操作。实际上，基于数组的循环队列，利用 CAS 原子操作，可以实现非常高效的并发队列。这也是循环队列比链式队列应用更加广泛的原因。



##### 队列在线程池等有限资源池中的应用

&emsp;&emsp;线程池没有空闲线程时，新的任务请求线程资源时，线程池该的处理一般有两种处理策略。第一种是非阻塞的处理方式，直接拒绝任务请求；另一种是阻塞的处理方式，将请求排队，等到有空闲线程时，取出排队的请求继续处理。

&emsp;&emsp;如果希望公平地处理每个排队的请求，先进者先服务，所以队列这种数据结构很适合来存储排队请求。队列有基于链表和基于数组这两种实现方式。这两种实现方式对于排队请求又有什么区别呢？

&emsp;&emsp;基于链表的实现方式，可以实现一个支持无限排队的无界队列（unbounded queue），但是可能会导致过多的请求排队等待，请求处理的响应时间过长。所以，针对响应时间比较敏感的系统，基于链表实现的无限排队的线程池是不合适的。

&emsp;&emsp;而基于数组实现的有界队列（bounded queue），队列的大小有限，所以线程池中排队的请求超过队列大小时，接下来的请求就会被拒绝，这种方式对响应时间敏感的系统来说，就相对更加合理。不过，设置一个合理的队列大小，也是非常有讲究的。队列太大导致等待的请求太多，队列太小会导致无法充分利用系统资源、发挥最大性能。

&emsp;&emsp;除了前面讲到队列应用在线程池请求排队的场景之外，队列可以应用在任何有限资源池中，用于排队请求，比如数据库连接池等。**实际上，对于大部分资源有限的场景，当没有空闲资源时，基本上都可以通过“队列”这种数据结构来实现请求排队。**



##### 总结

&emsp;&emsp;队列最大的特点就是先进先出，主要的两个操作是入队和出队。跟栈一样，它既可以用数组来实现，也可以用链表来实现。用数组实现的叫顺序队列，用链表实现的叫链式队列，特别是长得像一个环的循环队列，在数组实现队列的时候，会有数据搬移操作，要想解决数据搬移的问题，我们就需要像环一样的循环队列。

&emsp;&emsp;循环队列是重点。要想写出没有 bug 的循环队列实现代码，关键要确定好队空和队满的判定条件，(tail+1)%n=head。

&emsp;&emsp;高级的队列结构，阻塞队列、并发队列，底层都还是队列这种数据结构，只不过在之上附加了很多其他功能。阻塞队列就是入队、出队操作可以阻塞，并发队列就是队列的操作多线程安全。



##### 问题

&emsp;&emsp;除了线程池这种池结构会用到队列排队请求，你还知道有哪些类似的池结构或者场景中会用到队列的排队请求呢？
1.分布式应用中的消息队列，也是一种队列结构
2.http请求连接池是队列结构吗？

&emsp;&emsp;关于如何实现无锁并发队列，对这个问题你怎么看呢？
* 1.考虑使用CAS实现无锁队列，则在入队前，获取tail位置，入队时比较tail是否发生变化，如果否，则允许入队，反之，本次入队失败。出队则是获取head位置，进行cas，出队时比较head是否发生变化，如果否，则允许出队，反之本次出队失败。因为tail、head的变化都是取模运算，极端场景下，比如n=2，可能其他的线程执行完两个操作，tail或者head走了一圈又回来了。
* 2.循环队列的长度设定需要对并发数据有一定的预测，否则会丢失太多请求。
* 3.循环队列真的是比较牛逼的思路，尤其是linux内核源码的kfifo的实现，无论是取模运算转换成取与运算，还是考虑head，tail的溢出。
* 4.循环队列增加flag来避免浪费最后一个存储空间，flag本身也需要一个存储空间，所以这个说法并没有意义。
* 5.无锁化的并发其实就是不能接受锁的消耗，另外不恰当的编码会导致死锁，在并发场景下对业务是灾难性的影响。而无锁化的并发对容器的大小有严格的要求，需要对业务理解比较透彻，并且做好限流、资源监控等操作。





老师，循环队列的数组实现，在您的代码中，入队时会空留出一个位置，而且我感觉不太好理解。我定义一个记录队列大小的值size，当这个值与数组大小相等时，表示队列已满，当tail达到最底时，size不等于数组大小时，tail就指向数组第一个位置。当出队时，size—，入队时size++



循环队列真的是比较牛逼的思路，尤其是linux内核源码的kfifo的实现，无论是取模运算转换成取与运算，还是考虑head，tail的溢出，牛逼



很多同学都提到循环队列增加flag来避免浪费最后一个存储空间，flag本身也需要一个存储空间，并没有解决问题。



使用列表实现队列和循环队列，我用python实现了一遍，各位看官一起交流。
https://github.com/lipeng1991/testdemo/blob/master/38_array_implementation_queue.py
https://github.com/lipeng1991/testdemo/blob/master/39_array_implementation_loop_queue.py



一直想不明白为什么队列要空出一个空的格不存数据，不是可以直接入队存在tail里，tail＋＋再比较是否超出容量吗。

作者回复: 循环队列不行的 不然无法区分队空和队满