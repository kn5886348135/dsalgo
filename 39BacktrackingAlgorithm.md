回溯算法：从电影《蝴蝶效应》中学习回溯算法的核心思想
===

&emsp;&emsp;深度优先搜索算法利用的是回溯算法思想。这个算法思想非常简单，但是应用却非常广泛。它除了用来指导像深度优先搜索这种经典的算法设计之外，还可以用在很多实际的软件开发场景中，比如正则表达式匹配、编译原理中的语法分析等。

&emsp;&emsp; 很多经典的数学问题都可以用回溯算法解决，比如数独、八皇后、0-1 背包、图的着色、旅行商问题、全排列等等。

### 如何理解“回溯算法”？

&emsp;&emsp;回溯算法很多时候都应用在“搜索”这类问题上。不过这里说的搜索，并不是狭义的指前面讲过的图的搜索算法，而是在一组可能的解中，搜索满足期望的解。

&emsp;&emsp;回溯的处理思想，有点类似枚举搜索。枚举所有的解，找到满足期望的解。为了有规律地枚举所有可能的解，避免遗漏和重复，把问题求解的过程分为多个阶段。每个阶段，都会面对一个岔路口，先随意选一条路走，当发现这条路走不通的时候（不符合期望的解），就回退到上一个岔路口，另选一种走法继续走。

&emsp;&emsp;一个经典的回溯例子，八皇后问题。

&emsp;&emsp;有一个 8x8 的棋盘，希望往里放 8 个棋子（皇后），每个棋子所在的行、列、对角线都不能有另一个棋子。第一幅图是满足条件的一种方法，第二幅图是不满足条件的。八皇后问题就是期望找到所有满足这种要求的放棋子方式。

<img src="assert/EightQueenProblem.jpg" alt="八皇后问题" style="zoom:50%;" />

&emsp;&emsp;把这个问题划分成 8 个阶段，依次将 8 个棋子放到第一行、第二行、第三行……第八行。在放置的过程中，不停地检查当前的方法，是否满足要求。如果满足，则跳到下一行继续放置棋子；如果不满足，那就再换一种方法，继续尝试。

&emsp;&emsp;回溯算法非常适合用递归代码实现。

```Java
int[] result = new int[8];// 全局或成员变量, 下标表示行, 值表示 queen 存储在哪一列
public void cal8queens(int row) { // 调用方式：cal8queens(0);
  if (row == 8) { // 8 个棋子都放置好了，打印结果
    printQueens(result);
    return; // 8 行棋子都放好了，已经没法再往下递归了，所以就 return
  }
  for (int column = 0; column < 8; ++column) { // 每一行都有 8 中放法
    if (isOk(row, column)) { // 有些放法不满足要求
      result[row] = column; // 第 row 行的棋子放到了 column 列
      cal8queens(row+1); // 考察下一行
    }
  }
}

private boolean isOk(int row, int column) {// 判断 row 行 column 列放置是否合适
  int leftup = column - 1, rightup = column + 1;
  for (int i = row-1; i >= 0; --i) { // 逐行往上考察每一行
    if (result[i] == column) return false; // 第 i 行的 column 列有棋子吗？
    if (leftup >= 0) { // 考察左上对角线：第 i 行 leftup 列有棋子吗？
      if (result[i] == leftup) return false;
    }
    if (rightup < 8) { // 考察右上对角线：第 i 行 rightup 列有棋子吗？
      if (result[i] == rightup) return false;
    }
    --leftup; ++rightup;
  }
  return true;
}

private void printQueens(int[] result) { // 打印出一个二维矩阵
  for (int row = 0; row < 8; ++row) {
    for (int column = 0; column < 8; ++column) {
      if (result[row] == column) System.out.print("Q ");
      else System.out.print("* ");
    }
    System.out.println();
  }
  System.out.println();
}
```

### 两个回溯算法的经典应用

&emsp;&emsp;回溯算法的理论知识很容易弄懂。不过，对于新手来说，比较难的是用递归来实现。

1. 0-1 背包

&emsp;&emsp;0-1 背包是非常经典的算法问题，很多场景都可以抽象成这个问题模型。这个问题的经典解法是动态规划，不过还有一种简单但没有那么高效的解法，那就是回溯算法。

