[TOC]

# 第四章 Linux多线程开发

## 4.1 线程概述

> - 与进程类似，线程是**允许应用程序并发执行多个任务的一种机制**。一个进程可以包含多个线程，同一个程序中的所有线程**均会独立执行相同程序**，且共享同一份全局内存区域，其中包括**初始化数据段、未初始化数据段，以及堆内存段（传统意义上的 UNIX 进程只是多线程程序的一个特例，该进程只包含一个线程）**。
> - 进程是 OS 资源分配的最小单位，线程是 OS 调度执行的最小单位。
> - **线程是轻量级的进程（Light Weight Process, LWP）**，在 Linux 环境下，线程的本质仍是进程。
> - 查看指定进程的 LWP 号：`ps -Lf pid`

### 4.1.1 线程和进程的区别

> - 进程间的信息难以共享，由于除去只读代码段外，**父子进程并未共享内存**，因此**必须采用一些进程间通信方式**，在进程间进行信息交换。（第三章中，介绍了匿名管道、有名管道、内存映射和内存共享四种进程间通信方式）。
> - 调用`fork()`来创建进程的代价**相对较高**，即便利用读时共享、写时拷贝技术，仍然需要复制**诸如内存页表和文件描述符表之类的多种进程属性**。这意味着`fork()`调用在时间上的开销依然不菲。
> - 线程之间能够方便、快速地共享信息。**只需要将数据复制到共享（全局或堆）变量中即可**。
> - 创建线程比创建进程通常要快 10 倍甚至更多。**线程是共享虚拟地址空间的，无需采用写时复制来复制内存，也无需复制页表**。
> - 线程和进程的虚拟地址空间区别，如下图所示，**是三个进程的虚拟地址空间**，对于每一个进程中的线程，都是共享一个进程的虚拟地址空间。同一进程的不同的线程，都有自己的栈空间和`.text`段（代码段）。
>
> <center>
>   <img src = "./images/第四章 Linux多线程开发/4-1 线程和进程虚拟地址空间.png" width = "90%">
> </center>

### 4.1.2 线程之间共享和非共享资源

> <font color= green>共享资源</font>
>
> - 进程 ID 和父进程 ID
> - 进程组 ID 和会话 ID
> - 用户 ID 和用户组 ID
> - 文件描述符表
> - 信号处置
> - 文件系统的相关信息：文件权限掩码（`umask`）、当前工作目录
> - 虚拟地址空间（除栈、`.text`）
>
> <font color = green>非共享资源</font>
>
> - 线程 ID
> - 信号掩码
> - 线程特有数据
> - `error`变量（一般是函数调用，产生的错误号信息）
> - 实时调度策略和优先级
> - 栈，本地变量和函数的调用链接信息

## 4.2 NPTL

> - 当 Linux 最初开发时，在内核中并不能真正支持线程。但是可以通过 `clone()` 系统调用**将进程作为可调度的实体**。`clone()` 系统调用创建了调用进程（calling process）的一个拷贝，这个拷贝与调用进程共享相同的地址空间。`LinuxThreads` 项目使用的就是 `clone()` 系统调用，**在用户空间下模拟对线程的支持**。不幸的是，这种方法有一些缺点，尤其是在信号处理、调度和进程间同步等方面都存在问题。另外，这个线程模型也不符合 POSIX 的要求。
> - 要改进 `LinuxThreads`，需要内核的支持，并且重写线程库。这项工作由 Red Hat 的开发人员，开展的 <font color = green>NPTL</font> 项目完成。
> - <font color = green>NPTL</font>，全称为 Native POSIX Thread Library，是 Linux 线程的一个新实现，它克服了 `LinuxThreads` 的缺点，同时也符合 POSIX 的需求。与 `LinuxThreads` 相比，它在性能和稳定性方面都提供了重大的改进。
> - 查看当前 `pthread` 库版本：`getconf GNU_LIBPTHREAD_VERSION`

## 4.3 线程相关函数

> <font color = green>线程相关函数如下</font>
>
> ```c
> #include<pthread.h>
> 
> pthread_t pthread_self(void);
> int pthread_equal(pthread_t t1, pthread_t t2);
> int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg);
> void pthread_exit(void *retval);
> int pthread_join(pthread_t thread, void **retval);
> int pthread_detach(pthread_t thread);
> int pthread_cancel(pthread_t thread);
> 
> /*
> 		TIPS: 在链接包含 pthread 线程库的程序时，记得使用 gcc 命令
> 		的 -l 参数，并且指向 pthread 动态库，即 -lpthread
> */
> ```

### 4.3.1 `pthread_self(void)`和`pthread_equal()`

```c
/*
    pthread_t pthread_self(void);
    函数功能：获取调用线程的线程 ID
    返回值：线程 ID

    int pthread_equal(pthread_t t1, pthread_t t2);
    函数功能：比较两个线程 ID 是否相等
    函数参数
    - t1：比较的第一个线程 ID
    - t2：比较的第二个线程 ID
    返回值
    - 相等：非零的值
    - 不相等：0
    TIPS: 不同的 OS，pthread_t 类型的实现不一样，有的是无符号长整数，有的是结构体，所以不能直接使用 == 比较
*/
```

### 4.3.2 使用`pthread_create()`系统调用创建子线程

```c
/*
    一般情况下，main函数所在的线程，我们称之为主线程，其余创建的线程称之为子线程

    int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg);
    函数功能：创建一个子线程（Linux中，进程默认对应一个线程，创建子线程的叫做主线程）
    函数参数
    - thread：传出参数，线程创建成功后，子线程的线程 ID 被写到该变量
    - attr：设置线程属性，一般使用默认值，NULL
    - start_routine：函数指针，这个函数是子线程需要处理的逻辑代码
    - arg：给第三个参数使用，传参
    返回值
    - 成功：返回0
    - 失败：返回错误号，这个错误号和之前的 errno 有区别
    获取错误号的信息：char* strerror(int errnum)
*/
#include<stdio.h>
#include<pthread.h>
#include<string.h>  
#include<stdlib.h>
#include<unistd.h>

void* my_callback(void* arg) {
    printf("child pthread...\n");
    printf("child pthread arg: %d\n", *(int*)arg);

    // 返回值是void* 类型
    return NULL;
}

int main() {

    pthread_t tid;
    int num = 10;

    // 创建一个子线程
    int ret = pthread_create(&tid, NULL, my_callback, (void*)&num);

    // 创建子线程失败，返回一个错误号
    if (ret != 0) {
        char* str = strerror(ret);
        exit(0);
    }

    // 主线程执行代码
    for (int i = 0;i < 5;++i) {
        printf("%d\n", i);
    }
    sleep(1);

    return 0;
}
```

