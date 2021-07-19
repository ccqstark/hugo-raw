---
title: "[leetcode]1.两数之和"
date: 2021-03-31T21:07:00+08:00
draft: true
image: leetcode.png
slug: "two_sum"
tags:
    - leetcode
categories: 
    - 算法
--- 

## 题目

给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 的那 两个 整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。

你可以按任意顺序返回答案。

```java
示例 1:
输入: nums = [2,7,11,15], target = 9
输出: [0,1]
解释: 因为 nums[0] + nums[1] == 9, 返回 [0, 1].

示例 2: 
输入: nums = [3,2,4], target = 6
输出: [1,2]

示例 3: 
输入: nums = [3,3], target = 6
输出: [0,1]
```

提示：

- 2 <= nums.length <= 103
- -109 <= nums[i] <= 109
- -109 <= target <= 109
- 只会存在一个有效答案

## 分析

这道题比较容易，就是在数组中找到数值`x`和`target-x`

直接上代码

## 代码

### 方法一：暴力枚举

```java
public int[] twoSum(int[] nums, int target) {
    for (int i = 0; i < nums.length - 1; i++) {
        for (int j = i + 1; j < nums.length; j++) {
            if (nums[i] + nums[j] == target) {
                return new int[]{i, j};
            }
        }
    }
    return null;
}
```

这里我们只需关注一个点就是`j = i + 1` ，我们是选中其中一个数作为x，再去找target-x的，所谓为了避免重复组合，我们从x的下一个开始找，保证每个组合只试一遍，同时避免了x和找自己。

### 复杂度分析

- 时间复杂度：O(N^2)，其中N是数组中的元素数量。最坏情况下数组中任意两个数都要被匹配一次。
- 空间复杂度：O(1)。

### 方法二：哈希表

```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> hashtable = new HashMap<Integer, Integer>();
    for (int i = 0; i < nums.length; ++i) {
        if (hashtable.containsKey(target - nums[i])) {
            return new int[] {
                    hashtable.get(target - nums[i]), i
            } ;
        }
        hashtable.put(nums[i], i);
    }
    return new int[0];
}
```

利用哈希表我们可以提高找target-x的速度，从头到尾拿出一个数x，查看target-x是否存在于哈希表中，如果不存在就把自己也加入哈希表（x作为key，下标作为value）。这样就可以快速找到我们要的组合。

### 复杂度分析

- 时间复杂度：O(N)，其中N是数组中的元素数量。对于每一个元素 x，我们可以 O(1)地寻找target - x。
- 空间复杂度：O(N)，其中 N 是数组中的元素数量。主要为哈希表的开销。