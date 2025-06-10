# 第一章 STL 的基本使用（入门）

## 1.1 常见 STL 

```c++
#include<iostream>
#include<bits/stdc++.h>
using namespace std;

/*
    STL: 标准模板库
    1. vector and list
    2. stack and queue
    3. priority_queue and deque
    4. set and multiset and unordered_set
    5. map and multimap and unordered_map
*/

// vector
void Demo001(){
  // 创建指定大小的 vector 一维数组
  vector<int> res(10,0);
  
  // 创建指定大小的 vector 二维数组
  vector<vector<int>> res(10, vector<int>(10,0));
  
}


//pair
void Demo01() {
    vector<pair<int, int>> nums;
    nums.push_back(pair<int, int>(3, 2));
    nums.push_back(pair<int, int>(4, 1));
    nums.push_back(pair<int, int>(3, 1));

    for (auto i : nums) {
        cout << i.first << " " << i.second << endl;
    }

    //自定义排序规则对pair对象键排序
    sort(nums.begin(), nums.end(), [](pair<int, int> lhs, pair<int, int> rhs)->bool {return lhs.first > rhs.first;});
    for (auto i : nums) {
        cout << i.first << " " << i.second << endl;
    }
}

//unique的使用，unique会将重复的元素移动到vector末尾
void Demo02() {
    vector<int> v = { 1,2,2,3,3,4,5,6,6,7 };
    //vector去重，只对相邻元素有效，去重前可先排序
    for (auto i : v) {
        cout << i << " ";
    }
    cout << endl;
    //unique函数返回指向重复元素的第一个位置
    auto it = unique(v.begin(), v.end());
    for (auto i : v) {
        cout << i << " ";
    }
    cout << endl;
    cout << *it << endl;
    v.erase(it, v.end());
    for (auto i : v) {
        cout << i << " ";
    }
    cout << endl;
}

//list的使用
void Demo03() {
    list<int> l;
    l.push_back(1);
    //在1之前插入5个相同元素
    l.insert(l.begin(), 5, 2);
    //将l中的元素反转
    l.reverse();
    for (auto i : l) {
        cout << i << " ";
    }
    cout << endl;
}

//stack的使用
void Demo04() {
    stack<int> s;
    for (int i = 0;i < 5;++i) {
        s.push(i + 1);
    }
    cout << s.top() << endl;    //返回栈顶元素
    s.pop();
    cout << s.size() << endl;
}

//queue的使用
void Demo05() {
    queue<int> q;
    q.push(1);
    q.push(2);
    cout << q.front() << " " << q.back() << endl;
    q.pop();
    cout << q.front() << endl;
}

//priority_queue, 优先队列默认大根堆实现
void Demo06() {
    //自定义比较函数，小根堆实现
    auto cmp = [](int a, int b)->bool {return a > b;};
    priority_queue<int, vector<int>, decltype(cmp)> pq(cmp);
    for (int i = 0;i < 5;++i) {
        pq.push(i + 1);
    }
    cout << pq.top() << endl;   //返回优先队列顶部元素
    pq.pop();
    cout << pq.top() << endl;
}

//deque, 双端队列
void Demo07() {
    deque<int> dq;
    dq.push_front(1);
    dq.push_back(2);
    cout << dq.front() << endl;
    cout << dq.back() << endl;
    dq.pop_front();
    dq.pop_back();
    cout << dq.empty() << endl;
}

//set的应用，set底层实现是红黑树，默认是有序的
typedef struct Mycmp {
    //仿函数
    bool operator()(const int& a, const int& b) {
        return a > b;
    }
}Mycmp;
void Demo08() {
    set<int> s1;
    for (int i = 0;i < 5;++i) {
        s1.insert(i);
    }
    for (const auto& i : s1) {
        cout << i << " ";
    }
    cout << endl;

    //逆序集合
    set<int, Mycmp> s2;
    for (int i = 0;i < 5;++i) {
        s2.insert(i);
    }
    for (const auto& i : s2) {
        cout << i << " ";
    }
    cout << endl;
}

//multiset的使用，与set几乎一致，但是允许元素重复
void Demo09() {
    //仿函数自定义元素组合规则
    multiset<int, Mycmp> ms;
    for (int i = 0;i < 5;++i) {
        ms.insert(i + 1);
        ms.insert(i + 1);
    }
    for (const auto& i : ms) {
        cout << i << " ";
    }
    cout << endl;
}

//unordered_set，无序集合，哈希表实现
void Demo10() {
    vector<int> v = { 1,3,4,5,5,3,3,2,7,8,8,9 };
    unordered_set<int> us;
    for (const auto& i : v) {
        us.insert(i);
    }
    for (const auto& i : us) {
        cout << i << " ";
    }
    cout << endl;
    //存在对应的值，返回1，否则，返回0
    cout << us.count(5) << endl;
    cout << us.size() << endl;
    cout << endl;
}

//map, multimap, unordered_map的使用
void Demo11() {
    //仿函数自定义元素组合规则
    map<int, int, Mycmp> mp;
    mp.insert(pair<int, int>(3, 4));
    mp.insert(pair<int, int>(4, 4));
    mp.insert(pair<int, int>(5, 4));
    for (const auto& i : mp) {
        cout << i.first << " ";
    }
    cout << endl;

}

int main() {
    Demo11();
    system("pause");
    return 0;
}


```

## 1.2 `string`的基本使用



# 第三章 二分查找基础

## 3.1 正常二分

### [LC 704](https://leetcode.cn/problems/binary-search/description/)：简单二分查找

二分查找虽然简单，但是代码也很容易写错，导致边界值处理不了，从搜索区间（闭区间）的角度思考，代码就不会写错了。

## 3.2 二分查找的变式

变式主要是针对 begin、mid、end如何赋值。

### [LC 69](https://leetcode.cn/problems/sqrtx/description/)：求一个正整数X的平方根

属于二分查找的变种题目。

## 3.3 双指针（对撞指针）

### [LC 27](https://leetcode.cn/problems/remove-element/description/): 移除元素

充分利用题目条件（元素顺序可以打乱），采用最后一个元素覆盖法可以在较低的时间复杂度内完成

### [LC 11](https://leetcode.cn/problems/container-with-most-water/description/): 盛最多水的容器

> 方法一：利用组合的方法逐一例举出所有可能出现的情况（需要排除一些不可能的情况，不然会超时）；
>
> 方法二：利用双指针法，高较小的一端指针移动（只有这样才有可能找到更大的面积组合）。

### [LC 165](https://leetcode.cn/problems/compare-version-numbers/description/)：比较版本号

<font color =red>腾讯企业微信后端一面原题。</font>

## 3.4 滑动窗口（通过快慢指针实现）

### [LC 209](https://leetcode.cn/problems/minimum-size-subarray-sum/description/): 长度最小的子数组

通过快慢指针的移动，找到一个求和目标值最小区间。

## 3.5 巧用数组下标法

> 什么情况下可以用数组下标法: 
>
> - 数值范围比较小
> - 元素是 int，并且 >= 0（或者是一些变种）

### [LCR 120](https://leetcode.cn/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/description/): 寻找文件副本

## 3.6 递归入门与优化

### 3.6.1 递归三部曲

> 第一要素: 明确这个函数想要干嘛
>
> ```c
> // 计算 n!
> int f(int n){
>   
> }
> ```
>
> 第二要素: 寻找递归结束条件（可能会遗漏情况）
>
> ```c
> int f(int n){
>   if(n <= 1){
>     return 1;
>   }
> }
> ```
>
> 第三要素: 找出函数的等价关系式（非常难），进行递归转换
>
> `f(n) = n * f(n-1)`

### 3.6.2 字节真题: 仅用递归反转栈（对理解递归非常重要）

给你一个栈，需要你把里面的元素进行反转，比如栈一开始是 `1, 2, 3, 4, 5` ，反转后元素顺序变为 `5, 4, 3, 2, 1`。要求: 不能使用数组、队列或栈等数据结构，不过可以使用递归。

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

## 3.7 递归的优化

> - **状态保存**: 使用数据结构为辅助，保存每一次递归的状态，避免重复计算。
>
> ```cpp
> 
> ```
>
> - **自底向上**: 画出自顶向下递归的状态，如果底部的状态能很直观的得出，那么可以将自顶向下的问题转换为自底向上的问题（递归转循环），属于动态规划入门的一部分。
>
> ```cpp
> 
> ```

# 第四章 链表

## 4.1 关于链表解题的一些注意事项

> - 解题思路简单，但是容易出错；
>
> - 一定写出正确的代码；
> - 规定自己的习惯；
>   - 是否弄虚拟头节点；
> - 头节点是否有被删除的可能，next 节点是否可能为空等。

## 4.2 链表高频题目

### [LC 203](https://leetcode.cn/problems/remove-linked-list-elements/description/): 移除链表元素

方法 1: 制造虚拟头节点，使用双指针法（不需要考虑特殊情况）；

方法 2: 不制造虚拟头节点，但是需要列出三种情况: 

> - 头节点为空，直接返回；
>- 头节点需要删除；
> - 头节点不需要删除。
>

方法 3: 使用递归，和方法 1 类似。

### [LC 876](https://leetcode.cn/problems/middle-of-the-linked-list/description/): 找到链表的中间节点

方法1: 直接使用快慢指针即可，快指针的遍历速度是慢指针的两倍。

> - 注意特殊情况，空链表和节点数为 1 的链表即可。

### [LCR 140](https://leetcode.cn/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/description/) : 训练计划2，返回链表倒数第 k 个节点的子链表

方法1: 直接使用快慢指针即可，`low == fast == head`。