### 4.3.3 使用`pthread_exit()`终止一个线程的运行

```c
/*
    void pthread_exit(void *retval);
    函数功能：终止调用该函数的线程，终止的线程，在同一进程中的其他线程，可以通过 pthread_join(3) 加入
    函数参数
    - retval：传出参数，指针类型，作为一个返回值，可以在 pthread_join() 中获取到
*/

#include<stdio.h>
#include<pthread.h>
#include<string.h>
#include<stdlib.h>

void* callback(void* arg) {
    printf("child thread id = %ld\n", pthread_self());
    return NULL;
}

int main(int argc, char* argv[]) {

    // 创建一个子线程
    pthread_t tid;

    int ret = pthread_create(&tid, NULL, callback, NULL);

    if (ret != 0) {
        char* error = strerror(ret);
        exit(0);
    }

    for (int i = 0;i < 5;++i) {
        printf("%d\n", i);
    }
    printf("parent thread id = %ld, child thread id = %ld\n", pthread_self(), tid);

    // 让主线程退出，主线程退出不会影响其他线程的运行
    pthread_exit(NULL);

    printf("parent thread...\n");
    return 0;
}
```

### 4.3.4 使用`pthread_join()`和一个已经终止的线程进行连接

```c
/*
    #include <pthread.h>
    int pthread_join(pthread_t thread, void **retval);
    函数功能：和一个已经终止的线程进行连接，回收子线程资源
    - 该函数是阻塞函数，调用一次只能回收一个子线程
    - 一般在主线程中使用
    函数参数
    - thread：需要回收子线程的 ID
    - retval：接收子线程退出时的返回值，使用二级指针的原因是，pthread_exit()返回的子线程状态值是一级指针变量，作为传出参数
    		- 需要得到一级指针的地址，才能对栈空间中的一级指针变量进行赋值修改（和使用自定义函数改变main函数中的局部变量原理一致）
    返回值
    - 成功：0
    - 失败：非 0 值，即错误号
*/

#include<stdio.h>
#include<pthread.h>
#include<string.h>
#include<stdlib.h>
#include<unistd.h>

int child_thread;
void* callback(void* arg) {
    printf("child thread id = %ld\n", pthread_self());
    sleep(2);
    child_thread = 8;
    pthread_exit((void*)&child_thread);     // 利用全局变量记录线程函数栈空间的内容

    //return NULL;    //等价于 pthread_exit(NULL);
}

int main(int argc, char* argv[]) {

    // 创建一个子线程
    pthread_t tid;

    int ret = pthread_create(&tid, NULL, callback, NULL);

    if (ret != 0) {
        char* error = strerror(ret);
        printf("%s\n", error);
        exit(0);
    }

    for (int i = 0;i < 5;++i) {
        printf("%d\n", i);
    }

    printf("parent thread id = %ld, child thread id = %ld\n", pthread_self(), tid);

    // 回收子线程资源
    int* ret_value;
  
  	// 地址传递，才能改变 ret_value 的值
    int is_join = pthread_join(tid, (void**)&ret_value);
    if (is_join != 0) {
        char* error = strerror(is_join);
        printf("%s\n", error);
    }

    printf("recycle child thread success, recycle value = %d\n", *ret_value);

    // 让主线程退出
    pthread_exit(NULL);
    return 0;
}
```

### 4.3.5 使用`pthread_detach()`进行线程分离

```c
/*
    #include <pthread.h>
    int pthread_detach(pthread_t thread);
    函数功能：分离一个线程
    - 被分离的线程运行结束后，资源会被 OS 自动回收，无需其他线程调用 pthread_join()
    - 不能通过 pthread_join() 函数去连接一个已经分离的线程
    函数参数
    - thread：需要分离的线程 ID
    返回值
    - 成功：0
    - 失败：非 0 值，错误号
*/
#include<stdio.h>
#include<pthread.h>
#include<string.h>
#include<stdlib.h>
#include<unistd.h>

void* callback(void* arg) {
    printf("child thread id = %ld\n", pthread_self());
    sleep(2);
    pthread_exit(NULL);     // 利用全局变量记录线程函数栈空间的内容

    //return NULL;    //等价于 pthread_exit(NULL);
}

int main(int argc, char* argv[]) {

    // 创建一个子线程
    pthread_t tid;
    int ret = pthread_create(&tid, NULL, callback, NULL);
    if (ret != 0) {
        char* create_error = strerror(ret);
        printf("create error: %s\n", create_error);
        exit(0);
    }

    // 输出主线程和子线程的ID
    printf("child thread id = %ld, main thread id = %ld\n", tid, pthread_self());

    // 设置子线程分离，分离后，子线程对应的资源就不需要主线程释放
    ret = pthread_detach(tid);
    if (ret != 0) {
        char* detach_error = strerror(ret);
        printf("detach error: %s\n", detach_error);
    }

    // 对分离的子线程进行连接
    ret = pthread_join(tid, NULL);
    if (ret != 0) {
        char* join_error = strerror(ret);
        printf("join error: %s\n", join_error);
    }

    // 退出主线程
    pthread_exit(NULL);

    return 0;
}
```

### 4.3.6 使用`pthread_cancel()`取消线程，让线程终止运行

