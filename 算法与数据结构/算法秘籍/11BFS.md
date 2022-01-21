# BFS

BFS 算法起源于 **⼆叉树的层序遍历**，其核⼼是利⽤ **队列** 这种数据结构。

且 BFS 算法常⻅于求最值的场景，因为 BFS 的算法逻辑保证了算法第⼀次到达⽬标时的代价是最⼩的。

BFS 的核心思想应该不难理解的，就是把一些问题抽象成图，从一个点开始，向四周开始扩散。一般来说，我们写 BFS 算法都是用「队列」这种数据结构，每次将一个节点周围的所有节点加入队列。

BFS 相对 DFS 的最主要的区别是：**BFS 找到的路径一定是最短的，但代价就是空间复杂度可能比 DFS 大很多**

## 算法框架

先举例一下 BFS 出现的常见场景，**问题的本质就是让你在一幅「图」中找到从起点 `start` 到终点 `target` 的最近距离，这个例子听起来很枯燥，但是 BFS 算法问题其实都是在干这个事儿**。

这个广义的描述可以有各种变体，比如走迷宫，有的格子是围墙不能走，从起点到终点的最短距离是多少？如果这个迷宫带「传送门」可以瞬间传送呢？

再比如说两个单词，要求你通过某些替换，把其中一个变成另一个，每次只能替换一个字符，最少要替换几次？

再比如说连连看游戏，两个方块消除的条件不仅仅是图案相同，还得保证两个方块之间的最短连线不能多于两个拐点。你玩连连看，点击两个坐标，游戏是如何判断它俩的最短连线有几个拐点的？

这些问题都没啥奇技淫巧，**本质上就是一幅「图」，让你从一个起点，走到终点，问最短路径**。这就是 BFS 的本质。

框架：

```java
/**
     * BFS 遍历框架
     * 计算从起点 start 到终点 target 的最近距离
     *
     * @param start  起点
     * @param target 终点
     * @return 最短距离
     */
    public int BFS(Node start, Node target) {
        // 核心数据结构
        Queue<Node> q = new LinkedList<>();
        // 避免走回头路
        Set<Node> visited = new HashSet<>();

        // 将起点加入队列
        q.offer(start);
        visited.add(start);

        // 记录扩散的步数
        int step = 0;

        while (! q.isEmpty()) {
            int size = q.size();
            /* 将当前队列中的所有节点向四周扩散 */
            for (int i = 0; i < size; i++) {
                Node cur = q.poll();
                /* 划重点：这里判断是否到达终点 (根据题意写判断条件）*/
                if (cur == target) {
                    return step;
                }
                /* 将 cur 的相邻节点加入队列 */
//                for (Node x : cur.adj()) {
//                    // 避免回头
//                    if (! visited.contains(x)) {
//                        q.offer(x);
//                        visited.add(x);
//                    }
//                }
            }
            /* 划重点：更新步数在这里 */
            step++;
        }
        return step;
    }
```

队列 `q` 就不说了，BFS 的核心数据结构；`cur.adj()` 泛指 `cur` 相邻的节点，比如说二维数组中，`cur` 上下左右四面的位置就是相邻节点；`visited` 的主要作用是防止走回头路，大部分时候都是必须的，但是像一般的二叉树结构，没有子节点到父节点的指针，不会走回头路就不需要 `visited`。



## 二叉树的最小深度

