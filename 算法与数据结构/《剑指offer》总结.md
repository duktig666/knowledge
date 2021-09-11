

# 查找和排序

## 注意事项

1. 查找常用顺序查找、二分查找、哈希表查找和二叉排序树查找。
2. 无论使用循环还是递归，面试官都期待候选人**可以信手拈来写出二分查找法的代码**。
3. 算法题要求在**排序的数组（或者部分排序的数组）**中**查找一个数字**或者**统计某个数字出现的频次**，都可以尝试使用二分查找法来进行解题。
4. 一定要对各种排序算法的特点烂熟于胸，可以从额外空间消耗、平均时间复杂度、最差时间复杂度等方面比较其优缺点。

## 剑指Offer面试题11——旋转数组的最小数字

[https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)

> 把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。输入一个递增排序的数组的一个旋转，输出旋转数组的最小元素。例如，数组 [3,4,5,1,2] 为 [1,2,3,4,5] 的一个旋转，该数组的最小值为1。  
>
> **示例 1：**
>
> ```java
> 输入：[3,4,5,1,2]
> 输出：1
> ```
>
> **示例 2：**
>
> ```java
> 输入：[2,2,2,0,1]
> 输出：0
> ```

### 思路

旋转数组，分左右两段有序序列，可考虑使用二分查找法解题。

1. 二分查找法，左右指针找中间值。**右边的有序序列 >= 左边的有序序列**。

2. **中间值 >= 左指针的值**，证明中间值在左边序列，令**左指针 = 中间值**；**中间值 <= 右指针的值**，证明中间值在右边序列，令**右指针 = 中间值**。

3. 当**左右指针相邻**时，左指针指向左侧有序序列的最后一个元素，右指针指向右侧序列的第一个元素，此时说明**右指针的元素为最小值**。

4. 特殊情况1：**旋转数组旋转0次，数组只有一个递增序列**。中间值初始赋给左指针指向的元素，如果左指针指向的元素 < 右侧指针的元素，那么直接返回最小值即可。

5. 特殊情况2：**左指针的值 = 右指针的值 = 中间值**，则利用二分查找法继续找最小值，需要利用遍历法找最小值。

   > 举例：[0,1,1,1,1] 旋转为 [1,0,1,1,1] 和 [1,1,1,0,1]  无法确定索引下标2的元素到底属于左边序列，还是右边序列。

### 代码实现

