# 深度优先搜索

> **深度优先搜索** 算法（英语：Depth-First-Search，**DFS**）是一种用于遍历或搜索树或图的算法。这个算法会 **尽可能深** 的搜索树的分支。当结点 `v` 的所在边都己被探寻过，搜索将 **回溯** 到发现结点` v `的那条边的起始结点。这一过程一直进行到已发现从源结点可达的所有结点为止。如果还存在未被发现的结点，则选择其中一个作为源结点并重复以上过程，整个进程反复进行直到所有结点都被访问为止。

「回溯算法」与「深度优先遍历」都有「不撞南墙不回头」的意思。一个理解是：「回溯算法」强调了「深度优先遍历」思想的用途，用一个 **不断变化** 的变量，在尝试各种可能的过程中，搜索需要的结果。强调了 **回退** 操作对于搜索的合理性。而「深度优先遍历」强调一种遍历的思想，与之对应的遍历思想是「广度优先遍历」。

搜索引擎的「搜索」和「回溯搜索」算法里「搜索」的意思是一样的。

搜索问题的解，可以通过 **遍历** 实现。所以很多教程把「回溯算法」称为爆搜（暴力解法）。因此回溯算法用于 **搜索一个问题的所有的解** ，通过深度优先遍历的思想实现。

## 岛屿问题（网格）

我们所熟悉的 DFS（深度优先搜索）问题通常是在树或者图结构上进行的。而我们今天要讨论的 DFS 问题，是在一种「**网格**」结构中进行的。岛屿问题是这类网格 DFS 问题的典型代表。网格结构遍历起来要比二叉树复杂一些，如果没有掌握一定的方法，DFS 代码容易写得冗长繁杂。

### 网格类问题的 DFS 遍历方法

#### 网格问题的基本概念

首先明确一下岛屿问题中的网格结构是如何定义的？

网格问题是由 m×n 个小方格组成一个网格，每个小方格与其上下左右四个方格认为是相邻的，要在这样的网格上进行某种搜索。

岛屿问题是一类典型的网格问题。每个格子中的数字可能是 0 或者 1。我们把数字为 0 的格子看成海洋格子，数字为 1 的格子看成陆地格子，这样相邻的陆地格子就连接成一个岛屿。

