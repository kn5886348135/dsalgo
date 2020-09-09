二分查找（下）：如何快速定位IP对应的省份地址？
===

&emsp;&emsp;百度的通过 IP 地址来查找 IP 归属地的功能。

&emsp;&emsp;在庞大的地址库中逐一比对 IP 地址所在的区间，是非常耗时的。假设我们有 12 万条这样的 IP 区间与归属地的对应关系，如何快速定位出一个 IP 地址的归属地呢？

&emsp;&emsp;唐纳德·克努特（Donald E.Knuth）在《计算机程序设计艺术》的第 3 卷《排序和查找》中说到：“尽管第一个二分查找算法于 1946 年出现，然而第一个完全正确的二分查找算法实现直到 1962 年才出现。”

&emsp;&emsp;在不存在重复元素的有序数组中，查找值等于给定值的元素，是一种最简单的二分查找算法。

&emsp;&emsp;二分查找的变形问题很多。

<img src="assert/VarietyOfBinarySearch.jpg" alt="二分查找的变形问题" style="zoom:50%;" />

&emsp;&emsp;假设数据是从小到大排列的。



##### 变体一：查找第一个值等于给定值的元素

&emsp;&emsp;有序数据集合中存在重复的数据，希望找到第一个值等于给定值的数据。

&emsp;&emsp;比如这样一个有序数组，其中，a[5]，a[6]，a[7] 的值都等于 8，是重复的数据。我们希望查找第一个等于 8 的数据，也就是下标是 5 的元素。

<img src="assert/VarietyOfBinarySearcha.jpg" alt="二分查找的变种问题一" style="zoom:50%;" />

&emsp;&emsp;100 个人写二分查找就会有 100 种写法，下面这个写法尽管简洁，理解起来却非常烧脑，也很容易写错。

```
public int bsearch(int[] a, int n, int value) {
  int low = 0;
  int high = n - 1;
  while (low <= high) {
    int mid = low + ((high - low) >> 1);
    if (a[mid] >= value) {
      high = mid - 1;
    } else {
      low = mid + 1;
    }
  }
 
  if (low < n && a[low]==value) return low;
  else return -1;
}
```

&emsp;&emsp;容易理解的版本。

```
public int bsearch(int[] a, int n, int value) {
  int low = 0;
  int high = n - 1;
  while (low <= high) {
    int mid =  low + ((high - low) >> 1);
    if (a[mid] > value) {
      high = mid - 1;
    } else if (a[mid] < value) {
      low = mid + 1;
    } else {
      if ((mid == 0) || (a[mid - 1] != value)) return mid;
      else high = mid - 1;
    }
  }
  return -1;
}
```

&emsp;&emsp;a[mid] 跟要查找的 value 的大小关系有三种情况：大于、小于、等于。对于 a[mid]>value 的情况，我们需要更新 high= mid-1；对于 a[mid]$<$value 的情况，我们需要更新 low=mid+1。这两点都很好理解。如果我们求解的是第一个值等于给定值的元素，当 a[mid] 等于要查找的值时，我们就需要确认一下这个 a[mid] 是不是第一个值等于给定值的元素。

&emsp;&emsp;第 11 行代码。如果 mid 等于 0，那这个元素已经是数组的第一个元素，那它肯定是我们要找的；如果 mid 不等于 0，但 a[mid] 的前一个元素 a[mid-1] 不等于 value，那也说明 a[mid] 就是我们要找的第一个值等于给定值的元素。

&emsp;&emsp;如果经过检查之后发现 a[mid] 前面的一个元素 a[mid-1] 也等于 value，那说明此时的 a[mid] 肯定不是我们要查找的第一个值等于给定值的元素。那我们就更新 high=mid-1，因为要找的元素肯定出现在 [low, mid-1] 之间。

&emsp;&emsp;很多人都觉得变形的二分查找很难写，主要原因是太追求第一种那样完美、简洁的写法。而对于我们做工程开发的人来说，代码易读懂、没 Bug，其实更重要。



##### 变体二：查找最后一个值等于给定值的元素

