

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



# 动态规划

## 动态规划的基本思想

> 动态规划算法与分治法类似，其基本思想就是**将待求解问题分解成若干子问题，先求解子问题，然后从这些子问题的解得到原问题的解**。
>
> 与分治法不同的是，适合动态规划法求解的问题，**经分解得到的子问题往往不是相互独立的**。
>
> 若用分治法来解这类问题，则分解得到的子问题数目太多，以至于最后解决原问题需要耗费**指数时间**。然而，不同子问题的数目常常只有多项式量级。在用分治法求解时，有些子问题被重复结算了很多次。
>
> 如果我们能够**保存已经解决的子问题的答案，而在需要时再找出已求得的答案，这样就可以避免大量的重复计算，从而得到多项式时间复杂度的算法**。为了达到此目的，可以用一个表来记录所有已解决的子问题的答案，不管该子问题以后是否被用到，只要它被计算过，就将其结果填入表中。这就是动态规划的基本思想。

1. **求一个问题的最优解**。
2. **整体问题的最优解依赖于子问题的最优解**。
3. **把大问题分解成若干个小问题，这些小问题之间还有相互重叠的更小子问题**。
4. **从上往下分析问题，从下往上求解问题**。避免子问题在分解发问题的过程中重复出现，从下往上顺序先计算小问题的最优解并保存下来，再以此为基础求取大问题的最优解（**保存已经解决的子问题的答案，避免重复计算**）。

## 动态规划的基本要素

动态规划算法就是将待求解问题分解成若干子问题，先求解子问题并保存子问题的答案避免重复计算，然后从这些子问题的解得到原问题的解。而如何断定一个问题是否可以用动态规划来解决，就需要掌握动态规划的两个基本要素，**「最优子结构性质」** 和**「重叠子问题性质」** 。

### 最优子结构性质

设计动态规划算法的第一步通常是要刻画最优解的结构。**当问题的最优解包含了其子问题的最优解时，称该问题具有最优子结构性质** 。问题的最优子结构性质提供了该问题可用动态规划求解的重要线索。

例如，最短路径问题有如下的最优子结构：