```c
/*
    #include <pthread.h>
    int pthread_cancel(pthread_t thread);

    函数功能：取消线程（让线程终止运行）
    - 被指定线程取消的线程，并不会立马终止运行，而是执行到一个 pthread 指定的 cancellation point，线程才会终止
    - cancellation point：可以理解为系调用（线程进入内核态）
    函数参数
    - thread：需要进行线程取消的线程 ID
    返回值：
    - 成功：0
    - 失败：非 0 值，错误号
*/

#include<stdio.h>
#include<pthread.h>
#include<string.h>
#include<stdlib.h>
#include<unistd.h>

void* callback(void* arg) {
    for(int i = 0; i < 5; ++i){
      	// 如果主线程执行了 pthread_cancel()，会执行 cancellation point 系统调用，进入 OS 内核态
        printf("child thread: %d\n",i);
    }
    return NULL;    //等价于 pthread_exit(NULL);
}

int main(int argc, char* argv[]) {
    // 创建一个子线程
    pthread_t tid;
    int ret = pthread_create(&tid, NULL, callback, NULL);
    if (ret != 0) {
        char* create_error = strerror(ret);
        printf("create error: %s\n", create_error);
        exit(0);
    }

    // 取消线程
    pthread_cancel(tid);    

    // 输出主线程 ID
    for(int i=0;i<5;++i){
        printf("main thread: %d\n",i);
    }

    // 退出主线程
    pthread_exit(NULL);
    return 0;
}
```

## 4.4 线程属性相关函数

线程属性相关函数如下：

```c
#include <pthread.h>
// 线程属性类型：pthread_attr_t 
int pthread_attr_init(pthread_attr_t *attr);
int pthread_attr_destroy(pthread_attr_t *attr);
int pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate);
int pthread_attr_getdetachstate(const pthread_attr_t *attr, int *detachstate);
// ...还有很多，这里只举例部分 API
```

### 4.4.1 线程属性相关函数解析

```c
#include <pthread.h>
/*
    函数功能：初始化线程属性变量
    函数参数
    - attr：传出参数，线程属性变量
    返回值
    - 成功：0
    - 失败：非 0 错误号
*/
int pthread_attr_init(pthread_attr_t *attr);

/*
		函数功能：释放线程属性的资源
    函数参数
    - attr：传出参数，线程属性变量
    返回值
    - 成功：0
    - 失败：非 0 的错误号
*/
int pthread_attr_destroy(pthread_attr_t *attr);

/*
    函数功能：获取线程分离的状态属性
    函数参数：
    - attr：线程属性对象
    - detachstate：存储获取到的线程状态，常见的两种：PTHREAD_CREATE_JOINABLE, PTHREAD_CREATE_DETACHED
    返回值
    - 成功：0
    - 失败：非 0 的错误号
*/
int pthread_attr_getdetachstate(const pthread_attr_t *attr, int *detachstate);

/*
    函数功能：设置线程分离的状态属性
    函数参数
    - attr：线程属性对象
    - detachstate：创建线程之前，需要设置的线程分离状态（属于线程的属性之一），常见的两种：PTHREAD_CREATE_JOINABLE, PTHREAD_CREATE_DETACHED
    返回值
    - 成功：0
    - 失败：非 0 的错误号
*/
int pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate);
```

### 4.4.2 线程属性相关函数使用案例

```c
#include<stdio.h>
#include<pthread.h>
#include<string.h>
#include<stdlib.h>
#include<unistd.h>

void* callback(void* arg) {
    printf("child thread id = %ld\n", pthread_self());
    return NULL;    //等价于 pthread_exit(NULL);
}

int main(int argc, char* argv[]) {

    // 创建一个线程属性变量
    pthread_attr_t attr;

    // 初始化线程属性变量
    pthread_attr_init(&attr);
  
    // 设置线程属性
    pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);

    // 一般地，在创建线程之前，初始化和设置线程属性
    pthread_t tid;
    int ret1 = pthread_create(&tid, &attr, callback, NULL);
    if (ret1 != 0) {
        char* create_error = strerror(ret1);
        printf("create error: %s\n", create_error);
        exit(0);
    }

    // 获取子线程栈空间的大小
    size_t size;
    int ret2 = pthread_attr_getstacksize(&attr, &size);
    if (ret2 != 0) {
        char* get_stack_attr_error = strerror(ret2);
        printf("get_stack_attr_error: %s\n", get_stack_attr_error);
    }
    else {
        printf("child thread stack size = %ld\n", (long long)size);
    }

    // 获取子线程的线程分离属性
    int val = 0;
    int ret3 = pthread_attr_getdetachstate(&attr, &val);
    if (ret3 != 0) {
        char* get_detach_attr_error = strerror(ret3);
        printf("get_detach_attr_error: %s\n", get_detach_attr_error);
    }
    else {
        printf("child thread detach_attr: %d\n", val);
    }

    // 释放线程属性资源
    pthread_attr_destroy(&attr);

    // 退出主线程
    pthread_exit(NULL);

    return 0;
}
```

## 4.5 线程同步机制

> 线程的主要优势在于，**能够通过全局变量来共享信息**。不过，<font color = green>这种便捷的共享是有代价的</font>：
>
> - 必须确保多个线程不会同时修改同一变量，或者某一线程**不会读取正在由其他线程修改的变量**。
>
> <font color = green>临界资源和临界区</font>
>
> - 临界资源是指在某一时刻，**只允许一个线程访问并且对其进行操作的资源**。
> - 临界区是指访问某一共享资源的**代码片段**，**并且这段代码的执行应为原子操作**，也就是同时访问共享资源的**其他线程**，**不能终止**临界区对应代码片段的执行。
>
> <font color = green>线程同步</font>
>
> - 当有一个线程在对内存进行操作时，**其他线程都不可以对这个内存地址进行操作**，处于等待状态。直到该线程完成操作，其他线程才能对该内存地址进行操作。
> - UNIX OS 提供了**多种线程同步机制**
>   - 互斥量
>   - 读写锁
>   - 条件变量
>   - 信号量

## 4.6 互斥量

在 UNIX 内核中，使用了**互斥量机制解决线程同步问题**。

> <font color = green>互斥量</font>
>
> - 为避免线程更新共享变量时出现问题，可以使用**互斥量（mutex 是 mutual exclusion 的缩写）**来确保**同时只有一个线程可以访问某项共享资源**。同时，也可以使用互斥量来保证对任意共享资源的原子访问。
>- 互斥量有两种状态：**已锁定（locked）和未锁定（unlocked）**。任何时候，**至多只有一个线程可以锁定该互斥量**。试图对已经锁定的某一互斥量再次加锁，**会阻塞线程或者报错失败**，具体取决于加锁时使用的方法。
> - 一旦线程锁定互斥量，就成为了**该互斥量的所有者**，只有互斥量所有者**才能给互斥量解锁**。一般情况下，对每一个共享资源（可能由多个相关变量组成），**会分别使用不同的互斥量**，每一个线程在访问同一共享资源的时候，**将采用如下协议**：
>   - 针对共享资源锁定互斥量
>   - 访问共享资源
>   - 对互斥量解锁

