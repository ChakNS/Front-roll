### 496 - 下一个更大元素 I

#### 问题描述

[下一个更大元素 I](https://leetcode-cn.com/problems/next-greater-element-i/)

```textile
nums1 中数字 x 的 下一个更大元素 是指 x 在 nums2 中对应位置 右侧 的 第一个 比 x 大的元素。

给你两个 没有重复元素 的数组 nums1 和 nums2 ，下标从 0 开始计数，其中nums1 是 nums2 的子集。

对于每个 0 <= i < nums1.length 
找出满足 nums1[i] == nums2[j] 的下标 j 
并且在 nums2 确定 nums2[j] 的 下一个更大元素 
如果不存在下一个更大元素，那么本次查询的答案是 -1

返回一个长度为 nums1.length 的数组 ans 作为答案，满足 ans[i] 是如上所述的 下一个更大元素 。
```

#### 思路
- 单调栈及哈希表
- 先从尾部开始遍历nums2，并用哈希表存储每个元素对应的下一个更大元素
- 最后遍历nums1，从哈希表读出对应的结果

- 复杂度分析
  - 时间复杂度O(m + n) m为nums2个数，n为nums1个数
  - 空间复杂度O(m) 哈希表

```js
/**
 * @param {number[]} nums1
 * @param {number[]} nums2
 * @return {number[]}
 */
var nextGreaterElement = function(nums1, nums2) {
    const tempMap = new Map()
    const stack = []

    for (let i = nums2.length - 1; i >= 0; i--) {
        const num = nums2[i]
        // 如果栈顶元素比当前的元素要小，则弹出
        while (stack.length && stack[stack.length - 1] < num) stack.pop()

        tempMap.set(num, stack.length ? stack[stack.length - 1] : -1)
        stack.push(num)
    }

    return nums1.map(item => tempMap.get(item))
};
```
