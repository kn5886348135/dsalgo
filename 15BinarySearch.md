二分查找（上）：如何用最省内存的方式实现快速查找功能？
===

&emsp;&emsp;二分查找（Binary Search）算法是一种针对有序数据集合的查找算法，也叫折半查找算法。

&emsp;&emsp;假设我们有 1000 万个整数数据，每个数据占 8 个字节，**如何设计数据结构和算法，快速判断某个整数是否出现在这 1000 万数据中？** 我们希望这个功能不要占用太多的内存空间，最多不要超过 100MB。



##### 无处不在的二分思想

&emsp;&emsp;二分查找是一种非常简单易懂的快速查找算法。利用二分思想，每次都与区间的中间数据比对大小，缩小查找区间的范围。

##### &emsp;&emsp;二分查找针对的是一个有序的数据集合，查找思想有点类似分治思想。每次都通过跟区间的中间元素对比，将待查找的区间缩小为之前的一半，直到找到要查找的元素，或者区间被缩小为 0。



##### O($log{n}$) 惊人的查找速度

&emsp;&emsp;二分查找是一种非常高效的查找算法，时间复杂度是 O($log{n}$)，堆、二叉树的时间复杂度也是 O($log{n}$)。O($log{n}$) 这种**对数时间复杂度**，是一种极其高效的时间复杂度，有的时候甚至比时间复杂度是常量级 O(1) 的算法还要高效。因为用大 O 标记法表示时间复杂度的时候，会省略掉常数、系数和低阶。



##### 二分查找的递归与非递归实现

&emsp;&emsp;简单的二分查找并不难写，二分查找的变体问题，才是真正烧脑的。

&emsp;&emsp;最简单的情况就是有序数组中不存在重复元素，在其中用二分查找值等于给定值的数据。
```Java
public int bsearch(int[] a, int n, int value) {
  int low = 0;
  int high = n - 1;
 
  while (low <= high) {
    int mid = (low + high) / 2;
    if (a[mid] == value) {
      return mid;
    } else if (a[mid] < value) {
      low = mid + 1;
    } else {
      high = mid - 1;
    }
  }
 
  return -1;
}
```
&emsp;&emsp;low、high、mid 都是指数组下标，其中 low 和 high 表示当前查找的区间范围，初始 low=0， high=n-1。mid 表示 [low, high] 的中间位置。通过对比 a[mid] 与 value 的大小，来更新接下来要查找的区间范围，直到找到或者区间缩小为 0就退出。

&emsp;&emsp;容易出错的 3 个地方。

1. 循环退出条件是 low<=high，而不是 low<high。
2. mid 的取值
    mid=(low+high)$/$2 这种写法是有问题的。因为如果 low 和 high 比较大的话，两者之和就有可能会溢出。改进的方法是将 mid 的计算方式写成 low+(high-low)/2。更进一步，如果要将性能优化到极致的话，可以将这里的除以 2 操作转化成位运算 low+((high-low)>>1)。因为相比除法运算来说，计算机处理位运算要快得多。
3. low 和 high 的更新
    low=mid+1，high=mid-1。注意这里的 +1 和 -1，如果直接写成 low=mid 或者 high=mid，就可能会发生死循环。比如，当 high=3，low=3 时，如果 a[3] 不等于 value，就会导致一直循环不退出。

&emsp;&emsp;二分查找除了用循环来实现，还可以用递归来实现。

```Java
// 二分查找的递归实现
public int bsearch(int[] a, int n, int val) {
  return bsearchInternally(a, 0, n - 1, val);
}
 
private int bsearchInternally(int[] a, int low, int high, int value) {
  if (low > high) return -1;
 
  int mid =  low + ((high - low) >> 1);
  if (a[mid] == value) {
    return mid;
  } else if (a[mid] < value) {
    return bsearchInternally(a, mid+1, high, value);
  } else {
    return bsearchInternally(a, low, mid-1, value);
  }
}
```



##### 二分查找应用场景的局限性

&emsp;&emsp;二分查找的时间复杂度是 O($log{n}$)，查找数据的效率非常高。不过，并不是什么情况下都可以用二分查找，它的应用场景是有很大局限性的。

1. 二分查找依赖的是顺序表结构，简单点说就是数组。
    因为二分查找算法需要按照下标随机访问元素。数组按照下标随机访问数据的时间复杂度是 O(1)，而链表随机访问的时间复杂度是 O(n)。如果数据使用链表存储，二分查找的时间复杂就会变得很高。