### 4.6.1 互斥量相关函数

```c
#include<pthread.h>
/*
		互斥量：顾名思义，就是同一时刻只能有一个线程访问的量
		互斥量类型：pthread_mutex_t
*/

/*
    函数功能：初始化互斥量
    函数参数
    - mutex：需要初始化的互斥量对象
    - mutexattr：初始化的互斥量属性，一般默认使用NULL即可
    返回值
    - 只有 0，不管是成功还是失败
    TIPS
    - restrict：C语言的关键字（修饰符），被 restrict 修饰的指针所指向的内存，不能被另一个指针操作
*/
int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *mutexattr);

/*
		函数功能：释放互斥量资源
    函数参数
    - mutex：需要释放的互斥量对象
    返回值
    - 成功：0
    - 失败：返回 EBUSY 错误号，说明当前互斥量已经被上锁，无法释放
*/
int pthread_mutex_destroy(pthread_mutex_t *mutex);

/*
    函数功能：加锁，会造成阻塞，如果有一个线程加锁了，那么其他线程只能阻塞等待
    函数参数
    - mutex：需要进行加锁操作的互斥量
    返回值
    - 成功：0
    - 失败：返回错误号
        - EINVAL：mutex 没有被初始化
        - EDEADLK：mutex 已经被其他线程进行了上锁	
*/
int pthread_mutex_lock(pthread_mutex_t *mutex);

/*
		函数功能：尝试加锁，如果加锁失败，不会阻塞，会直接返回
    函数参数
    - mutex：尝试加锁的互斥量
    返回值
    - 成功：0
    - 失败：返回错误号
        - EINVAL：mutex 没有被初始化
        - EBUSY：mutex 当前已经被上锁，无法加锁
*/
int pthread_mutex_trylock(pthread_mutex_t *mutex);

/*
    函数功能：对互斥量进行解锁
    函数参数
    - mutex：需要进行解锁的互斥量
    返回值
    - 成功：0
    - 失败：返回错误号
    		- EINVAL：mutex 没有被初始化
    		- EPERM：调用解锁的线程没有 mutex 互斥量

    TIPS：加锁和解锁，都是互斥量的一种状态转换
*/
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

### 4.6.2 互斥量实现多窗口售票（线程同步问题）

> 首先，我们要知道多窗口售票的**基本需求**
>
> - 多窗口相当于多线程。
> - 票的总数相当于**共享资源**，并且**线程对这个共享资源的访问是原子性的**。
>
> <font color = green>多窗口售票实现基本思路</font>
>
> - 在主线程中创建多个子线程
> - 初始化共享资源（这里也就是初始化票的总数）
> - 在子线程中，使用互斥量，访问共享资源（原子操作，这里也就是访问存储了票总数的变量）

```c
#include<stdio.h>
#include<pthread.h>
#include<string.h>
#include<unistd.h>

// 全局变量（存放在静态存储区），票的总数，相对于多线程，属于共享资源
int tickets = 20;

// 创建互斥量
pthread_mutex_t mutex;

void* sell_tickets(void* arg) {
    while (1) {

        // 对临界资源的访问加锁
        pthread_mutex_lock(&mutex);

        if (tickets > 0) {
            printf("thread id = %ld, selling tickets num is %d.\n", pthread_self(), 20 - tickets + 1);
            --tickets;
        }
        else {
            // 访问完毕解锁
            pthread_mutex_unlock(&mutex);
            break;
        }

        // 访问完毕解锁
        pthread_mutex_unlock(&mutex);

        // microseconds
        //usleep(6000);
        sleep(1);
    }
    return NULL;
}


