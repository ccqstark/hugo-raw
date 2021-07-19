---
title: "贪心算法——会场安排问题"
date: 2020-10-31T09:45:00+08:00
draft: true
slug: "greedy_activity"
tags:
    - 贪心算法
categories:
    - 算法
---

### 题目

假设要在足够多的会场里安排一批活动，并希望使用尽可能少的会场。设计一个有效的 贪心算法进行安排。（这个问题实际上是著名的图着色问题。若将每一个活动作为图的一个 顶点，不相容活动间用边相连。使相邻顶点着有不同颜色的最小着色数，相应于要找的最小 会场数。）

### 输入格式:

第一行有 1 个正整数k，表示有 k个待安排的活动。 接下来的 k行中，每行有 2个正整数，分别表示 k个待安排的活动开始时间和结束时间。时间 以 0 点开始的分钟计。

### 输出格式:

输出最少会场数。

### 输入样例:

```in
5
1 23
12 28
25 35
27 80
36 50 
```

### 输出样例:

在这里给出相应的输出。例如：

```out
3
```



### 思路

首先这道题就很像书中那道在一个会场中安排尽可能多的活动，但是，不能完全按之前那个思路来做！

这里是要用尽可能少的会场，而且从题中可以看出会场的结束时间没有限制，只要活动的开始时间比上一场要晚就行。如果我们按书中的办法把活动先按结束时间从小到大排序，然后对当前未安排的活动用一个会场进行尽可能多的安排，之后若还每安排完在开辟一个新的会场继续之前的操作。这样的算法是有问题的，因为这样的在一个会场中尽可能多的安排活动，而从全局来看（还有这道题的特点：会场结束时间无限制），这种策略并不能保证把所有活动安排在最少的会场，所以两个问题并不能完全等同，这就是我一开始犯的错误。

其实原因就在于：会场结束时间无限制，要用最少的会场。

**正确解法**有2种：

1. 把活动**按开始时间**从小到大排，**当开始时间相同则结束时间早的优先**。遍历活动再用之前的那种在一个会场安排尽可能多的活动，完了之后开辟一个新的会场继续安排，直到全部活动安排完毕。
2. 把活动**按结束时间**从小到大排，**当结束时间相同则开始时间早的优先**。遍历活动，每次再遍历一次所有会场看结束时间是否满足可以安排下，都不能安排下就新开一个会场，**然后每次还要对所有已经开辟的会场按结束时间进行从大到小再次排序**，这样直到所有活动遍历安排完毕。

所以这道题是要把有限的活动尽量塞在最少的会场中，要从所有会场全局去考虑，而且这道题的特点是单个会场的结束时间没有限制，所以第一种解法是按开始时间排的而不用按结束时间。第二种按结束时间的话就需要每次重新遍历所有会场，每次还重排，保证从全局去考虑。



### 代码

#### 解法1：

```c++
// 按开始时间排序
#include <iostream>
#include <algorithm>
using namespace std;
#define MAX 666

struct activity
{
    int start;
    int end;
    int arrage;
} activities[MAX];
int n;

bool struct_compare(activity a, activity b)
{
    if (a.start != b.start)
        return a.start < b.start; //优先进行最先开始的活动
    else
        return a.end < b.end; //当开始时间相同时,优先进行最早结束的活动
}

int main()
{
    cin >> n;

    for (int i = 0; i < n; i++)
    {
        cin >> activities[i].start;
        cin >> activities[i].end;
        activities[i].arrage = 0;
    }
    // 初始化会场数
    int room = 0;

    // 对活动'开始时间'进行从小到大排序
    sort(activities, activities + n, struct_compare);

    // 记录当前安排会场的结束时间，被安排的会场数
    int lastest = 0, arrage_num = 0;

    // 如果会场还没有被全部安排完
    while (arrage_num != n)
    {
        // 新增一个会场
        room++;
        for (int i = 0; i < n; i++)
        {
            // 在一个会场中安排尽可能多的活动
            if (activities[i].arrage == 0 && activities[i].start >= lastest)
            {
                activities[i].arrage = 1; // 标记活动为已被安排
                arrage_num++;
                lastest = activities[i].end; // 更新当前会场的结束时间
            }
        }
        lastest = 0;
    }

    //另外一种写法，效果一样
    /*
    for (int i = 0; i < n; i++)
    {
        if (activities[i].arrage == 0)
        {
            room++;
            activities[i].arrage = 1;
            lastest = activities[i].end;
            for (int j = i + 1; j < n; j++)
            {
                if (activities[j].arrage == 0 && activities[j].start >= lastest)
                {
                    activities[j].arrage = 1;
                    lastest = activities[j].end;
                }
            }
        }
    }
    */

    cout << room;
    return 0;
}
```



#### 解法2：

```c++
//按结束时间排序
#include <iostream>
#include <algorithm>
using namespace std;
#define MAX 666

struct activity
{
    int start;
    int end;
} activities[MAX];
int n;
int end_time[MAX] = {0}; // 记录每个会场的结束时间
int room = 1;            // 会场数

bool struct_compare(activity a, activity b)
{
    if (a.end != b.end)
        return a.end < b.end; //优先进行最早结束的活动

    else
        return a.start < b.start; //当结束时间相同时,优先进行最早开始的活动
}

bool dcmp(int a, int b)
{
    return a > b; // 从大到小排序
}

int main()
{
    cin >> n;

    for (int i = 0; i < n; i++)
    {
        cin >> activities[i].start;
        cin >> activities[i].end;
    }

    // 对活动按'结束时间'进行排序
    sort(activities, activities + n, struct_compare);

    // 遍历活动
    for (int i = 0; i < n; i++)
    {
        int flag = 0;                   // 标记是否在已开辟的会场中被安排
        for (int j = 1; j <= room; j++) // 遍历已有会场寻找合适的
        {
            if (activities[i].start >= end_time[j])
            {
                end_time[j] = activities[i].end;
                flag = 1;
                break;
            }
        }

        // 已有会场找不到合适的就开辟一个新的
        if (!flag)
        {
            room++;
            end_time[room] = activities[i].end;
        }

        // 每次安排完一个活动都要对会场们按结束时间从大到小重排
        sort(end_time + 1, end_time + room + 1, dcmp);
    }

    cout << room;
    return 0;
}
```



[参考博文](https://blog.nowcoder.net/n/513cf3005bcf4093a738e722f6c463e3)