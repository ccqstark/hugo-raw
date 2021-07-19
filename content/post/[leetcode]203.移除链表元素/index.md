---
title: "[leetcode]203.移除链表元素"
date: 2021-04-11T00:00:00+08:00
draft: true
image: leetcode.png
slug: "remove_elements"
tags:
    - leetcode
categories: 
    - 算法
--- 

## 题目

给你一个链表的头节点 head 和一个整数 val ，请你删除链表中所有满足 Node.val == val 的节点，并返回 新的头节点 。

![203%20%E7%A7%BB%E9%99%A4%E9%93%BE%E8%A1%A8%E5%85%83%E7%B4%A0%203f587fa8f0f140a29b1404d3e15a151b/Untitled.png](203%20%E7%A7%BB%E9%99%A4%E9%93%BE%E8%A1%A8%E5%85%83%E7%B4%A0%203f587fa8f0f140a29b1404d3e15a151b/Untitled.png)

```java
示例 1:
输入: head = [1,2,6,3,4,5,6], val = 6
输出: [1,2,3,4,5]

示例 2:
输入: head = [], val = 1
输出: []

示例 3: 
输入: head = [7,7,7,7], val = 7
输出: []
```

## 分析

很普通的一道链表移除元素题目。这里我们主要考虑两种方式：

1. 直接在原链表上删除节点操作
2. 增加一个虚拟节点（也叫哨兵节点）来操作

第二种方法主要是为了统一所有的删除节点操作，而不用在遇到要删除头节点情况时要单独写一段代码来处理。

而对于普通节点，删除这个节点的操作一般是把自己的前驱节点的next指针指向自己的后驱节点，如图：

![203%20%E7%A7%BB%E9%99%A4%E9%93%BE%E8%A1%A8%E5%85%83%E7%B4%A0%203f587fa8f0f140a29b1404d3e15a151b/Untitled%201.png](203%20%E7%A7%BB%E9%99%A4%E9%93%BE%E8%A1%A8%E5%85%83%E7%B4%A0%203f587fa8f0f140a29b1404d3e15a151b/Untitled%201.png)

由于是单链表，我们不能获得一个被删节点的前驱节点，所以一般是判断一个节点的next的值是否为被删值。

> 对于C/C++，不能自动释放内存我们就要手动释放被删除的节点的内存，但Java会帮我们做好这一切。



## 代码

ListNode节点类的代码：

```java
public class ListNode {
  int val;
  ListNode next;
  ListNode() {}
  ListNode(int val) { this.val = val; }
  ListNode(int val, ListNode next) { this.val = val; this.next = next; }
}
```

### 方法一：直接在原链表上操作

```java
public ListNode removeElements1(ListNode head, int val) {

    ListNode ptr = head;

    while (ptr != null && ptr.next != null) {
        
        if (ptr == head && ptr.val == val) {
            // 删除头节点情况
            head = head.next;
            ptr = head;
        } else if (ptr.next.val == val) {
            ptr.next = ptr.next.next;
        } else {
            ptr = ptr.next;
        }
    }

    // 处理只有一个节点的情况
    if (head != null && head.next == null && head.val == val){
        return null;
    }

    return head;
}
```

这种方式对于要删除的点是头节点的情况就是把head节点往后移一位：

![203%20%E7%A7%BB%E9%99%A4%E9%93%BE%E8%A1%A8%E5%85%83%E7%B4%A0%203f587fa8f0f140a29b1404d3e15a151b/Untitled%202.png](203%20%E7%A7%BB%E9%99%A4%E9%93%BE%E8%A1%A8%E5%85%83%E7%B4%A0%203f587fa8f0f140a29b1404d3e15a151b/Untitled%202.png)

### 方法二：虚拟头节点

```java
public ListNode removeElements2(ListNode head, int val) {
    // 设置虚拟头节点
		ListNode virtualHead = new ListNode(0, head);
    ListNode ptr = virtualHead;

    while (ptr.next != null) {
        if (ptr.next.val == val) {
            ptr.next = ptr.next.next;
        } else {
            ptr = ptr.next;
        }
    }
    return virtualHead.next;
}
```

下面是有了虚拟头节点之后删除头节点的操作，发现和其它的非头节点操作是可以统一的：

![203%20%E7%A7%BB%E9%99%A4%E9%93%BE%E8%A1%A8%E5%85%83%E7%B4%A0%203f587fa8f0f140a29b1404d3e15a151b/Untitled%203.png](203%20%E7%A7%BB%E9%99%A4%E9%93%BE%E8%A1%A8%E5%85%83%E7%B4%A0%203f587fa8f0f140a29b1404d3e15a151b/Untitled%203.png)

### **复杂度分析**

- 时间复杂度：O(*N*)，只遍历了一次。
- 空间复杂度：O(1)。