int main(int argc, char* argv[]) {

    // 初始化互斥量
    pthread_mutex_init(&mutex, NULL);

    for (int i = 0;i < 3;++i) {
        pthread_t tid;

        // 创建线程属性
        pthread_attr_t attr;

        // 初始化和设置线程分离状态
        pthread_attr_init(&attr);
        pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);

        // 创建线程
        pthread_create(&tid, &attr, sell_tickets, NULL);

        // 销毁线程属性
        pthread_attr_destroy(&attr);

    }

    // 主线程退出
    pthread_exit(NULL);

    // 释放互斥量
    pthread_mutex_destroy(&mutex);

    return 0;
}
```

## 4.7 死锁

> <font color = green>死锁的概念</font>，是在**多线程访问多个共享资源，而多个共享资源需要多个互斥量**的前提下定义的。
>
> - 有时，一个线程需要**同时访问两个或更多不同的共享资源**，而每个共享资源又都由不同的互斥量管理。**当超过一个线程加锁<font color = green>同一组互斥量</font>时**，就有可能发生死锁（也有可能不发生死锁）。
> - 两个或两个以上的线程在执行过程中，**因争夺共享资源而造成的一种相互等待现象**，若无外力作用，它们都将无法推进下去。此时称，系统处于死锁状态或系统产生了死锁。
> - 死锁的几种场景：
>   - 忘记释放锁
>   - 重复加锁
>   - 多线程多锁，抢占锁资源

### 4.7.1 死锁案例

> 从上述对死锁的介绍，死锁出现有三种情况
>
> - 忘记释放锁
> - 重复加锁
> - 多线程多锁，抢占锁资源
>
> 前两种情况比较好理解，这里使用代码实现一下**多线程多锁，抢占锁资源的情况（即多个线程访问多个共享资源）**。
>
> ```c
> #include<stdio.h>
> #include<unistd.h>
> #include<pthread.h>
> #include<string.h>
> 
> 
> // 创建两个共享资源
> int share1 = 1;
> int share2 = 2;
> 
> // 创建两个互斥量
> pthread_mutex_t mutex1;
> pthread_mutex_t mutex2;
> 
> 
> // 子线程 A 逻辑
> void* workA(void* arg) {
>     // 假设该子线程先访问 share1，再访问 share2 共享资源
>     pthread_mutex_lock(&mutex1);
>     printf("share1 = %d\n", share1);
>     sleep(1);   // 阻塞 1s ，防止同时获得两个锁
> 
>     pthread_mutex_lock(&mutex2);
>     printf("share2 = %d\n", share2);
> 
>     // 解锁互斥量
>     pthread_mutex_unlock(&mutex2);
>     pthread_mutex_unlock(&mutex1);
> 
>     // 子线程退出
>     return NULL;
> }
> 
> // 子线程 B 逻辑
> void* workB(void* arg) {
> 
>     // 假设该子线程先访问 share2，再访问 share1 共享资源
>     pthread_mutex_lock(&mutex2);
>     printf("share2 = %d\n", share2);
>     sleep(1);   // 阻塞 1s ，防止同时获得两个锁
> 
>     pthread_mutex_lock(&mutex1);
>     printf("share1 = %d\n", share1);
> 
>     // 解锁互斥量
>     pthread_mutex_unlock(&mutex1);
>     pthread_mutex_unlock(&mutex2);
> 
>     // 子线程退出
>     return NULL;
> }
> 
> int main(int argc, char* argv[]) {
>     // 初始化互斥量
>     pthread_mutex_init(&mutex1, NULL);
>     pthread_mutex_init(&mutex2, NULL);
> 
>     // 创建线程属性
>     pthread_attr_t attr;
>     pthread_attr_init(&attr);
> 
>     // 设置线程属性
>     pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);
> 
>     // 创建两个线程
>     pthread_t tid1, tid2;
>     pthread_create(&tid1, &attr, workA, NULL);
>     pthread_create(&tid2, &attr, workB, NULL);
> 
>     // 释放线程属性
>     pthread_attr_destroy(&attr);
> 
>     // 主线程退出
>     pthread_exit(NULL);
> 
>     // 释放互斥量
>     pthread_mutex_destroy(&mutex1);
>     pthread_mutex_destroy(&mutex2);
> 
>     return 0;
> }
> ```
>
> 上述程序，就是**多线程多锁，抢占锁资源的情况**，如下图：
>
> <center>
>   <img src = "./images/第四章 Linux多线程开发/4-10 死锁1.png" width = "70%">
> </center>
>
>
> - 对于子线程 A 拿到了 `mutex1` 锁，而当想要再次去拿 `mutex2` 锁的时候，**`mutex2` 锁已经被子线程 B 拿到**，所以子线程 A 被阻塞。
> - 同理，对于子线程 B 拿到了 `mutex2` 锁，而当想要再次去拿 `mutex1` 锁的时候，**`mutex2` 锁已经被子线程 A 拿到**，所以子线程 B 也被阻塞。
>
> 两个阻塞的子线程，若无外力作用，都将无法执行后续的逻辑代码。

## 4.8 读写锁

> - 当一个线程已经持有互斥锁时，**互斥锁将所有试图进入临界区（共享资源）的线程都阻塞**。考虑一种情形，当前持有互斥锁的线程只是要读访问共享资源，而同时有其它几个线程也想读这个共享资源，**但是由于互斥锁的排他性，所有其它线程都无法获取锁，也就无法读访问共享资源了**。实际上，**多个线程同时读访问共享资源并不会导致问题**。
> - 对数据的读写操作中，**更多的是读操作**，写操作较少，例如对数据库的读写应用。为了满足**多线程可以同时读共享资源（不互斥），但只允许一个线程写共享资源（互斥）**，UNIX 提供了读写锁来实现。
> - <font color = green>读写锁的特点：</font>
>   - 如果有其它线程**读共享资源**，则**允许其它线程执行读操作，但不允许写操作**。
>   - 如果有其它线程**写共享资源**，则其它线程**不允许读、写操作**。
>   - 写是独占的（原子性，不允许中断），**写的优先级高**。

### 4.8.1 读写锁相关函数

```c
#include <pthread.h>
// 读写锁类型：pthread_rwlock_t

/*
		函数功能：初始化一个读写锁
		函数参数
		- rwlock：指针变量，指向需要初始化的读写锁变量
		- attr：初始化读写锁的属性，NULL即可，表示默认初始化
		返回值
		- 成功：0
		- 失败：非 0 值，表示错误号，可以通过 strerror() 得到
*/
int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock, const pthread_rwlockattr_t *restrict attr);

/*
		函数功能：释放读写锁资源
		函数参数
		- rwlock：指针变量，指向需要释放的读写锁变量
		返回值
    - 成功：0
		- 失败：非 0 值，表示错误号，可以通过 strerror() 得到
*/
int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);

/*
		函数功能：获取读写锁的读锁，允许多个线程同时读共享资源，阻塞写共享资源
		- 如果有线程在对共享资源进行写操作，那么该函数还是会阻塞
		函数参数
		- rwlock：指针变量，指向需要获取读锁的读写锁变量
		返回值
		- 成功：0
		- 失败：非 0 值，表示错误号，可以通过 strerror() 得到
*/
int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);

/*
		函数功能：尝试获取读写锁的读锁
		- 与 pthread_rwlock_rdlock() 不同的是，当读写锁的读锁不可用的时候，该函数不会阻塞，而是立即返回
		函数参数
		- rwlock：指针变量，指向需要尝试获取读锁的读写锁变量
    返回值
		- 成功：0
		- 失败：非 0 值，表示错误号，可以通过 strerror() 得到
*/
int pthread_rwlock_tryrdlock(pthread_rwlock_t *rwlock);

/*
		函数功能：获取读写锁的写锁，不允许其它线程对共享资源进行读、写
		- 如果有线程在对共享资源进行读或者写，该函数会阻塞
		函数参数
		- rwlock：指针变量，指向需要获取写锁的读写锁变量
		返回值
		- 成功：0
		- 失败：非 0 值，表示错误号，可以通过 strerror() 得到
*/
int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);

/*
		函数功能：尝试获取读写锁的写锁，不允许其它线程对共享资源进行读、写
		- 与 pthread_rwlock_wrlock() 不同的是，当读写锁的写锁不可用的时候，该函数不会阻塞，而是立即返回
		函数参数
		- rwlock：指针变量，指向需要获取写锁的读写锁变量
		返回值
		- 成功：0
		- 失败：非 0 值，表示错误号，可以通过 strerror() 得到
*/
int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock);