&emsp;&emsp;0-1 背包问题有很多变体，这里介绍一种比较基础的。有一个背包，背包总的承载重量是 Wkg。现在有 n 个物品，每个物品的重量不等，并且不可分割。现在期望选择几件物品，装载到背包中。在不超过背包所能装载重量的前提下，如何让背包中物品的总重量最大？

&emsp;&emsp;这个背包问题，物品是不可分割的，要么装要么不装，所以叫 0-1 背包问题。显然，这个问题已经无法通过贪心算法来解决。

&emsp;&emsp;对于每个物品来说，都有两种选择，装进背包或者不装进背包。对于 n 个物品来说，总的装法就有 $2^{n}$ 种，去掉总重量超过 Wkg 的，从剩下的装法中选择总重量最接近 Wkg 的。不过，如何才能不重复地穷举出这 2^n^种装法呢？

&emsp;&emsp;这里就可以用回溯的方法。可以把物品依次排列，整个问题就分解为了 n 个阶段，每个阶段对应一个物品怎么选择。先对第一个物品进行处理，选择装进去或者不装进去，然后再递归地处理剩下的物品。

&emsp;&emsp;这里还稍微用到了一点搜索剪枝的技巧，就是当发现已经选择的物品的重量超过 Wkg 之后，就停止继续探测剩下的物品。

```Java
public int maxW = Integer.MIN_VALUE; // 存储背包中物品总重量的最大值
// cw 表示当前已经装进去的物品的重量和；i 表示考察到哪个物品了；
// w 背包重量；items 表示每个物品的重量；n 表示物品个数
// 假设背包可承受重量 100，物品个数 10，物品重量存储在数组 a 中，那可以这样调用函数：
// f(0, 0, a, 10, 100)
public void f(int i, int cw, int[] items, int n, int w) {
  if (cw == w || i == n) { // cw==w 表示装满了 ;i==n 表示已经考察完所有的物品
    if (cw > maxW) maxW = cw;
    return;
  }
  f(i+1, cw, items, n, w);
  if (cw + items[i] <= w) {// 已经超过可以背包承受的重量的时候，就不要再装了
    f(i+1,cw + items[i], items, n, w);
  }
}
```
2. 正则表达式

&emsp;&emsp;正则表达式里最重要的一种算法思想就是回溯。

&emsp;&emsp;正则表达式中，最重要的就是通配符，通配符结合在一起，可以表达非常丰富的语义。为了方便讲解，我假设正表达式中只包含“$*$”和“$?$”这两种通配符，并且对这两个通配符的语义稍微做些改变，其中，“*”匹配任意多个（大于等于 0 个）任意字符，“?”匹配零个或者一个任意字符。基于以上背景假设，看下，如何用回溯算法，判断一个给定的文本，能否跟给定的正则表达式匹配？

&emsp;&emsp;依次考察正则表达式中的每个字符，当是非通配符时，就直接跟文本的字符进行匹配，如果相同，则继续往下处理；如果不同，则回溯。

&emsp;&emsp;如果遇到特殊字符的时候，就有多种处理方式了，也就是所谓的岔路口，比如“$*$”有多种匹配方案，可以匹配任意个文本串中的字符，就先随意的选择一种匹配方案，然后继续考察剩下的字符。如果中途发现无法继续匹配下去了，就回到这个岔路口，重新选择一种匹配方案，然后再继续匹配剩下的字符。

```Java
public class Pattern {
  private boolean matched = false;
  private char[] pattern; // 正则表达式
  private int plen; // 正则表达式长度

  public Pattern(char[] pattern, int plen) {
    this.pattern = pattern;
    this.plen = plen;
  }

  public boolean match(char[] text, int tlen) { // 文本串及长度
    matched = false;
    rmatch(0, 0, text, tlen);
    return matched;
  }

  private void rmatch(int ti, int pj, char[] text, int tlen) {
    if (matched) return; // 如果已经匹配了，就不要继续递归了
    if (pj == plen) { // 正则表达式到结尾了
      if (ti == tlen) matched = true; // 文本串也到结尾了
      return;
    }
    if (pattern[pj] == '*') { // * 匹配任意个字符
      for (int k = 0; k <= tlen-ti; ++k) {
        rmatch(ti+k, pj+1, text, tlen);
      }
    } else if (pattern[pj] == '?') { // ? 匹配 0 个或者 1 个字符
      rmatch(ti, pj+1, text, tlen);
      rmatch(ti+1, pj+1, text, tlen);
    } else if (ti < tlen && pattern[pj] == text[ti]) { // 纯字符匹配才行
      rmatch(ti+1, pj+1, text, tlen);
    }
  }
}
```