源码参看：[https://github.com/duktig666/algorithm/blob/main/src/offer/MinNumberInRotatedArray11.java](https://github.com/duktig666/algorithm/blob/main/src/offer/MinNumberInRotatedArray11.java)

```java
public class MinNumberInRotatedArray11 {

    /**
     * 查找旋转数组中的最小值
     */
    public int minArray(int[] numbers) {
        // 左右两个指针
        int index1 = 0, index2 = numbers.length - 1;
        // 初始中间值指向index1，数组只有一个升序序列，直接返回第一个元素（即旋转0个元素的情况）
        int mid = index1;
        while (numbers[index1] >= numbers[index2]) {
            // 左右指针相邻，index2右侧指针代表最小值
            if (index2 - index1 == 1) {
                mid = index2;
                break;
            }

            mid = index1 + ((index2 - index1) >> 1);

            // index1 = index2 = mid 无法用二分法进行判断，需要遍历找出最小值
            if (numbers[index1] == numbers[index2] && numbers[mid] == numbers[index1]) {
                return minInOrder(numbers, index1, index2);
            }
            //二分法指针调整
            if (numbers[mid] >= numbers[index1]) {
                index1 = mid;
            } else if (numbers[mid] <= numbers[index2]) {
                index2 = mid;
            }
        }
        return numbers[mid];
    }

    /**
     * 遍历法 找出指定序列的最小值
     */
    private int minInOrder(int[] numbers, int index1, int index2) {
        int min = numbers[index1];
        for (int i = index1 + 1; i < index2; i++) {
            if (min > numbers[i]) {
                min = numbers[i];
            }
        }
        return min;
    }

}
```

# 回溯法

> 回溯法 采用试错的思想，它尝试分步的去解决一个问题。在分步解决问题的过程中，当它通过尝试发现现有的分步答案不能得到有效的正确的解答的时候，它将取消上一步甚至是上几步的计算，再通过其它的可能的分步解答再次尝试寻找问题的答案。回溯法通常用最简单的递归方法来实现，在反复重复上述的步骤后可能出现两种情况：
>
> 1. 找到一个可能存在的正确的答案；
> 2. 在尝试了所有可能的分步方法后宣告该问题没有答案。

**解决一个回溯问题，实际上就是一个决策树的遍历过程**。你只需要思考 3 个问题：

**1、路径**：也就是已经做出的选择。

**2、选择列表**：也就是你当前可以做的选择。

**3、结束条件**：也就是到达决策树底层，无法再做选择的条件。

代码方面，回溯算法的框架：

```java
result = []
void backtrack(路径, 选择列表){
    if 满足结束条件:
        result.add(路径)
        return

    for (选择 : 选择列表){
        做选择
        backtrack(路径, 选择列表)
        撤销选择
    }
}
```

**其核心就是 for 循环里面的递归，在递归调用之前「做选择」，在递归调用之后「撤销选择」**，特别简单。

回溯法参看：[回溯算法详解（修订版）](https://www.cxyxiaowu.com/7103.html)

## 全排列问题

[https://leetcode-cn.com/problems/permutations/](https://leetcode-cn.com/problems/permutations/)

### 理解回溯

`n`个不重复的数，全排列共有 n! 个。***为了简单清晰起见，这次讨论的全排列问题不包含重复的数字***。

怎么穷举全排列的呢？比方说给三个数`[1,2,3]`，你肯定不会无规律地乱穷举，一般是这样：

先固定第一位为 1，然后第二位可以是 2，那么第三位只能是 3；然后可以把第二位变成 3，第三位就只能是 2 了；然后就只能变化第一位，变成 2，然后再穷举后两位……

其实这就是回溯算法，直接画出如下这棵回溯树：

![全排列 回溯树](https://cos.duktig.cn/typora/202109091433320.png)

只要从根遍历这棵树，记录路径上的数字，其实就是所有的全排列。**我们不妨把这棵树称为回溯算法的「决策树」**。

**为啥说这是决策树呢，因为你在每个节点上其实都在做决策**。

**可以把「路径」和「选择列表」作为决策树上每个节点的属性**，比如下图列出了几个节点的属性：

![回溯树详解](https://cos.duktig.cn/typora/202109091438882.jpeg)

**定义的`backtrack`方法其实就像一个指针，在这棵树上游走，同时要正确维护每个节点的属性，每当走到树的底层，其「路径」就是一个全排列**。

多叉树的遍历如下：

```java
void traverse(TreeNode root) {
    for (TreeNode child : root.childern)
        // 前序遍历需要的操作
        traverse(child);
        // 后序遍历需要的操作
}
```

而所谓的前序遍历和后序遍历，他们只是两个很有用的时间点，如下图：

![多叉树遍历](https://cos.duktig.cn/typora/202109091449683.png)

**前序遍历的代码在进入某一个节点之前的那个时间点执行，后序遍历代码在离开某个节点之后的那个时间点执行**。

「路径」和「选择」是每个节点的属性，函数在树上游走要正确维护节点的属性，那么就要在这两个特殊时间点搞点动作：

![路径和选择](https://cos.duktig.cn/typora/202109091450787.png)

总结搜索的方法：按顺序枚举每一位可能出现的情况，已经选择的数字在 **当前** 要选择的数字中不能出现。按照这种策略搜索就能够做到 **不重不漏**。这样的思路，可以用一个树形结构表示。

![全排列过程](https://cos.duktig.cn/typora/202109092021557.png)

### 全排列代码

#### 方法一

```java
List<List<Integer>> res = new LinkedList<>();

public List<List<Integer>> permute(int[] nums) {
    // 记录「路径」
    LinkedList<Integer> track = new LinkedList<>();
    backtrack(nums, track);
    return res;
}

/**
 * 回溯算法
 * 路径：track
 * 选择列表：nums中，不存在于 track 的那些元素
 * 结束条件：nums 中的元素全都在 track 中出现
 *
 * @param nums  选择列表，不存在于 track 的那些元素
 * @param track 记录的路径
 */
void backtrack(int[] nums, LinkedList<Integer> track) {
    // 触发结束条件
    if (track.size() == nums.length) {
        res.add(new LinkedList<>(track));
        return;
    }
    for (int num : nums) {
        // 排除不合法的选择
        if (track.contains(num)) {
            continue;
        }
        // 做选择
        track.add(num);
        // 进入下一层决策树
        backtrack(nums, track);
        // 取消选择
        track.removeLast();
    }
}
```

#### 方法二

```java
public List<List<Integer>> permute2(int[] nums) {
    // 存储所有种可能的情况
    List<List<Integer>> res = new ArrayList<>();
    // 添加所有的元素
    List<Integer> output = new ArrayList<>();
    for (int num : nums) {
        output.add(num);
    }
    // 全排列的元素个数
    int n = nums.length;
    // 深度优先搜索进行处理
    dfs(n, output, res, 0);
    return res;
}

public void dfs(int n, List<Integer> output, List<List<Integer>> res, int first) {
    // 所有数都填完了
    if (first == n) {
        res.add(new ArrayList<>(output));
    }
    for (int i = first; i < n; i++) {
        // 动态维护数组
        Collections.swap(output, first, i);
        // 继续递归填下一个数
        dfs(n, output, res, first + 1);
        // 撤销操作
        Collections.swap(output, first, i);
    }
}
```

#### 注意事项

执行 main 方法以后输出可能存在如下情况：

```
[[], [], [], [], [], []]
```


原因出现在递归终止条件这里：

```java
if (depth == len) {
    res.add(path);
    return;
}
```

变量 path 所指向的列表 在深度优先遍历的过程中只有一份 ，深度优先遍历完成以后，回到了根结点，成为空列表。

在 Java 中，参数传递是 值传递，对象类型变量在传参的过程中，复制的是变量的地址。这些地址被添加到 res 变量，但实际上指向的是同一块内存地址，因此我们会看到 66 个空的列表对象。解决的方法很简单，在 res.add(path); 这里做一次拷贝即可。

修改的部分：

```java
if (depth == len) {
    res.add(new ArrayList<>(path));
    return;
}
```

#### 为什么不是广度优先遍历

首先是正确性，只有遍历状态空间，才能得到所有符合条件的解，这一点 BFS 和 DFS 其实都可以；
在深度优先遍历的时候，不同状态之间的切换很容易 ，可以再看一下上面有很多箭头的那张图，每两个状态之间的差别只有 11 处，因此回退非常方便，这样全局才能使用一份状态变量完成搜索；
如果使用广度优先遍历，从浅层转到深层，状态的变化就很大，此时我们不得不在每一个状态都新建变量去保存它，从性能来说是不划算的；
如果使用广度优先遍历就得使用队列，然后编写结点类。队列中需要存储每一步的状态信息，需要存储的数据很大，真正能用到的很少 。
使用深度优先遍历，直接使用了系统栈，系统栈帮助我们保存了每一个结点的状态信息。我们不用编写结点类，不必手动编写栈完成深度优先遍历。

#### 不回溯可不可以

可以。搜索问题的状态空间一般很大，如果每一个状态都去创建新的变量，时间复杂度是 O(N)。在候选数比较多的时候，在非叶子结点上创建新的状态变量的性能消耗就很严重。

就本题而言，只需要叶子结点的那个状态，在叶子结点执行拷贝，时间复杂度是 O(N)。路径变量在深度优先遍历的时候，结点之间的转换只需要 O(1)。

最后，由于回溯算法的时间复杂度很高，因此在遍历的时候，如果能够提前知道这一条分支不能搜索到满意的结果，就可以提前结束，这一步操作称为 剪枝。

#### 剪枝

回溯算法会应用「剪枝」技巧达到以加快搜索速度。有些时候，需要做一些预处理工作（例如排序）才能达到剪枝的目的。预处理工作虽然也消耗时间，但能够剪枝节约的时间更多；
提示：剪枝是一种技巧，通常需要根据不同问题场景采用不同的剪枝策略，需要在做题的过程中不断总结。

由于回溯问题本身时间复杂度就很高，所以能用空间换时间就尽量使用空间。

#### 总结

做题的时候，建议 **先画树形图** ，画图能帮助我们想清楚递归结构，想清楚如何剪枝。拿题目中的示例，想一想人是怎么做的，一般这样下来，这棵递归树都不难画出。

在画图的过程中思考清楚：

- 分支如何产生；
- 题目需要的解在哪里？是在叶子结点、还是在非叶子结点、还是在从跟结点到叶子结点的路径？
- 哪些搜索会产生不需要的解的？例如：产生重复是什么原因，如果在浅层就知道这个分支不能产生需要的结果，应该提前剪枝，剪枝的条件是什么，代码怎么写？

**写`backtrack`函数时，需要维护走过的「路径」和当前可以做的「选择列表」，当触发「结束条件」时，将「路径」记入结果集**。

其实想想看，回溯算法和动态规划是不是有点像呢？我们在动态规划系列文章中多次强调，动态规划的三个需要明确的点就是「状态」「选择」和「base case」，是不是就对应着走过的「路径」，当前的「选择列表」和「结束条件」？

某种程度上说，动态规划的暴力求解阶段就是回溯算法。只是有的问题具有重叠子问题性质，可以用 dp table 或者备忘录优化，将递归树大幅剪枝，这就变成了动态规划。而今天的两个问题，都没有重叠子问题，也就是回溯算法问题了，复杂度非常高是不可避免的。



### n皇后问题

https://leetcode-cn.com/problems/n-queens/

![n皇后的决策树](https://cos.duktig.cn/typora/202109092139757.jpeg)

#### 基于数组拷贝实现

```java
public List<List<String>> solveNQueens(int n) {
    char[][] chess = new char[n][n];
    //初始化数组
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            chess[i][j] = '.';
        }
    }
    // 记录n皇后的每种符合条件的情况
    List<List<String>> res = new ArrayList<>();
    dfs(res, chess, 0);
    return res;
}

private void dfs(List<List<String>> res, char[][] chess, int row) {
    //终止条件，最后一行都走完了，说明找到了一组，把它加入到集合res中
    if (row == chess.length) {
        res.add(construct(chess));
        return;
    }
    //遍历每一列
    for (int col = 0; col < chess.length; col++) {
        //判断这个位置是否可以放皇后
        if (valid(chess, row, col)) {
            //数组复制一份
            char[][] temp = Arrays.copyOf(chess, chess.length);
            //在当前位置放个皇后
            temp[row][col] = 'Q';
            //递归到下一行继续
            dfs(res, temp, row + 1);
        }
    }
}

/**
 * 判断当前位置是否可以放置皇后
 *
 * @param chess /
 * @param row   第几行
 * @param col   第几列
 * @return 是否可以放置皇后
 */
private boolean valid(char[][] chess, int row, int col) {
    //判断当前列有没有皇后,因为他是一行一行往下走的，我们只需要检查走过的行数即可，通俗一点就是判断当前坐标位置的上面有没有皇后
    for (int i = 0; i < row; i++) {
        if (chess[i][col] == 'Q') {
            return false;
        }
    }
    //判断当前坐标的右上角有没有皇后
    for (int i = row - 1, j = col + 1; i >= 0 && j < chess.length; i--, j++) {
        if (chess[i][j] == 'Q') {
            return false;
        }
    }
    //判断当前坐标的左上角有没有皇后
    for (int i = row - 1, j = col - 1; i >= 0 && j >= 0; i--, j--) {
        if (chess[i][j] == 'Q') {
            return false;
        }
    }
    return true;
}

/**
 * 把数组转为list
 *
 * @param chess /
 * @return /
 */
private List<String> construct(char[][] chess) {
    List<String> path = new ArrayList<>();
    for (char[] chars : chess) {
        path.add(new String(chars));
    }
    return path;
}
```

#### 基于回溯实现

上述方法在使用深度优先搜索（DFS）时，每次都要拷贝数组`chess`，即需要保存一份临时变量，放置皇后，极大的浪费了空间性能。以下方法使用**回溯法**进行了改进，每次再递归执行操作后，撤销之前的操作。

```java
/**
 * 回溯解决n皇后问题（可替换上文的solve方法）
 *
 * @param res   /
 * @param chess /
 * @param row   /
 */
private void dfsBacktrack(List<List<String>> res, char[][] chess, int row) {
    if (row == chess.length) {
        res.add(construct(chess));
        return;
    }
    for (int col = 0; col < chess.length; col++) {
        if (valid(chess, row, col)) {
            chess[row][col] = 'Q';
            dfsBacktrack(res, chess, row + 1);
            chess[row][col] = '.';
        }
    }
}
```



### 剑指Offer面试题12——矩阵中的路径

[https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof/](https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof/)

请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一个格子开始，每一步可以在矩阵中向左，向右，向上，向下移动一个格子。如果一条路径经过了矩阵中的某一个格子，则该路径不能再进入该格子。

例如下面的矩阵包含了一条 bfce 路径。但是矩阵中不包含"abcb"路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入该格子。

![矩阵中的路径](https://cos.duktig.cn/typora/202109092157108.png)

#### 原版剑指Offer的Java代码实现（仅参考）

```java
/**
 * 判断矩阵中是否存在存在一条包含str所有字符的路径
 *
 * @param matrix 矩阵
 * @param rows   矩阵的行数
 * @param cols   矩阵的列数
 * @param str    待判断的字符串路径
 * @return 是否存在str的路径
 */
public boolean hasPath(char[] matrix, int rows, int cols, char[] str) {
    if (matrix == null || rows < 1 || cols < 1 || str == null) {
        return false;
    }
    // 定义布尔值矩阵（虽然是一个布尔值数组）
    boolean[] visited = new boolean[rows * cols];
    // 定义走到字符串的第几个字符
    int pathLength = 0;
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            // 选取一个字符为起点，判断是否存在判定路径
            if (hasPathCore(matrix, rows, cols, i, j, str, pathLength, visited)) {
                return true;
            }
        }
    }
    return false;
}

/**
 * 判断指定路经是否存在
 *
 * @param matrix     矩阵
 * @param rows       矩阵的行数
 * @param cols       矩阵的列数
 * @param row        当前元素行
 * @param col        当前元素列
 * @param str        待判断的字符串路径
 * @param pathLength 走到字符串的第几个字符
 * @param visited    布尔值矩阵
 * @return /
 */
public boolean hasPathCore(char[] matrix, int rows, int cols, int row, int col, char[] str, int pathLength,
                           boolean[] visited) {
    // 超出了字符串长度
    if (pathLength > str.length - 1) {
        return true;
    }
    // 定义是否存在路径
    boolean hasPath = false;
    // ①无越界 ②当前矩阵元素 = 当前第i个字符串路径的字符 ③当前路径未走过
    if (row >= 0 && row < rows && col >= 0 && col < cols && matrix[row * cols + col] == str[pathLength] && ! visited[row * cols + col]) {
        // 继续走
        ++ pathLength;
        // 标记上一个已走过
        visited[row * cols + col] = true;
        // 递归下去(上下右左都看一遍)
        hasPath = hasPathCore(matrix, rows, cols, row - 1, col, str, pathLength, visited)
                || hasPathCore(matrix, rows, cols, row + 1, col, str, pathLength, visited)
                || hasPathCore(matrix, rows, cols, row, col + 1, str, pathLength, visited)
                || hasPathCore(matrix, rows, cols, row, col - 1, str, pathLength, visited);
        // 路径未走通
        if (! hasPath) {
            // 回溯
            -- pathLength;
            visited[row * cols + col] = false;
        }
    }
    return hasPath;
}
```

#### LeetCode上代码实现

```java
/**
 * 判断指定路经是否存在
 *
 * @param board 矩阵
 * @param word  路径字符串
 * @return 是否存在路径
 */
public boolean exist2(char[][] board, String word) {
    char[] words = word.toCharArray();
    // 依次从每个节点进行深搜，判断是否可得到路径
    for (int i = 0; i < board.length; i++) {
        for (int j = 0; j < board[0].length; j++) {
            if (dfs(board, words, i, j, 0)) {
                return true;
            }
        }
    }
    return false;
}

/**
 * @param board 矩阵
 * @param word  字符串路径
 * @param i     矩阵行索引
 * @param j     矩阵列索引
 * @param k     当前目标字符在word中的索引
 * @return 是否存在路径
 */
boolean dfs(char[][] board, char[] word, int i, int j, int k) {
    //递归终止条件
    // 边界值判断 数组元素不等于第k个目标字符，直接剪枝
    if (i >= board.length || i < 0 || j >= board[0].length || j < 0 || board[i][j] != word[k]) {
        return false;
    }
    // 最后一个字符也相等，说明存在这样的一条路径
    if (k == word.length - 1) {
        return true;
    }
    // 当前矩阵元素标记为空字符，代表已经访问过（防止重复访问）
    board[i][j] = '\0';
    // 上下左右四个方向开启下层递归，只要有一个能找到，代表路径存在
    boolean res = dfs(board, word, i + 1, j, k + 1)
            || dfs(board, word, i - 1, j, k + 1)
            || dfs(board, word, i, j + 1, k + 1)
            || dfs(board, word, i, j - 1, k + 1);
    // 将当前元素再回复至初始值（只有相等时才会向下走，在这里时，之前的代码 board[i][j] = word[k] 是必然成立）
    board[i][j] = word[k];
    return res;
}
```

### 剑指Offer面试题13——机器人的运动范围

[https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/](https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/)

```java
public class RobotMove13 {

    public int movingCount(int m, int n, int k) {
        boolean[] visited = new boolean[m * n];
        int i = 0, j = 0;
        return backtrack(m, n, k, i, j, visited);
    }

    private int backtrack(int m, int n, int k, int i, int j, boolean[] visited) {
        if (i < 0 || i > m - 1 || j < 0 || j > n - 1 || visited[i * n + j]) {
            return 0;
        }
        //机器人到达格子的数量统计
        int count = 0;
        if (getDigitSum(i) + getDigitSum(j) <= k) {
            visited[i * n + j] = true;
            count = 1 + backtrack(m, n, k, i + 1, j, visited) +
                    backtrack(m, n, k, i - 1, j, visited) +
                    backtrack(m, n, k, i, j + 1, visited) +
                    backtrack(m, n, k, i, j - 1, visited);
        }
        return count;
    }

    /**
     * 计算数字各个位数之和
     */
    private int getDigitSum(int num) {
        int sum = 0;
        while (num > 0) {
            sum += num % 10;
            num /= 10;
        }
        return sum;
    }

    public static void main(String[] args) {
        RobotMove13 test = new RobotMove13();
        System.out.println(test.movingCount(2, 3, 1));
        System.out.println(test.movingCount(3, 1, 0));
    }

}
```