/*
		函数功能：对读写锁进行解锁
		函数参数
		- rwlock：指针变量，指向需要解锁的读写锁变量
		返回值
		- 成功：0
		- 失败：非 0 值，表示错误号，可以通过 strerror() 得到		
*/
int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);
```

### 4.8.2 读写锁案例

> <font color = green>读写锁案例</font>：创建八个线程，操作同一个全局变量（共享资源）。
>
> - 3 个线程写全局变量。
> - 8 个线程读全局变量。

```c
#include<stdio.h>
#include<unistd.h>
#include<pthread.h>
#include<string.h>
#include<stdlib.h>

// 全局变量（共享资源）
int global_variable = 100;

// 定义读写锁
pthread_rwlock_t rwlock;

// 读全局变量子线程
void* read_global(void* arg) {

    sleep(1);

    // 获得读写锁的读锁，阻塞写共享资源的线程
    pthread_rwlock_rdlock(&rwlock);

    printf("read global variable = %d.\n", global_variable);

    pthread_rwlock_unlock(&rwlock);

    return NULL;
}

// 写全局变量子线程
void* write_global(void* arg) {

    // 获得读写锁的写锁，阻塞读写共享资源的线程
    pthread_rwlock_wrlock(&rwlock);

    ++global_variable;
    printf("write global variable = %d.\n", global_variable);

    pthread_rwlock_unlock(&rwlock);

    return NULL;
}

int main(int argc, char* argv[]) {
    // 初始化读写锁
    pthread_rwlock_init(&rwlock, NULL);

    // 初始化线程属性
    pthread_attr_t attr;
    pthread_attr_init(&attr);

    // 设置线程分离
    pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);

    // 创建三个读全局变量子线程
    for (int i = 0;i < 3;++i) {
        pthread_t tid;
        pthread_create(&tid, &attr, read_global, NULL);
    }

    // 创建五个写全局变量子线程
    for (int i = 0;i < 5;++i) {
        pthread_t tid;
        pthread_create(&tid, &attr, write_global, NULL);
    }

    // 主线程退出
    pthread_exit(NULL);

    // 释放读写锁资源
    pthread_rwlock_destroy(&rwlock);

    return 0;
}
```

## 4.9 生产者消费者模型

> **使用互斥量**，实现**粗略版**的生产者消费者模型
>
> <font color = green>操作对象</font>
>
> - 生产者
> - 容器（可以选择是否限制容量）
>   - 不限制容量，**使用链表实现容器，互斥量限制生产和消费的原子性**
>   - 限制容器，**在 4.10 节信号量中实现**
> - 消费者

```c
#include<stdio.h>
#include<pthread.h>
#include<unistd.h>
#include<stdlib.h>

// 链表作为容器
typedef struct Node {
    int data;
    struct Node* next;
}Node;

// 创建头节点
Node* create_head() {
    Node* head = (Node*)malloc(sizeof(Node));
    head->data = -1;
    head->next = NULL;
    return head;
}

// 链表容器，全局变量（共享资源）
Node* head = NULL;

// 创建互斥量
pthread_mutex_t mutex;