2. 二分查找针对的是有序数据。
    二分查找要求数据必须是有序的。如果数据没有序，需要先排序，排序的时间复杂度最低是 O(n$log{n}$)。
    二分查找只能用在插入、删除操作不频繁，一次排序多次查找的场景中。针对动态变化的数据集合，二分查找将不再适用。那针对动态数据集合，可以使用二叉树快速查找某个数据。
3. 数据量太小不适合二分查找。
    如果要处理的数据量很小，完全没有必要用二分查找，顺序遍历就足够了。只有数据量比较大的时候，二分查找的优势才会比较明显。
    如果数据之间的比较操作非常耗时，不管数据量大小，都推荐使用二分查找。比如，数组中存储的都是长度超过 300 的字符串，如此长的两个字符串之间比对大小，就会非常耗时。需要尽可能地减少比较次数，而比较次数的减少会大大提高性能，这个时候二分查找就比顺序遍历更有优势。
4. 数据量太大也不适合二分查找。
    二分查找的底层需要依赖数组这种数据结构，而数组为了支持随机访问的特性，要求内存空间连续，对内存的要求比较苛刻。太大的数据用数组存储就比较吃力，也就不能用二分查找了。



##### 解答开篇

&emsp;&emsp;**如何在 1000 万个整数中快速查找某个整数？**

&emsp;&emsp;内存限制是 100MB，每个数据大小是 8 字节，最简单的办法就是将数据存储在数组中，内存占用差不多是 80MB，符合内存的限制。可以先对这 1000 万数据从小到大排序，然后再利用二分查找算法，就可以快速地查找想要的数据了。

&emsp;&emsp;用散列表、二叉树这些支持快速查找的动态数据结构解决这个问题实际上是不行的。

&emsp;&emsp;虽然大部分情况下，用二分查找可以解决的问题，用散列表、二叉树都可以解决。但是，不管是散列表还是二叉树，都会需要比较多的额外的内存空间。如果用散列表或者二叉树来存储这 1000 万的数据，用 100MB 的内存肯定是存不下的。而二分查找底层依赖的是数组，除了数据本身之外，不需要额外存储其他信息，是最省内存空间的存储方式，所以刚好能在限定的内存大小下解决这个问题。



##### 内容小结

&emsp;&emsp;二分查找是一种针对有序数据的高效查找算法，它的时间复杂度是 O($log{n}$)。

&emsp;&emsp;二分查找的核心思想有点类似分治思想。即每次都通过跟区间中的中间元素对比，将待查找的区间缩小为一半，直到找到要查找的元素，或者区间被缩小为 0。但是二分查找的代码实现比较容易写错。重点注意循环退出条件、mid 的取值，low 和 high 的更新。

&emsp;&emsp;二分查找虽然性能比较优秀，但应用场景也比较有限。底层必须依赖数组，并且还要求数据是有序的。对于较小规模的数据查找，我们直接使用顺序遍历就可以了，二分查找的优势并不明显。二分查找更适合处理静态数据，不能有频繁的数据插入、删除操作。



##### 课后思考

&emsp;&emsp;如何编程实现“求一个数的平方根”？要求精确到小数点后 6 位。

&emsp;&emsp;我刚才说了，如果数据使用链表存储，二分查找的时间复杂就会变得很高，那查找的时间复杂度究竟是多少呢？如果你自己推导一下，你就会深刻地认识到，为何我们会选择用数组而不是链表来实现二分查找了。




假设链表长度为n，二分查找每次都要找到中间点(计算中忽略奇偶数差异): 
第一次查找中间点，需要移动指针n/2次；
第二次，需要移动指针n/4次；
第三次需要移动指针n/8次；
......
以此类推，一直到1次为值

总共指针移动次数(查找次数) = n/2 + n/4 + n/8 + ...+ 1，这显然是个等比数列，根据等比数列求和公式：Sum = n - 1. 

最后算法时间复杂度是：O(n-1)，忽略常数，记为O(n)，时间复杂度和顺序查找时间复杂度相同

但是稍微思考下，在二分查找的时候，由于要进行多余的运算，严格来说，会比顺序查找时间慢



将mid = lo + (hi - lo) /2，将除法优化成移位运算时，得注意运算符的优先级，千万不能写成这样：mid = lo + (hi - lo) >> 1



二分法求一个数x的平方根y？
解答：根据x的值，判断求解值y的取值范围。假设求解值范围min < y < max。若0<x<1，则min=x，max=1；若x=1，则y=1；x>1，则min=1，max=x；在确定了求解范围之后，利用二分法在求解值的范围中取一个中间值middle=(min+max)÷2，判断middle是否是x的平方根？若(middle+0.000001)*(middle+0.000001)＞x且(middle-0.000001)*(middle-0.000001)<x，根据介值定理，可知middle既是求解值;若middle*middle > x，表示middle＞实际求解值，max=middle; 若middle*middle ＜ x，表示middle＜实际求解值，min =middle;之后递归求解！
备注：因为是保留6位小数，所以middle上下浮动0.000001用于介值定理的判断

