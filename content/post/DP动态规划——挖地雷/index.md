---
title: "DP动态规划——挖地雷"
date: 2020-10-22T01:05:00+08:00
draft: true
slug: "dp_digmines"
tags:
    - 动态规划
categories:
    - 算法
---

### 题目

在一个地图上有n个地窖（n≤200）,每个地窖中埋有一定数量的地雷。同时，给出地窖之间的连接路径，并规定路径都是单向的,且保证都是小序号地窖指向大序号地窖，也不存在可以从一个地窖出发经过若干地窖后又回到原来地窖的路径。某人可以从任意一处开始挖地雷，然后沿着指出的连接往下挖（仅能选择一条路径），当无连接时挖地雷工作结束。设计一个挖地雷的方案，使他能挖到最多的地雷。

#### 输入格式:

第一行：地窖的个数；

第二行：为依次每个地窖地雷的个数；

下面若干行：

xi yi //表示从xi可到yi，xi<yi。

最后一行为"0 0"表示结束。

#### 输出格式:

k1-k2−…−kv //挖地雷的顺序 挖到最多的雷。

#### 输入样例:

```in
6
5 10 20 5 4 5
1 2
1 4
2 4
3 4
4 5
4 6
5 6
0 0
```

#### 输出样例:

```out
3-4-5-6
34
```



### 代码

```c++
#include <iostream>
using namespace std;
#define MAX 203
int matrix[MAX][MAX];  // 存放通路情况
int mines[MAX];        // 存放各坑地雷数
int dp_mat[MAX][MAX];  // 存放子问题最优解
int path[MAX];         // 存放路径
int n, ans, last_update;    // last_update是最后一个更新最大值的点

void dig()
{
    // 一行行扫
    for (int i = 1; i <= n; i++)
    {
        // max_last是此点之前所有点可以挖到的最大地雷数
        int max_last = 0;
        for (int k = 1; k <= i - 1; k++)
        {
            // 判断之前所有可以通向现在的点中，可以挖到最大的地雷数的路径的最后一点
            if (matrix[k][i] == 1)
            {
                // 按列方向扫，可以通向本点的点
                if (dp_mat[k][i] > max_last)
                {
                    max_last = dp_mat[k][i];
                    path[i] = k;  // 路径是所连接的上一点
                }
            }
        }

        for (int j = i; j <= n; j++)
        {
            // max_last + 本点地雷数 = 以本点作为路径末点可以挖到的最大地雷数
            dp_mat[i][j] = max_last + mines[i];

            if (dp_mat[i][j] > ans)
            {
                // 更新最终答案的最大地雷数
                ans = dp_mat[i][j];
                // 记录最后更新最终答案的那个点，作为答案路径的末尾点，用数组回溯可以打印出完整路径
                last_update = i;
            }
        }
    }
}

// 递归回溯打印完整路径
void print_path(int point)
{

    if (point == 0)
        return;
    print_path(path[point]);

    if (point == last_update)
    {
        cout << point << endl;
    }
    else
    {
        cout << point << "-";
    }
}

int main()
{
    cin >> n;
    for (int i = 1; i <= n; i++)
    {
        cin >> mines[i];
    }

    int a, b;
    while (cin >> a >> b)
    {
        if (a == 0 && b == 0)
            break;
        matrix[a][b] = 1;
    }

    dig();

    print_path(last_update);

    cout << ans;
}
```