### 内容小结

&emsp;&emsp;回溯算法的思想非常简单，大部分情况下，都是用来解决广义的搜索问题，也就是，从一组可能的解中，选择出一个满足要求的解。回溯算法非常适合用递归来实现，在实现的过程中，剪枝操作是提高回溯效率的一种技巧。利用剪枝，并不需要穷举搜索所有的情况，从而提高搜索效率。

&emsp;&emsp;尽管回溯算法的原理非常简单，但是却可以解决很多问题，比如深度优先搜索、八皇后、0-1 背包问题、图的着色、旅行商问题、数独、全排列、正则表达式匹配等等。

### 课后思考

&emsp;&emsp;现在对 0-1 背包问题稍加改造，如果每个物品不仅重量不同，价值也不同。如何在不超过背包重量的情况下，让背包中的总价值最大？





回溯算法本质上就是枚举，优点在于其类似于摸着石头过河的查找策略，且可以通过剪枝少走冤枉路。它可能适合应用于缺乏规律，或还不了解其规律的搜索场景中。
作者回复: 👍



0-1 背包问题的回溯实现技巧：

第 11 行的递归调用表示不选择当前物品，直接考虑下一个（第 i+1 个），故 cw 不更新

第 13 行的递归调用表示选择了当前物品，故考虑下一个时，cw 通过入参更新为 cw + items[i]

函数入口处的 if 分支表明递归结束条件，并保证 maxW 跟踪所有选择中的最大值






0-1背包问题根据老师下边这句话的讲解，代码再加两行注释就非常容易理解了

可以把物品依次排列，整个问题就分解为了 n 个阶段，每个阶段对应一个物品怎么选择。先对第一个物品进行处理，选择装进去或者不装进去，然后再递归地处理剩下的物品。


```Java
public int maxW = Integer.MIN_VALUE; // 存储背包中物品总重量的最大值
// cw 表示当前已经装进去的物品的重量和；i 表示考察到哪个物品了；
// w 背包重量；items 表示每个物品的重量；n 表示物品个数
// 假设背包可承受重量 100，物品个数 10，物品重量存储在数组 a 中，那可以这样调用函数：
// f(0, 0, a, 10, 100)
public void f(int i, int cw, int[] items, int n, int w) {
  if (cw == w || i == n) { // cw==w 表示装满了 ;i==n 表示已经考察完所有的物品
    if (cw > maxW) maxW = cw;
    return;
  }
  f(i+1, cw, items, n, w); //当前物品不装进背包
  if (cw + items[i] <= w) {// 已经超过可以背包承受的重量的时候，就不要再装了
    f(i+1,cw + items[i], items, n, w); //当前物品装进背包
  }
}
```







0-1背包的递归代码里第11行非常巧妙，它借助回溯过程，实现了以每一个可能的物品，作为第一个装入背包的，以尝试所有物品组合。但如果仅按从前向后执行的顺序看，是不太容易发现这一点的。






看不懂背包问题代码同学，请好好仔细看看下面这句话，再结合代码你就看懂了

可以把物品依次排列，整个问题就分解为了 n 个阶段，每个阶段对应一个物品怎么选择。先对第一个物品进行处理，选择装进去或者不装进去，然后再递归地处理剩下的物品。
作者回复: 👍




总觉得背包问题11行代码应该写在14行后，那个if条件后面。