求平方根，可以参考0到99之间猜数字的思路，99换成x, 循环到误差允许内即可，注意1这个分界线。欢迎交流，Java如下
    public static double sqrt(double x, double precision) {
        if (x < 0) {
            return Double.NaN;
        }
        double low = 0;
        double up = x;
        if (x < 1 && x > 0) {
            /** 小于1的时候*/
            low = x;
            up = 1;
        }
        double mid = low + (up - low)/2;
        while(up - low > precision) {
            if (mid * mid > x ) {//TODO mid可能会溢出
                up = mid;
            } else if (mid * mid < x) {
                low = mid;
            } else {
                return mid;
            }
            mid = low + (up - low)/2;
        }
        return mid;
    }






1. 求平方根可以用二分查找或牛顿迭代法;
2. 有序链表的二分查找时间复杂度为 O(n)。



开篇的问题：1000w 个 8字节整数的中查找某个整数是否存在，且内存占用不超过100M ？ 我尝试延伸了一些解决方案：
1、由于内存限制，存储一个整数需要8字节，也就是 64 bit。此时是否可以考虑bitmap这样的数据结构，也就是每个整数就是一个索引下标，对于每一个索引bit，1 表示存在，0 表示不存在。同时考虑到整数的数据范围，8字节整数的范围太大，这是需要考虑压缩的问题，压缩方案可以参考 RoaringBitmap 的压缩方式。
2、我们要解决的问题，也就是判断某个元素是否属于某个集合的问题。这里是否可以和出题方探讨是否严格要求100%判断正确。在允许很小误差概率的情景下（比如判断是否在垃圾邮件地址黑名单中），可以考虑 BloomFilter 。
BloomFilter 存储空间更加高效。1000w数据、0.1%的误差下需要的内存仅为 17.14M 
时间复杂度上，上面两种都是 hashmap的变种，因此为 o(1)。



平方根C代码，precision位数，小数点后6位是0.000001
double squareRoot(double a , double precision){
    double low,high,mid,tmp;
    if (a>1){
        low = 1;
        high = a;
    }else{
        low = 1;
        high = a;
    }
    while (low<=high) {
        mid = (low+high)/2.000;
        tmp = mid*mid;
        if (tmp-a <= precision && tmp-a >= precision*-1){
            return mid;
        }else if (tmp>a){
            high = mid;
        }else{
            low = mid;
        }
    }
    return -1.000;
}
int main(int argc, const char * argv[]) {
    double num = squareRoot(2, 0.000001);
    printf("%f",num);
    return 0;
}




关于求平方根的题，我知道一种比较巧妙的方法，那就是利用魔数，时间复杂度是 O(1)，根据我测试，精度大概能精确到 5 位小数，也还不错。下面是 c 语言代码
float q_rsqrt(float number) {
    int i;
    float x2, y;
    const float threehalfs = 1.5;
    x2 = number * 0.5;
    y = number;
    i = *(int*)&y;
    i = 0x5f3759df - (i >> 1);
    y = *(float*)&i;
    y = y * (threehalfs - (x2 * y * y));
    y = y * (threehalfs - (x2 * y * y));
    y = y * (threehalfs - (x2 * y * y));

    return 1.0 / y;
}




二分查找Python实现：
1、非递归方式
def bsearch(ls, value):
    low, high = 0, len(ls)-1
    while low <= high:
        mid = low + (high - low) // 2
        if ls[mid] == value:
            return mid
        elif ls[mid] < value:
            low = mid + 1
        else:
            high = mid - 1
    return -1

2、递归方式
def bsearch(ls, value):
    return bsearch_recursively(ls, 0, len(ls)-1, value)
    
def bsearch_recursively(ls, low, high, value):
    if low > high:
        return -1
    mid = low + (high - low) // 2
    if ls[mid] == value:
        return mid
    elif ls[mid] < value:
        return bsearch_recursively(ls, mid+1, high, value)
    else:
        return bsearch_recursively(ls, low, mid-1, value)





def sqrt(x):
  '''
  求平方根，精确到小数点后6位
  '''
  low = 0
  mid = x / 2
  high = x
  while abs(mid ** 2 - x) > 0.000001:
    if mid ** 2 < x:
      low = mid
    else:
      high = mid
    mid = (low + high) / 2
  return mid