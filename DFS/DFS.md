# DFS

template：

1. 满足递归终止条件直接返回
2. 修改当前状态
3. 枚举可到达的所有下一状态，递归搜索

```jsx

function DFS(currentState) {
  if (isValid(currentState)) {
    return; // 当前状态满足递归终止条件就直接（记录）返回
  }
  modify(currentState); // 修改当前状态
  for(nextState of currentState.nextStates) { // 枚举当前状态能达到的所有nextState
    dfs(nextState) // 递归搜索下一个状态
  }
}
```

## [Leetcode 733](https://leetcode-cn.com/problems/flood-fill/)

输入和输出：

```
输入:
image = [[1,1,1],[1,1,0],[1,0,1]]
sr = 1, sc = 1, newColor = 2
输出: [[2,2,2],[2,2,0],[2,0,1]]
解析:
在图像的正中间，(坐标(sr,sc)=(1,1)),
在路径上所有符合条件的像素点的颜色都被更改成2。
注意，右下角的像素没有更改为2，
因为它不是在上下左右四个方向上与初始点相连的像素点。
```

> 设计dfs函数的参数，包括image，sr，sc，可以组合出currentState，此外还需要newColor模拟染色过程，oldColor作为优化参数进行剪枝。
> 
1. 递归终止条件（即搜索树的最底层节点）用到了所有参数，剪枝了数组越界，和已经被染色的情况
2. 然后修改当前状态（模拟染色）
3. 最后枚举所有可达到的下一状态，上下左右直接相连sr+-1， sc+-1，总共四个状态递归进行搜索，最终返回被修改的image

```jsx
var floodFill = function (image, sr, sc, newColor) {
  function dfs(image, sr, sc, newColor, oldColor) {
    if (
      sr < 0 ||
      sr >= image.length ||
      sc < 0 ||
      sc > image[0].length ||
      image[sr][sc] === newColor ||
      image[sr][sc] !== oldColor
    ) {
      return image;
    }
    image[sr][sc] = newColor;
    dfs(image, sr + 1, sc, newColor, oldColor);
    dfs(image, sr - 1, sc, newColor, oldColor);
    dfs(image, sr, sc + 1, newColor, oldColor);
    dfs(image, sr, sc - 1, newColor, oldColor);
    return image;
  }
  return dfs(image, sr, sc, newColor, image[sr][sc]);
};
```

## [Leetcode 695](https://leetcode-cn.com/problems/max-area-of-island/)

题目：

```markdown
给你一个大小为 `m x n` 的二进制矩阵 `grid` 。

**岛屿** 是由一些相邻的 `1` (代表土地) 构成的组合，
这里的「相邻」要求两个 `1` 必须在 **水平或者竖直的四个方向上** 相邻。
你可以假设 `grid` 的四个边缘都被 `0`（代表水）包围着。

岛屿的面积是岛上值为 `1` 的单元格的数目。

计算并返回 `grid` 中最大的岛屿面积。如果没有岛屿，则返回面积为 `0` 。
```

> 总体思路是枚举grid的所有坐标，对其跑DFS，再跑DFS的过程中，每修改当前状态把1变0一次，则记录+1，返回以该坐标开始DFS最多修改状态的次数，即岛屿的最大面积。最终复杂度为O(mn)，我们最多把m*n个1翻转成0，已经翻转成0的话可以剪枝掉。设计DFS函数时，参数包括grid，sr，sc，就可以维护出当前状态和剪枝情况
> 
1. 搜索树的最底层节点情况为数组越界或者此时已经为0，此时返回nums = 0
2. 修改当前状态把1变成0，此时记录nums = 1
3. 枚举所有可以到达的节点，上下左右，然后累加到nums上，最终返回nums

```jsx
function dfs(grid, sr, sc) {
  if (
    sr < 0 ||
    sc < 0 ||
    sr >= grid.length ||
    sc >= grid[0].length ||
    grid[sr][sc] === 0
  ) {
    return 0;
  }

  grid[sr][sc] = 0;
  let nums = 1;
  nums += dfs(grid, sr + 1, sc);
  nums += dfs(grid, sr - 1, sc);
  nums += dfs(grid, sr, sc + 1);
  nums += dfs(grid, sr, sc - 1);
  return nums;
}

var maxAreaOfIsland = function (grid) {
  let res = 0;

  for (let i = 0; i < grid.length; i++) {
    for (let j = 0; j < grid[0].length; j++) {
      let temp = dfs(grid, i, j);
      res = Math.max(temp, res);
    }
  }
  return res;
};
```

## [Leetcode 617](https://leetcode-cn.com/problems/merge-two-binary-trees/)

输入和输入：

```markdown
输入:
Tree 1                     Tree 2
1                         2
/ \                       / \
3   2                     1   3
/                           \   \
5                             4   7
输出:
合并后的树:
3
/ \
4   5
/ \   \
5   4   7
```

1. 递归终止条件是两个合并的树又一个是空，则返回另外一棵
2. 修改新树的值为tree的值+tree2的值
3. 在递归修改新树的左子树和右子树，最终返回新树的根节点

```jsx
var mergeTrees = function (root1, root2) {
  if (root1 === null) return root2;
  if (root2 === null) return root1;
  let newNode = new TreeNode();
  newNode.val = root1.val + root2.val;
  newNode.left = mergeTrees(root1.left, root2.left);
  newNode.right = mergeTrees(root1.right, root2.right);
  return newNode;
};
```

## [Leetcode 116](https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node/)

输入和输出：

![Untitled](DFS%2048e8c95f4082436ab0088c75e3c174bd/Untitled.png)

> DFS函数的参数包括左右子树，为了维护递归终止条件和第二部的修改状态
> 
1. 递归的终止条件为左子树或右子树有一个为空，则直接返回
2. 修改左子树的根节点，将其next指向右节点
3. 此时通过左右节点，访问到左右节点下一层的左右节点，递归将左节点的next指向右节点

```jsx
function DFS(left, right) {
  if (left === null || right === null) {
    return;
  }
  left.next = right;
  DFS(left.left, left.right);
  DFS(left.right, right.left);
  DFS(right.left, right.right);
}

var connect = function (root) {
  if (root === null) return null;
  DFS(root.left, root.right);
  return root;
};
```