![动态规划之最优子结构性质示例](https://cos.duktig.cn/typora/202109151442075.png)

结点 x 是从源结点 u 到目标结点 v 的最短路径上的结点，则源结点 u 到目标结点 v 的最短路径 7 就等于从源结点 u 到结点 x 的最短路径 5 加上从结点 x 到目标结点 v 的最短路径 2 的和。**源结点 u 到目标结点 v 的最短路径就是要求解的最优解**，源结点 u 到结点 x 的最短路径和从结点 x 到目标结点 v 的最短路径均为**子问题的最优解**，而**问题的最优解包含了其子问题的最优解**，则该问题具有**最优子结构性质**。

但是最长路径问题就不具有最优子结构性质，注意这里的最长路径指的是两个结点间的最长简单路径（即不存在环的路径）。

![不符合最优子结构性质示例](https://cos.duktig.cn/typora/202109151443379.png)

从结点 u 到结点 v 有两条最长路径，分别为 **u → s → v** 和 **u → t → v** ，但是与最短路径问题不同，这些最长路径不具有最优子结构性质。比如，从结点 u 到结点 v 有两条最长路径 **u → s → v** 并不等于从 u 到 s 的最长路径  **u → t → v → s** 与从 s 到 v 的最长路径 **s → u → t → v** 的加和。

![不符合最优子结构性质示例](https://cos.duktig.cn/typora/202109151444456.png)

### 重叠子问题性质

在用递归算法自顶向下解决一个问题时，每次产生的子问题并不总是新问题，有些子问题被反复计算多次。动态规划正是利用了这种**子问题的重叠性质**，**对每个子问题只解一次，而后将其解保存到一个表格中，当再次需要解此子问题时，只是简单地用常数时间查看一下结果**。

**动态规划经分解得到的子问题往往不是相互独立的** 。如果经分解得到的子问题的解之间相互独立，比如二分查找（Binary Search）经分解得到的子问题之间相互独立，不存在 **重叠子问题**，所以不适合用 **动态规划**，更适合分治算法。而斐波那契数列问题则更适用于动态规划，虽然严格意义上斐波那契数列的解决并不是动态规划的普适应用（**动态规划的一般形式是求最优值！**），但是对于我们理解动态规划的 **「重叠子问题性质」** 大有裨益！

关于斐波那契数列的递归实现，信手拈来：

```java
int fib(int n)
{
    if(n <= 1){
        return n;        
    }
 return fib(n-1) + fib(n-2);
}
```

虽然递归的代码简单易懂，但却极其低效。先不说这种递归实现造成栈空间的极大浪费，就仅仅该算法的时间复杂度已属于指数级别 了。

再来看看 时的递归树：

![斐波那契数列 n=5时的递归树](https://cos.duktig.cn/typora/202109151445349.png)

可以看到，fib(3) 被调用了两次，如果我们已经保存 fib(3) 的值，我们就可以复用保存的 fib(3) 的值，而不是重新计算，fib(2) 也是同样的道理。

保存重叠子问题的解（也就是 fib(3)）有以下两种方式 **DP table（自底向上）** 和 **备忘录方法（自顶向下）**。

#### DP table（自底向上）

DP table 就是动态规划算法自底向上建立的一个表格，用于保存每一个子问题的解，并返回表中的最后一个解。比如斐波那契数，我们先计算 fib(0)，然后 fib(1)，然后 fib(2)，然后 fib(3)，以此类推，直至计算出 fib(n)。

比如我们计算 fib(5)，先由 fib(0) + fib(1) 得到 fib(2)，再由 fib(1) + fib(2) 得到 fib(3)，再由 fib(2) + fib(3) 得到 fib(4)，最后由 fib(3) + fib(4) 得到 fib(5)。

![DP table（自底向上）](https://cos.duktig.cn/typora/202109151446875.png)

也就是说，我们只需要存储子问题的解，而不需要重复计算子问题的解，对上图进行简化：

![DP table（自底向上）简化](https://cos.duktig.cn/typora/202109151446725.png)

此图也就是动态规划法求斐波那契数的过程图。其实就实现而言，其实质上是斐波那契数的迭代实现，所以之前说斐波那契数严格意义上不是动态规划所解决的问题，但是其对于我们理解**「重叠子问题性质」**还有很有帮助的。

对斐波那契数列问题，动态规划法（自底而上）保存重叠子问题的解的更一般形式为：

![DP table（自底向上）](https://cos.duktig.cn/typora/202109151447607.png)

实现起来更是 so easy !

```java
public class Fibonacci { 
    int fib(int n) { 
        int f[] = new int[n+1]; //保存子问题的解
        f[0] = 0; 
        f[1] = 1; 
        for (int i = 2; i <= n; i++) 
            f[i] = f[i-1] + f[i-2]; 
        return f[n]; 
    } 
} 
```

请不要问我上面的代码空间复杂度明明可以用 就实现，为什么我要写成 。因为希望大家理解动态规划法的一个核心思想，**保存子问题的解，DP table 会保存所有子问题的解** 。你期望的可能是下面这样，那也 OK。

```java
public class Fibonacci { 
    int fib(int n) { 
        int result = 0;
        int fib0 = 0; 
        int fib1 = 1; 
        for (int i = 2; i <= n; i++){
            result = fib0 + fib1;
            fib0 = fib1;
            fib1 = result;
        }  
        return result; 
    } 
} 
```

#### 备忘录方法（自顶向下）

备忘录方法是动态规划算法的变形。与动态规划方法一样，备忘录方法用表格保存已解决的子问题的答案，在下次需要解此子问题时，只要简单地查看该子问题的答案，而不必重新计算。

与动态规划方法不同的是，备忘录方法的递归方式是自顶向下的，而动态规划算法则是自底向上递归的。因此，备忘录方法的控制结构与直接递归方法的控制结构相同，区别在于备忘录方法为每个解过的子问题建立了备忘录以备需要时查看，避免相同子问题的重复求解。

备忘录方法为每一个子问题建立一个记录项，初始时，该记录项存入一个特殊的值，表示该子问题尚未被解决（比如下面斐波那契数的备忘录版本中将其设置为 -1）。在求解过程中，对每个待求解的子问题，首先查看其相应的记录项。若记录项中存储的是初始化时存入的特殊值，则表示该子问题是第一次遇到，此时计算出该子问题的解，并保存在相应的记录项中，以备以后查看。若记录项中存储的已不是初始化时存入的特殊值，则表示该子问题已被计算过，其相应的记录项存储的是该子问题的答案。此时，只要从记录项中取出该子问题的答案即可，而不必重新计算。

以求解 fib(5) 为例，为求解 fib(5)，要先求解 fib(4)，要求解 fib(4)，要先求解 fib(3)，要求解 fib(2)，则要先求解 fib(1) 和 fib(0)，fib(1) = 1，fib(0) = 0，则直接返回，依次返回 fib(2) = 1，fib(3) = fib(2) + fib(1) = 2，fib(4) = fib(3) + fib(2) = 3，fib(5) = fib(4) + fib(3) = 5。

PS：递归调用可理解为入栈操作，而返回则为出栈操作。

![备忘录](https://cos.duktig.cn/typora/202109151448100.png)

由 fib(5) 的递归树，可以发现，备忘录方法与动态规划方法一样，把一颗存在巨量冗余的递归树进行剪枝，改造成了一颗不存在冗余的递归图。重叠子问题的解都被保存了起来，用到时直接查表即可，而不需要重新计算，这一点备忘录与动态规划方法一样。不同的是，**备忘录方法是自顶向下对问题求解，与直接递归方法的控制结构相同，而动态规划方法是自底向上对问题求解，与迭代实现方式的结构一致**。

以 fib(5) 为例，备忘录方法（自顶向下）最终就是这样：

![备忘录](https://cos.duktig.cn/typora/202109151449277.png)

对任意一个斐波那契数，更一般的备忘录方法则为下图这样，与动态规划法正好相反：

![备忘录](https://cos.duktig.cn/typora/202109151449840.png)

实现上只需要对递归实现进行稍加改动即可：

```java
public class Fibonacci
{
    final int MAX = 100; // 备忘录的大小
    final int NIL = -1; // 特殊值
    
    int lookup[] = new int[MAX]; // 备忘录数组
    
    //初始化备忘录中的值为特殊值 NIL
    void initializeTable(){
        for(int i = 0; i < MAX; i++){
            lookup[i] = NIL;
        }
    }
    
    //斐波那契数的备忘录版本
    int fib(int n){
        if(lookup[n] == NIL){
            if(n <= 1){
                lookup[n] = n;
            }
            else{
                lookup[n] = fib(n-1) + fib(n-2);
            }
        }
        return lookup[n];
    }
}
```

动态规划方法（DP table）与备忘录方法都是存储子问题解的方法，除了解决问题的逻辑不同之外（一个自底向上，一个自顶向下），两者在时间效率上较为相近。

## 如何解决动态规划问题？

解决动态规划问题四步法：

1. **辨别是不是一个动态规划问题**；
2. **确定状态**
3. **建立状态之间的关系**；
4. **为状态添加备忘录或者DP Table** 。

### 第一步：如何断定一个问题是动态规划问题？

一般情况下，需要求最优解的问题（最短路径问题，最长公共子序列，最大字段和等等，出现 **最** 字你就留意），在一定条件下对排列进行计数的计数问题（丑数问题）或某些概率问题都可以考虑用动态规划来解决。

所有的动态规划问题都满足重叠子问题性质，大多数经典的动态规划问题还满足最优子结构性质，当我们从一个给定的问题中发现了这些特性，就可以确定其可以用动态规划解决。

### 第二步：确定状态

DP 问题最重要的就是确定所有的状态和状态与状态之间的转移方程。确定状态转移方程是动态规划最难的部分，但也是最基础的，必须非常谨慎地选择状态，因为状态转移方程的确定取决于你对问题状态定义的选择。那么，状态到底是个什么鬼呢？

**「状态」** 可以视为一组可以唯一标识给定问题中某个子问题解的参数，这组参数应尽可能的小，以减少状态空间的大小。比如斐波那契数中，0 , 1, …, n 就可以视为参数，而通过这些参数定义出的 DP[0]，DP[1]，DP[2]，…，DP[n] 就是状态，而状态与状态之间的转移方程就是 `DP(n) = DP(n-1) + DP(n-2)` 。

再比如，经典的背包问题（Knapsack problem）中，状态通过 **index** 和 **weight** 两个参数来定义，即 **`DP[index][weight]`** 。`DP[index][weight]` 则表示当前从 0 到 index 的物品装入背包中可以获得的最大重量。因此，参数 index 和 weight 可以唯一确定背包问题的一个子问题的解。

所以，当确定给定的问题之后，首当其冲的就是**确定问题的状态**。动态规划算法就是将待求解问题分解成若干子问题，先求解子问题并保存子问题的答案避免重复计算，然后从这些子问题的解得到原问题的解。**既然确定了一个一个的子问题的状态，接下来就是确定前一个状态到当前状态的转移关系式，也称状态转移方程**。

### 第三步：构造状态转移方程

**构造状态转移方程是 DP 问题最难的部分，需要足够敏锐的直觉和观察力**，而这两者都是要通过**大量的练习**来获得。我们用一个简单的问题来理解这个步骤。

> 问题描述：给定 3 个数 {1，3，5}，请问使用这三个数，有多少种方式可以构造出一个给定的数 ‘n’.(允许重复和不同顺序)。
>
> 设 n = 6，使用 {1，3，5} 则共有 8 种方式可以构造出 ‘n’ ：
>
> 1+1+1+1+1+1
>
> 1+1+1+3
>
> 1+1+3+1
>
> 1+3+1+1
>
> 3+1+1+1
>
> 3+3
>
> 1+5
>
> 5+1

我们现在考虑用动态规划的方法论来解决。首先确定该问题的状态，由于参数 n 可以唯一标识任意一个子问题，我们用参数 n 来确定状态。所以，上述问题的状态就可以表示为 DP[n]，代表使用 {1，3，5} 作为元素可以形成 n 的总的序列数。

接下来就是计算 DP[n] 了，此时所谓的直觉就很关键了。

因为我们仅能使用  {1，3，5} 来形成给定的数字 n ，我们可以先考虑 n = 1,2,3,4,5,6 的结果，就是求出 n 等于 1，2，3，4，5，6 的状态值。

DP[n = 1] = 1，DP[n = 2] = 1, DP[n = 3] = 2，DP[n = 4] = 3，DP[n = 5] = 5，DP[n = 6] = 8。

现在，我们期望得到 DP[n = 7] 的值，我可以利用子状态的三种情况得到 n = 7 ：

一、给状态 DP[n = 6] 的序列均加 1，如下所示：

> (1+1+1+1+1+1) + 1
>
> (1+1+1+3) + 1
>
> (1+1+3+1) + 1
>
> (1+3+1+1) + 1
>
> (3+1+1+1) + 1
>
> (3+3) + 1
>
> (1+5) + 1
>
> (5+1) + 1

二、给状态 DP[4] 的序列均加 3

> (1+1+1+1) + 3
>
> (1+3) + 3
>
> (3 + 1) + 3

三、给状态 DP[2] 的序列均加 5

> (1 + 1) + 5

仔细检查并确认上述三种情况覆盖了所有可能形成和为 7 的序列，我们就可以说

`DP[7] = DP[6] + DP[4] + DP[2]` 或者 `DP[7] = DP[7 – 1] + DP[7 – 3] + DP[7-5]`

推广一下：`DP[n] = DP[n – 1] + DP[n – 3] + DP[n-5]`

此时我们就可以写出这样的代码了:

```java
int solve(int n){
    if(n < 0){
        return 0;
    }
    
    if(n == 1 || n == 0){
        return 1;
    }
    return solve(n-1) + solve(n-3) + solve(n-5);
}
```

但是这个递归解法的时间复杂度是指数级别的，因为递归解法重复计算了子问题的解。所以，第四步考虑加入备忘录或者DP Table 。

### 第四步：为状态添加备忘录或者 DP Table

这个可以说是动态规划最简单的部分，我们仅需要存储子状态的解，以便下次使用子状态时直接查表从内存中获得。

```java
// 备忘录的大小
final int MAX = 100; 
// 特殊值
final int NIL = - 1;


// 备忘录数组
int lookup[] = new int[MAX]; 

//初始化备忘录中的值为特殊值 NIL
void initialize() {
    for (int i = 0; i < MAX; i++) {
        lookup[i] = NIL;
    }
}

//备忘录版本
int solve(int n) {

    if (n < 0) {
        return 0;
    }
    if (n == 1 || n == 0) {
        return 1;
    }
    if (lookup[n] != NIL) {
        return lookup[n];
    }

    return lookup[n] = solve(n - 1) + solve(n - 3) + solve(n - 5);
}
```

添加 DP Table 的代码：

```java
// 备忘录的大小
final int MAX = 100; 

int solve(int n) {
    int[] dp = new int[MAX];
    dp[1] = 1;
    dp[2] = 1;
    dp[3] = 2;
    dp[4] = 3;
    dp[5] = 5;
    for (int i = 6; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 3] + dp[i - 5];
    }
    return dp[n];
}
```

## 备忘录 vs DP Table

被复用的子问题的解既可以使用备忘录进行存储，也可以使用 DP Table，那么到底哪种方法更好，两种方法应该如何抉择呢？

**DP Table 是自底向上的方式，备忘录是自顶向下的方式**。

### DP Table 法（自底向上的动态规划）

顾名思义，自底向上就是从底部（递归的出口，动态规划中称为 base case）开始，不断向上回溯，计算出问题的解。下面看一下 DP Table 的状态转移过程。

设 DP 问题的基态（Base State）为 **`dp[0]`** ，目标状态为 **`dp[n]`** 。

如果我们从基态 **`dp[0]`** 开始转移，在遵循状态转移方程的情况下到达目标状态 **`dp[n]`** ，则将其称为 “自底向上” 的方法。（`dp[0] → dp[n]`）

我们可以使用自底向上的方法计算一个数的阶乘。

定义状态：`dp[x]` 表示一个状态，即 x 的阶乘。

状态转移方程：`dp[x] = dp[x - 1] * x` .

```java
int dp[] = new int[MAX];
// base state
int dp[0] = 1; 

for(int i = 1; i <= n; i++){
    dp[i] = dp[i-1] * i;
}
```

上面的从基态 `dp[0]` 开始，经状态转移方程到达目标状态 `dp[n]` .

PS：注意这里的DP Table 是按照顺序填充的，并且我们直接从表中访问计算的子状态的解。

### 备忘录法（自顶向下的方法）

还是按照状态转移描述，我们从状态 **`dp[n]`** 开始，经状态转移向下寻找所需要的子状态的值，直到找到所有与状态  **`dp[n]`** 相关的子状态，并返回  **`dp[n]`** ，这就是自顶向下的备忘录方法。

我们用备忘录方法解决阶乘问题。

```java
int dp[] = new int[MAX];
for(int i = 0; i < MAX; i++){
 dp[i] = -1;
}

int solve(int x){
    if(x == 0){
        return 1;
    }
    if(dp[x] != -1){
        return dp[x];
    }
    return (dp[x] = x * solve(x-1));
}
```

对于阶乘问题，内存布局是线性的，所以 `dp[]` 表是按照线性进行填充的。但是当考虑二维数组的情况下，就像最小成本路径问题一样，此时的内存将不是按照顺序存储。

### 对比

状态：DP Table 状态转移关系较难确定，备忘录状态转移关系较易确定。你可以理解为自顶向下推导较为容易，自底向上推导较难。比如 `DP[n] = DP[n – 1] + DP[n – 3] + DP[n-5]` 的确定。

代码：当约束条件较多的情况下，DP Table 较为复杂；备忘录代码相对容易实现和简单，仅需对递归代码进行改造。

效率：**动态规划（DP Table）较快**，我们可以直接从表中获取子状态的解；备忘录由于大量的递归调用和返回状态操作，速度较慢。

子问题的解：当所有的子问题的解都至少要被解一遍，自底向上的动态规划算法通常比自顶向下的备忘录方法快常数量级；当求解的问题的子问题空间中的部分子问题不需要计算，仅需求解部分子问题就可以解决原问题，此时备忘录方法要优于动态规划，因为备忘录自顶向下仅存储与原问题求解相关的子问题的解。

表空间：DP Table 依次填充所有子状态的解；而备忘录不必填充所有子问题的解，而是按需填充。

至于两个该如何选择，我想你的心中也有数了，建议按照解动态规划的四步骤依次求解，至于第四步，你个人喜欢用 DP Table 就用 DP Table ，喜欢备忘录就用备忘录。



## 剑指offer14题——剪绳子（动态规划解法）

[https://leetcode-cn.com/problems/jian-sheng-zi-lcof/](https://leetcode-cn.com/problems/jian-sheng-zi-lcof/)

> 给你一根长度为 n 的绳子，请把绳子剪成整数长度的 m 段（m、n都是整数，n>1并且m>1），每段绳子的长度记为 k[0],k[1]...k[m-1] 。请问 k[0]*k[1]*...*k[m-1] 可能的最大乘积是多少？例如，当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此时得到的最大乘积是18。
>
> 示例 1：
>
> 输入: 2
> 输出: 1
> 解释: 2 = 1 + 1, 1 × 1 = 1
> 示例 2:
>
> 输入: 10
> 输出: 36
> 解释: 10 = 3 + 3 + 4, 3 × 3 × 4 = 36

**思路**

1. 我们想要求长度为`n`的绳子剪掉后的最大乘积，可以从前面比`n`小的绳子转移而来。
2. 用一个`dp数组`记录`从0到n`长度的绳子剪掉后的最大乘积，也就是`dp[i]`表示长度为`i`的绳子剪成`m`段后的最大乘积，初始化`dp[2] = 1`
3. 我们先把绳子剪掉第一段`（长度为j）`，如果只剪掉长度为1，对最后的乘积无任何增益，所以从长度为2开始剪。
4. 剪了第一段后，剩下(i - j)长度可以剪也可以不剪。如果不剪的话长度乘积即为`j * (i - j)`；如果剪的话长度乘积即为`j * dp[i - j]`。取两者最大值`max(j * (i - j), j * dp[i - j])`。
5. 第一段长度j可以取的区间为`[2,i)`，对所有j不同的情况取最大值，因此最终dp[i]的转移方程为
   `dp[i] = max(dp[i], max(j * (i - j), j * dp[i - j]))`
6. 最后返回`dp[n]`即可.

# 贪心算法

## 贪心算法的思想

贪心的意思在于**在作出选择时，每次都要选择对自身最为有利的结果，保证自身利益的最大化**。贪心算法就是利用这种贪心思想而得出一种算法。

贪心算法可以简单描述为：**大事化小，小事化了**。对于一个较大的问题，通过找到与子问题的重叠，把复杂的问题划分为多个小问题。并且对于每个子问题的解进行选择，找出最优值，进行处理，再找出最优值，再处理。也就是说贪心算法是一种在每一步选择中都采取在当前状态下最好或最优的选择，从而希望得到结果是最好或最优的算法。

贪心算法在对问题求解时，总是做出在当前看来是最好的选择。也就是说，不从整体最优上加以考虑，所做出的仅是在某种意义上的局部最优解。贪心算法不是对所有问题都能得到整体最优解，但对范围相当广泛的许多问题他能产生整体最优解或者是整体最优解的近似解。

## 算法流程

1. 建立数学模型来描述问题。
2. 把求解的问题分成若干个子问题。
3. 对每一子问题求解，得到**子问题的局部最优解**。
4. 把子问题的局部最优解合成原来问题的一个解。

## 伪代码

```java
从问题的某一初始解出发
    while (能朝给定总目标前进一步) 
        do
            选择当前最优解作为可行解的一个解元素；
    由所有解元素组合成问题的一个可行解。
```

## 示例

> 小明手中有 1，5，10，50，100 五种面额的纸币，每种纸币对应张数分别为 5，2，2，3，5 张。若小明需要支付 456 元，则需要多少张纸币？

![贪心算法示例](https://cos.duktig.cn/typora/202109151611941.jpg)

### 题目分析

（1）**建立数学模型**
设小明每次选择纸币面额为 Xi ，需要的纸币张数为 n 张，剩余待支付金额为 V ，则有：`X1 + X2 + … + Xn = 456`

（2）**问题拆分为子问题**
小明选择纸币进行支付的过程，可以划分为n个子问题，即每个子问题对应为：**在未超过456的前提下，在剩余的纸币中选择一张纸币**。

（3）**制定贪心策略，求解子问题**

制定的贪心策略为：在允许的条件下选择面额最大的纸币。则整个求解过程如下：

- 选取面值为 100 的纸币，则 `X1 = 100, V = 456 – 100 = 356`；
- 继续选取面值为 100 的纸币，则 `X2 = 100, V = 356 – 100 = 256`；
- 继续选取面值为 100 的纸币，则 `X3 = 100, V = 256 – 100 = 156`；
- 继续选取面值为 100 的纸币，则 `X4 = 100, V = 156 – 100 = 56`；
- 选取面值为 50 的纸币，则 `X5 = 50, V = 56 – 50 = 6`；
- 选取面值为 5 的纸币，则 `X6 = 5, V = 6 – 5 = 1`；
- 选取面值为 1 的纸币，则 `X7 = 1, V = 1 – 1 = 0`；求解结束

（4）**将所有解元素合并为原问题的解**

小明需要支付的纸币张数为 7 张，其中面值 100 元的 4 张，50 元 1 张，5 元 1 张，1 元 1 张。

### 代码实现

```java
int greedyPayment(int payMoney) {
    // 纸币的类型
    int[] moneyType = new int[] {1, 5, 10, 50, 100};
    // 每种纸币的张数
    int[] count = new int[] {5, 2, 2, 3, 5};
    int sum = 0;
    for (int i = moneyType.length - 1; i >= 0 && payMoney > 0; i--) {
        // 贪心算法，每次选取最大的纸币，计算需要的张数
        int temp = Math.min(payMoney / moneyType[i], count[i]);
        // 计算剩余待支付的金额
        payMoney -= temp * moneyType[i];
        sum += temp;
    }
    // 计算完，不够支付，返回-1
    if (payMoney > 0) {
        return - 1;
    }
    return sum;
}
```



## 剑指offer14题——剪绳子（贪心算法解法）

[https://leetcode-cn.com/problems/jian-sheng-zi-lcof/](https://leetcode-cn.com/problems/jian-sheng-zi-lcof/)

**思路**

推论参看：[https://leetcode-cn.com/problems/jian-sheng-zi-lcof/solution/mian-shi-ti-14-i-jian-sheng-zi-tan-xin-si-xiang-by/](https://leetcode-cn.com/problems/jian-sheng-zi-lcof/solution/mian-shi-ti-14-i-jian-sheng-zi-tan-xin-si-xiang-by/)

**数学推论：**

> **推论一：** 将绳子 **以相等的长度等分为多段** ，得到的乘积最大。
>
> **推论二：** 尽可能将绳子以长度 33 等分为多段时，乘积最大。

**贪心推论**：

> **推论一：** 合理的切分方案可以带来更大的乘积。
>
>  **推论二：** 若切分方案合理，绳子段切分的越多，乘积越大。
>
> **推论三：** **为使乘积最大，只有长度为 2和 3的绳子不应再切分，且 3比 2更优** 。
>
> **所以，尽可能以3进行切割，3切割完最后一段如果等于4，那么平分成2段。**

**代码实现**

可以统计次数最后再计算，也可以每次剪的时候就计算好。

```java
/**
 * 贪心算法实现 优化后
 */
public int cuttingRopeByGreedy(int n) {
    /*
     *  三种特殊情况：
     *  1、长度为1时，没法剪，最大乘积为0
     *  2、长度为2时，最大乘积为1 × 1 = 1
     *  3、长度为3时，最大乘积为1 × 2 = 2
     */
    if (n <= 3) {
        return n - 1;
    }
    // 计算绳子每次剪3后的乘积
    int res = 1;
    while (n > 4) {
        res *= 3;
        n -= 3;
    }
    // 最后再乘上不能剪成3的，和可以剪成4（平分的）
    return res * n;
}

/**
 * 贪心算法实现 详细
 */
public int cuttingRopeByGreedyDetail(int n) {
    /*
     *  三种特殊情况：
     *  1、长度为1时，没法剪，最大乘积为0
     *  2、长度为2时，最大乘积为1 × 1 = 1
     *  3、长度为3时，最大乘积为1 × 2 = 2
     */
    if (n <= 3) {
        return n - 1;
    }

    // 尽可能多的，计算绳子每次剪3 的次数
    int threeCount = 0;
    while (n > 4) {
        n -= 3;
        threeCount++;
    }
    return (int) Math.pow(3, threeCount) * n;
}
```



# 位运算

## 剑指offer15题——二进制中1的个数

[https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)

### 方法一：循环检查二进制位

**思路**：

循环检查给定整数 n 的二进制位的每一位是否为 1 （二进制最高为2^31^-1，所以循环条件 < 32）
缺点：无论二进制有多少个1，都要循环32次，虽然时间复杂度为O(1)，但是实际上还是有很大的优化空间

**代码实现**：

```java
public int hammingWeight2(int n) {
    int count = 0;
    for (int i = 0; i < 32; i++) {
        // 这里的判断条件不能是 == 1
        if ((n & (1 << i)) != 0) {
            count++;
        }
    }
    return count;
}
```

**注意**：

**循环中的判断条件`(n & (1 << i))`不能是`==1`，必须要是`!=0`才行？**

- 用`n=11`做测试，操作：`System.out.println((n & (1 << i)));`，当前位为1，发现输出的结果并不是1，而是`1,2,8`。
- 又根据 `101 & 010 = 000 = 0`  和 `111 & 010 = 010 = 2`，所以确定高位为1时的条件应该为`(n & (1 << i)) != 0`。

### 方法二：位运算优化

**思路**：

1. 最后一位是0，减1后，最后一位变成0，其他不变
2. 最后一位不是0，假设最右边1位于m位，减去1，第m为变成0，m位之后都由0变成1，m位之前不变
3. 减去1之后的数 与 n 进行取余，第m位之后的数变成0。 即结果： n最右边为1的位变成0

所以，先减1，然后结果对n取余。

举例：

- `1100 -1 = 1011`
- `1011 & 1100 = 1000`

**代码实现**：

```java
public int hammingWeight(int n) {
    int count = 0;
    while (n != 0) {
        ++ count;
        // 可以简写为 n &= (n-1);
        n = (n - 1) & n;
    }
    return count;
}
```

