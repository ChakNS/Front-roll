### 28 - 实现 strStr()

#### 问题描述

[实现 strStr()](https://leetcode-cn.com/problems/implement-strstr/)

```textile
实现 strStr() 函数。

给你两个字符串 haystack 和 needle ，请你在 haystack 字符串中找出 needle 字符串出现的第一个位置（下标从 0 开始）。如果不存在，则返回  -1 。

 

说明：

当 needle 是空字符串时，我们应当返回什么值呢？这是一个在面试中很好的问题。

对于本题而言，当 needle 是空字符串时我们应当返回 0 。这与 C 语言的 strstr() 以及 Java 的 indexOf() 定义相符。
```

#### 思路

- 暴力解法
- KMP算法
  - 遍历模式串获取next队列，即最长公共前后缀的前缀下标
  - 双指针遍历主串，相同则一起往前，坏字符则拿当前索引查找next，找到最长公共前缀下标，赋值给模式串指针，继续比对

- 复杂度分析
  
  - 时间复杂度O(m + n)
  - 空间复杂度O(m)

```js
/**
 * @param {string} haystack
 * @param {string} needle
 * @return {number}
 */
var strStr = function(haystack, needle) {
    const n = haystack.length, m = needle.length;
    if (m === 0) {
        return 0
    }

    const next = new Array(m).fill(0)

    // next
    for (let i = 1, j = 0; i < m; i++) {
        while (j > 0 && needle[i] !== needle[j]) {
            j = next[j - 1]
        }

        if (needle[i] === needle[j]) {
            j++
        }

        next[i] = j
    }

    // 匹配
    for (let i = 0, j = 0; i < n; i++) {
        while (j > 0 && haystack[i] !== needle[j]) {
            j = next[j - 1]
        }
        
        if (haystack[i] === needle[j]) {
            j++
        }

        if (j === m) {
            return i - m + 1
        }
    }

    return -1
};
```
