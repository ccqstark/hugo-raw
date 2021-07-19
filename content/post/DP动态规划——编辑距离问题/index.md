---
title: "DP动态规划——编辑距离问题"
date: 2020-10-31T17:03:00+08:00
draft: true
slug: "dp_edit_distance"
tags:
    - 动态规划
categories:
    - 算法
---

### 问题

设A和B是2个字符串。要用最少的字符操作将字符串A转换为字符串B。这里所说的字符操作包括 (1)删除一个字符； (2)插入一个字符； (3)将一个字符改为另一个字符。 将字符串A变换为字符串B所用的最少字符操作数称为字符串A到 B的编辑距离，记为d(A,B)。 对于给定的字符串A和字符串B，计算其编辑距离 d(A,B)。

### 输入格式:

第一行是字符串A，文件的第二行是字符串B。

提示：字符串长度不超过2000个字符。

### 输出格式:

输出编辑距离d(A,B)

### 输入样例:

在这里给出一组输入。例如：

```in
fxpimu
xwrs 
```

### 输出样例:

在这里给出相应的输出。例如：

```out
5
```



### 思路

用动态规划算法可以将问题分解出最优子结构。

设`dp[i][j]`表示把A字符串前i个字符组成的字符串转变为B字符串前j个字符组成的字符串所需的最少的字符操作数

如果A字符串的第i个字符与B字符串的第j个字符串相同，则这个位置不需要操作，所需的操作等于`dp[i-1][j-1]`，否则需要进行修改，操作数就要+1

由于每个位置都可以进行修改、删除、插入三种操作，因此需要把这三种操作中编辑距离最小的作为`dp[i][j]`的值

递推公式(代码表示)：

```c++
if (A[i - 1] == B[j - 1]) // dp矩阵以1开始，字符数组是0开始，因此对应的话要-1
	dp[i][j] = dp[i - 1][j - 1]; // 如果对应的位置相同就不用操作，否则要修改所以要+1
else
	dp[i][j] = dp[i - 1][j - 1] + 1;

//              修改              删除              插入
dp[i][j] = min(dp[i][j], min(dp[i - 1][j] + 1, dp[i][j - 1] + 1));
```

表的维度：二维

填表的范围：（len_A和len_B分别为字符串A、B的长度）

i：1 ~ len_A

j：1 ~ len_B

填表顺序：从左至右，自顶向下（i与j的递增方向）

时间复杂度：由于填写的是二维表，需要二重循环，所以时间复杂度是O(n^2)

空间复杂度：需要一个二维数组，因而是O(n^2)



### 代码

```c++
// 编辑距离问题
#include <iostream>
#include <cstring>
#include <algorithm>
using namespace std;
#define MAX 2002
char A[MAX];
char B[MAX];
int dp[MAX][MAX];

int calculate_distance()
{

    // a、b字符串的长度
    int len_a = strlen(A);
    int len_b = strlen(B);

    // 边界初始化
    for (int i = 0; i <= len_a; i++)
        dp[i][0] = i;
    for (int j = 0; j <= len_b; j++)
        dp[0][j] = j;

    for (int i = 1; i <= len_a; i++)
    {
        for (int j = 1; j <= len_b; j++)
        {
            // dp矩阵以1开始，字符数组是0开始，因此对应的话要-1
            if (A[i - 1] == B[j - 1])
                // 如果对应的位置相同就不用操作，否则要修改所以要+1
                dp[i][j] = dp[i - 1][j - 1];
            else
                dp[i][j] = dp[i - 1][j - 1] + 1;

            //              修改              删除              插入
            dp[i][j] = min(dp[i][j], min(dp[i - 1][j] + 1, dp[i][j - 1] + 1));
        }
    }

    return dp[len_a][len_b];
}

int main()
{
    cin >> A >> B;

    cout << calculate_distance();
}
```