> - 快指针先走 k 步；
>- 边界条件 `fast == nullptr` 的时候，`low` 指针就找到了倒数第 k 个节点。

### [LC 19](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/description/) : 删除链表的倒数第 N 个节点

单向链表中，需要删除一个节点，一定是需要找到其前驱节点的，所以，特殊情况的分类是，判断是否每个节点都有前驱节点。

方法1: 快慢指针法，和 LC 140 一样，找到需要删除节点的前驱节点。

> - 如果删除的是第一个节点（倒数最后一个节点），是没有前驱的，直接删除即可
>- 如果删除的是其它节点，可以找到前驱节点，改变前驱节点的指向即可
> 
> 注意: 当快指针先走 N 步之后，`fast == nullptr` 时，说明删除的是第一个节点。
>

方法2（推荐）: 设置虚拟头指针，然后使用快慢指针法，这个时候所有节点都有前驱节点，不需要考虑特殊情况。

### [LCR 123](https://leetcode.cn/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/description/): 图书整理Ⅰ

题目要求是反向打印链表，直接打印即可。

### [LC 160](https://leetcode.cn/problems/intersection-of-two-linked-lists/description/): 相交链表

本题比较考验技巧性，在时间复杂度为`O(m + n)`，空间复杂度为`O(1)`的前提下，通过此题。

> 技巧性: 两次交叉遍历链表，可以抵消链表的长度差异，如果存在交叉节点，两个指针一定会撞上。

### [LC 206](https://leetcode.cn/problems/reverse-linked-list/): 反转链表