```
public int bsearch(int[] a, int n, int value) {
  int low = 0;
  int high = n - 1;
  while (low <= high) {
    int mid =  low + ((high - low) >> 1);
    if (a[mid] > value) {
      high = mid - 1;
    } else if (a[mid] < value) {
      low = mid + 1;
    } else {
      if ((mid == n - 1) || (a[mid + 1] != value)) return mid;
      else low = mid + 1;
    }
  }
  return -1;
}
```
&emsp;&emsp;第 11 行代码。如果 a[mid] 这个元素已经是数组中的最后一个元素了，那它肯定是我们要找的；如果 a[mid] 的后一个元素 a[mid+1] 不等于 value，那也说明 a[mid] 就是我们要找的最后一个值等于给定值的元素。

&emsp;&emsp;如果经过检查之后，发现 a[mid] 后面的一个元素 a[mid+1] 也等于 value，那说明当前的这个 a[mid] 并不是最后一个值等于给定值的元素。我们就更新 low=mid+1，因为要找的元素肯定出现在 [mid+1, high] 之间。



##### 变体三：查找第一个大于等于给定值的元素

&emsp;&emsp;在有序数组中，查找第一个大于等于给定值的元素。比如，数组中存储的这样一个序列：3，4，6，7，10。如果查找第一个大于等于 5 的元素，那就是 6。

```Java
public int bsearch(int[] a, int n, int value) {
  int low = 0;
  int high = n - 1;
  while (low <= high) {
    int mid =  low + ((high - low) >> 1);
    if (a[mid] >= value) {
      if ((mid == 0) || (a[mid - 1] < value)) return mid;
      else high = mid - 1;
    } else {
      low = mid + 1;
    }
  }
  return -1;
}
```
&emsp;&emsp;如果 a[mid] 小于要查找的值 value，那要查找的值肯定在 [mid+1, high] 之间，所以，我们更新 low=mid+1。

&emsp;&emsp;对于 a[mid] 大于等于给定值 value 的情况，我们要先看下这个 a[mid] 是不是我们要找的第一个值大于等于给定值的元素。如果 a[mid] 前面已经没有元素，或者前面一个元素小于要查找的值 value，那 a[mid] 就是我们要找的元素。这段逻辑对应的代码是第 7 行。

&emsp;&emsp;如果 a[mid-1] 也大于等于要查找的值 value，那说明要查找的元素在 [low, mid-1] 之间，所以，我们将 high 更新为 mid-1。



##### 变体四：查找最后一个小于等于给定值的元素

&emsp;&emsp;查找最后一个小于等于给定值的元素。比如，数组中存储了这样一组数据：3，5，6，8，9，10。最后一个小于等于 7 的元素就是 6。

```Java
public int bsearch7(int[] a, int n, int value) {
  int low = 0;
  int high = n - 1;
  while (low <= high) {
    int mid =  low + ((high - low) >> 1);
    if (a[mid] > value) {
      high = mid - 1;
    } else {
      if ((mid == n - 1) || (a[mid + 1] > value)) return mid;
      else low = mid + 1;
    }
  }
  return -1;
}
```


##### 解答开篇

&emsp;&emsp;如何快速定位出一个 IP 地址的归属地？

&emsp;&emsp;如果 IP 区间与归属地的对应关系不经常更新，我们可以先预处理这 12 万条数据，让其按照起始 IP 从小到大排序。IP 地址可以转化为 32 位的整型数。所以，可以将起始地址，按照对应的整型值的大小关系，从小到大进行排序。

&emsp;&emsp;然后，这个问题就可以转化为第四种变形问题“在有序数组中，查找最后一个小于等于某个给定值的元素”了。

&emsp;&emsp;当我们要查询某个 IP 归属地时，我们可以先通过二分查找，找到最后一个起始 IP 小于等于这个 IP 的 IP 区间，然后，检查这个 IP 是否在这个 IP 区间内，如果在，我们就取出对应的归属地显示；如果不在，就返回未查找到。



##### 内容小结

&emsp;&emsp;凡是用二分查找能解决的，绝大部分我们更倾向于用散列表或者二叉查找树。即便是二分查找在内存使用上更节省，但是毕竟内存如此紧缺的情况并不多。

&emsp;&emsp;实际上，求“值等于给定值”的二分查找确实不怎么会被用到，二分查找更适合用在“近似”查找问题，在这类问题上，二分查找的优势更加明显。比如这几种变体问题，用其他数据结构，比如散列表、二叉树，就比较难实现了。

