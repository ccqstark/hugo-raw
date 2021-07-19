---
title: "DP动态规划——单调递增最长子序列"
date: 2020-10-21T21:07:00+08:00
draft: true
slug: "dp_increasing"
tags:
    - 动态规划
categories:
    - 算法
---

### 题目

设计一个O(n2)时间的算法，找出由n个数组成的序列的最长单调递增子序列。

#### 输入格式:

输入有两行： 第一行：n，代表要输入的数列的个数 第二行：n个数，数字之间用空格格开

#### 输出格式:

最长单调递增子序列的长度

#### 输入样例:

在这里给出一组输入。例如：

```in
5
1 3 5 2 9
```

#### 输出样例:

在这里给出相应的输出。例如：

```out
4
```



### 思路

用动态规划的思想，利用子问题的最优解求更大一点的子问题。

设一个数组`dp[i]`用于存放数组中从0到i下标的序列中，最长的递增子序列的长度

双重遍历，如果arr[j]小于arr[i]，则`dp[i]`为`dp[j]+1`和`dp[i]`中较大的那个。即

`dp[i] = max{ dp[j]+1, dp[i] }`

由于每次重头又遍历了一次，并每次都分析最优解，避免了`1 2 3 9 6 7 `这样在最大数后面还有2个较小的数可以产生更长递增子序列的情况可能犯的错误。

这也说明这个算法的时间复杂度只能是O(n2)



### 代码

```c++
// 单调递增最长子序列
#include <iostream>
using namespace std;
#define MAX 666
int arr[MAX];
int dp[MAX];
int n;

int longest_increasing(){

    // 初始化第一个
    dp[0] = 1;

    // 双重遍历
    for (int i = 0;i<n;i++){
        for (int j = 0;j<i;j++){
            // 利用子问题最优解
            if(arr[i]>arr[j]){
                dp[i] = (dp[j]+1>dp[i])?dp[j]+1:dp[i];
            }
        }
    }

    // 找出dp[]中最大的那个作为答案
    int max_len = 1;
    for (int i = 0;i<n;i++){
        max_len = (dp[i]>max_len)?dp[i]:max_len;
    }

    return max_len;
}

int main()
{
    cin>>n;
    for (int i = 0;i<n;i++){
        cin>>arr[i];
    }

    cout<<longest_increasing();
}
```