![岛屿问题示例](https://cos.duktig.cn/typora/202112181649619.jpeg)

在这样一个设定下，就出现了各种岛屿问题的变种，包括岛屿的数量、面积、周长等。不过这些问题，基本都可以用 DFS 遍历来解决。

#### 网格问题的 DFS 基本结构

网格结构要比二叉树结构稍微复杂一些，它其实是一种简化版的图结构。要写好网格上的 DFS 遍历，我们首先要理解二叉树上的 DFS 遍历方法，再类比写出网格结构上的 DFS 遍历。我们写的 **二叉树 DFS 遍历** 一般是这样的：

```java
void traverse(TreeNode root) {
    // 判断 base case
    if (root == null) {
        return;
    }
    // 访问两个相邻结点：左子结点、右子结点
    traverse(root.left);
    traverse(root.right);
}
```

可以看到，二叉树的 DFS 有两个要素：「**访问相邻结点**」和「**判断 base case**」。

第一个要素是访问相邻结点。二叉树的相邻结点非常简单，只有左子结点和右子结点两个。二叉树本身就是一个递归定义的结构：一棵二叉树，它的左子树和右子树也是一棵二叉树。那么我们的 DFS 遍历只需要递归调用左子树和右子树即可。

第二个要素是 判断 base case。一般来说，二叉树遍历的 base case 是 `root == null`。这样一个条件判断其实有两个含义：一方面，这表示 root 指向的子树为空，不需要再往下遍历了。另一方面，在 `root == null` 的时候及时返回，可以让后面的 root.left 和 root.right 操作不会出现空指针异常。

对于网格上的 DFS，我们完全可以参考二叉树的 DFS，写出网格 DFS 的两个要素：

首先，网格结构中的格子有多少相邻结点？答案是上下左右四个。对于格子 (r, c) 来说（r 和 c 分别代表行坐标和列坐标），四个相邻的格子分别是 (r-1, c)、(r+1, c)、(r, c-1)、(r, c+1)。换句话说，网格结构是「四叉」的。

![网格结构中四个相邻的格子](https://cos.duktig.cn/typora/202112181656265.jpeg)

其次，网格 DFS 中的 base case 是什么？从二叉树的 base case 对应过来，应该是网格中不需要继续遍历、`grid[r][c]` 会出现数组下标越界异常的格子，也就是那些超出网格范围的格子。

![网格 DFS 的 base case](https://cos.duktig.cn/typora/202112181657709.jpeg)

这一点稍微有些反直觉，坐标竟然可以临时超出网格的范围？这种方法我称为「**先污染后治理**」—— 甭管当前是在哪个格子，先往四个方向走一步再说，如果发现走出了网格范围再赶紧返回。这跟二叉树的遍历方法是一样的，先递归调用，发现 `root == null` 再返回。

#### 如何避免重复遍历

网格结构的 DFS 与二叉树的 DFS 最大的不同之处在于，**遍历中可能遇到遍历过的结点**。这是因为，网格结构本质上是一个「图」，我们可以把每个格子看成图中的结点，每个结点有向上下左右的四条边。在图中遍历时，自然可能遇到重复遍历结点。

这时候，DFS 可能会不停地「兜圈子」，永远停不下来，如下图所示：

![DFS 遍历可能会兜圈子（动图）](https://cos.duktig.cn/typora/202112181659979.gif)

如何避免这样的重复遍历呢？答案是标记已经遍历过的格子。以岛屿问题为例，我们需要在所有值为 1 的陆地格子上做 DFS 遍历。每走过一个陆地格子，就把格子的值改为 2，这样当我们遇到 2 的时候，就知道这是遍历过的格子了。也就是说，每个格子可能取三个值：

- 0 —— 海洋格子
- 1 —— 陆地格子（未遍历过）
- 2 —— 陆地格子（已遍历过）

这样，我们就得到了一个岛屿问题、乃至各种网格问题的通用 DFS 遍历方法。

```java
// 实质上在遍历过后，将网格置为2，意为将这个岛屿淹没
void dfs(int[][] grid, int r, int c) {
    // 判断 base case
    if (!inArea(grid, r, c)) {
        return;
    }
    // 如果这个格子不是岛屿，直接返回
    if (grid[r][c] != 1) {
        return;
    }
    grid[r][c] = 2; // 将格子标记为「已遍历过」
    
    // 访问上、下、左、右四个相邻结点
    dfs(grid, r - 1, c);
    dfs(grid, r + 1, c);
    dfs(grid, r, c - 1);
    dfs(grid, r, c + 1);
}

// 判断坐标 (r, c) 是否在网格中
boolean inArea(int[][] grid, int r, int c) {
    return 0 <= r && r < grid.length 
        	&& 0 <= c && c < grid[0].length;
}
```

> 小贴士：
>
> 在一些题解中，可能会把「已遍历过的陆地格子」标记为和海洋格子一样的 0，美其名曰「陆地沉没方法」，即遍历完一个陆地格子就让陆地「沉没」为海洋。这种方法看似很巧妙，但实际上有很大隐患，因为这样我们就无法区分「海洋格子」和「已遍历过的陆地格子」了。如果题目更复杂一点，这很容易出 bug。

### 岛屿的最大面积

> [695. 岛屿的最大面积](https://leetcode-cn.com/problems/max-area-of-island/)
>
> 给定一个包含了一些 0 和 1 的非空二维数组 grid，一个岛屿是一组相邻的 1（代表陆地），这里的「相邻」要求两个 1 必须在水平或者竖直方向上相邻。你可以假设 grid 的四个边缘都被 0（代表海洋）包围着。
>
> 找到给定的二维数组中最大的岛屿面积。如果没有岛屿，则返回面积为 0 。

这道题目只需要对每个岛屿做 DFS 遍历，求出每个岛屿的面积就可以了。求岛屿面积的方法也很简单，代码如下，每遍历到一个格子，就把面积加一。

```java
/** 0代表海洋 */
final int ZERO = 0;
/** 1代表未被遍历过的陆地 */
final int ONE = 1;
/** 2代表已被遍历过的陆地 */
final int TWO = 2;

/**
 * 计算 岛屿面积最大值
 *
 * @param grid 岛屿和海洋的二维网格
 * @return 岛屿面积最大值
 */
public int maxAreaOfIsland(int[][] grid) {
    // 记录结果
    int res = 0;
    // 一次遍历网格（area方法中将遍历过的网格设为2，避免重复遍历）
    for (int r = 0; r < grid.length; r++) {
        for (int c = 0; c < grid[0].length; c++) {
            if (grid[r][c] == ONE) {
                // 计算当前岛屿的面积
                int a = area(grid, r, c);
                res = Math.max(res, a);
            }
        }
    }
    return res;
}

/**
 * 递归计算岛屿的面积（将遍历过的网格设为2，避免重复计算）
 *
 * @param grid 岛屿和海洋的二维网格
 * @param row  行索引
 * @param col  列索引
 * @return 当前岛屿的面积
 */
private int area(int[][] grid, int row, int col) {
    // 越界，停止递归
    if (! inArea(grid, row, col)) {
        return 0;
    }
    // 当前网格遍历过，停止递归
    if (grid[row][col] != ONE) {
        return 0;
    }
    // 将遍历过的网格，设为2，避免下次重复遍历
    grid[row][col] = TWO;

    // 递归计算岛屿的面积
    return 1
            + area(grid, row - 1, col)
            + area(grid, row + 1, col)
            + area(grid, row, col - 1)
            + area(grid, row, col + 1);
}

/**
 * 判断网格遍历时，是否越界
 *
 * @param grid 岛屿和海洋的二维网格
 * @param row  行索引
 * @param col  列索引
 * @return 是否越界
 */
private boolean inArea(int[][] grid, int row, int col) {
    return 0 <= row && row < grid.length && 0 <= col && col < grid[0].length;
}
```



### 岛屿的周长

>  [463. 岛屿的周长](https://leetcode-cn.com/problems/island-perimeter/)
>
>  给定一个包含 0 和 1 的二维网格地图，其中 1 表示陆地，0 表示海洋。网格中的格子水平和垂直方向相连（对角线方向不相连）。整个网格被水完全包围，但其中恰好有一个岛屿（一个或多个表示陆地的格子相连组成岛屿）。
>
>  岛屿中没有“湖”（“湖” 指水域在岛屿内部且不和岛屿周围的水相连）。格子是边长为 1 的正方形。计算这个岛屿的周长。
>
>  ![题目示例](https://cos.duktig.cn/typora/202112181710343.jpeg)

实话说，这道题用 DFS 来解并不是最优的方法。对于岛屿，直接用数学的方法求周长会更容易。不过这道题是一个很好的理解 DFS 遍历过程的例题。

可以看到，dfs 函数直接返回有这几种情况：

- `!inArea(grid, r, c)`，即坐标 (r, c) 超出了网格的范围，也就是我所说的「先污染后治理」的情况
- `grid[r][c] != 1`，即当前格子不是岛屿格子，这又分为两种情况：
  - `grid[r][c] == 0`，当前格子是海洋格子
  - `grid[r][c] == 2`，当前格子是已遍历的陆地格子

那么这些和我们岛屿的周长有什么关系呢？实际上，**岛屿的周长是计算岛屿全部的「边缘」，而这些边缘就是我们在 DFS 遍历中，dfs 函数返回的位置**。观察题目示例，我们可以将岛屿的周长中的边分为两类，如下图所示。黄色的边是与网格边界相邻的周长，而蓝色的边是与海洋格子相邻的周长。

![将岛屿周长中的边分为两类](https://cos.duktig.cn/typora/202112181714017.jpeg)

当我们的 dfs 函数因为「坐标 (r, c) 超出网格范围」返回的时候，实际上就经过了一条黄色的边；而当函数因为「当前格子是海洋格子」返回的时候，实际上就经过了一条蓝色的边。这样，我们就把岛屿的周长跟 DFS 遍历联系起来了，我们的题解代码也呼之欲出：

```java
/** 0代表海洋 */
final int ZERO = 0;
/** 1代表未被遍历过的陆地 */
final int ONE = 1;
/** 2代表已被遍历过的陆地 */
final int TWO = 2;

/**
 * 计算岛屿的周长
 */
public int islandPerimeter(int[][] grid) {
    for (int r = 0; r < grid.length; r++) {
        for (int c = 0; c < grid[0].length; c++) {
            if (grid[r][c] == ONE) {
                // 题目限制只有一个岛屿，计算一个即可
                return dfs(grid, r, c);
            }
        }
    }
    return 0;
}

/**
 * DFS遍历
 */
private int dfs(int[][] grid, int r, int c) {
    // 函数因为「坐标 (r, c) 超出网格范围」返回，对应一条黄色的边
    if (! inArea(grid, r, c)) {
        return 1;
    }
    // 函数因为「当前格子是海洋格子」返回，对应一条蓝色的边
    if (grid[r][c] == ZERO) {
        return 1;
    }
    // 函数因为「当前格子是已遍历的陆地格子」返回，和周长没关系
    if (grid[r][c] != ONE) {
        return 0;
    }
    grid[r][c] = TWO;
    return dfs(grid, r - 1, c)
            + dfs(grid, r + 1, c)
            + dfs(grid, r, c - 1)
            + dfs(grid, r, c + 1);
}

/**
 * 判断坐标 (r, c) 是否在网格中
 */
private boolean inArea(int[][] grid, int r, int c) {
    return 0 <= r && r < grid.length && 0 <= c && c < grid[0].length;
}
```

### 岛屿数量

> [200. 岛屿数量](https://leetcode-cn.com/problems/number-of-islands/)

```java
/** 海洋 */
final int ZERO = '0';
/** 未遍历过的岛屿 */
final int ONE = '1';
/** 已遍历过的岛屿 */
final int TWO = '2';

public int numIslands(char[][] grid) {
    int res = 0;
    for (int r = 0; r < grid.length; r++) {
        for (int c = 0; c < grid[0].length; c++) {
            // 遇到一个岛屿直接标记，有标记 2 ，其他岛屿都进不来，直接标记即可
            if (grid[r][c] == ONE) {
                isLand(grid, r, c);
                res++;
            }
        }
    }
    return res;
}

private void isLand(char[][] grid, int r, int c) {
    if (! inArea(grid, r, c)) {
        return;
    }

    if (grid[r][c] != ONE) {
        return;
    }
    // 将遍历过的网格，设为2，避免下次重复遍历
    grid[r][c] = TWO;

    isLand(grid, r - 1, c);
    isLand(grid, r + 1, c);
    isLand(grid, r, c - 1);
    isLand(grid, r, c + 1);

}

/**
 * 判断是否在网格中
 */
private boolean inArea(char[][] grid, int row, int col) {
    return 0 <= row && row < grid.length && 0 <= col && col < grid[0].length;
}
```

### 统计封闭岛屿的数目

> [1254. 统计封闭岛屿的数目](https://leetcode-cn.com/problems/number-of-closed-islands/)
>
> 有一个二维矩阵 grid ，每个位置要么是陆地（记号为 0 ）要么是水域（记号为 1 ）。
>
> 我们从一块陆地出发，每次可以往上下左右 4 个方向相邻区域走，能走到的所有陆地区域，我们将其称为一座「岛屿」。
>
> 如果一座岛屿 完全 由水域包围，即陆地边缘上下左右所有相邻区域都是水域，那么我们将其称为 「封闭岛屿」。
>
> 请返回封闭岛屿的数目。

**那么如何判断「封闭岛屿」呢？其实很简单，把上⼀题中那些靠边的岛屿排除掉，剩下的不就是「封闭岛屿」了吗？**

```java
/** 海洋 */
final int OCEAN = 1;
/** 未遍历过的岛屿 */
final int LAND = 0;
/** 已遍历过的岛屿 */
final int LAND_SEARCHED = 2;

/**
 * 主函数
 */
public int closedIsland(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    // 上下边缘 淹没
    for (int i = 0; i < m; i++) {
        dfs(grid, i, 0);
        dfs(grid, i, n - 1);
    }
    // 左右边缘 淹没
    for (int i = 0; i < n; i++) {
        dfs(grid, 0, i);
        dfs(grid, m - 1, i);
    }
    // 记录封闭岛屿数量
    int res = 0;
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (grid[i][j] == LAND) {
                dfs(grid, i, j);
                res++;
            }
        }
    }
    return res;
}

/**
 * 岛屿的通用遍历方法
 * 统计过一片岛屿后，将其淹没
 */
private void dfs(int[][] grid, int r, int c) {
    // 不在网格直接返回
    if (! inArea(grid, r, c)) {
        return;
    }
    // 不是陆地直接返回
    if (grid[r][c] != LAND) {
        return;
    }
    // 遍历过的格子 置为 2
    grid[r][c] = LAND_SEARCHED;

    // 递归走四个方向
    dfs(grid, r + 1, c);
    dfs(grid, r - 1, c);
    dfs(grid, r, c + 1);
    dfs(grid, r, c - 1);
}

/**
 * 判断是否在网格当中
 */
private boolean inArea(int[][] grid, int r, int c) {
    return r >= 0 && r < grid.length
            && c >= 0 && c < grid[0].length;
}
```

只要提前把靠边的陆地都淹掉，然后算出来的就是封闭岛屿了。

### 飞地的数量

> [1020. 飞地的数量](https://leetcode-cn.com/problems/number-of-enclaves/)    

与上题解法基本一致，只是在循环中不再进行dfs遍历（淹没）。

```java
/** 海洋 */
final int OCEAN = 0;
/** 未遍历过的岛屿 */
final int LAND = 1;
/** 已遍历过的岛屿 */
final int LAND_SEARCHED = 2;

/**
  * 主函数
  */
public int numEnclaves(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    // 上下边缘 淹没
    for (int i = 0; i < m; i++) {
        dfs(grid, i, 0);
        dfs(grid, i, n - 1);
    }
    // 左右边缘 淹没
    for (int i = 0; i < n; i++) {
        dfs(grid, 0, i);
        dfs(grid, m - 1, i);
    }
    // 记录封闭岛屿数量
    int res = 0;
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (grid[i][j] == LAND) {
                res++;
            }
        }
    }
    return res;
}

/**
  * 岛屿的通用遍历方法
  * 统计过一片岛屿后，将其淹没
  */
private void dfs(int[][] grid, int r, int c) {
    // 不在网格直接返回
    if (! inArea(grid, r, c)) {
        return;
    }
    // 不是陆地直接返回
    if (grid[r][c] != LAND) {
        return;
    }
    // 遍历过的格子 置为 2
    grid[r][c] = LAND_SEARCHED;

    // 递归走四个方向
    dfs(grid, r + 1, c);
    dfs(grid, r - 1, c);
    dfs(grid, r, c + 1);
    dfs(grid, r, c - 1);
}

/**
  * 判断是否在网格当中
  */
private boolean inArea(int[][] grid, int r, int c) {
    return r >= 0 && r < grid.length
        && c >= 0 && c < grid[0].length;
}
```

## 统计子岛屿

> [1905. 统计子岛屿](https://leetcode-cn.com/problems/count-sub-islands/)
>
> 给你两个 m x n 的二进制矩阵 grid1 和 grid2 ，它们只包含 0 （表示水域）和 1 （表示陆地）。一个 岛屿 是由 四个方向 （水平或者竖直）上相邻的 1 组成的区域。任何矩阵以外的区域都视为水域。
>
> 如果 grid2 的一个岛屿，被 grid1 的一个岛屿 完全 包含，也就是说 grid2 中该岛屿的每一个格子都被 grid1 中同一个岛屿完全包含，那么我们称 grid2 中的这个岛屿为 子岛屿 。
>
> 请你返回 grid2 中 子岛屿 的 数目 。

**这道题的关键在于，如何快速判断⼦岛屿？**

什么情况下 grid2 中的⼀个岛屿 B 是 grid1 中的⼀个岛屿 A 的⼦岛？

当岛屿 B 中所有陆地在岛屿 A 中也是陆地的时候，岛屿 B 是岛屿 A 的⼦岛。

**反过来说，如果岛屿 B 中存在⼀⽚陆地，在岛屿 A 的对应位置是海⽔，那么岛屿 B 就不是岛屿 A 的⼦岛。**

那么，我们只要遍历 grid2 中的所有岛屿，把那些不可能是⼦岛的岛屿排除掉，剩下的就是⼦岛。

```java
/** 海洋 */
final int OCEAN = 0;
/** 未遍历过的岛屿 */
final int LAND = 1;
/** 已遍历过的岛屿 */
final int LAND_SEARCHED = 2;

/**
 * 主函数
 */
public int countSubIslands(int[][] grid1, int[][] grid2) {
    int m = grid2.length, n = grid2[0].length;
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (grid1[i][j] == OCEAN && grid2[i][j] == LAND) {
                // 这个岛屿肯定不是⼦岛，淹掉
                dfs(grid2, i, j);
            }
        }
    }
    // 记录封闭岛屿数量
    int res = 0;
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (grid2[i][j] == LAND) {
                dfs(grid2, i, j);
                res++;
            }
        }
    }
    return res;
}

/**
 * 岛屿的通用遍历方法
 * 统计过一片岛屿后，将其淹没
 */
private void dfs(int[][] grid, int r, int c) {
    // 不在网格直接返回
    if (! inArea(grid, r, c)) {
        return;
    }
    // 不是陆地直接返回
    if (grid[r][c] != LAND) {
        return;
    }
    // 遍历过的格子 置为 2
    grid[r][c] = LAND_SEARCHED;

    // 递归走四个方向
    dfs(grid, r + 1, c);
    dfs(grid, r - 1, c);
    dfs(grid, r, c + 1);
    dfs(grid, r, c - 1);
}

/**
 * 判断是否在网格当中
 */
private boolean inArea(int[][] grid, int r, int c) {
    return r >= 0 && r < grid.length
            && c >= 0 && c < grid[0].length;
}
```



### 不同的岛屿数量

> ⼒扣第 694 题「不同的岛屿数量」，题⽬还是输⼊⼀个⼆维矩阵，0 表示海⽔，1 表示陆地，这次让你计算
> 不同的 (distinct) 岛屿数量，函数签名如下：
>
> ```java
> int numDistinctIslands(int[][] grid)
> ```
>
>   ⽐如题⽬输⼊下⾯这个⼆维矩阵：
>
> ![image-20220120214133747](https://cos.duktig.cn/typora/202201202141219.png)
>
> 其中有四个岛屿，但是左下⻆和右上⻆的岛屿形状相同，所以不同的岛屿共有三个，算法返回 3。

很显然我们得想办法把⼆维矩阵中的「岛屿」进⾏转化，变成⽐如字符串这样的类型，然后利⽤ HashSet 这样的数据结构去重，最终得到不同的岛屿的个数。

如果想把岛屿转化成字符串，说⽩了就是序列化，序列化说⽩了就是遍历嘛，之前的 **⼆叉树的序列化和反序列化** 讲了⼆叉树和字符串互转，这⾥也是类似的。

**⾸先，对于形状相同的岛屿，如果从同⼀起点出发，dfs 函数遍历的顺序肯定是⼀样的。**

因为遍历顺序是写死在你的递归函数⾥⾯的，不会动态改变。

```java
void dfs(int[][] grid, int i, int j) { 
    // 递归顺序： 
    dfs(grid, i - 1, j); // 上 
    dfs(grid, i + 1, j); // 下 
    dfs(grid, i, j - 1); // 左 
    dfs(grid, i, j + 1); // 右 
} 
```

所以，遍历顺序从某种意义上说就可以⽤来描述岛屿的形状，⽐如下图这两个岛屿：

<img src="https://cos.duktig.cn/typora/202201202152367.png" alt="image-20220120215207962" style="zoom:80%;" />

假设它们的遍历顺序是：

> 下，右，上，撤销上，撤销右，撤销下

如果我⽤分别⽤ 1, 2, 3, 4 代表上下左右，⽤ -1, -2, -3, -4 代表上下左右的撤销，那么可以这样表示它们的遍历顺序：

> 2, 4, 1, -1, -4, -2

你看，这就相当于是岛屿序列化的结果，只要每次使⽤ dfs 遍历岛屿的时候⽣成这串数字进⾏⽐较，就可以计算到底有多少个不同的岛屿了。

**这就相当于是岛屿序列化的结果，只要每次使⽤ dfs 遍历岛屿的时候⽣成这串数字进⾏⽐较，就可以计算到底有多少个不同的岛屿了。**

我们需要稍微改造 dfs 函数，添加⼀些函数参数以便记录遍历顺序：

```java
/** 海洋 */
final int OCEAN = 0;
/** 未遍历过的岛屿 */
final int LAND = 1;
/** 已遍历过的岛屿 */
final int LAND_SEARCHED = 2;

/**
 * 主函数
 * HashSet记录岛屿的序列化，去重，统计个数
 */
public int numDistinctIslands(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    // 记录所有岛屿的序列化结果
    Set<String> lands = new HashSet<>();
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (grid[i][j] == LAND) {
                // 淹掉这个岛屿，同时存储岛屿的序列化结果
                StringBuilder sb = new StringBuilder();
                // 初始的⽅向可以随便写，不影响正确性
                dfs(grid, i, j, sb, 666);
                lands.add(sb.toString());
            }
        }
    }
    // 不相同的岛屿数量
    return lands.size();
}

/**
 * 将岛屿遍历，转为序列化字符串 上（1） 下（2） 左（3） 右（4）
 *
 * @param grid 岛屿
 * @param r    /
 * @param c    /
 * @param sb   岛屿序列化字符串
 * @param dir  方向
 */
private void dfs(int[][] grid, int r, int c, StringBuilder sb, int dir) {
    // 不在网格直接返回
    if (! inArea(grid, r, c)) {
        return;
    }
    // 不是陆地直接返回
    if (grid[r][c] != LAND) {
        return;
    }
    // 遍历过的格子 置为 2
    grid[r][c] = LAND_SEARCHED;

    sb.append(dir).append(',');

    // 上
    dfs(grid, r - 1, c, sb, 1);
    // 下
    dfs(grid, r + 1, c, sb, 2);
    // 左
    dfs(grid, r, c - 1, sb, 3);
    // 右
    dfs(grid, r, c + 1, sb, 4);

    // 后序遍历位置：离开 (i, j)
    sb.append(- dir).append(',');
}

/**
 * 判断是否在网格当中
 */
private boolean inArea(int[][] grid, int r, int c) {
    return r >= 0 && r < grid.length
            && c >= 0 && c < grid[0].length;
}
```