&emsp;&emsp;变体的二分查找算法写起来非常烧脑，很容易因为细节处理不好而产生 Bug，这些容易出错的细节有：**终止条件、区间上下界更新方法、返回值选择**。



##### 课后思考

&emsp;&emsp;如果有序数组是一个循环有序数组，比如 4，5，6，1，2，3。针对这种情况，如何实现一个求“值等于给定值”的二分查找算法呢？



有三种方法查找循环有序数组
一、
 1. 找到分界下标，分成两个有序数组

 2. 判断目标值在哪个有序数据范围内，做二分查找
    二、

 3. 找到最大值的下标 x;

 4. 所有元素下标 +x 偏移，超过数组范围值的取模;

 5. 利用偏移后的下标做二分查找；

 6. 如果找到目标下标，再作 -x 偏移，就是目标值实际下标。
     两种情况最高时耗都在查找分界点上，所以时间复杂度是 O(N）。

      复杂度有点高，能否优化呢？
     
     三、
     我们发现循环数组存在一个性质：以数组中间点为分区，会将数组分成一个有序数组和一个循环有序数组。
      如果首元素小于 mid，说明前半部分是有序的，后半部分是循环有序数组；
      如果首元素大于 mid，说明后半部分是有序的，前半部分是循环有序的数组；
      如果目标元素在有序数组范围中，使用二分查找；
      如果目标元素在循环有序数组中，设定数组边界后，使用以上方法继续查找。
      时间复杂度为 O(logN)。

思考题对应leetcode 33题，大家可以去练习



将12w条归属地与IP区间的开始、结束存入数据库中。
数据库表ip_table有如下字段：area_name | start_ip | end_ip ，start_ip及end_ip 均建立索引
SQL语句：
select area_name from ip_table where input_ip >= start_ip and input_ip <= end_ip;
作者回复: 数据库可以 但性能会受限




用JavaScript实现的最基本的思考题：
array是传入的数组，value是要查找的值
思路是通过对比low,high的值来判断value所在的区间，不用多循环一遍找偏移量了~
```JavaScript
function search(array,value){
        let low = 0;
        let high = array.length - 1;
        while(low <= high){
            let mid = low + ((high - low) >> 1);
            if(value == array[low]) return low;
            if(value == array[high]) return high;
            if(value == array[mid]) return mid;
    
            if(value > array[mid] && value > array[high] && array[mid] < array[low]){
                high = mid - 1;
            }else if(value < array[mid] && value < array[low] && array[mid] < array[low]){
                high = mid - 1;
            }else if(value < array[mid] && value > array[low]){
                high = mid - 1;
            }else{
                low = mid + 1;
            }
        }
    
        return -1
    }
```







第一段代码有漏洞，且不说int能不能表示数组的下标问题，毕竟这个数组能越界说明相当庞大了；
主要问题在于，如果我给定的数大于任何一个数组元素，low就会等于n，n是数组越界后的第一个元素，如果它刚好是要查找的值呢？？
作者回复: 谢谢指正 我稍后改下






1.如果不知道分界点，找寻分界点没有意义，不如直接遍历。
2.如果知道分界点，查看在哪一边，然后二分法，或者偏移量计算,二分法
老师,我今天这种可以吗:
```Java
/**
     * 功能描述:查找第一个大于等于给定值的元素
     *
     * @param null
     * @return
     * @author xiongfan
     * @date 2018/12/7 9:43:00
     */
    public static int getFirstGreaterValue(int[] array,int value) {
        int low = 0;
        int high = array.length - 1;

        while (low <= high) {
            int mid = low + (high - low) >> 1;
            if (array[mid] < value) {
                low = mid + 1;
            } else if (array[mid] > value) {
                high = mid - 1;
            } else {
    
                if (mid == 0 || array[mid - 1] < array[mid]) {
                    return mid;
                }
                high = mid - 1;
    
            }
        }
    
        return low>array.length-1?-1:low;
    }
    
    /**
     * 功能描述:查找最后一个小于等于给定值的元素
     *
     * @param null
     * @return
     * @author xiongfan
     * @date 2018/12/7 10:03:00
     */
    public static int getLastLessValue(int[] array,int value) {
        int low = 0;
        int high = array.length - 1;
    
        while (low <= high) {
            int mid = low + (high - low) >> 1;
            if (array[mid] > value) {
                high = mid - 1;
            } else if (array[mid] < value) {
                low = mid + 1;
            } else {
                if (mid > array.length-1 || array[mid] < array[mid + 1]) {
                    return mid;
                }
                low = mid + 1;
            }
        }
    
        return high<0?-1:high;
    }
```

```Java
/**
     * 例如： 4 5 6 1 2 3
     * 循环数组的二分查找 总体时间复杂度O(n)
     */
    public static int forEqualsThan(int[] arr, int num) {
        if (arr[0] == num) {
            return 0;
        }
        int length = arr.length;
        int low = 0;
        int high = length - 1;
        //找到循环节点
        for (int i = 0; i < length; i++) {
            if (i == length - 1) {
                if (arr[i] > arr[0]) {
                    low = i;
                    high = 0;
                    break;
                }
            } else {
                if (arr[i] > arr[i + 1]) {
                    low = i;
                    high = low + 1;
                    break;
                }
            }
        }
        //判断第一个节点的大小位置，确定low和high的值，转变为正常有序的二分查找
        if (arr[0] < num) {
            high = low;
            low = 0;
        }
        if (arr[0] > num) {
            low = high;
            high = length - 1;
        }
        while (low <= high) {
            int index = low + ((high - low) >> 1);
            if (arr[index] > num) {
                high = index - 1;
            }
            if (arr[index] < num) {
                low = index + 1;
            }
            if (arr[index] == num) {
                return index;
            }
        }
        return -1;
    }
```



1. 先二分遍历找到分隔点index，特征是<pre, >=next;
2. 把数组分成二个部分，[0,index-1], [index,length-1];
3. 分别使用二分查找，找到给定的值。
时间复杂度为2*log(n). 不确定有什么更好的办法。





给原来的index加上偏移量。
比如原来的二分查找代码从0开始到n-1结束，现在为x到x - 1 (即n-1+x-n)。
x为开始循环处的索引，例子里为3 （1所在索引）。需要扫描一遍数组找到x，复杂度O(n)。其余和普通二分查找一样，需要多判断index not out of bound。如果索引超过n了要减n。
总的复杂度还是O(n)



```Java
public int search(int[] nums, int target) {
        if(nums.length ==0) return -1;
        if(nums.length ==1){
            if(nums[0] == target) return 0;
            else return -1;
        }
        int low = 0;
        int high = nums.length - 1;
        int index = subIndex(nums,low,high);
        if(index != -1){
            int val = binarySearch(nums,low,index,target);
            if (val != -1) return val;
            return binarySearch(nums,index+1,high,target);
        }
        return binarySearch(nums,low,high,target);
    }
    
    public static int subIndex(int [] nums,int low,int high){
        while (low <= high){
            int mid = low + ((high - low )>> 1);
            if(nums.length < 1) return -1;
            if(nums[mid] > nums[mid+1]) return mid;
            else if( nums[mid] < nums[low] ) high = mid ;
            else if (nums[mid] > nums[high]) low = mid ;
            else return -1; 
        }
        return -1;
    }
    
    public static int binarySearch(int[] nums,int low,int high,int target){
        while (low <= high){
            int mid = low + ((high - low)>>1);
            if(nums[mid] == target) return mid;
            else if (nums[mid] < target) low = mid + 1;
            else high = mid -1; 
        }
        return -1;
    }
```




二分的四种变种写法。个人觉得都是分三种情况进行讨论，再多注意判断边界值，三种情况为：
a[mid]<value
a[mid]=value
a[mid]>value； 



置顶的同学的思路一，即先找分界再判断在哪个数组，再二分，其实是可以做到O(Log N)的，找分界的点的规则就是找到首个小于a[0]的元素，思路用老师4个转换问题的解法就可以。




可以考虑将数组分为N个有序数组，分别进行二分查找。
```Java
 public int circleBinarySearch(int[] a, int value){
        int low = 0, high=0;
        for(int i=0;i<a.length-1;i++){
            //找到有序数组的下标
            if(a[i]<a[i+1]){
                high=i+1;
            }else{
                //有序数组到顶，二分查找
                int i1 = binarySearch(low, high, a, value);
                if(-1 != i1){
                    return i1;
                }else{
                    low = high+1;
                }
            }
            //high已经到最后一个位置
            if(a.length-1 == high){
                return binarySearch(low, high, a, value);
            }
        }
        return -1;
    }

    private int binarySearch(int low, int high, int[] a, int value){
        for(;low<=high;){
            int middle = low+((high-low)>>1);
            if(a[middle] == value){
                return middle;
            }
            if(a[middle] > value){
                high = middle -1;
            }else{
                low = middle +1;
            }
        }
        return -1;
    }
```






```Python
#查找第一个值等于给定值的元素
def bsearchFirst(nums, val):
    low, high = 0, len(nums) - 1
    while low <= high:
        mid = low + ((high - low) >> 1)
        if nums[mid] >= val:
            high = mid - 1
        else:
            low = mid + 1
    if nums[low] == val:
        return low
    else:
        return None

#查找最后一个值等于给定值的元素
def bsearchLast(nums, val):
    low, high = 0, len(nums) - 1
    while low <= high:
        mid = low + ((high - low) >> 1)
        if nums[mid] <= val:
            low = mid + 1
        else:
            high = mid - 1
    if nums[high] == val:
        return high
    else:
        return None

#查找第一个大于等于给定值的元素
def bsearchFirstLargerEqual(nums, val):
    low, high = 0, len(nums) - 1
    while low <= high:
        mid = low + ((high - low) >> 1)
        if nums[mid] >= val:
            high = mid - 1
        else:
            low = mid + 1
    if nums[low] >= val:
        return low
    else:
        return None

#查找最后一个小于等于给定值的元素
def bSearchLastsmallerEqual(nums, val):
    low, high = 0, len(nums) - 1
    while low <= high:
        mid = low + ((high - low) >> 1)
        if nums[mid] <= val:
            low = mid + 1
        else:
            high = mid - 1
    if nums[high] <= val:
        return high
    else:
        return None

#循环数组的二分查找
def bSearchRecycle(nums, val):
    n, offset = len(nums), 3
    low, high = 0, n - 1 
    while low <= high:
        mid = low + ((high - low) >> 1)
        midIdx = (mid + offset) % n
        if nums[midIdx] == val:
            return midIdx
        elif nums[midIdx] < val:
            low = mid + 1
        else:
            high = mid - 1
    return None

if __name__ == '__main__':
    nums = [0, 0, 2, 2, 3, 3]
    val = 1
    print(bSearchLastsmallerEqual(nums, val))

    nums = [4, 5, 6, 1, 2, 3]
    val = 3
    print(bSearchRecycle(nums, val))
```



关于循环有序数组的问题，假设array无重复元素的话，我们可以先二分法将array分成两个递增数组，再分别采用二分法。

    public static int search(int x, int[] array) {
        if (null == array || array.length == 0) {
            return -1;
        }
        /** Step1. 先找到循环队列新一轮从小变大的起始位置*/
        int low = 1;
        int high = array.length - 1;
        int mid;
        int firstVal = array[0];
        /*到firstVal之后第一个<firstVal的position */
        int dividePos = -1;
        while(low <= high) {
            mid = low + ((high - low)>>1);
            if (array[mid] > firstVal) {
                low = mid + 1;
            } else {
                if (mid == 1 || array[mid - 1] > firstVal) {
                    dividePos = mid;
                    break;
                } else {
                    high = mid -1;
                }
            }
        }
        /** Step2. 对dividePos两边分别进行二分搜索 */
        int targetPos = -1;
        if (dividePos >= 1) {
            targetPos = bSearch(x, array, 0, dividePos - 1);
            if (-1 != targetPos) {
                return targetPos;
            } else {
                targetPos = bSearch(x, array, dividePos, array.length - 1);
            };
        } else {
            targetPos = bSearch(x, array, 0, array.length - 1);
        }
        return targetPos;
    }
    
    /** 普通的二分搜索 */
    public static int bSearch(int x, int[] array, int low, int high) {
        if (null == array || array.length == 0 || low < 0 || high >= array.length) {
            return -1;
        }
        int mid;
        while(low <= high) {
            mid = low + ((high - low)>>1);
            if (array[mid] > x) {
                high = mid - 1;
            } else if (array[mid] < x) {
                low = mid + 1;
            } else {
                return mid;
            }
        }
        return -1;
    }


