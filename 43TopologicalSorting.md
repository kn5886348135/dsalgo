拓扑排序：如何确定代码源文件的编译依赖关系？
===

&emsp;&emsp;如何确定代码源文件的编译依赖关系？

&emsp;&emsp;一个完整的项目往往会包含很多代码源文件。编译器在编译整个项目的时候，需要按照依赖关系，依次编译每个源文件。比如，A.cpp 依赖 B.cpp，那在编译的时候，编译器需要先编译 B.cpp，才能编译 A.cpp。

&emsp;&emsp;编译器通过分析源文件或者程序员事先写好的编译配置文件（比如 Makefile 文件），来获取这种局部的依赖关系。那编译器又该如何通过源文件两两之间的局部依赖关系，确定一个全局的编译顺序呢？

<img src="assert\SourceFileDependenciesA.jpg" alt="源文件的依赖关系" style="zoom:50%;" />

#### 算法解析

&emsp;&emsp;这个问题的解决思路与“图”这种数据结构的一个经典算法“拓扑排序算法”有关。那什么是拓扑排序呢？

&emsp;&emsp;在穿衣服的时候都有一定的顺序，可以把这种顺序想成，衣服与衣服之间有一定的依赖关系。比如说，你必须先穿袜子才能穿鞋，先穿内裤才能穿秋裤。假设现在有八件衣服要穿，它们之间的两两依赖关系已经很清楚了，那如何安排一个穿衣序列，能够满足所有的两两之间的依赖关系？

&emsp;&emsp;这就是个拓扑排序问题。从这个例子中，你应该能想到，在很多时候，拓扑排序的序列并不是唯一的。你可以看我画的这幅图，我找到了好几种满足这些局部先后关系的穿衣序列。

<img src="assert\SourceFileDependenciesB.jpg" alt="源文件的依赖关系" style="zoom:50%;" />

&emsp;&emsp;可以把源文件与源文件之间的依赖关系，抽象成一个有向图。每个源文件对应图中的一个顶点，源文件之间的依赖关系就是顶点之间的边。

&emsp;&emsp;如果 a 先于 b 执行，也就是说 b 依赖于 a，那么就在顶点 a 和顶点 b 之间，构建一条从 a 指向 b 的边。而且，这个图不仅要是有向图，还要是一个有向无环图，也就是不能存在像 a->b->c->a 这样的循环依赖关系。因为图中一旦出现环，拓扑排序就无法工作了。实际上，拓扑排序本身就是基于有向无环图的一个算法。

```Java
public class Graph {
  private int v; // 顶点的个数
  private LinkedList<Integer> adj[]; // 邻接表

  public Graph(int v) {
    this.v = v;
    adj = new LinkedList[v];
    for (int i=0; i<v; ++i) {
      adj[i] = new LinkedList<>();
    }
  }

  public void addEdge(int s, int t) { // s 先于 t，边 s->t
    adj[s].add(t);
  }
}
```
&emsp;&emsp;拓扑排序有两种实现方法，它们分别是**Kahn 算法**和**DFS 深度优先搜索算法**。

##### 1.Kahn 算法

&emsp;&emsp;Kahn 算法实际上用的是贪心算法思想，思路非常简单、好懂。

&emsp;&emsp;定义数据结构的时候，如果 s 需要先于 t 执行，那就添加一条 s 指向 t 的边。所以，如果某个顶点入度为 0， 也就表示，没有任何顶点必须先于这个顶点执行，那么这个顶点就可以执行了。

&emsp;&emsp;先从图中，找出一个入度为 0 的顶点，将其输出到拓扑排序的结果序列中（对应代码中就是把它打印出来），并且把这个顶点从图中删除（也就是把这个顶点可达的顶点的入度都减 1）。循环执行上面的过程，直到所有的顶点都被输出。最后输出的序列，就是满足局部依赖关系的拓扑排序。

&emsp;&emsp;这段代码实现更有技巧一些，并没有真正删除顶点的操作。

```Java
public void topoSortByKahn() {
  int[] inDegree = new int[v]; // 统计每个顶点的入度
  for (int i = 0; i < v; ++i) {
    for (int j = 0; j < adj[i].size(); ++j) {
      int w = adj[i].get(j); // i->w
      inDegree[w]++;
    }
  }
  LinkedList<Integer> queue = new LinkedList<>();
  for (int i = 0; i < v; ++i) {
    if (inDegree[i] == 0) queue.add(i);
  }
  while (!queue.isEmpty()) {
    int i = queue.remove();
    System.out.print("->" + i);
    for (int j = 0; j < adj[i].size(); ++j) {
      int k = adj[i].get(j);
      inDegree[k]--;
      if (inDegree[k] == 0) queue.add(k);
    }
  }
}
```
##### 2.DFS 算法

