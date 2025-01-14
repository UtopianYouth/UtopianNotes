# Leetcode 刷题记录

## 第三章 二分查找基础

### 3.1 正常二分

[LC 704](https://leetcode.cn/problems/binary-search/description/)：简单二分查找。Tips：二分查找虽然简单，但是代码也很容易写错，导致边界值处理不了，从搜索区间（闭区间）的角度思考，代码就不会写错了。

### 3.2 二分查找的变式

变式主要是针对 begin、mid、end如何赋值。

[LC 69](https://leetcode.cn/problems/sqrtx/description/)：求一个正整数X的平方根，属于二分查找的变种题目。

### 3.3 双指针（对撞指针）

[LC 27](https://leetcode.cn/problems/remove-element/description/)：移除元素。充分利用题目条件（元素顺序可以打乱），采用最后一个元素覆盖法可以在较低的时间复杂度内完成。

[LC 11](https://leetcode.cn/problems/container-with-most-water/description/)：盛最多水的容器。方法一：利用组合的方法逐一例举出所有可能出现的情况（需要排除一些不可能的情况，不然会超时）；方法二：利用双指针法，高较小的一端指针移动（只有这样才有可能找到更大的面积组合）。

### 3.4 滑动窗口（通过快慢指针实现）

[LC 209](https://leetcode.cn/problems/minimum-size-subarray-sum/description/)：长度最小的子数组。通过快慢指针的移动，找到一个求和目标值最小区间。

### 3.5 巧用数组下标法

> 什么情况下可以用数组下标法：
>
> - 数值范围比较小
> - 元素是 int，并且 >= 0（或者是一些变种）

[LCR 120](https://leetcode.cn/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/description/)：寻找文件副本。

### 3.6 递归入门与优化

#### 3.6.1 递归三部曲

> 第一要素：明确这个函数想要干嘛
>
> ```c
> // 计算 n!
> int f(int n){
>   
> }
> ```
>
> 第二要素：寻找递归结束条件（可能会遗漏情况）
>
> ```c
> int f(int n){
>   if(n <= 1){
>     return 1;
>   }
> }
> ```
>
> 第三要素：找出函数的等价关系式（非常难），进行递归转换
>
> `f(n) = n * f(n-1)`

#### 3.6.2 字节真题：仅用递归反转栈（对理解递归非常重要）

给你一个栈，需要你把里面的元素进行反转，比如栈一开始是 `1, 2, 3, 4, 5` ，反转后元素顺序变为 `5, 4, 3, 2, 1`。要求：不能使用数组、队列或栈等数据结构，不过可以使用递归。

```cpp
#include<bits/stdc++.h>
using namespace std;


stack<int> s;
void top_to_bottom(stack<int>& s, int top);
int bottom_to_top(stack<int>& s);

// 方法 1
void reverse_stack1(stack<int>& s) {
 // 结束条件
 if (s.size() == 1 || s.empty()) {
     return;
 }

 // 递推式减小问题规模 
 int top = s.top();
 s.pop();
 reverse_stack1(s);

 // 功能（干什么）
 top_to_bottom(s, top);

 return;
}

// 将元素 top 放到栈底
void top_to_bottom(stack<int>& s, int top) {
 if (s.empty()) {
     s.push(top);
     return;
 }
 int tmp = s.top();
 s.pop();
 top_to_bottom(s, top);    // 减小问题规模
 s.push(tmp);
}

// 方法 2
void reverse_stack2(stack<int>& s) {
 // 结束条件
 if (s.size() == 1 || s.empty()) {
     return;
 }

 // 递推式减小问题规模 
 int tmp = bottom_to_top(s);     // 拿到栈底元素，减小问题规模
 reverse_stack2(s);

 // 功能（干什么）
 s.push(tmp);
}

// 将栈底元素放到栈顶
int bottom_to_top(stack<int>& s) {
 if (s.size() == 1) {
     int bottom = s.top();
     s.pop();
     return bottom;
 }

 int top = s.top();
 s.pop();
 int tmp = bottom_to_top(s);
 s.push(top);
 return tmp;
}


int main(int argc, char* argv[]) {
 for (int i = 0; i < 5; ++i) {
     s.push(i);
 }

 reverse_stack2(s);
 reverse_stack1(s);

 for (int i = 0; i < 5; ++i) {
     cout << s.top() << " ";
     s.pop();
 }
 cout << endl;

 getchar();
 return 0;
}
```

### 3.7 递归的优化

> - **状态保存**：使用数据结构为辅助，保存每一次递归的状态，避免重复计算。
>
> ```cpp
> 
> ```
>
> - **自底向上**：画出自顶向下递归的状态，如果底部的状态能很直观的得出，那么可以将自顶向下的问题转换为自底向上的问题（递归转循环），属于动态规划入门的一部分。
>
> ```cpp
> 
> ```

## 第四章 理解链表

### 4.1 关于链表解题的一些注意事项

> - 解题思路简单，但是容易出错
>
> - 一定写出正确的代码
> - 规定自己的习惯
>   - 是否弄虚拟头节点
> - 头节点是否有被删除的可能，next 节点是否可能为空等

### 4.2 一些题目

[LC 203](https://leetcode.cn/problems/remove-linked-list-elements/description/)：移除链表元素

> 方法 1：制造虚拟头节点，使用双指针法（不需要考虑特殊情况）
>
> 方法 2：不制造虚拟头节点，但是需要列出三种情况
>
> - 头节点为空，直接返回
> - 头节点需要删除
> - 头节点不需要删除
>
> 方法 3：使用递归，和方法 1 类似

[LC 876](https://leetcode.cn/problems/middle-of-the-linked-list/description/)：找到链表的中间节点

> 方法1：直接使用快慢指针即可，快指针的遍历速度是慢指针的两倍
>
> - 注意特殊情况，空链表和节点数为 1 的链表即可

[LCR 140](https://leetcode.cn/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/description/) : 训练计划2，返回链表倒数第 k 个节点的子链表

> 方法1：直接使用快慢指针即可，`low == fast == head`。
>
> - 快指针先走 k 步
> - 边界条件 `fast == nullptr` 的时候，`low` 指针就找到了倒数第 k 个节点

[LC 19](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/description/) : 删除链表的倒数第 N 个节点

单向链表中，需要删除一个节点，一定是需要找到其前驱节点的，所以，特殊情况的分类是，判断是否每个节点都有前驱节点。

> 方法1：快慢指针法，和 LC 140 一样，找到需要删除节点的前驱节点。
>
> - 如果删除的是第一个节点（倒数最后一个节点），是没有前驱的，直接删除即可
> - 如果删除的是其它节点，可以找到前驱节点，改变前驱节点的指向即可
>
> 注意：当快指针先走 N 步之后，`fast == nullptr` 时，说明删除的是第一个节点。
>
> 方法2（推荐）：设置虚拟头指针，然后使用快慢指针法，这个时候所有节点都有前驱节点，不需要考虑特殊情况。

[LCR 123](https://leetcode.cn/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/description/)：图书整理Ⅰ

题目要求是反向打印链表，直接打印即可。

