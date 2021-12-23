# BFS

## Template

- 定义一个队列维护节点的按层次展开的顺序
    - 对于基础的二维矩阵上的BFS，节点的结构一般为[ 横坐标, 纵坐标, 第几层]
- 首先遍历所以节点，将初始状态的节点都全部加入队列
- 从队列取出头节点，寻找头节点相邻的合法节点
    - 合法节点需要是没有被访问过的，在二维矩阵上可以通过设计矩阵的值来判断，其他场景需要一个visited数组记录
    - 再把这些合法节点从尾部塞入队列，直到队列为空。

```jsx
function bfs() {
  let q = [];
  for (node of nodes) {
    if (isEntry(node)) {
      q.push(node);
    }
  }

  while (q.length) {
    let pos = q.shift();
    for (neighbor of pos.neighbors) {
      q.push(neighbor);
    }
  }
}
```

## Leetcode 542

题目：

给定一个由 `0` 和 `1` 组成的矩阵 `mat` ，请输出一个大小相同的矩阵，其中每一个格子是 `mat` 中对应位置元素到最近的 `0` 的距离。

示例：

```jsx
// 输入
[[0,0,0],
 [0,1,0],
 [1,1,1]]

// 输出
[[0,0,0],
 [0,1,0],
 [1,2,1]]
```

题目要求将矩阵的值更新成距离最近0的距离，即从值为0的坐标开始BFS，把值不为0的更新成BFS的层级数

- 首先遍历二维数组，将所有值为0的位置入队，且层级为0，同时更新所有不为0的值为Infinity，再之后寻找下一层级的合法节点时可以用来做判断
- 可以定义一个方向的helper数组方便写代码 `dir = [[-1, 0],[1, 0],[0, -1],[0, 1]];`
- 从队列中取出此时的头节点，如果原数组中头节点坐标对应的值大于头节点的层次，则更新成头节点的层次
    - 一开始是Infinity，更新成层级数
    - 有可能有个1在一个0的旁边，在另一个0旁边的旁边，所以要维护成最小值
- 构造下一层节点next，位置通过方向数组构造，层级+1，`next = [pos[0] + d[0], pos[1] + d[1], pos[2] + 1]`
- 判断next节点的合法性，需要判断下标是否越界还有是否访问过（值是否是Infinity），然后加入队尾

```jsx
var updateMatrix = function (matrix) {
  let q = []; // 维护层级顺序的队列
  for (let i = 0; i < matrix.length; i++) {
    for (let j = 0; j < matrix[0].length; j++) {
      if (matrix[i][j] === 0) {
        q.push([i, j, 0]); // 从值为0的节点为起点开始BFS
      } else {
        matrix[i][j] = Infinity;// 把值不为0的节点置为一个不影响的特殊值，可以判断出未访问过
      }
    }
  }
  let dir = [
    [-1, 0],
    [1, 0],
    [0, -1],
    [0, 1],
  ]; // 维护上下左右方向偏移
  while (q.length) {
    let pos = q.shift(); // 取出此时的头节点
    if (matrix[pos[0]][pos[1]] > pos[2]) {
      matrix[pos[0]][pos[1]] = pos[2];
    } // matric需要维护距离最近0的距离，存在若干距离0的步数，所以维护最小值
    dir.forEach((d) => {
      let next = [pos[0] + d[0], pos[1] + d[1], pos[2] + 1];// 构造下一层节点
      if (
        next[0] > -1 &&
        next[0] < matrix.length &&
        next[1] > -1 &&
        next[1] < matrix[0].length
      ) {
        if (matrix[next[0]][next[1]] === Infinity) {
          q.push(next);// 如果节点坐标没越界，值也是初始化的特殊值代表未访问，则加入队尾
        }
      }
    });
  }
  return matrix;
};
```

## [Leetcode 994](https://leetcode-cn.com/problems/rotting-oranges)

题目：

在给定的网格中，每个单元格可以有以下三个值之一：

值 0 代表空单元格；
值 1 代表新鲜橘子；
值 2 代表腐烂的橘子。
每分钟，任何与腐烂的橘子（在 4 个正方向上）相邻的新鲜橘子都会腐烂。

返回直到单元格中没有新鲜橘子为止所必须经过的最小分钟数。如果不可能，返回 -1

示例：

![Untitled](BFS%20cd60c8df0a8f4fdba1415d3df6ef2368/Untitled.png)

```
输入：[[2,1,1],[1,1,0],[0,1,1]]
输出：4
```

和542一样的代码，不同处在于

从所有值为2的的位置开始bfs，层级为0，将所有1更新成Infinity，2更新成-Infinity

然后正常跑BFS模拟橘子腐烂的过程

最后的结果从matrix数组中取得，如果还存在infinity则代表有🍊未腐烂，返回-1，否则返回`Math.min(...matrix.flat(), 0)`

```jsx
var orangesRotting = function (matrix) {
  let q = [];
  for (let i = 0; i < matrix.length; i++) {
    for (let j = 0; j < matrix[0].length; j++) {
      if (matrix[i][j] === 2) {
        q.push([i, j, 0]);
      } else if (matrix[i][j] === 1) {
        matrix[i][j] = Infinity;
      } else {
        matrix[i][j] = -Infinity;
      }
    }
  }
  const dir = [
    [-1, 0],
    [1, 0],
    [0, 1],
    [0, -1],
  ];
  while (q.length) {
    let pos = q.shift();
    if (matrix[pos[0]][pos[1]] > pos[2]) {
      matrix[pos[0]][pos[1]] = pos[2];
    }
    dir.forEach((d) => {
      let next = [pos[0] + d[0], pos[1] + d[1], pos[2] + 1];
      if (
        next[0] > -1 &&
        next[0] < matrix.length &&
        next[1] > -1 &&
        next[1] < matrix[0].length
      ) {
        if (matrix[next[0]][next[1]] === Infinity) {
          q.push(next);
        }
      }
    });
  }
  if (matrix.flat().some((_) => _ === Infinity)) {
    return -1;
  } else {
    return Math.max(...matrix.flat());
  }
};
```