&emsp;&emsp;拓扑排序也可以用深度优先搜索来实现，更加确切的说法应该是深度优先遍历，遍历图中的所有顶点，而非只是搜索一个顶点到另一个顶点的路径。

```Java
public void topoSortByDFS() {
  // 先构建逆邻接表，边 s->t 表示，s 依赖于 t，t 先于 s
  LinkedList<Integer> inverseAdj[] = new LinkedList[v];
  for (int i = 0; i < v; ++i) { // 申请空间
    inverseAdj[i] = new LinkedList<>();
  }
  for (int i = 0; i < v; ++i) { // 通过邻接表生成逆邻接表
    for (int j = 0; j < adj[i].size(); ++j) {
      int w = adj[i].get(j); // i->w
      inverseAdj[w].add(i); // w->i
    }
  }
  boolean[] visited = new boolean[v];
  for (int i = 0; i < v; ++i) { // 深度优先遍历图
    if (visited[i] == false) {
      visited[i] = true;
      dfs(i, inverseAdj, visited);
    }
  }
}

private void dfs(
    int vertex, LinkedList<Integer> inverseAdj[], boolean[] visited) {
  for (int i = 0; i < inverseAdj[vertex].size(); ++i) {
    int w = inverseAdj[vertex].get(i);
    if (visited[w] == true) continue;
    visited[w] = true;
    dfs(w, inverseAdj, visited);
  } // 先把 vertex 这个顶点可达的所有顶点都打印出来之后，再打印它自己
  System.out.print("->" + vertex);
}
```
&emsp;&emsp;这个算法包含两个关键部分。

1. 通过邻接表构造逆邻接表。邻接表中，边 s->t 表示 s 先于 t 执行，也就是 t 要依赖 s。在逆邻接表中，边 s->t 表示 s 依赖于 t，s 后于 t 执行。为什么这么转化呢？这个跟这个算法的实现思想有关。

2. 这个算法的核心，也就是递归处理每个顶点。对于顶点 vertex 来说，先输出它可达的所有顶点，也就是说，先把它依赖的所有的顶点输出了，然后再输出自己。

&emsp;&emsp; Kahn 算法和 DFS 算法求拓扑排序的时间复杂度分别是多少呢？

&emsp;&emsp;从 Kahn 代码中可以看出来，每个顶点被访问了一次，每个边也都被访问了一次，所以，Kahn 算法的时间复杂度就是 O(V+E)（V 表示顶点个数，E 表示边的个数）。

&emsp;&emsp;DFS 算法的时间复杂度之前分析过。每个顶点被访问两次，每条边都被访问一次，所以时间复杂度也是 O(V+E)。

&emsp;&emsp;注意，这里的图可能不是连通的，有可能是有好几个不连通的子图构成，所以，E 并不一定大于 V，两者的大小关系不确定。所以，在表示时间复杂度的时候，V、E 都要考虑在内。

### 总结引申

&emsp;&emsp;图的定义和存储、图的广度和深度优先搜索，图的拓扑排序。

&emsp;&emsp;拓扑排序应用非常广泛，解决的问题的模型也非常一致。凡是需要通过局部顺序来推导全局顺序的，一般都能用拓扑排序来解决。除此之外，拓扑排序还能检测图中环的存在。对于 Kahn 算法来说，如果最后输出出来的顶点个数，少于图中顶点个数，图中还有入度不是 0 的顶点，那就说明，图中存在环。

&emsp;&emsp;关于图中环的检测，在递归那一节讲过一个例子，在查找最终推荐人的时候，可能会因为脏数据，造成存在循环推荐，比如，用户 A 推荐了用户 B，用户 B 推荐了用户 C，用户 C 又推荐了用户 A。如何避免这种脏数据导致的无限递归？

&emsp;&emsp;实际上，这就是环的检测问题。因为每次都只是查找一个用户的最终推荐人，所以，并不需要动用复杂的拓扑排序算法，而只需要记录已经访问过的用户 ID，当用户 ID 第二次被访问的时候，就说明存在环，也就说明存在脏数据。