> [111. 二叉树的最小深度](https://leetcode-cn.com/problems/minimum-depth-of-binary-tree/)
>
> 给定一个二叉树，找出其最小深度。
>
> 最小深度是从根节点到最近叶子节点的最短路径上的节点数量。
>
> **说明：**叶子节点是指没有子节点的节点。

怎么套到 BFS 的框架里呢？首先明确一下起点 `start` 和终点 `target` 是什么，怎么判断到达了终点？

**显然起点就是 `root` 根节点，终点就是最靠近根节点的那个「叶子节点」嘛**，叶子节点就是两个子节点都是 `null` 的节点：

```java
if (cur.left == null && cur.right == null) 
    // 到达叶子节点
```

代码实现：

```java
public int minDepth(TreeNode root) {
    if (root == null) {
        return 0;
    }
    Queue<TreeNode> q = new LinkedList<>();
    // 最小深度 (root本身就是一层，初始化为1)
    int minDepth = 1;
    q.offer(root);

    while (! q.isEmpty()) {
        int size = q.size();
        /* 将当前队列中的所有节点向四周扩散 */
        for (int i = 0; i < size; i++) {
            TreeNode cur = q.poll();
            /* 判断是否到达终点 */
            if (cur.left == null && cur.right == null) {
                return minDepth;
            }
            /* 将 cur 的相邻节点加入队列 */
            if (cur.left != null) {
                q.offer(cur.left);
            }
            if (cur.right != null) {
                q.offer(cur.right);
            }
        }
        /* 这里增加步数 */
        minDepth++;
    }

    return minDepth;
}
```

这里注意这个 `while` 循环和 `for` 循环的配合，**`while` 循环控制一层一层往下走，`for` 循环利用 `sz` 变量控制从左到右遍历每一层二叉树节点**：

![image-20220121151258634](https://cos.duktig.cn/typora/202201211513211.png)

**1、为什么 BFS 可以找到最短距离，DFS 不行吗**？

首先，你看 BFS 的逻辑，`depth` 每增加一次，队列中的所有节点都向前迈一步，这保证了第一次到达终点的时候，走的步数是最少的。

DFS 不能找最短路径吗？其实也是可以的，但是时间复杂度相对高很多。你想啊，DFS 实际上是靠递归的堆栈记录走过的路径，你要找到最短路径，肯定得把二叉树中所有树杈都探索完才能对比出最短的路径有多长对不对？而 BFS 借助队列做到一次一步「齐头并进」，是可以在不遍历完整棵树的条件下找到最短距离的。

形象点说，DFS 是线，BFS 是面；DFS 是单打独斗，BFS 是集体行动。这个应该比较容易理解吧。

**2、既然 BFS 那么好，为啥 DFS 还要存在**？

BFS 可以找到最短距离，但是空间复杂度高，而 DFS 的空间复杂度较低。

还是拿刚才我们处理二叉树问题的例子，假设给你的这个二叉树是满二叉树，节点数为 `N`，对于 DFS 算法来说，空间复杂度无非就是递归堆栈，最坏情况下顶多就是树的高度，也就是 `O(logN)`。

但是你想想 BFS 算法，队列中每次都会储存着二叉树一层的节点，这样的话最坏情况下空间复杂度应该是树的最底层节点的数量，也就是 `N/2`，用 Big O 表示的话也就是 `O(N)`。

由此观之，BFS 还是有代价的，一般来说在找最短路径的时候使用 BFS，其他时候还是 DFS 使用得多一些（主要是递归代码好写）。

## 打开转盘锁

**解开密码锁的最少次数**

> [752. 打开转盘锁](https://leetcode-cn.com/problems/open-the-lock/)
>
> 你有一个带有四个圆形拨轮的转盘锁。每个拨轮都有10个数字： '0', '1', '2', '3', '4', '5', '6', '7', '8', '9' 。每个拨轮可以自由旋转：例如把 '9' 变为 '0'，'0' 变为 '9' 。每次旋转都只能旋转一个拨轮的一位数字。
>
> 锁的初始数字为 '0000' ，一个代表四个拨轮的数字的字符串。
>
> 列表 deadends 包含了一组死亡数字，一旦拨轮的数字和列表里的任何一个元素相同，这个锁将会被永久锁定，无法再被旋转。
>
> 字符串 target 代表可以解锁的数字，你需要给出解锁需要的最小旋转次数，如果无论如何不能解锁，返回 -1 。
>

题目中描述的就是我们生活中常见的那种密码锁，若果没有任何约束，最少的拨动次数很好算，就像我们平时开密码锁那样直奔密码拨就行了。

但现在的难点就在于，不能出现 `deadends`，应该如何计算出最少的转动次数呢？

**第一步，我们不管所有的限制条件，不管 `deadends` 和 `target` 的限制，就思考一个问题：如果让你设计一个算法，穷举所有可能的密码组合，你怎么做**？

穷举呗，再简单一点，如果你只转一下锁，有几种可能？总共有 4 个位置，每个位置可以向上转，也可以向下转，也就是有 8 种可能对吧。

比如说从 `"0000"` 开始，转一次，可以穷举出 `"1000", "9000", "0100", "0900"...` 共 8 种密码。然后，再以这 8 种密码作为基础，对每个密码再转一下，穷举出所有可能…

**仔细想想，这就可以抽象成一幅图，每个节点有 8 个相邻的节点**，又让你求最短距离，这不就是典型的 BFS 嘛，框架就可以派上用场了。

先写出一个「简陋」的 BFS 框架代码再说别的：

```java
// 将 s[j] 向上拨动一次
String plusOne(String s, int j) {
    char[] ch = s.toCharArray();
    if (ch[j] == '9')
        ch[j] = '0';
    else
        ch[j] += 1;
    return new String(ch);
}
// 将 s[i] 向下拨动一次
String minusOne(String s, int j) {
    char[] ch = s.toCharArray();
    if (ch[j] == '0')
        ch[j] = '9';
    else
        ch[j] -= 1;
    return new String(ch);
}

// BFS 框架，打印出所有可能的密码
void BFS(String target) {
    Queue<String> q = new LinkedList<>();
    q.offer("0000");
    
    while (!q.isEmpty()) {
        int sz = q.size();
        /* 将当前队列中的所有节点向周围扩散 */
        for (int i = 0; i < sz; i++) {
            String cur = q.poll();
            /* 判断是否到达终点 */
            System.out.println(cur);

            /* 将一个节点的相邻节点加入队列 */
            for (int j = 0; j < 4; j++) {
                String up = plusOne(cur, j);
                String down = minusOne(cur, j);
                q.offer(up);
                q.offer(down);
            }
        }
        /* 在这里增加步数 */
    }
    return;
}
```

> PS：这段代码当然有很多问题，但是我们做算法题肯定不是一蹴而就的，而是从简陋到完美的。不要完美主义，咱要慢慢来，好不。

**这段 BFS 代码已经能够穷举所有可能的密码组合了，但是显然不能完成题目，有如下问题需要解决**：

1、会走回头路。比如说我们从 `"0000"` 拨到 `"1000"`，但是等从队列拿出 `"1000"` 时，还会拨出一个 `"0000"`，这样的话会产生死循环。

2、没有终止条件，按照题目要求，我们找到 `target` 就应该结束并返回拨动的次数。

3、没有对 `deadends` 的处理，按道理这些「死亡密码」是不能出现的，也就是说你遇到这些密码的时候需要跳过。

代码实现：

```java
public int openLock(String[] deadends, String target) {
    // 死亡密码
    Set<String> deads = new HashSet<>();
    Collections.addAll(deads, deadends);

    // 队列 进行BFS 寻找最小次数
    Queue<String> q = new LinkedList<>();
    q.offer("0000");

    // 避免 重复操作
    Set<String> visited = new HashSet<>();
    visited.add("0000");

    // 记录扩散的步数
    int step = 0;

    while (! q.isEmpty()) {
        int size = q.size();
        /* 扩散 */
        for (int i = 0; i < size; i++) {
            String cur = q.poll();
            /* 划重点：这里判断是否到达终点 (根据题意写判断条件） */
            if (deads.contains(cur)) {
                continue;
            }
            if (Objects.requireNonNull(cur).equals(target)) {
                return step;
            }
            /* 将 cur 的相邻节点加入队列 */
            for (int j = 0; j < 4; j++) {
                String plusOne = plusOne(cur, j);
                if (! visited.contains(plusOne)) {
                    q.offer(plusOne);
                    visited.add(plusOne);
                }
                String minusOne = minusOne(cur, j);
                if (! visited.contains(minusOne)) {
                    q.offer(minusOne);
                    visited.add(minusOne);
                }
            }
        }
        /* 在这里增加步数 */
        step++;
    }
    // 如果穷举完都没找到目标密码，那就是找不到了
    return - 1;
}

/**
 * 将 s[j] 向上拨动一次
 */
private String plusOne(String s, int j) {
    char[] ch = s.toCharArray();
    if (ch[j] == '9') {
        ch[j] = '0';
    } else {
        ch[j] += 1;
    }
    return new String(ch);
}

/**
 * 将 s[i] 向下拨动一次
 */
private String minusOne(String s, int j) {
    char[] ch = s.toCharArray();
    if (ch[j] == '0') {
        ch[j] = '9';
    } else {
        ch[j] -= 1;
    }
    return new String(ch);
}
```

至此，我们就解决这道题目了。有一个比较小的优化：可以不需要 `dead` 这个哈希集合，可以直接将这些元素初始化到 `visited` 集合中，效果是一样的，可能更加优雅一些。

## 双向 BFS 优化

你以为到这里 BFS 算法就结束了？恰恰相反。BFS 算法还有一种稍微高级一点的优化思路：**双向 BFS**，可以进一步提高算法的效率。

区别：**传统的 BFS 框架就是从起点开始向四周扩散，遇到终点时停止；而双向 BFS 则是从起点和终点同时开始扩散，当两边有交集的时候停止**。

为什么这样能够能够提升效率呢？其实从 Big O 表示法分析算法复杂度的话，它俩的最坏复杂度都是 `O(N)`，但是实际上双向 BFS 确实会快一些，参看下图：

![image-20220121160528881](https://cos.duktig.cn/typora/202201211605185.png)

![image-20220121160542387](https://cos.duktig.cn/typora/202201211605217.png)

图示中的树形结构，如果终点在最底部，按照传统 BFS 算法的策略，会把整棵树的节点都搜索一遍，最后找到 `target`；而双向 BFS 其实只遍历了半棵树就出现了交集，也就是找到了最短距离。从这个例子可以直观地感受到，双向 BFS 是要比传统 BFS 高效的。

**不过，双向 BFS 也有局限，因为你必须知道终点在哪里**。比如我们刚才讨论的二叉树最小高度的问题，你一开始根本就不知道终点在哪里，也就无法使用双向 BFS；但是第二个密码锁的问题，是可以使用双向 BFS 算法来提高效率的，代码稍加修改即可：

```java
/**
 * 双向 bfs
 */
public int openLock2(String[] deadends, String target) {
    Set<String> deads = new HashSet<>();
    Collections.addAll(deads, deadends);

    // 用集合不用队列，可以快速判断元素是否存在
    Set<String> q1 = new HashSet<>();
    Set<String> q2 = new HashSet<>();
    Set<String> visited = new HashSet<>();

    int step = 0;

    q1.add("0000");
    q2.add(target);

    while (! q1.isEmpty() && ! q2.isEmpty()) {
        // 哈希集合在遍历的过程中不能修改，用 temp 存储扩散结果
        Set<String> temp = new HashSet<>();

        /* 将 q1 中的所有节点向周围扩散 */
        for (String cur : q1) {
            /* 判断是否到达终点 */
            if (deads.contains(cur)) {
                continue;
            }
            if (q2.contains(cur)) {
                return step;
            }
            visited.add(cur);

            /* 将一个节点的未遍历相邻节点加入集合 */
            for (int j = 0; j < 4; j++) {
                String up = plusOne(cur, j);
                if (! visited.contains(up)) {
                    temp.add(up);
                }
                String down = minusOne(cur, j);
                if (! visited.contains(down)) {
                    temp.add(down);
                }
            }
        }
        /* 在这里增加步数 */
        step++;
        // temp 相当于 q1
        // 这里交换 q1 q2，下一轮 while 就是扩散 q2
        q1 = q2;
        q2 = temp;
    }
    return - 1;
}
```

双向 BFS 还是遵循 BFS 算法框架的，只是**不再使用队列，而是使用 HashSet 方便快速判断两个集合是否有交集**。

为什么这是一个优化呢？

因为按照 BFS 的逻辑，队列（集合）中的元素越多，扩散之后新的队列（集合）中的元素就越多；在双向 BFS 算法中，如果我们每次都选择一个较小的集合进行扩散，那么占用的空间增长速度就会慢一些，效率就会高一些。

不过话说回来，**无论传统 BFS 还是双向 BFS，无论做不做优化，从 Big O 衡量标准来看，时间复杂度都是一样的**，只能说双向 BFS 是一种 trick，算法运行的速度会相对快一点，掌握不掌握其实都无所谓。最关键的是把 BFS 通用框架记下来，反正所有 BFS 算法都可以用它套出解法。

## 滑动谜题

>  [773. 滑动谜题](https://leetcode-cn.com/problems/sliding-puzzle/)
>
> 给你一个 2x3 的滑动拼图，用一个 2x3 的数组`board`表示。拼图中有数字 0~5 六个数，其中数字 0 就表示那个空着的格子，你可以移动其中的数字，当`board`变为`[[1,2,3],[4,5,0]]`时，赢得游戏。
>
> 请你写一个算法，计算赢得游戏需要的最少移动次数，如果不能赢得游戏，返回 -1。
>
> 比如说输入的二维数组`board = [[4,1,2],[5,0,3]]`，算法应该返回 5：
>
> ![image-20220121161757191](https://cos.duktig.cn/typora/202201211618661.png)
>
> 如果输入的是`board = [[1,2,3],[5,4,0]]`，则算法返回 -1，因为这种局面下无论如何都不能赢得游戏。

那么这种游戏怎么玩呢？我记得是有一些套路的，类似于魔方还原公式。但是我们今天不来研究让人头秃的技巧，**这些益智游戏通通可以用暴力搜索算法解决，用 BFS 算法框架来秒杀这些游戏**。

对于这种计算最小步数的问题，我们就要敏感地想到 BFS 算法。

这个题目转化成 BFS 问题是有一些技巧的，我们面临如下问题：

1、一般的 BFS 算法，是从一个起点`start`开始，向终点`target`进行寻路，但是拼图问题不是在寻路，而是在不断交换数字，这应该怎么转化成 BFS 算法问题呢？

2、即便这个问题能够转化成 BFS 问题，如何处理起点`start`和终点`target`？它们都是数组哎，把数组放进队列，套 BFS 框架，想想就比较麻烦且低效。

首先回答第一个问题，**BFS 算法并不只是一个寻路算法，而是一种暴力搜索算法**，只要涉及暴力穷举的问题，BFS 就可以用，而且可以最快地找到答案。

我们的问题转化成了：**如何穷举出`board`当前局面下可能衍生出的所有局面**？这就简单了，看数字 0 的位置呗，和上下左右的数字进行交换就行了：

![image-20220121162135970](https://cos.duktig.cn/typora/202201211621124.png)

这样其实就是一个 BFS 问题，每次先找到数字 0，然后和周围的数字进行交换，形成新的局面加入队列…… 当第一次到达`target`时，就得到了赢得游戏的最少步数。

对于第二个问题，我们这里的`board`仅仅是 2x3 的二维数组，所以可以压缩成一个一维字符串。**其中比较有技巧性的点在于，二维数组有「上下左右」的概念，压缩成一维后，如何得到某一个索引上下左右的索引**？

很简单，我们只要手动写出来这个映射就行了：

```
vector<vector<int>> neighbor = {
    { 1, 3 },
    { 0, 4, 2 },
    { 1, 5 },
    { 0, 4 },
    { 3, 1, 5 },
    { 4, 2 }
};
```

**这个含义就是，在一维字符串中，索引`i`在二维数组中的的相邻索引为`neighbor[i]`**，：

![image-20220121163126304](https://cos.duktig.cn/typora/202201211631247.png)

至此，我们就把这个问题完全转化成标准的 BFS 问题了，直接就可以套出解法代码了：

```java
public int slidingPuzzle(int[][] board) {
    int m = 2, n = 3;
    StringBuilder sb = new StringBuilder();
    String target = "123450";
    // 将 2x3 的数组转化成字符串作为 BFS 的起点
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            sb.append(board[i][j]);
        }
    }
    String start = sb.toString();

    // 记录一维字符串的相邻索引
    int[][] neighbor = new int[][] {
            {1, 3},
            {0, 4, 2},
            {1, 5},
            {0, 4},
            {3, 1, 5},
            {4, 2}
    };

    /* BFS 算法框架开始 */
    Queue<String> q = new LinkedList<>();
    HashSet<String> visited = new HashSet<>();
    // 从起点开始 BFS 搜索
    q.offer(start);
    visited.add(start);

    int step = 0;
    while (! q.isEmpty()) {
        int sz = q.size();
        for (int i = 0; i < sz; i++) {
            String cur = q.poll();
            // 判断是否达到目标局面
            if (target.equals(cur)) {
                return step;
            }
            // 找到数字 0 的索引
            int idx = 0;
            while (Objects.requireNonNull(cur).charAt(idx) != '0') {
                idx++;
            }

            // 将数字 0 和相邻的数字交换位置
            for (int adj : neighbor[idx]) {
                String newBoard = swap(cur.toCharArray(), adj, idx);
                // 防止走回头路
                if (! visited.contains(newBoard)) {
                    q.offer(newBoard);
                    visited.add(newBoard);
                }
            }
        }
        step++;
    }
    return - 1;
}

private String swap(char[] chars, int i, int j) {
    char temp = chars[i];
    chars[i] = chars[j];
    chars[j] = temp;
    return new String(chars);
}
```











