# BackTracking

## Template

```jsx
function Solve(total) {
  // total代表题目给出的总的步骤数
  let res = []; // 全局变量用于保存所有合法结果
  function dfs(currentState, currentStep) {
    if (isValid(currentState)) {
      res.push(copy(currentState)); // 如果是合法状态则保存进res，需要注意非primitive的结果需要拷贝一下
      return;
    }
    for (nextStepValue of currentStep.nextSteps) {
      // 这里currentStep一般是从 0到total，代表DFS进行到第几层
      // nextStepValue是枚举该层可供选择的值
      modify(currentState, nextStepValue); // 选择之后修改当前状态
      dfs(currentState, currentStep + 1); // 状态更新，递归搜索下一层
      undo(currentState, nextStepValue); //回溯， 撤销之前的状态更新
    }
  }
  dfs(initialState, initialStep); // 调用DFS，给出初始状态和初始步骤，一般是dfs([], 0) 或者dfs('', 0);
  return res;
}
```

题目会给出一个搜索空间集合和合法结果，其中搜索空间集合对应nextStepValue，生成合法结果的步骤对应initialStep ⇒ total

## [Leetcode 77](https://leetcode-cn.com/problems/combinations/)

题目：

给定两个整数 `n` 和 `k`，返回范围 `[1, n]` 中所有可能的 `k` 个数的组合。

示例：

```
输入：n = 4, k = 2
输出：
[
  [2,4],
  [3,4],
  [2,3],
  [1,2],
  [1,3],
  [1,4],
]
```

> 题目要求返回范围 `[1, n]` 中所有可能的 `k` 个数的组合。带入模版，则nextStepValue的值为`[1, n]` ，currentStep将从0枚举至k。设计DFS函数的参数为当前状态和当前的步骤。
> 
1. 如果当前状态满足递归终止条件（包含K个数的组合），则保存答案。
2. 枚举当前步骤时，范围1到n的值作为nextStepValue，此时可以根据当前的步骤cur.length来进行剪枝，避免选择到重复的数字
3. 用当前状态和下一步的值合成新的状态递归搜索，再回溯撤销修改
4. 最后调用DFS，初始状态为[]，初始步骤为1，最终返回res

```jsx
var combine = function (n, k) {
  let res = [];

  function dfs(cur, index) {
    if (cur.length === k) {
      res.push(cur.slice());
      return;
    }
    for (let i = index; i <= cur.length + 1 + n - k; i++) {
      cur.push(i);
      dfs(cur, i + 1);
      cur.pop();
    }
  }
  dfs([], 1);
  return res;
};
```

## [Leetcode 46](https://leetcode-cn.com/problems/permutations/)

题目：给定一个不含重复数字的数组 `nums` ，返回其 **所有可能的全排列**

示例：

```
输入：nums = [1,2,3]
输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

> 题目要求返回`nums`的所有全排列，则nextStepValue的枚举值则为`nums`，currentStep则从0到`nums.length` 。由于生成全排列要求使用到的nextStepValue都用到且只用到一次，需要一个`visited`数组标记是否使用过。
> 
1. 如果当前的长度等于合法排列的长度，则保存答案
2. 枚举当前步骤时，可以使用的nextStepValue，如果被使用过了则跳过
3. 用当前状态和下一步的值合成新的状态，同时标记nextStepValue的下标被使用过，再用新的状态递归搜索。
4. 回溯撤销新的状态，再将nextStepValue的标记置回false
5. 调用DFS函数，初始状态为[]，初始步骤为0，返回res

```jsx
var permute = function (letters) {
  let res = [];
  let visited = Array(letters.length).fill(false);

  function dfs(index, cur) {
    if (cur.length === letters.length) {
      res.push(cur.slice());
      return;
    }
    for (let i = 0; i < letters.length; i++) {
      if (visited[i]) {
        continue;
      }
      cur.push(letters[i]);
      visited[i] = true;
      dfs(i, cur);
      cur.pop();
      visited[i] = false;
    }
  }

  dfs(0, []);
  return res;
};
```

## [leetcode 784](https://leetcode-cn.com/problems/letter-case-permutation/)

题目：给定一个字符串`S`，通过将字符串`S`中的每个字母转变大小写，我们可以获得一个新的字符串。返回所有可能得到的字符串集合。

示例：

```
输入：S = "a1b2"
输出：["a1b2", "a1B2", "A1b2", "A1B2"]
```

> 题目中的nextStepValue需要分情况判断，如果是字母，则为char.lowerCase和char.upperCase，如果是数字，则只有char。题中的步骤则是从0到S.length
> 
1. 如果当前状态的长度等于S的长度，则保存进res
2. 枚举当前步骤时可选的nextStepValue，如果是字母，则DFS当前状态+char.upperCase和DFS当前状态+char.lowerCase；如果是数字，则DFS
3. 当前状态+char。不需要回溯。
4. 调用DFS，初始状态’’，初始步骤0，最后返回res；

```jsx
var letterCasePermutation = function (S) {
  let res = [];

  function isAlpha(ch) {
    return /[a-zA-Z]/i.test(ch);
  }

  function dfs(cur ,index) {
    if(cur.length === S.length ) {
      res.push(cur);
      return;
    }
    for(let i =index;i<S.length;i++) {
      if(isAlpha(S[i])) {
        dfs(cur+S[i].toLowerCase(), i+1);
        dfs(cur+S[i].toUpperCase(), i+1);
      } else {
        dfs(cur+S[i], i+1);
      }
    }
  }

  dfs('', 0);
  return res;
};
```

## [LeetCode 22](https://leetcode-cn.com/problems/generate-parentheses/)

题目：

数字 `n` 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 **有效的** 括号组合

示例：

```
输入：n = 3
输出：["((()))","(()())","(())()","()(())","()()()"]
```

> 题目中的nextStepValue为`(`或者`)` ，枚举的steps需要两个：left和right，可以理解成当前用了多少个左括号和右括号，都是从0到n，剪枝的条件是right>left，即右括号多于左括号时非法。
> 
1. 递归的非法结束状态是right大于left，需要剪枝直接返回；
2. 递归的合法结束条件是当前状态的长度等于2*n，或者leftStep和rightStep都等于n，保存当前状态。
3. 如果左括号的使用数小于n，可以一直当前状态+`(` ，left递增，right保持不变
4. 如果右括号的使用数小于n，再当前状态一直+`)` ，right递增，left保持不变
5. 调用DFS，初始状态’’，初始步骤left为0，right也为0，返回res；

## [LeetCode 17](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/)

题目：

给定一个仅包含数字 `2-9` 的字符串，返回所有它能表示的字母组合

数字到字符串的映射如下

```json
digitMap = {
    2: "abc",
    3: "def",
    4: "ghi",
    5: "jkl",
    6: "mno",
    7: "pqrs",
    8: "tuv",
    9: "wxyz",
  };
```

示例：

```
输入：digits = "23"
输出：["ad","ae","af","bd","be","bf","cd","ce","cf"]
```

> 题目的`currentStep`的枚举值从0到`digits.length`，对应的`nextStepValue`则为 `digitMap[digits[index]]` ，即当`index`为0时，`digits[index]`为`2`， `digitMap[digits[index]]` 为`’abc’` 。需要加个特殊判断`digits`为空时返回空数组
> 
1. 合法的递归结束条件为当前状态的长度等于digits的长度（或currentStep的值等于digits长度），保存当前状态到res
2. 在当前步骤的情况下，枚举可选的nextStepValue，组合当前状态和nextValue进行递归搜索
3. 调用DFS，初始步骤0，初始状态’’，最后返回res