```Java
HashSet<Integer> hashTable = new HashSet<>(); // 保存已经访问过的 actorId
long findRootReferrerId(long actorId) {
  if (hashTable.contains(actorId)) { // 存在环
    return;
  }
  hashTable.add(actorId);
  Long referrerId = 
       select referrer_id from [table] where actor_id = actorId;
  if (referrerId == null) return actorId;
  return findRootReferrerId(referrerId);
}
```
&emsp;&emsp;如果把这个问题改一下，想要知道，数据库中的所有用户之间的推荐关系了，有没有存在环的情况。这个问题，就需要用到拓扑排序算法了。把用户之间的推荐关系，从数据库中加载到内存中，然后构建成今天讲的这种有向图数据结构，再利用拓扑排序，就可以快速检测出是否存在环了。

### 课后思考

&emsp;&emsp;在今天的讲解中，用图表示依赖关系的时候，如果 a 先于 b 执行，就画一条从 a 到 b 的有向边；反过来，如果 a 先于 b，画一条从 b 到 a 的有向边，表示 b 依赖 a，那 Kahn 算法和 DFS 算法还能否正确工作呢？如果不能，应该如何改造一下呢？

&emsp;&emsp;今天讲了两种拓扑排序算法的实现思路，Kahn 算法和 DFS 深度优先搜索算法，如果换做 BFS 广度优先搜索算法，还可以实现吗？

  





算法的由来，也就是背景，某个算法的创造者为什么会发明某个算法，怎么能够发明出某个算法

 





1. a先于b执行，也就说b依赖于a，b指向a，这样构建有向无环图时，要找到出度为0的顶点，然后删除
2. BFS也能实现，因为遍历只是实现拓扑排序的一个“辅助手段”，本质上是帮助找到优先执行的顶点







课后思考里“BFS 深度优先搜索算法”是否应该是“BFS 广度优先搜索算法”？BFS: Breadth-first Search






1.kahn算法找出度为0的节点删除。dfs算法直接用正邻接表即可。

2. BFS也可以。其实与DFS一样，BFS也是从某个节点开始，找到所有与其相连通的节点。区别在于BFS是一层一层找（递归函数在for循环外），DFS是先一杆子插到底，再回来插第二条路、第三条路等等（递归函数在for循环内）。






刚解决完工作中类似的问题 老师的文章就来了，然后才知道那个算法叫kahn






老师，好像数据结构少了B+树的讲解啊，B+不准备讲吗？





NeverMore
1、反过来的话计算的就不是入度了，可以用出度来判断；
2、BFS的话，则需要记录上一个节点是哪个，可以实现，但是比DFS要麻烦些。
还请老师指点。
老师之后能不能给思考题一个答疑？






老师，我觉得这里BFS和Kahn算法基本可以说是一样的，本身Kahn贪婪算法运用queue实现的过程就是一个典型的BFS范式。采用BFS就应该按照入度一层一层遍历，一层遍历完了的同时把下一层的顶点push进入queue中。





可汗算法里面，需要把入度所有的都存到数组中吗？
当求出所有节点的入度时，求出入度最大的数，然后依次从0到这个最大数依次打印节点
这样的结果也是正确的吧





https://leetcode-cn.com/problems/course-schedule-ii/





问题1:khan改为出度为0，深度优先搜索改为先打印
问题2:广度优先遍历节点保存到双向链表中，然后逆序输出






编译过程如果存在两个互相依赖的类呢？






我怎么觉得这个kahn算法其实就是BFS算法





老师您好，还看到过另一个深度优先遍历的方法，是通过将节点涂不同的颜色判断是否在遍历的时候遇到了环，这种方法看着应该很明了，但是好像很少看到有人这么写程序，不知道是什么原因呢？
作者回复: 那个比较大而全，所以不经常用。







思考题：
1、思路是找出度为0的节点然后打印出来。kahn算法可以和上面的类似通过构建逆临接表找出入度为0的节点，其余都一样。dfs和讲解中代码一致只是不需要再构建逆邻接表了。
2、bfs解的思路感觉就和kahn一样。找入度为0的节点放入queue再取出找到它的邻接节点入度减1，如果减1后等于0再放入queue。依此类推。






kahn算法中统计每个顶点的入度，有两层循环，时间复杂度为什么不是O(V*E)呢？
作者回复: 第一层是v 但第二层不是E呢






这个问题有没有可能通过hashmap来做？用每一个事件之前的一个事件作为key, 事件本身作为value，然后遍历一遍