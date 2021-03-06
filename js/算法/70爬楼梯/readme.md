# 爬楼梯

这道题最初是在一个孙红雷演的电影《少年班》里面看到的，当时没搞懂怎么解决，但有了一个印象，后来再遇到的时候就感觉很有趣，值得研究一下。

假设你正在爬楼梯。需要 n  阶你才能到达楼顶。

每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？

注意：给定 n 是一个正整数。

示例 1：

```js
输入： 2
输出： 2
解释： 有两种方法可以爬到楼顶。
1.  1 阶 + 1 阶
2.  2 阶
```

示例 2：

```js
输入： 3
输出： 3
解释： 有三种方法可以爬到楼顶。
1.  1 阶 + 1 阶 + 1 阶
2.  1 阶 + 2 阶
3.  2 阶 + 1 阶
```

来源：[LeetCode](https://leetcode-cn.com/problems/climbing-stairs)

## 思路

这道题的难度是 Easy，但是我第一次见到的时候并没有想到办法，后来看到别人的解法才感到恍然大悟，我们一步步来。

说下这道题的思路：上 第 N 阶台阶的话，最后一步只能是迈 1 阶或者 2 阶这两种情况，所以到 N 阶台阶的所有方法等于到 N-1 阶和 到 N-2 阶的所有方法的和。

转化成公式就是：`f(N)=f(N-1)+f(N-2)`，`f(N)`的意思是到 N 阶台阶的所有方法数

已知：`f(1)=1`和`f(2)=2`，上代码

```js
var climbStairs = function (n) {
  if (n === 1) {
    return 1;
  }
  if (n === 2) {
    return 2;
  }
  return climbStairs(n - 1) + climbStairs(n - 2);
};
```

测试下，结果在算 45 阶的时候超时了，下一步需要考虑怎么优化

## 优化

我们可以看到，按照上面的思路是没有问题的，只是运算超时了，那么就需要考虑下怎么加快运算，我们可以尝试画出求解的整个过程，来寻找优化的思路。

![分解45](https://i.loli.net/2020/08/17/h3zWpLJwc8NQ9jY.png)

通过上图我们可以看到，`f(45) = f(43)+f(44)` ,然后可以继续分解：

```js
f(45) = f(43) + f(44);
f(45) = f(41) + f(42) + f(42) + f(43);
// .....
// .....
// .....
// 最后分解到了已知值，即：
f(1) = 1;
f(2) = 2;
```

在整个分解的过程中，我们会发现其实有一些重复的值，比如上面的`f(42)`求解了两次，而且`f(42)`和`f(43)`会继续分解出重复的值。

那么我们可以想办法不去求解重复的值，利用已经求解完毕的值做一个备份，发现已经求解的直接拿来用，上代码。

```js
let dp = new Map();
dp.set(1, 1).set(2, 2);

var climbStairs = function (n) {
  if (dp.has(n)) {
    return dp.get(n);
  }
  let result = climbStairs(n - 1) + climbStairs(n - 2);
  dp.set(n, result);
  return result;
};
```

提交后的结果：

- 45/45 cases passed (92 ms)
- Your runtime beats 9.54 % of javascript submissions
- Your memory usage beats 42.58 % of javascript submissions (37.5 MB)

还可以，终于不是超时了，但是结果来看，我们的排名并不靠前，那么还有继续优化的空间了吗？

## 继续优化

我们发现，求解完毕一个 `f(45)` 之后，dp 里面记录了从 `f(1)`到 `f(45)`的所有的值，实际上我们只想要`f(45)`的值。

我们换个思路，求`f(45)`的过程，分解为先求`f(3)`，然后再求`f(4)`......一直到`f(45)`，上代码：

```js
var climbStairs = function (n) {
  if (n === 1 || n === 2) {
    return n;
  }
  // 前一个值
  let pre = 2;
  // 前一个的前一个的值
  let beforePre = 1;
  // 中间变量
  let temp = null;
  for (let index = 3; index <= n; index++) {
    temp = pre;
    pre = pre + beforePre;
    beforePre = temp;
  }
  return pre;
};
```

提交下看看结果：

- 45/45 cases passed (76 ms)
- Your runtime beats 42.21 % of javascript submissions
- Your memory usage beats 79.72 % of javascript submissions (37.4 MB)

感觉还不错，上面的方法只用了 3 个变量，就从`f(3)`一直算到了 f`(45)`，时间复杂度和空间复杂度都很低
