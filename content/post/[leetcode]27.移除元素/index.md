---
title: "[leetcode]27.移除元素"
date: 2021-04-02T21:07:00+08:00
draft: true
image: leetcode.png
slug: "remove_element"
tags:
    - leetcode
categories: 
    - 算法
--- 

## 题目

给你一个数组 nums 和一个值 val，你需要 `原地` 移除所有数值等于 val 的元素，并返回移除后数组的新长度。

不要使用额外的数组空间，你必须仅使用 O(1) 额外空间并 原地 修改输入数组。

元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。

## 分析

对于数组而言，我们原地移除元素的话就肯定要把被移除的元素后面的全部元素都往前挪一个位，这是最基本的操作。所以最普通的暴力解法就是删除一个，整体挪动一个位。优化一点的话就是当有几个需要删除的数连在一起时，我们找到边界后一起挪动n个位，减少整体挪动的次数。

最优的是经常被用到的双指针法，下面再解释。

## 代码

### 混合暴力法

```java
public int removeElement(int[] nums, int val) {
    int len = nums.length;
    for (int left = 0; left < len; left++) {
        if (nums[left] == val) {
            // left位于最后一个元素时
            if (left == len - 1) {
                return len - 1;
            }
            int right = left + 1;
            while (right <= len - 1 && nums[right] == val) right++;
            if (right == len - 1 && nums[right] == val) {
                // right超过长度时
                return len - (right - left);
            }
            //  把元素right处整体前移到left
            moveUp(left, right, nums, len);
            len = len - (right - left);
        }
    }
    return len;
}

// 该函数用于将right及后面的元素完前移到left位置
public void moveUp(int left, int right, int[] nums, int len) {
    int step = len - 1 - right;
    for (int i = 0; i <= step; i++) {
        nums[left + i] = nums[right + i];
    }
}
```

![27%20%E7%A7%BB%E9%99%A4%E5%85%83%E7%B4%A0%205a59d90099fd47aa96428284ab561018/Untitled.png](27%20%E7%A7%BB%E9%99%A4%E5%85%83%E7%B4%A0%205a59d90099fd47aa96428284ab561018/Untitled.png)

此方法其实是初步优化的暴力法，假设我们要移除值为2的元素，left指针找到第一被删除的元素，right指向left后第一个不等于2的元素，然后把从right开始的后面全部元素都往前挪至left位置。

这种方法在被删除元素都是在连续位置时可以很好提高速度。但是会有一些边界条件需要考虑，在上面代码中用注释强调了。

### 双指针法

```java
public int removeElement(int[] nums, int val) {
  int slowIndex = 0;
  for (int fastIndex = 0; fastIndex < nums.length; fastIndex++) {
      if (nums[fastIndex] != val) {
          nums[slowIndex++] = nums[fastIndex];
      }
  }
  return slowIndex;
}
```

双指针法用一个快指针和一个慢指针，都是从0开始。

当快指针遇到要被删除的元素时，让慢指针不动，自己继续往后走，直到遇到一个非被删元素。

然后就把快指针所处位置的值覆盖慢指针所处位置的值，然后两个指针一起向后走。

最终慢指针的位置就是去掉所有要删除的元素的数组的末端，当快指针移到原数组最后一个元素时结束整个流程。

其实慢指针就是用来一点点构建最终数据的指针，快指针就是用来找那些存在于最终数组中的元素，慢指针停留在要被删除的元素的位置时，就把快指针位置的元素搬过去覆盖，本质就是把最终会存在于数组中的（没有被删的）元素移到一起连成数组。

### **复杂度分析**

- 时间复杂度：O(n)，假设数组总共有 n 个元素，i 和 j 至少遍历 2n 步。
- 空间复杂度：O(1)。