老师，你好，背包问题，貌似只记录了可以放进去的最大值，没有记录放进最大值对应的放法，我稍微
改了下，算出了最大值对应的所有放法，不知道可行不，希望老师回复下。
```Java
private int maxW = Integer.MIN_VALUE; // 存储背包中物品总重量的最大值
    // 下标表示物品序号，值表示是否放进背包:1放，0不放
    private int[] currentAnswer;
    //存储所有解(map key表示放进去的重量，value表示对应重量的物品放法)，
    //最终所有最优解为bestAnswerMap.get(maxW)
    private Map<Integer, List<int[]>> bestAnswerMap = new HashMap();

    // cw 表示当前已经装进去的物品的重量和；i 表示考察到哪个物品了；
    // w 背包重量；items 表示每个物品的重量；n 表示物品个数
    // 假设背包可承受重量 100，物品个数 10，物品重量存储在数组 a 中，那可以这样调用函数：
    // f(0, 0, a, 10, 100)
    public void f(int i, int cw, int[] items, int n, int w) {
        if(currentAnswer == null){
            currentAnswer = new int[n];
        }
    
        if (cw == w || i == n) { // cw==w 表示装满了 ;i==n 表示已经考察完所有的物品
            if (cw >= maxW) {
                maxW = cw;
                int[] bestAnswer = new int[currentAnswer.length];
                for(int j=0; j<currentAnswer.length; j++){
                    bestAnswer[j] = currentAnswer[j];
                }
                if(bestAnswerMap.containsKey(cw)){
                    bestAnswerMap.get(cw).add(bestAnswer);
                }else{
                    List<int[]> list = new ArrayList<int[]>();
                    list.add(bestAnswer);
                    bestAnswerMap.put(cw, list);
                }
            }
            return;
        }
        currentAnswer[i] = 0;
        f(i+1, cw, items, n, w);
        if (cw + items[i] <= w) {// 已经超过可以背包承受的重量的时候，就不要再装了
            currentAnswer[i] = 1;
            f(i+1,cw + items[i], items, n, w);
        }
    }
```
最终maxW 对应的所有最优解为bestAnswerMap.get(maxW)




回溯就是暴力枚举的解法吧？遍历所有情况，当满足情况就停止遍历（剪枝）。
作者回复: 是的





流程大概就是：
第一个不放，第二个不放，……，第n-1个不放，第n个不放。
第一个不放，第二个不放，……，第n-1个不放，
第n个放。
第一个不放，第二个不放， ……，第n-1个放，
第n个不放。
第一个不放，第二个不放， ……，第n-1个放，
第n个放。
……
以此类推
感觉这些问题就是将概率论知识转化成代码实现。





0-1背包python实现：
```Python
maxW = -1 # tracking the max weight

def backpack(i, cw, items, w):
    ''' 
    # i: the ith item, integer
    # cw: current weight, integer
    # items: python list of item weights
    # w: upper limit weight the backpack can load
    '''
    global maxW
    
    if cw==w or i==len(items): # base case
        if cw > maxW:
            maxW = cw
        return
    
    # There are 2 states, traverse both!!!
    backpack(i+1, cw, items, w) # do not choose
    if (cw + items[i] <= w):
        backpack(i+1, cw+items[i], items, w) # choose

# how to use
items = [2, 2, 4, 6, 3]
backpack(0, 0, items, 10)
print(maxW)
```










今天正好发现一个算法的示例，大家结合看看，应该能更好的理解https://algorithm-visualizer.org/backtracking/n-queens-problem









背包问题：if (cw > maxW) maxW = cw;这样不是超重了嘛？还有代码里cw一直没有赋值操作啊




老师，我经过查资料，找到，其实判断是否在一条斜线上还有更加简便的做法，就是如果行互减的绝对值等于列互减的绝对值，那么就是在一条斜线上的。
if (Math.abs(row - i) == Math.abs(column - result[i])) {
                return false;
            }
作者回复: 是的，我写的时候也查过资料。





0-1背包问题为什么不是这样的代码呢？不超过就装，超过了就不装？
if (items[i] + cw <= w) {
        backpack(i + 1, cw + items[i], items, n, w)
} else {
backpack(i + 1, cw, items, n, w)
}





老师，这中类型的题有时候跟着感觉也就写出来了，运行也ok，但是不知道为什么对，老师你有这种感觉吗？




0-1 背包那个问题，没有地方存放，哪个地方存储选取的数组呢。结果输出在哪？希望老师指导下
