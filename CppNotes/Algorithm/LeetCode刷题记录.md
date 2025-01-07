# Leetcode 刷题记录

## 第三章 二分查找基础

### 3.1 正常二分

LC 704：简单二分查找。Tips：二分查找虽然简单，但是代码也很容易写错，导致边界值处理不了，从搜索区间（闭区间）的角度思考，代码就不会写错了。

### 3.2 二分查找的变式

变式主要是针对 begin、mid、end如何赋值。

LC 69：求一个正整数X的平方根，属于二分查找的变种题目。

### 3.3 双指针（对撞指针）

LC 27：移除元素。充分利用题目条件（元素顺序可以打乱），采用最后一个元素覆盖法可以在较低的时间复杂度内完成。

LC 11：盛最多水的容器。方法一：利用组合的方法逐一例举出所有可能出现的情况（需要排除一些不可能的情况，不然会超时）；方法二：利用双指针法，高较小的一端指针移动（只有这样才有可能找到更大的面积组合）。

### 3.4 滑动窗口（通过快慢指针实现）

LC 209：长度最小的子数组。通过快慢指针的移动，找到一个求和目标值最小区间。

### 3.5 巧用数组下标法

> 什么情况下可以用数组下标法：
>
> - 数值范围比较小
> - 元素是 int，并且 >= 0（或者是一些变种）

LC 120：寻找文件副本。

### 3.6 递归入门与优化

递归三部曲

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
>
> **字节真题：仅用递归反转栈（对理解递归非常重要）**
>
> 给你一个栈，需要你把里面的元素进行反转，比如栈一开始是 `1, 2, 3, 4, 5` ，反转后元素顺序变为 `5, 4, 3, 2, 1`。要求：不能使用数组、队列或栈等数据结构，不过可以使用递归。
>
> ```c
> #include<bits/stdc++.h>
> using namespace std;
> 
> 
> stack<int> s;
> void top_to_bottom(stack<int>& s, int top);
> int bottom_to_top(stack<int>& s);
> 
> // 方法 1
> void reverse_stack1(stack<int>& s) {
>     // 结束条件
>     if (s.size() == 1 || s.empty()) {
>         return;
>     }
> 
>     // 递推式减小问题规模 
>     int top = s.top();
>     s.pop();
>     reverse_stack1(s);
> 
>     // 功能（干什么）
>     top_to_bottom(s, top);
> 
>     return;
> }
> 
> // 将元素 top 放到栈底
> void top_to_bottom(stack<int>& s, int top) {
>     if (s.empty()) {
>         s.push(top);
>         return;
>     }
>     int tmp = s.top();
>     s.pop();
>     top_to_bottom(s, top);    // 减小问题规模
>     s.push(tmp);
> }
> 
> // 方法 2
> void reverse_stack2(stack<int>& s) {
>     // 结束条件
>     if (s.size() == 1 || s.empty()) {
>         return;
>     }
> 
>     // 递推式减小问题规模 
>     int tmp = bottom_to_top(s);     // 拿到栈底元素，减小问题规模
>     reverse_stack2(s);
> 
>     // 功能（干什么）
>     s.push(tmp);
> }
> 
> // 将栈底元素放到栈顶
> int bottom_to_top(stack<int>& s) {
>     if (s.size() == 1) {
>         int bottom = s.top();
>         s.pop();
>         return bottom;
>     }
> 
>     int top = s.top();
>     s.pop();
>     int tmp = bottom_to_top(s);
>     s.push(top);
>     return tmp;
> }
> 
> 
> int main(int argc, char* argv[]) {
>     for (int i = 0; i < 5; ++i) {
>         s.push(i);
>     }
> 
>     reverse_stack2(s);
>     reverse_stack1(s);
> 
>     for (int i = 0; i < 5; ++i) {
>         cout << s.top() << " ";
>         s.pop();
>     }
>     cout << endl;
> 
>     getchar();
>     return 0;
> }
> ```

## 第四章 理解链表