和 [LCR 123](https://leetcode.cn/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/description/) 一样的思路，两种解法本质是一样的，需要找到前驱节点，思路如下。

> - 解法1: 采用递归的方式，需要栈空间的辅助，空间复杂度为`O(n)`，牢记递归三部曲即可；
> - 解法2: 利用双指针法，实现链表的原地反转，空间复杂度为`O(1)`。

### [LC 92](https://leetcode.cn/problems/reverse-linked-list-ii/description/): 反转链表Ⅱ

和上一题不一样的是，这一题需要局部反转链表，但是换汤不换药，本质还是一样的。

> - 首先应该限定住需要反转的局部部分，反转完成后，需要完成拼接
>
> - 当反转的局部部分左边是第一个节点的时候，左边是不需要拼接的

### [LC 25](https://leetcode.cn/problems/reverse-nodes-in-k-group/description/): K个一组反转链表

和上一个题目不一样的是，这一个反转链表属于局部反转，但是需要反转多个局部位置。

> - 通过 K ，限定住需要进行翻转链表的位置
> - 遍历链表，每 K 步之后翻转一次
> - 链表反转的核心算法可以用递归实现
> - 如果反转是的链表的局部部分，应该考虑一下，反转之后的拼接问题

这个题目刷完之后，链表反转的问题基本就解决了，总结如下: 

> - 链表反转的本质可以用递归的思路解决（这基本是最优解，利用函数栈帧，完成链表的原地反转）；
>
> - 不仅要学会整体的链表反转，还要学会局部的链表反转；
> - 针对局部的链表反转，链表反转之后，需要考虑整体链表连接的问题。

### [LC 141](https://leetcode.cn/problems/linked-list-cycle/description/): 环形链表

利用快慢指针，如果链表有环，快指针一定会追上慢指针。

### [LC 142](https://leetcode.cn/problems/linked-list-cycle-ii/description/): 环形链表Ⅱ

和上一题本质一样，但是需要找到进入环形链表的第一个节点。

> - 判断环形链表比较容易，但是需要找到环形链表的第一个节点，需要一点技巧；
> - 第一步，计算出环形链表的节点个数 K ；
> - 第二步，设置快慢指针，快指针先走 K 步，即可找到环形链表的入口。

<font color = red>这道题主要考察思路，代码实现不难</font>。

## 4.3 链表低频题目

### [LC 21](https://leetcode.cn/problems/merge-two-sorted-lists/description/): 合并两个有序链表

本质考察归并排序，思路比较简单。

### [LC 61](https://leetcode.cn/problems/rotate-list/description/): 旋转链表

和反转链表类似，需要改变内部节点的指向，但是**比反转链表要简单一些**。

> - 方法一: 快慢指针法
>   - 快指针先走 k 步；
>
>   - 当快指针走到最后一个节点时，连上起始节点；
>
>   - 将慢指针的 next 置为 nullptr ；
>
> - 方法二: 链表成环法，推荐，较为简单
>   - 找到最后一个节点，并且连上起始节点；
>   - 起始节点往后遍历 len-k-1 个位置，找到旋转后的起始节点前一个节点；
>   - 断连即可。
>

<font color = red>注意: </font>该题目有陷阱，链表向右移动的位置 K 可能大于链表的节点个数，所以需要求出链表中节点的个数。

### [LC 148](https://leetcode.cn/problems/sort-list/description/): 排序链表

这个题目大致思路是采用归并（递归实现）的方法来做，是一个比较综合的题目，涉及了找到链表中间节点、合并两个有序链表两个操作。

> - 首先，根据归并的思想，将链表拆分成两部分（也就是找到链表的中间节点）；
>   - 找中间节点有一个细节，如果是偶数个节点，中间节点有两个，应该找第一个作为拆分；
> - 不断拆分链表，直到只剩一个节点（这个时候必然有序），然后合并有序链表即可。

<font color = red>这个题目还能很好的锻炼操作链表指针的能力。</font>

### [LC 382](https://leetcode.cn/problems/linked-list-random-node/description/): 链表随机节点

这个题目主要考察的是一种解题技巧**蓄水抽样法**，但是使用普通方法也能做出来

> **方法1**: 统计链表节点个数 n，生成 $[1, n]$ 的随机数，返回随机数对应随机数位置的节点值即可，这种方法需要遍历 2 次链表。
>
> **方法2**: 蓄水抽样法，只需要遍历一次链表，在遍历链表的过程中，不断地计算出第 $i$ 个节点返回的概率是 $1/i$ ，随着链表所有节点遍历完毕，计算出的每一个节点被选中的概率是 $1/n$ ，具体如下: 
>
> - 链表有一个节点，最后一个节点返回的概率是1；
> - 链表有两个节点，最后一个节点返回的概率是1/2，第一个节点返回的概率是 $1 * (1/2)$ ；
> - 链表有三个节点，最后一个节点返回的概率是1/3，第二个节点返回的概率是 $(1/2) * (2/3)$，第一个节点返回的概率是 $1 * (1/2) * (2/3)$；
> - 以此类推......
>
> 那么，怎么确定最后一个节点返回的概率是 $1/i$ 呢？
>
> - 通过生成的随机数 `(rand() % i + 1) == i` 判断即可，左边等概率生成 $[1,i]$ 之间的随机数。
>
> ```c++
> // 生成 0~100的随机数生成方法
> // 1. 定义随机数种子
> int local_num = 0;
> // 2. 生成随机数
> while(1){
>   srand(local_num++);
>   int num = rand()%101;
> }
> ```
>
> 

### [LC 138](https://leetcode.cn/problems/copy-list-with-random-pointer/description/): 随机链表的复制

本题属于链表复制的变体，每一个链表节点多了一个随机指针，所以，随机指针的指向也需要被复制，增大了题目的难度（因为随机指针的复制，需要提前将链表的整体）。

> 方法 1 : 常规方法，整体为 3 步，时间复杂度为 `O(n^2)`。
>
> - 遍历原链表，复制原链表的 next 指向部分；
> - 遍历原链表，判断原链表中节点的 random 指针是否为空；
>      - 不为空: 存储该节点 random 指针指向节点的位置（索引）；
>      - 为空: 不做任何操作；
> - 遍历复制的链表，根据原链表得出每一个节点 random 指针指向的位置，将复制的链表 random 指针指向正确位置。
>
> 方法 2: **该方法是基于新的链表复制方法衍生出来的**，时间复杂度为`O(n)`，推荐。
>
> - 在原链表上复制链表，比如原链表为: `A->B->C->D->NULL`，那么复制后的链表为: `A->A1->B->B1->C->C1->D->D1->NULL`；
> - 在第一步的基础上，由于位置关系已经有了，可以直接复制随机指针 random 的指向，这里只需要遍历一遍即可，时间复杂度为`O(n)`；
> - 将复制完的链表拆分即可。

### [LC_146](https://leetcode.cn/problems/lru-cache/description/): LRU 缓存

本题目是属于链表的一个综合性题目，注重链表的操作，题目的思路不难，但是操作较为繁琐（很容易出错，需要理清思路慢慢做）。

需要注意的是，题目要求：每一次对 LRU 数据结构的操作，都必须是`O(1)`。

> 首先，我们需要明确的是，LRU 的实现使用链表作为数据结构为基础，需要实现三个函数，思路如下：
>
> `LRUCache(int capacity)`函数：主要完成 LRU 缓存容量等的初始化；
>
> `int get(int key)`函数：通过 key 从 LRU 缓存中找到对应的 value，本质就是遍历 LRU 缓存；
>
> - 通过 key 找到了对应的 value，返回 value，并且将当前节点从该位置删除，插入到链表的头节点（满足 LRU 的定义）；
> - 通过 key 没找到 value，返回 -1。
>
> `void put(int key, int value)`函数：将 <key, value> 键值对放入到 LRU 缓存中去，操作如下：
>
> **LRU 容量没满**
>
> 	- 先遍历链表，看是否有对应的 key 值节点
> 		- 没有：直接将 <key, value> 组成的新节点插入到链表头部，头部表示最新操作的 <key, value>，并且此时要记录 LRU 的容量减1；
> 	- 有：修改对应 key 值节点的 value 值，并且将该节点从原来位置删除，插入到链表头部（满足 LRU 的定义）。
>
> **LRU 容量已满**
>
> - 先遍历链表，看是否有对应的 key 值节点
> 	- 没有：直接将 <key, value> 组成的新节点插入到链表头部，头部表示最新操作的 <key, value> 节点，删除链表的尾部节点；
> 	- 有：修改对应 key 值节点的 value 值，并且将该节点从原来位置删除，插入到链表头部（满足 LRU 的定义）。
>
> 限于 `get()`和`put()`函数只能在 O(1) 的时间复杂度内完成，所以基于上述思路，对于其具体操作，<font color = red>需要优化</font>。
>
> - `get()`优化
>   - 在 O(1) 时间复杂度内完成查找，**第一时间想到使用哈希表实现**；
>   - 完成查找之后，需要将该节点从找到的位置删除，并且放到链表头部，我们知道，传统的单链表，删除节点需要找到其前驱节点，这里想到**使用双链表的实现**。
> - `put()`优化：和`get()`函数一样，涉及到查找和删除操作，都必须以 O(1) 的时间复杂度内完成，并且由于插入节点是放在链表头部，可以在 O(1) 内完成。

# 第五章 栈和队列

栈和队列的学习中，主要抓住以下三点：

> - 栈和队列更多的是以辅助工具来考察的。
>
> - 栈和队列特殊的变体：**单调栈和优先队列**。
>
> - 注意点：要求会自己默写栈、队列的创建及其方法的使用。

## 5.1 栈的简单应用

### [LC 232](https://leetcode.cn/problems/implement-queue-using-stacks/description/)：用栈实现队列

这一个题目是栈的应用最经典的题目，比较简单，但是也**很容易下意识的想不到最优解**，主要针对的是出队和取队头元素（两个操作基本一样），其中一个栈是天然的队列出队顺序，不要复原到另一个栈中！！！！

### [LC 225](https://leetcode.cn/problems/implement-stack-using-queues/description/)：用队列实现栈

注意审题，题目要求仅仅使用两个队列实现栈，这个题目有两种方法，第一种方法用一个队列实现，第二种方法用两个队列实现。

> 方法一：使用一个队列
>
> - push：直接调用 queue 的 push 即可；
>
> - pop：根据 queue 的 size，进行 size-1 次的出队和入队，得到的是栈顶元素，最后一次只出队，不入队；
>
> - top：先进行 size-1 次的出队和入队，得到栈顶元素，再进行一次出队入队，恢复队列的结构；
>
> - empty：直接调用 queue 的 size 或者 empty 函数即可。
>
> 方法二：使用两个队列实现，核心思想是，push 操作永远插入到为空的队列，然后，再将不为空的队列元素放入到开始为空的队列，这样就保证了，**最后一次插入的元素永远是最先出队**。

### [LC 20](https://leetcode.cn/problems/valid-parentheses/description/)：有效的括号

很经典的一个栈的应用题目。

> 传统的方法，遍历字符串：
>
> - 左括号
>   - 入栈；
> - 右括号
>   - 栈为空
>     - 不是有效括号；
>   - 栈不为空
>     - 匹配，出栈；
>     - 不匹配，说明不是有效括号；
> - 遍历完毕后
>   - 为空：有效括号；
>   - 不为空：非有效括号。
>
> **另一个新巧的思路**，其实思想是一样的，遍历字符串：
>
> - 左括号
>   - 入栈其对应的右括号；
> - 右括号
>   - 栈为空
>     - 不是有效括号；
>   - 栈不为空
>     - 相等，出栈；
>     - 不相等，说明不是有效括号；
> - 遍历完毕后
>   - 为空：有效括号；
>   - 不为空：非有效括号。

### [LC 150](https://leetcode.cn/problems/evaluate-reverse-polish-notation/description/)：逆波兰表达式

这道题属于栈的应用，非常经典，思路如下：

> 遍历逆波兰表达式
>
> - 数字
>   - 直接入栈即可
> - 运算符
>   - 相继出栈，注意，后出栈的数字在运算符的左边
>   - 将计算的结果放回到栈中

### [LC 155](https://leetcode.cn/problems/min-stack/description/)：最小栈

这个题目比较有意思，一开始不太会，思路如下：

> 定义栈 ordered_nums，该栈的栈顶永远存放着当前栈状态的最小值，另一个栈 nums 存放栈的元素；
> - push
>     - ordered_nums
>         - 当前元素小于等于栈顶元素，入栈；
>         - 否则，不做操作；
>     - nums：正常入栈；
> - pop
>     - ordered_nums
>         - nums.top() == ordered_nums.top()，出栈；
>         - 否则，不做操作 ；
>     - nums：正常出栈；
> - 特殊情况，第一个元素入栈 nums 时，ordered_nums 为空，也要入栈
>     - 此时，最小元素就是其本身，只要 nums 不为空，ordered_nums 也一定不为空，getMin() 函数一定能取到值；

### [LCR 148](https://leetcode.cn/problems/zhan-de-ya-ru-dan-chu-xu-lie-lcof/description/)：验证图书取出顺序

这个题目是入栈和出栈的模拟题，当入栈的顺序和出栈的结果给出后，栈的入栈和出栈的操作是唯一的。

> 基本思路：
>
> - 一开始是没有思路的，想尝试从出栈的结果出发，看是否符合入栈的顺序，这种方式是不可取的，因为找不到一种判断规则，先入栈的元素也可能先出栈。
>
> 模拟题的本质就是，可以通过给出的入栈顺序和出栈结果，通过辅助栈进行模拟，如果辅助栈最后为空，即模拟成功。

## 5.2 优先队列

**STL 操作**

```c
//priority_queue, 优先队列默认大根堆实现

// 小根堆的最简化定义
priority_queue<int,vector<int>, greater<int>> pq1;

// 大根堆的最简化定义
priority_queue<int> pq2;


void Demo06() {
    //自定义比较函数，小根堆实现
    auto cmp = [](int a, int b)->bool {return a > b;};
    priority_queue<int, vector<int>, decltype(cmp)> pq(cmp);
    for (int i = 0;i < 5;++i) {
        pq.push(i + 1);
    }
    cout << pq.top() << endl;   //返回优先队列顶部元素
    pq.pop();
    cout << pq.top() << endl;
}

```

**底层实现：大根堆和小根堆**

**出队和入队时间复杂度：`O(logn)`，主要是调整堆的结构。**

**适合用的题型（部分）**

> - 求第 k 大的最值；
> - 前 k 大的值都求出来。

**注意事项**

> - 知道基本的 API 操作；
> - 知道自定义排序规则。

### [LC 215](https://leetcode.cn/problems/kth-largest-element-in-an-array/description/)：数组中的第 K 个最大元素

这个题目属于小根堆的应用，题目思路很简单，维护一个大小为 k 小根堆即可，把数组遍历完毕之后，维护的小根堆的 `top()` 值就是要找的元素。

### [LC 347](https://leetcode.cn/problems/top-k-frequent-elements/description/)：前 K 个高频元素

和上一题目的思路基本一致，这里题目明确了时间复杂度的要求，优于`O(n*logn)`，基本思路如下：

> 使用哈希表 `unordered_map<int, int>` 存放每一个数组出现的次数（题目也保证了每个不同的数字，出现的频次一定不一样）；
>
> - 第一步，遍历数组中的元素，构造哈希表；
> - 第二步，通过哈希表中，每一个数字出现的频次，构造大小为 k 的小根堆，遍历完哈希表之后，小根堆的所有值就是，前 k 个高频元素出现的次数。
>
> <font color =red>注意：</font>题目需要的是前 k 个高频元素的数字，不是对应出现的次数，可以通过哈希表的值，去寻找对应的键。

### [LC295](https://leetcode.cn/problems/find-median-from-data-stream/description/)：数据流的中位数

这个题目比较有意思，正常找数据流的中位数：使用 vector 存储数据流，每执行一次 addNum() 都对数组进行排序，但由于题目数据量和函数调用次数的限制，会超时，**这里需要用到非常规的思路**。

> ​    改进思路：
> - addNum() 和 findMedian() 函数中，算法的时间复杂度不能超过 O(logn)，题目的重点，只关注中位数，也就是，中位数以外的数据，不需要有序，所以常规的排序思路没必要；
> - 想到使用优先队列，优先队列能找到局部数据中的一个最值；
> - 优先队列只关注最小值和最大值：
>     - 中位数介于最小值和最大值之间，想到使用两个优先队列，每个优先队列存储一半的值；
>     - 大根堆存储前半部分较小的值，如果数据流是奇数个，大根堆多存放一个；
>     - 小根堆存储后半部分较大的值；
>     - 中位数要么是大根堆的第一个值，要么是大小根堆 `top()` 的平均值。

## 5.3 单调栈 & 队列

是一种思想，可能在某种题目中可以使用，优化时间复杂度，经典题目：找出数组中每一个元素左边比它大的第一个元素。

> 单调栈：栈中的元素是单调递增或者递减；
>
> 单调队列：队列中的元素是单调递增或者递减。

### [LC 1475](https://leetcode.cn/problems/final-prices-with-a-special-discount-in-a-shop/description/): 商品折扣后的最终价格

**单调栈的应用**，题目意思很简单，就是找到数组中元素右边第一个比它小的元素。

> - 常规思路，暴力法，这个题目没有时间复杂度的限制，O(n^2)；
>
> 优化：利用单调栈，时间复杂度为 O(n)。如何知道需要优化：考虑极端的例子：1, 2, 3, 4, 5：
>
> - 暴力法很明显存在重复计算，因为在找第一个元素右边第一个比它小的值时，就已经知道了所有元素都不存在右边比它小的值，暴力法后续的遍历都是**重复计算**；
>
> - 单调栈思路：维护一个递增的单调栈，从右往左遍历，单调栈的栈顶存放了右边比该元素小的第一个值。

### [LC 239](https://leetcode.cn/problems/sliding-window-maximum/description/): 滑动窗口的最大值

**单调队列的应用**，和上一题一样，题目的意思很简单，就是找到每一个滑动窗口中的最大值。

> - 常规思路，暴力法，O(k*n)，超出了题目限制的时间复杂度；
> - 暴力法很明显存在重复计算，因为找到一个滑动窗口的最大值时，该最大值可能也是后续滑动窗口的最大值，所以后续滑动窗口不需要重复查找；
>
> 优化：利用单调队列，时间复杂度为 O(n)，空间复杂度为 O(k);
>
> - 单调队列思路：维护一个单调递减的队列（边界值两个元素相等可以存在队列中，也可以不存在队列中）：
>   - 单调递减队列中，队头元素永远存放了当前滑动窗口的最大值；
>   - 队头元素后面的元素是后续滑动窗口可能的最大值；
>   - 需要注意的是，滑动窗口存放的值是遍历数组元素的索引，当队头元素索引不在滑动窗口的范围内时，要及时出队；
> - 单调队列数据结构选择：使用双端队列 `deque` ，因为出队操作可能在队头（索引值超过滑动窗口的范围），也可能在队尾（不可能成为滑动窗口的最大值）。

# 第六章 二叉树

二叉树考察频率较高，基本考察原题，算法题主要考点：

> - 遍历与构造类
>   - 递归遍历
>   - 队列辅助
> - 路径与求和类
>   - DFS 和 BFS
> - 其它

## 6.1 二叉树的遍历

用栈实现二叉树的遍历，先序遍历比较好理解，中序遍历和后序遍历不太好理解

### [LC144](https://leetcode.cn/problems/binary-tree-preorder-traversal/description/)：二叉树的先序遍历

先序遍历的思路比较简单，因为访问二叉树的节点顺序和输出二叉树的节点顺序是一致的。

### [LC94](https://leetcode.cn/problems/binary-tree-inorder-traversal/description/)：二叉树的中序遍历

中序遍历的思路比较复杂，因为访问二叉树节点的顺序和打印二叉树节点的顺序不一致。

### [LC145](https://leetcode.cn/problems/binary-tree-postorder-traversal/description/)：二叉树的后序遍历

后序遍历和中序遍历一样，比较复杂，但是后序遍历有一个比较简单的处理方式，如下：

> 思路一：
>
>         - 不断迭代，根节点最先入栈（最后打印），然后先处理左子树，左子树节点入栈，左子树为空的前提下，右子树入栈
>             - 找到了局部根节点，如果没有左右子树，输出该节点，并且判断该节点是否是其父节点的左孩子
>                 - 是：访问父节点的右子树
>                 - 否：继续输出该节点的父节点（也就是上一个局部根节点）
>
> 思路二：由于后序遍历是左右根，前序遍历是根左右，只需要需改前序遍历的输出为根右左，然后反转输出的结果就是后序遍历。

### [LCR149](https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/description/)：二叉树的层序遍历

思路很简单，对于层序遍历，使用队列辅助即可，也是 BFS 的模板。

> 二叉树的层序遍历基本思路：
>    - 使用队列辅助，一个 while 循环处理二叉树的每一个节点
>         - 队头元素节点出队后，如果该节点有左右孩子，左右孩子节点应该入队
>         - 直到队列为空，退出循环

变种题目：[LC102](https://leetcode.cn/problems/binary-tree-level-order-traversal/description/)，[LC107](https://leetcode.cn/problems/binary-tree-level-order-traversal-ii/description/)，两个都是对层序遍历输出结果进行处理即可。

### [LC 104](https://leetcode.cn/problems/maximum-depth-of-binary-tree/description/)：求二叉树的最大深度

遍历二叉树即可，使用递归三部曲，代码简洁。

> - 找到递归结束条件；
> - 将大规模的问题拆解，寻找当前问题和子问题的递推式：
>   - 这里的递推式是：`h(n) = 1 + max(left_h(n-1), right_h(n-1))`;
> - 编写递归业务逻辑。

### [LC 110](https://leetcode.cn/problems/balanced-binary-tree/description/)：平衡二叉树

题目思路很简单，就是求二叉树最大深度的变种。

> 平衡二叉树的定义：严格地说，每一个节点的左右子树高度差不超过1。
>
> 方法一：时间复杂度为：O(n^2)，注意，计算递归的时间复杂度，可以画出递归树，查看每一个节点被访问的次数，同理，二叉树也是一样的。
>
> - 节点开始，求左右子树的高度，判断其是否为平衡二叉树的节点；
>     - 之后继续判断左右子树的节点是否满足平衡二叉树的定义；
>     
>
> 方法二：优化到时间复杂度为 O(n)。
>
> - 用一个全局变量做标记，在计算二叉树的高度时，每一次计算完左右子树的高度都进行比较，如果存在不满足平衡二叉树的节点，将标记改为 false，表示不是平衡二叉树。

### [LCR 144](https://leetcode.cn/problems/er-cha-shu-de-jing-xiang-lcof/description/)：翻转二叉树

题目思路简单，翻转每一个二叉树节点的左右孩子即可，加深 Cpp 中指针的概念！

### [LC 101](https://leetcode.cn/problems/symmetric-tree/description/)：对称二叉树

题目思路比较奇妙，可以说是翻转二叉树的应用，但是比翻转二叉树难，思路不是很容易想到。

> 基本思路：
>
> - 如果二叉树是对称二叉树，那么其对应的镜像节点的值相等，所以只需要判断互为镜像的节点值是否相等即可；
> - 利用递归可以实现每一个镜像节点的遍历：
>   - 注意，第一个根节点是没有镜像节点的，所以第一个节点不用处理，需要自定义一个函数。

### [LC 199](https://leetcode.cn/problems/binary-tree-right-side-view/description/)：二叉树的右视图

题目思路正常，属于层序遍历的变种题目，输出每一层的最后一个节点即可。

### [LC 662](https://leetcode.cn/problems/maximum-width-of-binary-tree/description/)：二叉树的最大宽度

题目思路正常，属于层序遍历的变种题目，需要注意的是，每一层的最大宽度定义是每一层最右边的节点位置减去每一层最左边节点位置加1。

以后遇到类似求二叉树最大宽度的题目，第一反应想到只能通过数组下标法来解决。

> - 每一层最右边和最左边节点位置的计算，可以通过满二叉树的公式计算：
>   - 左孩子节点位置 = 父节点位置 * 2；
>   -  右孩子节点位置 = 父节点位置 * 2 + 1；

<font color = red>注意：</font>层序遍历的时候，不能简单的使用空节点代替不存在的满二叉树节点，会造成内存溢出，所以需要一个记录每一个节点索引的队列辅助。

### [LC 105](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/description/)：从前序与中序遍历序列构造二叉树

题目考察了二叉树的概念，需要知道二叉树的三种遍历之间的关系，通过前序遍历和中序遍历可以确定一颗二叉树，题目思路不难，但是很久不做容易忘记。

> 基本思路：
>
> - 通过前序遍历找到根节点；
> - 通过根节点，在中序遍历中，找到左右子树；
> - 递归的构造左右子树即可。

### [LC 106](https://leetcode.cn/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)：从中序与后序遍历序列构造二叉树

和上一题思路一致，题目本质是一样的，可以通过中序遍历和后序遍历确定一颗二叉树。

### [LC 230](https://leetcode.cn/problems/kth-smallest-element-in-a-bst/description/)：二叉搜索树

理解二叉搜索树的概念，对于二叉搜索树每一个节点，其左子树的值都小于该节点的值，其右子树的值都大于该节点的值，所以二叉搜索树的中序遍历是一个有序的序列。

### [LCR 156](https://leetcode.cn/problems/xu-lie-hua-er-cha-shu-lcof/description/)：序列化与反序列化二叉树

属于开放性题目，类似于加密和解密算法，但是题目对序列化和反序列化没有要求，可以自由发挥。

> 思路：
>
> - 第一时间想到的是，通过存储二叉树的先序遍历和中序遍历结果进行序列化，然后再对先序遍历和中序遍历的结果反序列化构造二叉树，（但是这需要一个前提：二叉树中不存在重复的节点值，不然的话，无法在中序遍历中定位根节点），所以该方法不行。
>
> 进一步构思：使用层序遍历的方法，具体如下：
>
> - 在序列化的过程中，使用队列实现对二叉树的层序遍历（这里需要做一下修改，空节点也要入队列中），将层序遍历的结果存放到字符串中（空节点用一个特殊的标记记录即可）。
> - 在反序列化过程中，还是使用队列 + 序列化的结果实现，因为序列化的结果字符串中，存放了层次遍历的结果，需要借助队列通过层序遍历的结果，构造二叉树：
>   - 细节1：在反序列化构造二叉树的过程中，空节点不需要入队处理。

### [LC 112](https://leetcode.cn/problems/path-sum/)：路径总和Ⅰ

题目意思简单，从二叉树中找到一条路径（从根节点到叶子节点），该路径上的所有节点值的总和等于一个目标值。

> 目标值的一个新思路：
>
> - 第一个方法是从0开始，逐渐加到目标值；
> - 第二个方法是从目标值开始，逐渐减到0，且到叶子节点，表示该路径符合要求（该题目建议）。

### [LC 113](https://leetcode.cn/problems/path-sum-ii/description/)：路径总和Ⅱ

和上一题思路一致，只是返回的答案需要记录路径，并且可能有多条路径（用容器存储）。

### [LC 235](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-search-tree/description/)：二叉搜索树的最近公共祖先

该题目比较考验技巧性（可能想不到的思路），注意需要用到二叉搜索树的条件。

> 最开始思路1，也是很容易想到，没有用到二叉搜索树的条件：
>
> - 分别打印出两个节点的遍历路径；
> - 通过两个路径比较，最后一个相同的节点就是公共祖先。
>
> 思路2，利用二叉搜索树的条件，遍历二叉搜索树的节点
>
> - 如果两个节点的值都小于当前遍历的节点，说明最近公共祖先一定在当前节点的左子树；
> - 如果两个节点的值都大于当前遍历的节点，说明最近公共祖先一定在当前节点的右子树；
> - 其它情况：说明找到了公共祖先（题目有说明，节点本身也可以是自己的祖先节点）。

## 第七章 位运算与数学专题

> 难度：easy & medium；
>
> 考察频率：不怎么高；
>
> 特点：如果没有了解过，可能想不出最优解或者做不出来。
>
> 位运算特点：高效
>
> - 判断一个数是奇数还是偶数，把`n%2 == 0`转换位`n>>1`；
> - `n/2`转换位`n>>1`。
>
> **Cpp中位运算表示**
>
> ```c++
> // 位操作可以实现高效计算
> // 优先级：~ > 算术运算符 > (<<, >>) > 比较运算符 > & > ^ > | > 逻辑运算符
> int a, b;
> int c;
> c = a&b;		// 按位与
> c = a|b;		// 按位或
> c = a^b;		// 按位异或
> c = ~a;			// 按位取反
> c = a<<2;		// 左移两位，相当于乘2的平方（前提是没有溢出）
> c = a>>2;		// 右移两位，相当于除2的平方
> ```
>
> 位运算的一些技巧，可以实现高效计算
>
> - 异或技巧
>
> ```c++
> int a,b;
> // 一个数与其本身异或的值恒为0
> // 任何一个数与0异或还是其本身
> a^0 == a;
> b^b == 0;
> a^b^b == a;
>  
> ```
>
> - 与技巧
>
> ```c++
> // 对于一个数n，与其 n-1 的值相与，可以消除 n 的二进制中，最右边的1，因为n-1操作，对n二进制中，最右边的1进行了借位
> int n = 0b10010;
> int tmp = n & (n-1);
> // tmp 的值为 0b10000
> ```
>
> - m的n次方技巧
>
> ```c++
> /*
> 	n = 13 = 1101 = 1+4+8;
> 	m^n = (m^1)*(m^100)*(m^1000)
> */
> int pow(int m,int n){
>   int res = 1;
>   int tmp = m;
>   
>   while(n>0){
>     if(n&1 == 1){
>       sum = sum*tmp;
>     }
>     tmp *= tmp;	// 关键的一步，相当于 tmp, tmp^2, tmp^4
>     n = n>>1;
>   }
>   
> }
> ```
>
> 数学相关
>
> - 大树运算

### [LC 136](https://leetcode.cn/problems/single-number/description/)：只出现一次的数字

题目思路简单，属于异或运算的应用，但是如果不知道的话，很难做出来。

> 思路：这题考察位运算中，异或运算的应用。
>
> - 两个相同的数异或值为0；
> - 任何数与0异或的值为其本身；
> - 异或支持交换律。

### [LCR 133](https://leetcode.cn/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/description/)：位 1 的个数

题目思路简单，属于与运算的应用，当然也可以使用传统的方法来解题，只是效率没有位运算高，<font color = red>这个题目很重要，提供了统计整数中二进制位1的个数的方法</font>。

> 思路：这个题目的本质是，统计4字节无符号整数的二进制表示中位1的个数，可以使用与运算技巧。

### [LC 260](https://leetcode.cn/problems/single-number-iii/)：只出现一次的数字Ⅲ

和[LC 136](https://leetcode.cn/problems/single-number/description/)不同的是，数组中只出现一次的数字需要统计两个，所以不能使用异或运算简单的把只出现一次的数字找出来，属于思路题。

> 思路：
>
> - 对数组中所有数进行异或操作得到数 z，数 z 的本质就是两个不同数的异或结果；
> - 对数 z 从右往左，找到第一个为 1 的二进制位（技巧，通过定义一个变量和数 z 进行与操作），将找到的二进制位组成新的数m；
> - 将数 m 和数组中所有的数进行与操作，得到为0的结果分为新的一组，得到不为0的结果分为另一组；
> - 拆分的数组中，分别存在只出现一次的数，这时候就可以用异或操作得出结果了。

### [LC 137](https://leetcode.cn/problems/single-number-ii/description/)：只出现一次的数字Ⅱ

和 [LC 136](https://leetcode.cn/problems/single-number/description/) 不同的是，数组中其它数字重复出现了3次，只出现一次的数字只有一个，也不能用异或的运算得出结果。

> 思路：
>
> - 通过对4字节每一个二进制位，统计数组中所有数字在指定二进制位是否为1，如果为1，则在该二进制为的统计结果中加1；
> - 4字节每一个二进制位统计的结果对3取模，得到的值就是只出现一次数字的二进制位结果；
> - 对结果进行输出即可。
>
> <font color = red>这个思路也可以解题 [LC 136](https://leetcode.cn/problems/single-number/description/) ，只需要第二步改为对2取模即可</font>

### [LC 172](https://leetcode.cn/problems/factorial-trailing-zeroes/description/)：阶乘后的零

题目本质也是一个数学问题，如果不知道思路，也很难想到

> 暴力法：单纯计算大数相乘是无法通过全部案例的
> 思路转换：
>
> - 计算阶乘后的 0，本质是计算 n 个数字中，一共有多少对 `2*5 = 10`，因为 10 可以产生一个0，5 = 20 也可以产生0，但是是可以分解为：`20 = 2*10 = 2*2*5` ；
> - n 个数字中，因数 5 的个数一定比因数 2 的个数少，所以只需要计算 5 的因数个数即可，需要注意的是，n中可能可以分解出多个5。

### [LC 169](https://leetcode.cn/problems/majority-element/description/)：多数元素

简单题，暴力法可以通过，这里了解一下摩尔投票法。

> 暴力法1：使用 unordered_map 统计每一个数出现的次数，时间复杂度：O(n)，空间复杂度：O(n)；
>
> 暴力法2：使用排序，由于要找的数出现的次数大于 n/2, 所以中间的数一定是我们要找的数，时间复杂度：O(N*LOGN)，空间复杂度O(1)；
>
> 优化思路：**摩尔投票法**
>
> - 通过设计一个容器，存储遍历的数，如果遇到相同的数，加入到容器中，遇到不同的数，从容器中抵消一个数（删除），最后剩下的数字就是我们要找的数；
> - 虽然使用了容器，但是时间复杂度是O(N)，空间复杂度是O(1)。

### [LCR 186](https://leetcode.cn/problems/bu-ke-pai-zhong-de-shun-zi-lcof/description/)：文物朝代判断

判断一串数字，是否为连续的情况，且一串数字中 0 可以替代任何数字。

> 在数字没有重复的前提下，且不对 0 做处理，这个时候简单的找出一串数字中的最大值 max 和最小值 min，然后 `max-min<5`则说明一串数字是连续的。
>
> 但是本题目的测试用例中存在重复数字，需要使用 set 存储数字，判断是否有重复的情况，有重复的情况**说明不是连续的**。
>
> - 找出除 0 之外的最大值和最小值；
> - 用 set 去重。

### [LC 343](https://leetcode.cn/problems/integer-break/)：整数拆分

这一题属于找数字的数学规律题目，对于 int 范围内的整数，只要大于等于 5 的数，拆分成以 3 为基准的数字，乘积最大。

> 基本思路：找规律
> - 拆分成两个数之和：两个数尽可能的均匀；
> - 归纳到多个数：多个数尽可能的均匀；
> - 从小数拆分开始找规律
>      - 1~4之间的数字：拆分成两个数的乘积小于等于其本身；
>      - 5：拆分成 2 和 3，乘积大于其本身；
> - 大数找规律
>      - 6：拆分成 3 和 3 乘积最大；
> - 综合：以拆分3为基准
>      - case1：完全拆分；
>      - case2：还剩下2；
>      - case3：还剩下1，从上一个拆分的 3 中均分为 2 和 2；

### [LCR 132](https://leetcode.cn/problems/jian-sheng-zi-ii-lcof/description/)：砍竹子Ⅱ

本题考察了数值运算中，取模与乘法一起的分配律问题。

> 本题和整数拆分本质是一样的，但是需要考虑数值溢出的问题，考察如何在计算大数的过程中，取模的问题。
>
> - 取模与乘法一起，在两个乘数都大于取模的数时，符合分配律，`(6%5)*(7%5) == (6*7)%5`。

# 第八章 贪心

> 属于一种思想，题目没有什么套路。
>
> 贪心：只关注局部最优解/当前状态的最优解，不关注全局最优解。
>
> 贪心的一般使用场景：希望能够通过局部最优解 ==》推算出全局最优解。
>
> **套路**
>
> 基本没有，对于一道题目的描述，很难看出是否要使用贪心。
>
> - 只能靠直觉和多做题目。
>
> **面试**
>
> 贪心算法考的不多，重点掌握七八道核心题即可。

### [LC 455](https://leetcode.cn/problems/assign-cookies/description/)：分发饼干

题目思路比较简单，就是尽可能地**把小饼干分配给胃口小的孩子（贪心思想，满足局部最优）**。

> - 对孩子胃口和饼干大小进行排序；
> - 统计能满足孩子胃口的个数即可。

### [LC 376](https://leetcode.cn/problems/wiggle-subsequence/description/)：摆动序列

题目要求出数组中，最长的摆动子序列。

> 基本思路：
>
> - 计算出数组中每两个数之间的差值，组成一个新的数组 tmps，每一个差值代表一次摆动（以数值为参考量画出折线图比较直观）；
>
> - 利用双指针法，在 tmps 数组中寻找不一样符号的数对。
>
> 不一样符号数对的结果 + 2 就是答案。

### [LC 55](https://leetcode.cn/problems/jump-game/description/)：跳跃游戏

这题如果想着用贪心的思想做，好像无法解题。

> 常规思路1：为了能够尽快到达最后一个下标，每次都选择最大的跳跃步数（基于局部考虑，贪心思想），但是不能解题。
>
> 常规思路2：遍历 nums 数组即可，求出当前格子能够跳到最大位置处（没想到）想到了的话，非常好理解，用一个变量 `max_idx` 记录前面格子的最大跳跃数：
>
> - 如果`max_idx>=i`，说明能继续跳下去，否则，无法继续跳跃；
> - 当前格子能够跳跃的位置：`i + nums[i]`，与 `max_idx` 做比较，如果大于，需要重新赋值，`max_idx` 表示跳跃在当前位置下，能够跳到的最大位置。

### [LC 45](https://leetcode.cn/problems/jump-game-ii/description/)：跳跃游戏Ⅱ

和上一题不一样的是，这一题给的测试用例一定能跳到数组的终点，需要求出最小跳跃次数。

> 常规思路1：既然是最小跳跃次数，那么每次尽可能的跳多一点（贪心思想），无法解题；
>
> 常规思路2：求当前能跳跃的格子中，支持最大的跳跃步数，然后当前节点跳跃到那个格子中（贪心思想），遍历数组即可，利用双指针法，标记每一次跳跃所能达到的最后一个位置。
>
> - 考虑当前局部的选择，能获得的最大收益，即当前跳到哪一个格子，能使得下一次跳跃更远。

### [LC 621](https://leetcode.cn/problems/task-scheduler/description/)：任务调度器

> 题意：执行一个任务需要一个时间间隔，任务执行完后需要冷却时间（像是思路题）；
>
> 初步思路（暴力法）：使用键值对统计每一种任务的个数，一个一个任务的分析（非常复杂，貌似写不出来）；
>
> 进一步思考：统计出现次数最多的任务，通过 n 计算出时间间隔
>
> - 如果时间间隔够插入其它的所有任务，返回出现次数最多的任务的最短执行时间间隔即可；
>
> - 如果时间间隔不够插入其它的所有任务，返回数组的长度即可
>
> 需要注意出现次数最多的任务可能有多种。（😄😄）

### [LC 435](https://leetcode.cn/problems/non-overlapping-intervals/description/)：无重叠区间

> 初步思路：通过对区间的start进行排序，如果存在重复区间，那么会存在多个start，然后删除多个start，上述思路不正确。
>
> 题意不仅需要考虑区间不重叠，还需要考虑移除区间的最小数量。
>
> **转化思路**：保留最多的区间数量 max，使得区间不重叠，`intervals.size() - max`即为答案。
>
> 例子：intervals = [[1,2],[2,3],[3,4],[1,3], [2,5]]；
>
> - 根据每个区间的 end 从小到大排序每一个区间；
>
> - 尽可能的选择区间长度小的区间，因为区间长度小，对于其它区间的覆盖概率就减小了（这里符合贪心思想）。

### [LC 135](https://leetcode.cn/problems/candy/description/)：分发糖果

> 第一眼直接没思路，暴力法也想不出来。
>
> 思路：拆分相邻孩子的影响为左右规则，使用分类谈论法：
> - 只考虑左规则的影响，也就是只有当前元素比左边元素大的时候，才分配多一个糖果，否则只提供一个糖果（这里感觉是贪心思想）；
> - 同理只考虑右规则的影响；
> - 得到两个不同规则的数组，每一个元素取较大值即可满足左右规则；
>
> 本题的难点在于思路比较独特，题目意思比较好理解（属于思路想不到，无法用暴力法解决）。
>
> <font color = red>本题也提供了一种新的思路</font>：就是对于相邻元素的条件，可以尝试分类讨论，只考虑左右一边的影响，然后将分类讨论的结果合并即可，合并的结果能够同时满足左右（相邻）元素的条件（具体通过题意选择合并的思路）。

# 第九章 回溯

回溯的字面意思：对于解决问题的其中一步处理了之后，再 [撤销] 处理，[撤销] 这个动作就是回溯。

**回溯常用解决的问题**

> 1. 棋盘问题
> 2. 路径搜索问题
> 3. 组合问题
> 4. 排列问题
> 5. 子集问题
>
> 对回溯问题进行求解时，可以尝试画出递归树型结构（可视化），方便理解回溯的过程，特别是递归时存在限制条件，比如：组合、排列和子集问题中，存在重复元素的情况。

**回溯的效率**

非常差，回溯相当于暴力法，因为它会尝试枚举每一种可能的情况。

**回溯通用模板**

> ```c++
> /*
> 		回溯算法是用递归实现的，所以回溯模板可以参考递归
> 			- 找到回溯的起点（很重要）
> 			- 列出所有可能的向下递归情况（选择路径）
> 			- 找到递归结束条件
> 			- 处理完递归情况之后，不要忘记回溯（撤销已选路径）
> */
> void backtrace(parameter...){
>   // 回溯结束条件
>   if(condition){
>     return;
>   }
> 
>   // 对可能的情况进行回溯
>   for(all the 'case'){
>     // 标记回溯路径
>     res.push('case');
>     // 递归指定的情况
>     backtrace(parameter);
> 
>     // 递归完指定情况后，撤销回溯的路径（回溯的关键）
>     res.pop();
>   }
> 
> }
> ```
>
> 一般地，回溯相当于暴力法，所以，解题完成后，可以考虑进行优化，回溯中的优化通过<font color = red>剪枝</font>实现，因为正常的回溯题目，能可视化为树结构，所以称为剪枝。

**回溯在面试中的考察频率**

考察频率非常高，不管是机试还是笔试，一般地，有些题目可以用回溯解决，但是由于暴力法的原因，拿不到 100% 的分数，考的比较多的是**组合和排列问题**。

**学好回溯的关键**

> - 对递归有深入的了解，回溯可以简单的理解为递归操作返回之后，加了一步撤销；
> - 递归三部曲 ==> 回溯三部曲
>   - 确定递归结束条件；
>   - 寻找递归的递推式；
>   - 寻找在递推式中，需要回溯的关键点
> - 刚刚开始做回溯题目的时候，不一定要弄清楚细节，先学会**结合回溯模板做题**，很容易和递归一样，想着想着进入死结，脑洞爆炸警告⚠⚠⚠；
> - 掌握常见的 15+ 道题目即可。

### [LC 77](https://leetcode.cn/problems/combinations/description/)：组合

### [LC 216](https://leetcode.cn/problems/combination-sum-iii/description/)：组合总和Ⅲ

### [LC 40](https://leetcode.cn/problems/combination-sum-ii/description/)：组合总和Ⅱ

### [LC 39](https://leetcode.cn/problems/combination-sum/description/)：组合总和

### [LC 78](https://leetcode.cn/problems/subsets/description/)：子集

### [LC 90](https://leetcode.cn/problems/subsets-ii/description/)：子集Ⅱ

### [LC 46](https://leetcode.cn/problems/permutations/description/)：全排列

### [LC 47](https://leetcode.cn/problems/permutations-ii/description/)：全排列Ⅱ

### [LC 51](https://leetcode.cn/problems/n-queens/description/)：N皇后

### [LC 93](https://leetcode.cn/problems/restore-ip-addresses/description/)：复原 IP 地址

<font color = red>腾讯企业微信后端一面原题。</font>

# 第十章 动态规划

## 10.1 动态规划基础

**动态规划主要解决的问题**

> - 求最值问题，比如最大值和最小值（理解为最优解）；
> - 字符串问题
>   - 包含 80% 左右；
>   - 如果是两个字符串类型的题目，95%以上都可以用递归解决；
>   - 字符串问题中，不使用递归解决的，往往比较简单；
> - 动态规划求解的问题一般可以使用暴力法，但是暴力法时间复杂度高，只能通过部分案例。

**如何理解动态规划**

> 1. 动态规划一般是通过**使用一定的空间**，得到时间复杂度的最优解；
> 2. 通过利用空间存储的历史记录，寻找出一些规律，进而规划当前状态的选择，最后实现避免重复计算。

**如何学好动态规划**

> - 一般地，在刚刚学习动态规划的时候，无从下手，但是能看懂动态规划的答案；
> - 多刷题，通过套路 or 模板，刚开始不理解套路 or 模板没关系，做多了就会自然理解。

## 10.2 动态规划与递归的一些关系

**从递归理解动态规划**

> 例题：**斐波那契数** （通常用 `F(n)` 表示）形成的序列称为 **斐波那契数列** 。该数列由 **0** 和 **1** 开始，后面的每一项数字都是前面两项数字的和。
>
> ```c++
> int fib(int n) {
> 	if(n == 0){
>     return 0;
>   }
>   if(n ==1 || n==2){
>     return 1;
>   }
>   
>   int sum = f(n-1) + f(n-2);
>   return sum;
> }
> ```
>
> 常见的做法是使用递归，但是，通过可视化递归树，发现存在很多重复没必要的计算。
>
> **优化版本**
>
> 通过辅助空间，进行**状态保存**，避免重复计算。
>
> ```c++
> vector<int> arr(n,0);
> int fib(int n) {
> 	if(n == 0){
>     return 0;
>   }
>   
>   if(n ==1 || n==2){
>     return 1;
>   }
>   
>   if(arr[n]!=0){
>     return arr[n];
>   }
>   
>   int sum = f(n-1) + f(n-2);
>   arr[n] = sum;
>   
>   return sum;
> }
> ```
>
> 优化版本的题解，通过辅助空间进行了状态保存，但是我们可视化出递归树可以发现，问题的求解是自顶向下的。
>
> **关于自顶向下和自底向上的一些关系**：
>
> 那么，针对斐波那契数列这道题，可以将问题的求解从自顶向下转换为自底向上吗？
>
> 显然是可以的，转换为自底向上的过程，就是动态规划的过程，因为自底向上，保存了历史记录，我们可以通过历史记录，规划当前的状态，这就是一个简单的**动态规划例子**。
>
> ```c++
> int main(){
>   vector<int>fib(n,0);
>   fib[1] = 1;
>   
>   for(int i=2;i<=n;++i){
>     fib[i] = fib[i-1] + fib[i-2];
>   }
>   return fib[n];
> }
> ```
>
> 将上述代码中的 `fib` 变量改为 `dp` ，就是一个简单的动态规划案例啦，`dp` 也称之为状态数组，存储了历史记录。

## 10.3 动态规划三部曲

**动态规划的解题模板**

> 1. 定义 `dp[]` 数组的含义；
> 2. 找出 `dp[]` 数组**当前状态和历史状态之间的关系式**，`dp(i) = f(dp(i-1))`，这一步最重要，找到了题目也就解决了；
> 3. 找出 `dp[]` 数组的初始值（底），因为**动态规划是自底向上的**，只有初始化了（底），向上的推导才能进行下去。
>
> <font color = red>当然，`dp` 数组也可能是二维数组</font>。
>
> 以斐波那契数列题目为例：
>
> ```c++
> int main(){
>   // 1. 定义 dp 数组的含义：我们想要求什么，就定义 dp 为什么（针对较简单题目）
>   vector<int>dp(n,0);
>   
>   // 2. 找出 dp 数组之间的关系式，用于自底向上递推
>   // dp[i] = dp[i-1] + dp[i-2];
>   
>   // 3.找出初始值
>   dp[1] = 1;
>   
>   for(int i=2;i<=n;++i){
>     dp[i] = dp[i-1] + dp[i-2];
>   }
>   
>   return dp[n];
> }
> ```
>
> 一般地，动态规划的题目可以优化（针对空间），比如`O(n^2) ==> O(n)`。

## 10.4 动态规划题目

### [LC 70](https://leetcode.cn/problems/climbing-stairs/description/)：爬楼梯

### [LC 746](https://leetcode.cn/problems/min-cost-climbing-stairs/description/)：使用最小花费爬楼梯

### [LC 198](https://leetcode.cn/problems/house-robber/description/)：打家劫舍

### [LC 62](https://leetcode.cn/problems/unique-paths/description/)：不同路径

### [LC 64](https://leetcode.cn/problems/minimum-path-sum/description/)：最小路径和

### [LC 63](https://leetcode.cn/problems/unique-paths-ii/description/)：不同路径Ⅱ

## 10.5 动态规划的优化

一般地，动态规划定义的`dp`数组进行优化时，可以降低空间复杂度，不会降低时间复杂度。

> - 二维数组优化成一维数组；
>
> - 一维数组优化成常数级空间。

### [01背包问题](https://www.acwing.com/problem/content/description/2/)

> 动态规划三部曲基本思路：
>
> 优化：

```c++
/*
    01 背包问题
*/
#include<bits/stdc++.h>
using namespace std;

int main() {
    int N, V;
    cin >> N >> V;

    vector<int> v(N + 1, 0);
    vector<int> w(N + 1, 0);

    for (int i = 1;i <= N;++i) {
        cin >> v[i] >> w[i];
    }

    // 定义 dp 数组的含义顺便初始化 dp 数组
    vector<int> dp(V + 1, 0);

    for (int i = 1;i <= N;++i) {
      	// 从后往前推，保证的前面的 dp 值是可选物品范围为 i-1 的情况
        for (int j = V;j >= 1;--j) {
            if (j >= v[i]) {
                dp[j] = max(dp[j], dp[j - v[i]] + w[i]);
            }
            else {
                dp[j] = dp[j];
            }
        }
    }

    cout << dp[V] << endl;

    return 0;
}
```

### [完全背包问题](https://www.acwing.com/problem/content/description/3/)

> 动态规划三部曲基本思路：
>
> ```c++
> /*
>     完全背包问题
>     - 和01背包不同的是，每一件物品可以选择无限次（在背包容量够的前提下）
> 
> */
> #include<bits/stdc++.h>
> using namespace std;
> 
> int main() {
>     int N, V;
>     cin >> N >> V;
> 
>     vector<int> v(N + 1, 0);
>     vector<int> w(N + 1, 0);
> 
>     for (int i = 1;i <= N;++i) {
>         cin >> v[i] >> w[i];
>     }
> 
>     // 定义 dp 数组的含义顺便初始化 dp 数组
>     vector<int> dp(V + 1, 0);
> 
> 
>     for (int i = 1;i <= N;++i) {
>        	// 从前往后递推，保证了前面的 dp[j] 已经更新为可选物品范围是 i 的值
>         for (int j = 1;j <= V;++j) {
>             // 每一件物品可以选择多次，那么可以理解为，选完第i个物品后排，可以继续在i个背包的物品范围内选择
>             if (j >= v[i]) {
>                 dp[j] = max(dp[j], dp[j - v[i]] + w[i]);
>             }
>             else {
>                 dp[j] = dp[j];
>             }
>         }
>     }
> 
>     cout << dp[V] << endl;
> 
> 
>     return 0;
> }
> ```
>
> 优化1：从递推式进行详细分析，先优化三层循环，降低时间复杂度
>
> 优化2：

### [多重背包问题](https://www.acwing.com/problem/content/description/4/)

> 动态规划三部曲基本思路：
>
> ```c++
> /*
>        多重背包问题
>     - 和01背包不同的是，每一件物品可以选择有限次
> */
> #include<bits/stdc++.h>
> using namespace std;
> 
> int main() {
>     int N, V;
>     cin >> N >> V;
> 
>     vector<int> v(N + 1, 0);
>     vector<int> w(N + 1, 0);
>     vector<int> s(N + 1, 0);
> 
>     for (int i = 1;i <= N;++i) {
>         cin >> v[i] >> w[i] >> s[i];
>     }
> 
>     // 定义 dp 数组的含义顺便初始化 dp 数组
>     vector<int> dp(V + 1, 0);
> 
>     for (int i = 1;i <= N;++i) {
>         for (int j = V;j >= 1;--j) {
>             // 每一件物品可以选择多次，通过递推式得到最大值
>             for (int k = 0;k <= s[i] && k * v[i] <= j;++k) {
>                 dp[j] = max(dp[j], dp[j - v[i] * k] + w[i] * k);
>             }
>         }
>     }
> 
>     cout << dp[V] << endl;
> 
>     return 0;
> }
> ```
> 
> 优化：

### [LC 5](https://leetcode.cn/problems/longest-palindromic-substring/description/)：最长回文子串

> 

### [LC 718](https://leetcode.cn/problems/maximum-length-of-repeated-subarray/description/)：最长重复子数组

### [LC 300](https://leetcode.cn/problems/longest-increasing-subsequence/description/)：最长递增子序列

### [LC 72](https://leetcode.cn/problems/edit-distance/description/)：编辑距离

使用最少操作将一个字符串转换为另一个字符串

### [LC 10](https://leetcode.cn/problems/regular-expression-matching/description/)：正则表达式匹配

### [LC 122](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/description/)：买卖股票的最佳时机Ⅱ

### [LC 714](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/description/)：买卖股票的最佳时机含手续费

### [LC 121](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/description/)：买卖股票的最佳时机

### [LC 123](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iii/description/)：买卖股票的最佳时机Ⅲ

### [LC 188](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iv/description/)：买卖股票的最佳时机Ⅳ

### [LC 309](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-cooldown/description/)：买卖股票的最佳时机含冷冻期

### [LC 322](https://leetcode.cn/problems/coin-change/description/)：零钱兑换

# 高频题库

### [LC 160](https://leetcode.cn/problems/intersection-of-two-linked-lists/description/)：相交链表

### [LC 206](https://leetcode.cn/problems/reverse-linked-list/description/)：反转链表

两种方法，递归和双指针法，建议使用双指针。

### [LC 234](https://leetcode.cn/problems/palindrome-linked-list/description/)：回文链表

> 方法1：链表中的数字用数组存储，首尾双指针法判断即可，时间复杂度：O(N)；
>
> 方法2：快慢双指针法，把后半段链表反转，然后和前半段链表逐一比较即可，时间复杂度：O(1)。

### [LC 2](https://leetcode.cn/problems/add-two-numbers/description/)：两数相加



# 第十一章 图论

## 11.1 图存储方式

邻接矩阵：二维数组的方式，空间复杂度：`O(v^2)`，v 表示节点个数。

邻接表：数组 & 链表，空间复杂度：`O(v + e)`，v 表示节点个数，e 表示边的个数。

> 邻接矩阵的优点：
>
> - 表达方式简单，易于理解；
> - 检查任意两个顶点间是否存在边的操作非常快；
> - 适合稠密图，在边数接近顶点数平方的图中，邻接矩阵是一种空间效率较高的表示方法。
>
> 缺点：
>
> - 遇到稀疏图，会导致申请过大的二维数组造成空间浪费 且遍历 边 的时候需要遍历整个n * n矩阵，造成时间浪费。
>
> 
>
> 邻接表的优点：
>
> - 对于稀疏图的存储，只需要存储边，空间利用率高；
> - 遍历节点连接情况相对容易，时间复杂度相对较低 n * v，n表示节点数量，v表示节点连接的边。
>
> 缺点：
>
> - 检查任意两个节点间是否存在边，效率相对低，需要 O(V)时间，V表示某节点连接其他节点的数量；
> - 实现相对复杂，不易理解。

## 11.2 图的遍历方式

> 图的遍历方式基本是两大类：
>
> - 深度优先搜索（dfs）；
> - 广度优先搜索（bfs）。
>
> 二叉树的递归遍历，是dfs 在二叉树上的遍历方式，二叉树的层序遍历，是bfs 在二叉树上的遍历方式，dfs 和 bfs 是一种搜索算法，可以在不同的数据结构上进行搜索。

### 11.2.1 dfs 和 bfs 模板题

适应一下 ACM 模式吧！！！

#### [KamaCoder 98](https://kamacoder.com/problempage.php?pid=1170)：所有可达路径

#### [KamaCoder 99 ](https://kamacoder.com/problempage.php?pid=1171)：岛屿问题

#### [KamCoder 99 ](https://kamacoder.com/problempage.php?pid=1171)：岛屿问题

#### [KamaCoder 100](https://kamacoder.com/problempage.php?pid=1172) 岛屿的最大面积

#### [KamaCoder 101](https://kamacoder.com/problempage.php?pid=1173) 孤岛的总和

#### [KamaCoder 102](https://kamacoder.com/problempage.php?pid=1174)：沉没孤岛

#### [KamaCoder 103](https://kamacoder.com/problempage.php?pid=1175)：水流问题

#### [KamaCoder 104](https://kamacoder.com/problempage.php?pid=1176)：建造最大岛屿

#### [KamaCoder 106](https://kamacoder.com/problempage.php?pid=1178)：岛屿的周长

本题目属于简单的遍历题目，不需要用到 dfs 和 bfs

#### [KamaCoder 110](https://kamacoder.com/problempage.php?pid=1183)：字符串接龙

> 阶段总结：
>
> 无向图的最短路径：直接用 dfs 或 bfs 即可；
>
> 有向图的最短路径：用最短路径算法；
>
> 一般地，在 dfs 和 bfs 的过程中，需要 visited 数组做辅助，防止往回搜索的情况（死循环），这也是 dfs 和 bfs 模板中不可缺少的一部分。

#### [KamaCoder 105](https://kamacoder.com/problempage.php?pid=1177)：有向图的完全联通

## 11.3 并查集相关

**背景**

> 并查集常用来解决连通性问题，当需要判断两个元素是否在同一个集合里的时候，就要想到用并查集。
>
> 并查集主要有两个功能：
>
> - 将两个元素添加到一个集合中；
> - 判断两个元素在不在同一个集合。
>
> <font color = red>并查集可以求出图的连通分量个数</font>。

**路径压缩算法**

由于对节点进行根节点查找的时候，存在较多的递归调用，可以采用路径压缩算法，使得查的过程时间复杂度优化到：O(1)。

#### [KamaCoder 107](https://kamacoder.com/problempage.php?pid=1179)：寻找存在的路径

#### [KamaCoder 108](https://kamacoder.com/problempage.php?pid=1181)：冗余的边

#### [KamaCoder 109](https://kamacoder.com/problempage.php?pid=1182)：冗余的边Ⅱ

## 11.4 最小生成树之 prim 算法

#### [KamaCoder 53](https://kamacoder.com/problempage.php?pid=1053)：寻宝

该题目是最小生成树算法的模板题。

> ​    算法三部曲：
> ​    1. 建议邻接矩阵存储无向图的信息，定义 minDist 数组存放非生成树节点到生成树节点的最小距离，定义 visited 数组存放生成树节点和非生成树节点；
> ​        - minDist 数组都初始化为 INT_MAX 值，visited 数组都初始化为 false，表示此时生成树中没有节点；
> ​    2. 根据 visited 数组和 minDist 数组，从非生成树节点中选择一个距离生成树最近的节点，加入到生成树中；
> ​        - 将加入到生成树节点的 visited 值改为 true；
> ​        - 根据新加入的生成树节点，结合无向图边的信息，更新 minDist 数组（只需要遍历新加入生成树节点作为起始节点的边信息即可）；
> ​    3. 根据 minDist 数组， 计算出最小生成树的权值；
>
> 4. 拓展内容：如何存储最小生成树的每一条边信息。

## 11.5 最小生成树之 kruskal 算法

> kruskal 最小生成树算法（无向图）：对比 prim 算法，该算法更容易理解，步骤如下：
> - 维护边节点的信息，并且对边的权重进行排序；
>
> - 每一次都选择最小的权重边加入到最小生成树中，直到选择了 n-1 条最小的权重边连接了 n 个节点；
>   - 这里需要借助并查集来进行辅助，过滤掉同一个根节点的冗余边；
>
> - 根据选择的边节点信息，输出最小生成树的权重；
>
> - 拓展内容：kruskal 算法天然是基于边信息运行的，所以很容易输出最小生成树的边信息（需要用自定义数据类型存放边信息）；
>
> 时间复杂度：排序 O(e*loge)，并查集 O(loge)，其中 e 是边的条数。

## 11.6 最小生成树算法选择

如果节点多边少（稀疏图），选择 kruskal 算法即可，如果边多节点少（稠密图），选择 prim 算法即可。

> 两个算法代码量差不多，prim 是基于节点运行的算法（将节点分为生成树和非生成树两个部分），kruskal 算法是基于边运行的算法。

## 11.7 拓扑排序

#### [KamaCoder 117](https://kamacoder.com/problempage.php?pid=1191)：软件构建

拓扑排序常见背景：

> 拓扑排序在文件处理上有应用，在做项目安装文件包的时候，经常发现复杂的文件依赖关系， A依赖B，B依赖C，B依赖D，C依赖E 等等。给出一条线性的依赖顺序来下载这些文件，就是拓扑排序。
>
> - 这一种线性依赖构成了有向无环图，拓扑排序就是如何将有向无环图中的所有节点选出来，每一次选择都只能选入度为 0 的节点；
> - 在拓扑排序算法执行的过程中，如果不能选择出所有节点，说明该图是一个**有向无环图**。

拓扑排序基本思路：

> **利用 BFS 来实现**：
>
> - 找到入度为 0 的节点，加入到 BFS 队列中，并且删除和该节点有关的边的信息；
>
> - 循环上一步，继续寻找入度为 0 的节点，加入到 BFS 队列中；
>
> - BFS 队列中即是需要拓扑排序的结果。

## 11.8 Dijkstra 算法

> - dijkstra 算法可以同时求 起点到所有节点的最短路径；
> - 权值不能为负数；

#### [KamaCoder 47](https://kamacoder.com/problempage.php?pid=1047)：参加科学大会

# 第十二章 Hot100补充

## 12.1 哈希

### [LC 1](https://leetcode.cn/problems/two-sum/?envType=study-plan-v2&envId=top-100-liked)：两数之和

> 暴力法：时间复杂度 O(n^2)，每一次遍历到当前元素都去寻找其它元素是否能拼凑成 target；
>
> 哈希法：时间复杂度 O(n)，本质思路还是和暴力法一样，只是借助哈希表的查找为O(1)，所以时间复杂度降了，但是空间复杂度变高了。

### [LC 49](https://leetcode.cn/problems/group-anagrams/description/?envType=study-plan-v2&envId=top-100-liked)：字母异位词

用到了哈希，但是感觉还是暴力法，本质就是通过对每一个单词进行字典序的排序，将排序的结果作为构建哈希的键值，将单词字典序排序的结果为判断，结果相同的分到同一个字符串数组中。

### [LC 128](https://leetcode.cn/problems/longest-consecutive-sequence/description/?envType=study-plan-v2&envId=top-100-liked)：最长连续序列

> 第一时间想到暴力法，排序然后找最长子序列，但是时间复杂度为 O(n*logn)，不符合题目要求；
>
> 优化：借助哈希表存储数组的值，然后遍历哈希表，因为哈希表的查找为O(1)，所以时间复杂度降为O(n)了，具体如下：
>
> - 遍历哈希表，当前值为 num
>   - 如果不能在哈希表找到 num-1 的值，说明肯定不是最长连续序列，不做操作；
>   - 如果能在哈希表找到 num-1 的值，说明当前 num 可能是最长连续序列的开始位置，往后遍历，求出以 num 为起始位置的最长连续序列即可；
>
> 分析优化后算法的时间复杂度，连续的数字部分被访问了2次，其它的数字被访问了1次，整体的时间复杂度为 O(n)，空间复杂度为 O(n)。
>
> 需要注意题目的数字去重即可，不过 unordered_set 自动对数字进行了去重。

## 12.2 双指针

### [LC 283](https://leetcode.cn/problems/move-zeroes/description/?envType=study-plan-v2&envId=top-100-liked)：移动零

### [LC 42](https://leetcode.cn/problems/trapping-rain-water/description/?envType=study-plan-v2&envId=top-100-liked)：接雨水
