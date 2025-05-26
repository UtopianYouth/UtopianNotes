# 字节跳动抖音基础技术部二面复盘

## 自我介绍

自我介绍非常重要。

## 多任务 RHEED 图像分析方法研究

因为面试的岗位是客户端，所以和一面不同的是，这次直接从客户端入手。

**问题1：**项目用到了 MVVM 模式，可以解释一下为什么使用 MVVM 模式吗，顺便聊一下为什么不使用如 MVC 等模式，有对比过差异吗？



**问题2：**关于设计模式，了解过哪些（好像除了工厂者模式，其它都不会）？



**问题3：**从项目2入手，简单说一下这个项目的主要逻辑是什么吧？（项目2 需要去详细的复盘一下）



**问题4：**基于问题 4，说到了数据抽取部分，为什么使用 XML 文件的格式，将从其它数据库服务器或开放性网站抽取的数据进行存储？



**问题5：**你说到了使用内存映射的方式，从磁盘读取 XML 文件到内存中，说一下内存映射和传统的文件 IO 在网络编程通信中有什么优势？



**问题6：**说一说内存映射的具体流程是怎么样的，对于 XML 大文件的内存映射，又是一个什么样的机制？



**问题7：**绕过了客户端和项目相关的问题，这次直接从 c++ 来做切入点，你了解 c++ 吗，顺便解释一下什么是多态吧？



**问题8：**基于问题 7 的深入，既然说到了虚函数表，那么，在构造函数中可以执行虚函数吗？



**问题9：**你知道 new 和 malloc 的区别吗？（这个问题不应该答不出，看过，但是没形成深刻的印象）



**问题10：**HTTP 和 HTTPS 的区别，客户端如何验证服务器端的 CA 证书是否有效？（这个问题真的是高频题目了）



**问题11：**了解什么是函数代理吗？



**问题12：**知道 c++ 的智能指针吗，对于智能指针  `shared_ptr<T>` 是线程安全的吗，如果不是，为什么？



## **手撕题目**

三个线程依次打印 1~100 的数字（如此简单的题目，但却由于 API 的调用问题，卡住并且不会了）。

```c++
#include<iostream>
#include<thread>
#include<mutex>
#include<condition_variable>
#include<vector>

using namespace std;

mutex mtx;
condition_variable cv;
int num = 1;
const int MAX_NUM = 100;
int cur_thread_id = 0;     // 轮换线程

void function(int thread_id) {
    while (1) {
        // 互斥锁，unique_lock 属于 RAII 资源，当前作用域执行完自动释放
        unique_lock<mutex> lock(mtx);
        // 条件变量，获得互斥锁的线程，会执行条件变量检查函数，当条件变量检查函数返回true时，阻塞解除
        cv.wait(lock, [thread_id]()->bool {
            return (num > MAX_NUM) || (cur_thread_id == thread_id);
            });

        if (num > MAX_NUM) {
            cv.notify_all();    // 通知所有线程退出，执行wait()函数
            break;
        }

        cout << "Thread id " << thread_id << ": " << num++ << endl;
        cur_thread_id = (cur_thread_id + 1) % 3;
        cv.notify_all();
    }
}

int main() {
    const int nums = 3;
    vector<thread> threads;

    for (int i = 0;i < nums;++i) {
        threads.emplace_back(function, i);
    }

    for (auto& thread : threads) {
        thread.join();      // 等待所有子线程结束
    }
  
    system("pause");
    return 0;
}
```

线程池

```c++

```

## 反问环节

问了一下关于学习的问题，也就是对于基础的知识和项目经验，面试官更看重哪一点？

忘记具体怎么回答的了，反正大概意思就是，这两者的学习不冲突，通过做项目遇到的难点，来对一个关键基础知识技术点进行详细拆解，关键是要学会理解其原理。