void* producer(void* arg) {
    // 生产者不断的创建新的节点，添加到链表中
    while (1) {
        sleep(1);

        pthread_mutex_lock(&mutex);
        Node* new_node = (Node*)malloc(sizeof(Node));
        new_node->data = random() % 1000;
        new_node->next = head->next;
        head->next = new_node;
        printf("producer p_tid = %ld, adding node num = %d\n", pthread_self(), new_node->data);
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

void* consumer(void* arg) {
    // 消费者不断从链表头部取节点
    while (1) {
        sleep(1);

        pthread_mutex_lock(&mutex);
        if (head->next != NULL) {
            Node* tmp = head->next;
            head->next = head->next->next;
            printf("consumer c_tid = %ld, deleting node num = %d\n", pthread_self(), tmp->data);
            free(tmp);
        }
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

int main() {
    // 初始化互斥量
    pthread_mutex_init(&mutex, NULL);

    // 初始化链表头节点
    head = create_head();

    // 创建5个生产者线程，5个消费者线程
    pthread_t p_tids[5], c_tids[5];
    for (int i = 0;i < 5;++i) {
        pthread_create(p_tids + i, NULL, producer, NULL);
        pthread_create(c_tids + i, NULL, consumer, NULL);
    }

    // 线程分离
    for (int i = 0;i < 5;++i) {
        pthread_detach(p_tids[i]);
        pthread_detach(c_tids[i]);
    }

    // 保证主线程不退出
    while (1) {
        sleep(10);
    }

    // 释放互斥量
    pthread_mutex_destroy(&mutex);

    pthread_exit(NULL);
    return 0;
}
```

## 4.10 条件变量

> 在 4.8 节中实现的生产者消费者模型，<font color = green>存在一定的问题</font>：
>
> - 对于消费者线程（即`void* consumer(void* arg)`函数），**如果链表中没有数据**，程序也会一直执行下去，不断地执行`while(1){}`循环，造成 CPU资源的浪费。
>
> 针对 4.8 节出现的 CPU 资源浪费，UNIX 内核中，**提供了条件变量来解决**。
>
> -  **条件变量不是锁**，主要用途是**基于条件阻塞和唤醒**线程，结合互斥量使用。

### 4.10.1 条件变量相关函数

```c
// 条件变量的类型：pthread_cond_t

/*
    函数功能：初始化条件变量
    函数参数
    - cond：指针，指向需要进行初始化的条件变量
    - cond_attr：指针，指向条件变量初始化的属性对象，默认NULL即可
    返回值
    - 成功：0
    - 失败：非 0 的错误号
*/
int pthread_cond_init(pthread_cond_t *cond, pthread_condattr_t *cond_attr);

/*
    函数功能：唤醒一个或多个阻塞等待的线程
    函数参数
    - cond：指针，指向条件变量
    返回值
    - 成功：0
    - 失败：非 0 的错误号
*/
int pthread_cond_signal(pthread_cond_t *cond);

/*
    函数功能：唤醒所有阻塞等待的线程
    函数参数
    - cond：指针，指向条件变量
    返回值
    - 成功：0
    - 失败：非 0 的错误号
*/
int pthread_cond_broadcast(pthread_cond_t *cond);

/*
    函数功能：阻塞，调用该函数的线程会阻塞，但是会将互斥量 mutex 的状态设置为 unlock，其他线程可以获得互斥量
    函数参数
    - cond：指针，指向条件变量
    - mutex：互斥量对象
    返回值
    - 成功：0
    - 失败：非 0 的错误号
*/
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);

/*
    函数功能：线程阻塞指定的时间，调用该函数的线程，会阻塞，直到指定的时间结束
    函数参数
    - cond：指针，指向条件变量
    - mutex：互斥量
    - abstime：设置线程阻塞的指定时间
    返回值
    - 成功：0
    - 失败：非 0 的错误号
*/
int pthread_cond_timedwait(pthread_cond_t *cond, pthread_mutex_t *mutex, const struct timespec *abstime);

/*
    函数功能：释放条件变量资源
    函数参数
    - cond：需要释放的条件变量对象
    返回值
    - 成功：0
    - 失败：非 0 的错误号
*/
int pthread_cond_destroy(pthread_cond_t *cond);
```

### 4.10.2 基于条件变量，优化生产者消费者模型

```c
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<pthread.h>

// 定义条件变量
pthread_cond_t condition;

// 定义互斥量
pthread_mutex_t mutex;

// 定义共享资源，链表头部添加删除节点
typedef struct LinkNode {
    int data;
    struct LinkNode* next;
}LinkNode, LinkList;

LinkList* head = NULL;

// 创建链表，初始化头节点
void createLinkList() {
    head = (LinkNode*)malloc(sizeof(LinkNode));
    head->data = -1;
    head->next = NULL;
}


// 消费者子线程
void* consumer(void* arg) {

    // 消费者线程不断从链表头部取节点
    while (1) {
        sleep(1);
        pthread_mutex_lock(&mutex);
        if (head->next != NULL) {
            LinkNode* tmp = head->next;
            head->next = tmp->next;
            printf("consumer thread, LinkNode data = %d\n", tmp->data);
            free(tmp);
        }
        else {
            // 链表中没有节点，使用条件变量阻塞消费者进程，并且释放互斥量的锁，避免 CPU 资源的浪费
            // 执行该函数的线程，会使互斥量解锁，相当于执行 pthread_mutex_unlock() 操作，被唤醒后，又会对互斥量上锁
            pthread_cond_wait(&condition, &mutex);
        }
        pthread_mutex_unlock(&mutex);
    }

    return NULL;
}

// 生产者子线程
void* producer(void* arg) {

    // 生产者不断从链表头部插入节点
    while (1) {
        sleep(1);
        pthread_mutex_lock(&mutex);
        LinkNode* tmp = (LinkNode*)malloc(sizeof(LinkNode));
        tmp->data = rand() % 1000;
        tmp->next = head->next;
        head->next = tmp;
        printf("producer thread, LinkNode data = %d\n", tmp->data);

        // 生产了链表节点，可以唤醒消费者进程
        pthread_cond_signal(&condition);
        pthread_mutex_unlock(&mutex);
    }

    return NULL;
}


int main(int argc, char* argv[]) {

    // 初始化互斥量
    pthread_mutex_init(&mutex, NULL);

    // 初始化条件变量
    pthread_cond_init(&condition, NULL);

    // 初始化共享资源
    createLinkList();

    // 创建5个消费者线程和5个生产者线程
    pthread_t c_tid[5], p_tid[5];
    for (int i = 0;i < 5;++i) {
        pthread_create(p_tid + i, NULL, producer, NULL);
        pthread_create(c_tid + i, NULL, consumer, NULL);
    }

    // 设置线程分离
    for (int i = 0;i < 5;++i) {
        pthread_detach(p_tid[i]);
        pthread_detach(c_tid[i]);
    }

    // 主线程不退出
    while (1) {
        sleep(1);
    }

    // 释放互斥量
    pthread_mutex_destroy(&mutex);

    // 释放条件变量
    pthread_cond_destroy(&condition);

    return 0;
}
```

## 4.11 信号量

> 在 OS 中，信号量机制也是实现线程同步的一种方式，与互斥量不同的是，**信号量可以管理多个线程间，同步访问多个共享资源**。而互斥量，**是管理多个线程间，同步访问一个共享资源**。
>
> 我们假设信号量初始值$sem = y$，表示共享资源量为 $y$（信号量的初始值不可能大于共享资源的个数）。此时，$sem$ 的**取值有三种**情况，代表着**共享资源的三种状态**：
>
> - $sem < 0$，表示此时 $y$ 个**共享资源**都被 $y$ 个线程占用着，且有 $|sem|$ 个线程因为需要访问该共享资源被阻塞。
> - $sem = 0$，表示此时 $y$ 个**共享资源**刚好被 $y$ 个线程占用着，而且没有线程因为需要访问共享资源被阻塞。
> - $sem > 0$，表示此时 $y$ 个**共享资源**还剩下 $sem$ 个，即有 $y - sem$ 个线程正在占用共享资源。 
>
> **线程在对共享资源进行访问时**，$sem$ 值的变化有**两种情况**：
>
> - 线程**访问共享资源**，信号量的值 $sem$  **减 1**。
> - 线程**释放共享资源**，信号量的值 $sem$ **加 1**。

### 4.11.1 信号量相关函数

> UNIX OS 中，提供了**信号量相关的操作函数**

```c
#include <semaphore.h>
// 信号量的类型：sem_t

/*
    函数功能：初始化信号量
    函数参数
    - sem：需要初始化的信号量对象
    - pshared：赋值为 0，表示线程之间的信号量，赋值为非 0，表示进程之间的信号量
    - value：信号量的值，可以理解为资源的个数
    返回值
    - 成功：0
    - 失败：-1，并且设置错误号
*/
int sem_init(sem_t *sem, int pshared, unsigned int value);

/*
    函数功能：申请一个信号量资源，对信号量加锁，并且进行减一操作，如果 sem->value <= 0，调用该函数的线程阻塞
    函数参数
    - sem：信号量资源对象
    返回值
    - 成功：0
    - 失败：-1，并且设置错误号，信号量对象 sem 的 value 值不改变
*/
int sem_wait(sem_t *sem);

/*
    函数功能：和 sem_wait() 一样，申请一个信号量资源，不同的是，如果 sem->value <= 0，不会阻塞线程，直接返回 EAGAIN 错误号
    函数参数
    - sem：信号量资源对象
    返回值
    - 成功：0
    - 失败：-1，并且设置相应的错误号，信号量对象 sem 的 value 值不改变
*/
int sem_trywait(sem_t *sem);

/*
    函数功能：和 sem_wait() 一样，申请一个信号量资源，不同的是，如果 sem->value <= 0，阻塞线程，并且 abs_timeout 指定阻塞的时间
    - 如果在 abs_timeout 指定的时间内，sem->value 的值还是 <=0，结束线程阻塞，返回 ETIMEDOUT 错误号
    - 如果在 abs_timeout 指定的时间内，sem->value 的值 >0，进行信号量资源减一
    函数参数
    - sem：信号量资源对象
    - abs_timeout：指定线程能够接受最大的阻塞时间
    返回值
    - 成功：0
    - 失败：-1，并且设置相应的错误号，信号量对象 sem 的 value 值不改变
*/
int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);

/*
    函数功能：释放一个信号量资源，对信号量解锁，并且进行加一操作，对因为调用 sem_wait() 阻塞的线程进行唤醒
    函数参数
    - sem：信号量资源对象
    返回值
    - 成功：0
    - 失败：-1，并且设置错误号，信号量对象 sem 的 value 值不改变
*/
int sem_post(sem_t *sem);

/*
    函数功能：获取信号量对象的 value 值，赋值给 sval
    - 如果存在因为 sem 对象阻塞的线程，POSIX.1 允许 sval 的值有两种情况，可以是 0，也可以是 value 的绝对值（value 为负数时，说明存在阻塞进程）
    函数参数
    - sem：信号量对象
    - sval：接收 sem->value 的值
    返回值
    - 成功：0
    - 失败：-1，并且设置错误号 EINVAL，表示无效的 sem
*/
int sem_getvalue(sem_t *sem, int *sval);

/*
    函数功能：释放信号量资源
    函数参数
    - sem：需要释放的信号量对象
    返回值
    - 成功：0
    - 失败：-1，并且设置错误号
*/
int sem_destroy(sem_t *sem);
```

### 4.11.2 信号量实现生产者消费者模型

```c
#include<stdio.h>
#include<pthread.h>
#include<semaphore.h>
#include<unistd.h>
#include<stdlib.h>


typedef struct Node {
    int data;
    struct Node* next;
}Node;

// 创建头节点
Node* create_head() {
    Node* head = (Node*)malloc(sizeof(Node));
    head->data = -1;
    head->next = NULL;
    return head;
}
Node* head = NULL;

// 创建互斥量，保证线程数据安全（线程同步，保证同一时刻，只能有一个线程操作链表）
pthread_mutex_t mutex;

// 创建生产者和消费者信号量
sem_t producer_sem;
sem_t consumer_sem;

void* producer(void* arg) {
    // 生产者不断的创建新的节点，添加到链表中
    while (1) {
        usleep(1000);

        // 这里要注意，是先拿信号量，还是先拿互斥量，顺序不对，会造成死锁
        sem_wait(&producer_sem);        // 消耗一个生产者信号量

        pthread_mutex_lock(&mutex);

        Node* new_node = (Node*)malloc(sizeof(Node));
        new_node->data = rand() % 1000;
        new_node->next = head->next;
        head->next = new_node;
        printf("producer p_tid = %ld, adding node num = %d\n", pthread_self(), new_node->data);

        pthread_mutex_unlock(&mutex);

        sem_post(&consumer_sem);    // 生成一个消费者信号量

    }
    return NULL;
}

void* consumer(void* arg) {
    // 消费者不断从链表头部取节点
    while (1) {
        usleep(1000);

        sem_wait(&consumer_sem);    // 消耗一个消费者信号量

        pthread_mutex_lock(&mutex);

        Node* tmp = head->next;
        head->next = tmp->next;
        printf("consumer c_tid = %ld, deleting node num = %d\n", pthread_self(), tmp->data);
        free(tmp);

        pthread_mutex_unlock(&mutex);

        sem_post(&producer_sem);    // 生成一个生产者信号量

    }
    return NULL;
}

int main() {

    // 初始化链表头节点
    head = create_head();

    // 初始化互斥量
    pthread_mutex_init(&mutex, NULL);

    // 初始化生产者消费者信号量
    sem_init(&producer_sem, 0, 8);      // 生产者信号量的 value 值，决定了容器的上限
    sem_init(&consumer_sem, 0, 0);


    // 创建5个生产者线程，5个消费者线程
    pthread_t p_tids[5], c_tids[5];
    for (int i = 0;i < 5;++i) {
        pthread_create(p_tids + i, NULL, producer, NULL);
        pthread_create(c_tids + i, NULL, consumer, NULL);
    }

    // 线程分离
    for (int i = 0;i < 5;++i) {
        pthread_detach(p_tids[i]);
        pthread_detach(c_tids[i]);
    }

    // 保证主线程不退出
    while (1) {
        sleep(10);
    }

    // 释放互斥量
    pthread_mutex_destroy(&mutex);

    // 释放信号量
    sem_destroy(&producer_sem);
    sem_destroy(&consumer_sem);

    pthread_exit(NULL);
    return 0;
}
```



## 4.12 一些小的知识点补充

### 4.12.1 关于 Linux 开发环境问题

> 在`VSCode`的linux开发环境下，进行**进程、多线程开发的时候**，一些系统调用或者定义比如`mq_timedsend()`，`pthread_rwlock_t`等，编辑器无法解析出来，**经常会出现无法跳转函数定义、飘红等**，除了没有 include 相关的头文件之外，很可能是没有进行宏定义导致的 `VSCode` 解析器无法识别，解决方案如下：
>
> - 打开 `c_cpp_properties.json` 文件，在 `"defines"` 字段中加入 `_POSIX_C_SOURCE=200809L`。
>

### 4.12.2 关于 Linux 开发程序错误问题

> 在`Linux`中，**运行可执行文件出现的错误**，可以通过如下指令，查看错误的**详细信息**，主要原理是**基于`core`文件和`gdb`调试工具**。
>
> ```bash
> # 第一步，设置 core file size 为 unlimited
> ulimit -c unlimited
> # 第二步，通过如下指令查看设置是否成功
> ulimit -a
> # 第三步，通过 gcc 编译生成可调式的文件
> gcc demo.c -o demo -g
> # 第四步，运行 demo 可执行文件
> ./demo
> # 第五步，生成core文件，使用 gdb 命令
> gdb core
> # 第六步，进入 gdb 工具界面后，使用如下指令
> core-file core
> #生成相应的详细错误信息
> ```
>

