### 1. 岛屿的周长

 给定一个包含 0 和 1 的二维网格地图，其中 1 表示陆地 0 表示水域。

网格中的格子水平和垂直方向相连（对角线方向不相连）。整个网格被水完全包围，但其中恰好有一个岛屿（或者说，一个或多个表示陆地的格子相连组成的岛屿）。

岛屿中没有“湖”（“湖” 指水域在岛屿内部且不和岛屿周围的水相连）。格子是边长为 1 的正方形。网格为长方形，且宽度和高度均不超过 100 。计算这个岛屿的周长。

**示例 :**

```
输入:
[[0,1,0,0],
 [1,1,1,0],
 [0,1,0,0],
 [1,1,0,0]]

输出: 16

解释: 它的周长是下面图片中的 16 个黄色的边：
```

![](img\09.png)

解法：使用DFS

如何在网格上做 DFS
网格问题是这样一类搜索问题：有 m×n 个小方格，组成一个网格，每个小方格与其上下左右四个方格认为是相邻的，要在这样的网格上进行某种搜索。这种题目乍一看可能有点麻烦，实际上非常简单，尤其是用 DFS 的方法。题目没有限制的话，我们尽量用 DFS 来写代码。

下面我们一步步地构造出方格类 DFS 的代码。

首先，每个方格与其上下左右的四个方格相邻，则 DFS 每次要分出四个岔：

```java
// 基本的 DFS 框架：每次搜索四个相邻方格
void dfs(int[][] grid, int r, int c) {
    dfs(grid, r - 1, c); // 上边相邻
    dfs(grid, r + 1, c); // 下边相邻
    dfs(grid, r, c - 1); // 左边相邻
    dfs(grid, r, c + 1); // 右边相邻
}
```


但是，对于网格边缘的方格，上下左右并不都有邻居。一种做法是在递归调用之前判断方格的位置，例如位于左边缘，则不访问其左邻居。但这样一个一个判断写起来比较麻烦，我们可以用“先污染后治理”的方法，先做递归调用，再在每个 DFS 函数的开头判断坐标是否合法，不合法的直接返回。同样地，我们还需要判断该方格是否有岛屿（值是否为 1），否则也需要返回。

```Java
// 处理方格位于网格边缘的情况
void dfs(int[][] grid, int r, int c) {
    // 若坐标不合法，直接返回
    if (!(0 <= r && r < grid.length && 0 <= c && c < grid[0].length)) {
        return;
    }
    // 若该方格不是岛屿，直接返回
    if (grid[r][c] != 1) {
        return;
    }
    dfs(grid, r - 1, c);
    dfs(grid, r + 1, c);
    dfs(grid, r, c - 1);
    dfs(grid, r, c + 1);
}
```

但是这样还有一个问题：DFS 可能会不停地“兜圈子”，永远停不下来，如下图所示：

![](C:\Users\VSUS\Desktop\笔记\LeetCode\img\10.gif)

那么我们需要标记遍历过的方格，保证方格不进行重复遍历。标记遍历过的方格并不需要使用额外的空间，只需要改变方格中存储的值就可以。在这道题中，值为 0 表示非岛屿（不可遍历），值为 1 表示岛屿（可遍历），我们用 2 表示已遍历过的岛屿。

这样，我们就得到了网格 DFS 遍历的框架代码：

```Java
// 标记已遍历过的岛屿，不做重复遍历
void dfs(int[][] grid, int r, int c) {
    if (!(0 <= r && r < grid.length && 0 <= c && c < grid[0].length)) {
        return;
    }
    // 已遍历过（值为2）的岛屿在这里会直接返回，不会重复遍历
    if (grid[r][c] != 1) {
        return;
    }
    grid[r][c] = 2; // 将方格标记为"已遍历"
    dfs(grid, r - 1, c); 
    dfs(grid, r + 1, c);
    dfs(grid, r, c - 1);
    dfs(grid, r, c + 1);
}
```


如何在 DFS 遍历时求岛屿的周长？求岛屿的周长其实有很多种方法，如果用 DFS 遍历来求的话，有一种很简单的思路：岛屿的周长就是岛屿方格和非岛屿方格相邻的边的数量。注意，这里的非岛屿方格，既包括水域方格，也包括网格的边界。我们可以画一张图，看得更清晰：

![](img\11.jpg)

将这个“相邻关系”对应到 DFS 遍历中，就是：每当在 DFS 遍历中，从一个岛屿方格走向一个非岛屿方格，就将周长加 1。代码如下：

```Java
int dfs(int[][] grid, int r, int c) {
    // 从一个岛屿方格走向网格边界，周长加 1
    if (!(0 <= r && r < grid.length && 0 <= c && c < grid[0].length)) {
        return 1;
    }
    // 从一个岛屿方格走向水域方格，周长加 1
    if (grid[r][c] == 0) {
        return 1;
    }
    if (grid[r][c] != 1) {
        return 0;
    }
    grid[r][c] = 2;
    return dfs(grid, r - 1, c)
        + dfs(grid, r + 1, c)
        + dfs(grid, r, c - 1)
        + dfs(grid, r, c + 1);
}
```


最终，我们得到完整的题解代码：

```Java
public int islandPerimeter(int[][] grid) {
    for (int r = 0; r < grid.length; r++) {
        for (int c = 0; c < grid[0].length; c++) {
            if (grid[r][c] == 1) {
                // 题目限制只有一个岛屿，计算一个即可
                return dfs(grid, r, c);
            }
        }
    }
    return 0;
}

int dfs(int[][] grid, int r, int c) {
    if (!(0 <= r && r < grid.length && 0 <= c && c < grid[0].length)) {
        return 1;
    }
    if (grid[r][c] == 0) {
        return 1;
    }
    if (grid[r][c] != 1) {
        return 0;
    }
    grid[r][c] = 2;
    return dfs(grid, r - 1, c)
        + dfs(grid, r + 1, c)
        + dfs(grid, r, c - 1)
        + dfs(grid, r, c + 1);
}
```

# 

