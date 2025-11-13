[TOC]

# 第三章 Linux多进程开发

## 3.1 进程概述

相信大家对进程一点都不陌生，当我们打开Windows OS的任务管理器，如下图所示，就能在**进程页面**看到很多用户进程和系统进程，并且我们还能发现，它是**动态占用CPU和内存等硬件资源的**，所以，这里得到**进程和程序（我们写的代码）的最大区别**，也是进程的最主要特征，它是**动态的**、可以发生**状态转换的**、有**生命周期的**。我们也可以简单的理解为进程是**程序 + 数据**的动态运行。

<center><img src="./images/第三章 Linux多进程开发/3-1 进程概述.png" width="90%"></center>

## 3.2 进程的状态转换

进程在 OS 运行的过程中，有五种常见的状态，分别是：**创建态、就绪态、运行态、阻塞态和结束态**，**这五种状态之间的转换关系以及转换发生的条件**如下图所示。

<center>
  <img src = "./images/第三章 Linux多进程开发/3-2 进程状态迁移图.png" width = "95%">
</center>


`linux`中，提供了`top`命令，**实时显示进程的运行状态**，`top`命令的解释如下：

> top 命令：实时显示进程运行状态（所占系统资源）
>
> - 在使用 top 命令的时候，可以加上 `-d`来指定**进程信息**更新的的时间间隔，在 top 命令执行之后，可以按以下按键对显示的结果进行排序
>   - `M`：根据**内存使用量**排序进程信息
>   - `P`：根据**CPU占有率**排序进程信息
>   - `T`：根据**进程运行时间长短**排序
>   - `U`：根据**用户名**来筛选进程
>   - `K`：输入指定的**进程`PID`**杀死进程
>
> 

`linux`中，提供了**进程号**相关的操作函数，具体的解释如下：

> 每一个进程都由进程号来标识，其类型为`pid_t`整型变量，进程号的**范围为 0 ~ 32767（4字节无符号整数）**，进程号总是唯一的，**但可以重用**，当一个进程终止后，其进程号就可以再次使用。
>
> 任何进程（除`init`进程）都是由另一个进程创建，创建子进程的进程叫做**父进程**，对应的进程号为`PPID`，被创建的进程叫做**子进程**。
>
> <font color = red>进程组</font>是一个或多个进程的集合。他们之间相互关联，**进程组**可以接收同一终端的各种信号，**相互关联的进程**有一个进程组号`PGID`，默认情况下，**一个进程自己组成进程组**，当前的`PID`会当作当前的`PGID`。
>
> **进程号和进程组相关函数：**
>
> `pid_t getpid(void);`
>
> - 函数功能：获取调用该函数进程的`PID`
> - 返回值：进程的`PID`
>
> `pid_t getppid(void);`
>
> - 函数功能：获取当前进程的父进程`PID`
> - 返回值：父进程的`PID`
>
> `pid_t getpgid(pid_t pid);`
>
> - 函数功能：获取当前进程所在进程组的`PGID`
> - 函数参数
>   - pid：当前进程的`PID`
> - 返回值：进程组的`PGID`

`linux`中**进程控制块 (Processing Control Block，PCB)**的相关概念如下：

> - OS 为了管理正在运行中进程，内核必须对每个进程**所做的事情进行清楚的描述**。所以，内核就使用了`PCB`这个结构体，维护了进程相关的信息，linux内核**进程控制块结构体名是`task_struct`**。
> - 在`/usr/src/linux-headers-xxx/include/linux/sched.h`文件中可以查看`struct task_struct` 结构体的定义。其内部成员有很多，我们只需要掌握部分即可：
>   - 进程`PID`
>   - 进程状态：创建、就绪、运行、阻塞和结束
>   - 进程切换时需要保护和恢复的一些 CPU 寄存器
>   - 描述虚拟地址空间的信息
>   - 描述控制终端的信息
>   - 当前的工作目录 (Current Working Directory)
>   - `umask` 掩码
>   - 文件描述符表，包含很多指向 file 结构体的指针，文件描述符（file descriptor）不仅仅记录的是打开的文件，还有记录内存缓冲区等。
>   - 信号相关的信息
>   - 用户 id 和组 id
>   - 会话（Session）和进程组
>   - 进程可以使用的资源上限（Resource Limit）

`linux`中，使用`ps`命令可以查看**当前系统的进程状态（静态显示）**，其中，`PS`命令和`STAT`参数各项意义如下：

> `ps`命令：静态查看进程运行状态
>
> - 使用方法：`ps aux`或者`ps ajx`
>   - a：显示终端上的所有进程，包括其他用户的进程
>   - u：显示进程的详细信息
>   - x：显示没有控制终端的进程
>   - j：列出与作业控制相关的信息
>
> 显示进程状态的静态信息，`STAT`参数的各项意义
>
> - D：不可中断 `Uninterruptible`(usually IO)
> - R：正在运行，或在队列中的进程
> - S（大写）：处于休眠状态
> - T：停止运行或被追踪
> - Z：`zombie` 僵尸进程
> - W：进入内存交换
> - X：死掉的进程
> - <：高优先级
> - N：低优先级
> - s：包含子进程
> - +：位于前台的进程组

`linux`中，提供了`kill`命令，向指定进程发送`signal`

> - `kill [-signal] pid`：向指定 pid 进程发送指定信号
> - `kill -l`：列出所有可发送信号
> - `kill -SIGKILL pid`：向指定 pid 进程发送`SIGKILL`命令，强制杀死进程
> - `kill -9 pid`：和如上命令等价
> - `killall name`：根据进程名杀死进程

## 3.3 进程创建

`linux`中，提供了**进程创建的函数`fork()`**，可以通过`man 2 fork`指令查看具体使用。

```c
#include<sys/types.h>
#include<unistd.h>
pid_t fork(void);
/*
  函数功能：创建子进程
  返回值：创建子进程成功之后，fork的返回值会返回两次，一次是在父进程中，一次是在子进程中
    - 在父进程中返回创建子进程的PID > 0
    - 在子进程中返回0
    - 通过fork的返回值区分父进程和子进程
    - 创建子进程失败之后，向父进程返回-1，并且设置 errno 错误号	
    
*/


/*
	argc 和 argv 都是用来处理命令行的参数
  argc: 是一个整数，表示传递给程序的命令行参数数量
  argv: 字符指针数组，包含了所有命令行参数的值，每一个元素都是指向字符串首地址的指针
*/
int main(int argc, char* argv[]) {
    int num = 10;
    pid_t pid = fork();
    if (pid > 0) {
			// 父进程逻辑....
    }
    else if (pid == 0) {
      // 子进程逻辑....
    }
    else if (pid == -1) {
      // 创建子进程失败
    }
    return 0;
}
```

## 3.4 父子进程虚拟地址空间情况

使用`fork()`创建的子进程，与**父进程具有相同的用户区**，同时，子进程**内核区的一部分也会从父进程拷贝过来（并非全部拷贝，比如`pid`等需要重新赋值）。**

关于**子进程对用户空间写时拷贝**，在linux中，`fork()`使用的是写时拷贝（copy-on-write）实现的，写时拷贝是一种**可以推迟甚至避免拷贝数据的技术（有时候，我们的子进程不存在写操作，这时，拷贝一份父进程的用户空间是没有必要的，会造成内存的浪费）**。

具体地，在`fork()`时，系统仅仅为子进程创建**内核空间**，其余的用户区、栈空间和共享库等地址空间**和父进程一起共享**，只有在子进程**需要写入的时候**，才会**复制其余的地址空间**。

<font color = red>注意</font>，执行`fork()`函数之后，父子进程共享文件，`fork()`产生的子进程和父进程**具有相同的文件描述符，指向相同的文件表**，引用计数增加，共享文件偏移指针。

## 3.5 父子进程关系及GDB多进程调试

### 3.5.1 `fork()`函数下父子进程之间的关系

`linux`中，父进程通过`fork()`函数创建的子进程有以下关系：

> 父进程与子进程之间的关系
> 区别：
>
> - fork()函数返回值不同
> - PCB 中的一些数据，如`pid`和`umask`等等
>
> 共同点（当子进程没有写数据时）：
>
> - 用户区的数据
> - 文件描述符表等
>
>   父子进程之间的变量共享问题：
>    - 读时共享（除了内核区）
>    - 写时拷贝（子进程需要对非内核区的变量进行写操作时）	

### 3.5.2 `GDB(GNU Debugger)`多进程调试

在使用`GDB`调试的时候，默认只能跟踪一个进程，**可以在`fork()`函数调用之前，设置断点**，然后通过指令设置`GDB`**是跟踪父进程逻辑还是子进程逻辑**，默认跟踪父进程逻辑，使用`GDB`进行调试的基本流程如下：

> - 使用`gdb`命令打开需要调试的程序：`gdb process_name`
> - `l`命令查看需要调试程序的内容
> - `b line_num`设置断点
> - 设置调试父进程或者子进程逻辑：`set follow-fork-mode [parent | child]`
>   - 默认为调试父进程逻辑
> - 设置调试模式：`set detach—on-fork [on | off]`
>   - 默认为`on`，表示调试当前进程的时候，其他进程继续运行，如果为`off`，调试当前进程的时候，其他进程被`GDB`挂起
> - `r`命令运行程序，直到遇到断点，程序停止运行
> - `n`命令运行下一步
>
> 其他的`GDB`相关 命令：
>
> - 查看调试的进程：`info inferiors`
> - 切换当前调试的进程：`inferior id`
> - 使进程脱离`GDB`调试：`detach inferiors id`

## 3.6 `exec(executable)`函数族（函数重载）

`exec`函数族的作用是根据指定的文件名找到可执行文件，并用它来取代调用进程的内容，**也可以理解为在调用进程内部执行一个可执行程序**。

<font color= red>注意：</font>`exec()`函数族函数执行成功后，**不会返回到原来调用函数的上下文继续执行**，因为调用进程的**实体，包括代码段、数据段和堆栈等都已经被新的内容取代**，只保留了进程 ID 等一些表面上的信息。只有调用失败了，才会返回 -1，从原程序的调用点接着往下执行。

```c
#include <unistd.h>

/*
	man 3 execl 
  函数功能：在一个进程内部执行可执行文件
  函数参数：
  	- path：可执行文件的路径 + 文件名，推荐使用绝对路径
  	- arg：可执行文件所需的参数列表，后面省略号代表可变参数
  		- 第一个参数可以理解为参数列表集合的名称，没什么作用，为了方便，一般写可执行程序的名称
    	- 从第二个参数开始往后，就是程序执行所需要的参数列表
    	- 参数最后需要以NULL结束（哨兵）
  返回值
  	- 调用成功：没有返回值（很好理解，进程的虚拟地址空间，除了内核区，都被调用的可执行文件覆盖了，找不到现场，无法返回）
  	- 调用失败：返回值为-1，并且设置errno
*/
int execl(const char *path, const char *arg, ...);

/*
  函数功能：和上面函数一样，唯一不同的是，调用参数
  函数参数：
  	- file: 需要执行可执行文件的文件名（与execl的区别是，不需要指定可执行文件的绝对路径，自动在系统的环境变量中寻找），适合执行内核相关的可执行文件
    - arg: 可执行文件所需要的参数列表，后面省略号代表可变参数
    	- 第一个参数可以理解为参数列表集合的名称，没什么作用，为了方便，一般写可执行程序的名称
      - 从第二个参数开始往后，就是程序执行所需要的参数列表
      - 参数最后需要以NULL结束（哨兵）
	返回值
  	- 调用成功，没有返回值（很好理解，进程的虚拟地址空间，除了内核区，都被调用的可执行文件覆盖了，找不到现场，无法返回）
  	- 调用失败，返回值为-1，并且设置errno
*/
int execlp(const char *file, const char *arg, ...);


#include <unistd.h>

int main(int argc, char* argv[]) {
    size_t pid = fork();
    if (pid > 0) {
      // ...
    }
    else if (pid == 0) {
        // 子进程执行execl函数，对应进程虚拟地址空间，除了内核区，全部被可执行文件覆盖
        execl("/bin/ps", "ps", "aux", NULL);
        perror("execl");
    }
    return 0;
}


int main(int argc, char* argv[]) {
    size_t pid = fork();
    if (pid > 0) {
      // ...
    }
    else if (pid == 0) {
        // 子进程执行execlp函数，对应进程虚拟地址空间，除了内核区，全部被可执行文件覆盖
        execlp("ps", "ps", "aux", NULL);
        perror("execl");
    }
    return 0;
}
```

## 3.7 进程控制（孤儿进程、僵尸进程、进程退出）

> <font color = green>孤儿进程：</font>父进程运行结束，但是子进程还在运行，这样的子进程就称为**孤儿进程（Orphan Process）**。
>
> - 每当出现一个孤儿进程时，内核就会把孤儿进程的父进程**设置为`init`进程，也就是 1 号进程**，然后，`init`进程会循环的`wait()`它已经退出的子进程，这样，当一个孤儿进程结束运行后，`init`进程就会负责回收孤儿进程的资源。因此，**孤儿进程不会有什么危害**。
>
> <font color = green>僵尸进程：</font>子进程运行结束，但是父进程还在运行，而每个进程结束之后，**都会释放自己地址空间中的用户数据，而内核区的 PCB 没有办法自己释放掉**，需要父进程去释放。如果父进程一直运行不结束，**且不调用`wait()`等函数，去释放子进程占用的内核资源**，就会导致子进程变成僵尸进程。
>
> - 僵尸进程不能被`kill -9 PID`杀死。
> - 试想一下，**如果父进程一直不调用`wait()`或`waitpid()`函数**，那么该父进程下的所有子进程，其 PCB 占用的内核资源**一直得不到释放**，就会导致**所有的 PCB 资源用尽（`PID`是有取值范围的）**，进而不能产生新的进程。因此，<font color = red>僵尸进程是有危害的</font>。
>
> <font color = green>进程退出：</font>进程退出的概念，主要是针对 IO 缓冲区的，linux 中有两个系统调用可以使进程退出，分别是`exit()`和`_exit()`，两者的区别如下：
>
> - `exit()`函数**会刷新 IO 缓冲区和关闭文件描述符**，然后调用`_exit()`函数。
> - `_exit()`函数直接将进程结束，**不会刷新IO缓冲区和关闭文件描述符**，相比`exit()`函数，它是一种高效的进程结束方法，但是，**会导致 IO 缓冲区数据泄露和文件未正确关闭**等问题。
>
> 所以，如果一个进程结束时，调用的是`_exit()`函数，**就是进程退出**。
>
> <center>
>   <img src = "./images/第三章 Linux多进程开发/3-7 进程退出1.png" width = 45%/>
> </center>

<font color = green>孤儿进程示例：</font>

```c
#include<sys/types.h>
#include<unistd.h>
#include<stdio.h>

int main(int argc, char* argv[]) {
  
    pid_t pid = fork();

    if (pid > 0) {
      	// ...
    }
    else if (pid == 0) {
      	// ...
        sleep(10);		// 让父进程先运行完
    }
    else {
        printf("create child process failure....\n");
    }
    for (int i = 0;i < 5;++i) {
      	// 这里 getppid() 返回的是 init 进程号
        printf("i = %d, pid = %d, ppid = %d\n", i, getpid(), getppid());		
    }
    return 0;
}
```

<font color = green>僵尸进程示例：</font>注意，不提倡在父进程中这样做，为了制造僵尸进程，在父进程中，我们不调用`wait()`和`waitpid()`系统调用，**并且在父进程中设置一个死循环，使得父进程一直运行**。

```c
#include<sys/types.h>
#include<unistd.h>
#include<stdio.h>

/*
		可以通过命令 `ps aux | grep 子进程的PID` 查看子进程的STAT，如果是Z，表示子进程属于僵尸进程
*/

int main(int argc, char* argv[]) {
    pid_t pid = fork();
  
    if (pid > 0) {
      	//父进程死循环
        while (1) {
            printf("parent process, pid = %d, ppid = %d\n", getpid(), getppid());
            sleep(1);
        }
    }
    else if (pid == 0) {
      	// 子进程运行完毕后，由于父进程没有结束运行，成为僵尸进程
        printf("child process, pid = %d, ppid = %d\n", getpid(), getppid());
        sleep(1);
    }
    else {
        printf("create child process failure....\n");
    }
  
    return 0;
}
```

<font color = green>进程退出示例：</font>

```c
#include<stdlib.h>
#include<unistd.h>
#include<stdio.h>

/*
	#include<stdlib.h>
  void exit(int status);
  函数功能：分三步执行
  进程运行
  - 调用进程退出处理函数
  - 刷新I/O缓冲，关闭文件描述符
  - 调用_exit()系统调用
  进程终止运行
  函数参数
  - status：进程退出时的状态码，可自定义，父进程回收子进程资源的时候可以获取到

	#include<unistd.h>
  void _exit(int status);
  函数功能：分一步执行
  进程运行
  - 调用_exit()系统调用
  进程终止运行
  函数参数
  - status：进程退出时的状态码，可自定义，父进程回收子进程资源的时候可以获取到
*/

int main(int argc, char* argv[]) {
    printf("hello\n");      // ‘\n’ 会自动刷新I/O缓冲区，输出到控制台
    printf("world");

    //exit(0);                // 刷新I/O缓冲区，world会输出到控制台中，status = 0 返回给父进程

    _exit(0);               // 不会刷新I/O缓冲区，world不会输出
    return 0;
}
```

## 3.8 进程回收、`wait()`和`waitpid()`系统调用

在谈`wait()`和`waitpid()`这两个系统调用的作用之前，我们先来谈谈<font color = green>什么是进程回收？</font>

> - 在每个进程运行完退出的时候，内核释放该进程的所有资源，包括打开的文件和占用的内存等。**但仍然会为其保留一定的信息，这些信息主要指 PCB 相关的，如进程号、退出状态和运行时间等**，<font color = red>这就是内核的进程回收机制</font>。
> - 父进程可以通过`wait()`或`waitpid()`系统调用，**得到子进程的退出状态，并且彻底清除掉这个子进程所占的系统资源**。

<font color = green>进程回收之后</font>，仍然会保留一些如 PCB 等的信息，需要父进程使用`wait()`和`waitpid()`这两个系统调用，**来实现这些信息资源的回收**。`wait()`和`waitpid()`系统调用如下：

> ```C
> #include <sys/types.h>
>    #include <sys/wait.h>
>    
> /*
>     函数功能：等待任意一个子进程结束，回收子进程的所有资源
>     函数参数：
>  		- wstatus，子进程退出时的状态信息
> 	返回值
>     - 成功：返回被回收的子进程pid
>     - 失败：-1，设置错误号（所有子进程都结束了且没有可以回收的子进程资源，或者是调用函数失败）
> 
>     wait函数对父进程状态的影响
>     - 调用 wait 函数的父进程会被挂起（阻塞），直到它的一个子进程结束或者收到一个不可忽略的信号才被唤醒
>     - 没有子进程，立刻返回-1，父进程从阻塞态变为就绪态
>     - 调用一次 wait 函数，只能回收一个子进程 
> */
> pid_t wait(int *wstatus);
> 
> /*
>     函数功能：父进程通过参数 pid 和 options，选择特定的方式回收子进程资源，可以设置父进程是否阻塞
>     函数参数：
>     - pid_t pid
>     	- pid > 0 : 回收指定pid的子进程
>     	- pid = 0 : 回收当前进程组的所有子进程
>     	- pid = -1: 回收所有子进程资源，相当于wait()函数，最常用
>     	- pid < -1: 回收pgid = |pid|的所有子进程资源，限制了进程组范围
> 		- wstatus，进程退出时的状态信息
>     - int options，设置父进程阻塞和非阻塞
>       - 0: 阻塞
>       - WNOHANG : 非阻塞（如果不存在结束的子进程，waitpid()函数立即返回），常用，是和 wait() 的主要区别
>     返回值
>       - >0：返回回收资源的子进程pid
>       - 0: 如果 options = WNOHANG, 表示还有子进程没有运行结束
>       - -1：所有子进程都结束了且没有可以回收的子进程资源，或者是调用函数失败
> 
>     与 wait() 函数不同的是，waitpid() 在等待子进程结束之前，可以设置非阻塞，提高了父进程的执行效率，并且，可以指定回收进程组的进程
>     - 与 wait() 函数一样，waitpid() 调用一次，只能回收一个子进程资源
> */
> pid_t waitpid(pid_t pid, int *wstatus, int options);
> ```

<font color = green>`wait()`系统调用示例：</font>

```c
#include<sys/types.h>
#include<sys/wait.h>
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>

int main(int argc, char* argv[]) {
    pid_t pid;
    //创建5个子进程
    for (int i = 0;i < 5;++i) {
        pid = fork();
        if (pid == 0) {
            break;
        }
        else if (pid < 0) {
            perror("fork");
          	exit(-1);
        }
    }
    if (pid > 0) {
        while (1) {
            printf("parent process, pid = %d\n", getpid());
            int status;
          	//调用wait阻塞函数，一直阻塞等待子进程结束
            int ret = wait(&status);       
            if (ret == -1) {
                printf("no child process need recycle\n");
                break;
            }
            if (WIFEXITED(status)) {
                //正常退出
                printf("退出的状态码：%d\n", WEXITSTATUS(status));
            }
            if (WIFSIGNALED(status)) {
                //异常终止
                printf("异常终止状态码：%d\n", WTERMSIG(status));
            }
            printf("child process die, child pid = %d\n", ret);
            sleep(1);
        }
    }
    else if (pid == 0) {
      	printf("child process, pid = %d\n", getpid());
      	sleep(1);
    }
    return 0;
}
```

<font color = green>`waitpid()`系统调用示例：</font>

```c
#include<sys/types.h>
#include<sys/wait.h>
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>

int main(int argc, char* argv[]) {
    pid_t pid;
    //创建5个子进程
    for (int i = 0;i < 5;++i) {
        pid = fork();
        if (pid == 0) {     
            break;
        }
        else if (pid < 0) {
            perror("fork");
          	exit(-1);
        }
    }
    if (pid > 0) {
        while (1) {
            printf("parent process, pid = %d\n", getpid());
            int status;
       			//options = WNOHANG，父进程不阻塞
            int ret = waitpid(-1, &status, WNOHANG);      
            if (ret == -1) {
                printf("no child process need recycle\n");
                break;
            }
            else if (ret == 0) {
              	// 当 options = WNOHANG，因为是非阻塞，子进程没有运行结束，与 wait() 不同的是，ret 有返回值为 0 的情况
                printf("having child process\n");
            }
            else {
                printf("child process die, child pid = %d\n", ret);
                if (WIFEXITED(status)) {
                    // 正常退出
                    printf("退出的状态码：%d\n", WEXITSTATUS(status));
                }
                if (WIFSIGNALED(status)) {
                    // 异常终止
                    printf("异常终止状态码：%d\n", WTERMSIG(status));
                }
            }
            sleep(1);
        }
    }
    else if (pid == 0) {
        // 子进程
        printf("child process, pid = %d\n", getpid());
        sleep(1);
    }
    return 0;
}
```

在使用`wait()`和`waitpid()`两个系统调用的时候，我们通过传出参数`status`可以得到**子进程的返回状态编号，通过 linux 提供的进程退出相关宏函数**，传入状态编号，<font color = green>得到进程退出的相关信息：</font>

> - `WIFEXITED(status)`：返回值非0，进程正常退出
> - `WEXITSTATUS(status)`：如果上宏为真（可以理解为返回值非0），获取进程退出的状态（exit的函数）
> - `WIFSIGNALED(status)`：返回值非0，进程异常终止
> - `WTERMSIG(status)`：如果上述宏为真，获取使进程终止的信号编号
> - `WIFSTOPPED(status)`：非0，进程处于暂停状态
> - `WSTOPSIG(status)`：如果上述宏为真，获取使进程暂停的信号的编号
> - `WIFCONTINUED(status)`：非0，进程暂停后已经继续运行

## 3.9 进程间通信简介

linux 中，进程是一个独立的资源分配单元，不同进程之间的资源是相互独立的，**不能在一个进程中直接访问另一个进程的资源**。

但是，进程在运行的过程中，又不是孤立的，**不同进程之间需要进行信息的交互和状态的传递等**，因此需要<font color= red>进程间通信（Inter Processes Communication, IPC）</font>。

### 3.9.1 linux 进程间通信的几种方式

> 进程之间**进行通信的主要目的有如下：**
>
> - 数据传输：一个进程需要将它的数据发送给另一个进程
> - 通知事件：一个进程**需要向另一个或一组进程发送消息**，通知它们发生了某种事件（如进程终止时需要通知父进程）
> - 资源共享：多个进程之间**共享同样的资源**，为了做到这一点，OS 的内核需要提供互斥和同步机制
> - 进程控制：有些进程希望**完全控制另一个进程的执行（如 Debug 进程）**，此时控制进程希望能够拦截另一个进程的所有陷入和异常，并能够及时知道它的状态改变。
>
> **进程间通信的几种方式**，大致可以总结为以下几种：
>
> <center>
>   <img src = "./images/第三章 Linux多进程开发/3-10 linux进程通信方式.png" width = "70%">
> </center>

## 3.10 匿名管道概述

> <font color =green>管道介绍</font>
>
> - 它是 UNIX 系统 IPC 的最古老形式，所有的 UNIX 系统都支持这种通信机制
> - 举例子：使用一个 shell 命令`ls | wc -l`统计一个目录中文件的数目，为了执行该命令， shell 创建了两个进程，分别执行 `ls` 和 `wc`：
>
> <center>
>   <img src = "./images/第三章 Linux多进程开发\3-10 匿名管道1.png" width = "80%"> 
> </center>

### 3.10.1 管道特点

> - 管道其实是一个在内存的内核空间中，维护的一个缓冲器（缓冲区），这个缓冲器的**存储能力是有限的**，不同的操作系统大小不一定相同。
> - 管道拥有文件的特质：读操作、写操作，**匿名管道没有文件实体，有名管道有文件实体（通过文件描述符实现），但不存储数据**，可以按照操作文件的方式，对管道进程操作。
> - 一个管道可以理解为是一个字节流序列，使用管道的过程中，**不存在消息或者消息边界的概念**，从管道读取数据的进程，**可以读取任意大小的数据块，而不管写入进程写入管道的数据块大小是多少**。
> - **通过管道传递的数据是顺序的**，从管道中读取出来的字节顺序，和它们被写入管道的顺序是完全一样的。
> - 在管道中，**数据的传递方向是单向的，一端用于写入，一端用于读取，管道是半双工的**。
> - 从管道中读取数据是一次性操作，**数据一旦被读走，它就从管道中被抛弃，释放空间以便写更多的数据**，在管道中，无法使用 `lseek()`来随机访问管道里面的数据。
> - 匿名管道只能在**具有公共祖先的进程（父子进程，兄弟进程等）**之间使用。
> - 管道的实现使用的数据结构是循环队列。
>
> <center>
>   <img src = "./images/第三章 Linux多进程开发/3-11 匿名管道的特点3.png" width = "80%">
> </center>

### 3.10.2 匿名管道的通信原理

前面 <font color = red>3.10.1</font> 中我们说到，**匿名管道只能在具有公共祖先的进程之间使用**，这是为什么呢？

> 首先，匿名管道没有文件实体，其次，在不同的**非亲子进程**之间，**它们的文件描述符表是不共享的**，这就导致了，**每一个非亲子进程**，创建匿名管道对应的文件描述符都是独立的。
>
> 但是，在具有公共祖先的进程中，**它们 PCB 中的文件描述符表是共享的**，如果我们在祖先进程中，创建匿名管道，**并且通过文件描述符表标记匿名管道的读端和写端**。这样，通过祖先进程创建的其他子进程，**由于具有相同的文件描述符表，就可以找到祖先进程创建的匿名管道**，进而实现 IPC。
>
> <center>
>   <img src = "./images/第三章 Linux多进程开发/3-11 匿名管道通信原理图1.png" width = "80%">
> </center>

### 3.10.3 匿名管道的使用

> <font color = green>创建匿名管道</font>
>
> ```c
> #include <unistd.h>
> 
> /*
> 函数功能：创建一个匿名管道，用来进程间通信
>  函数参数
>     - pipefd[2]: 传出参数
>     - pipefd[0]: 对应管道的读端
>     - pipefd[1]: 对应管的道写端
>     返回值
>     - 成功，返回 0
>     - 失败，返回 -1
> 
>     tips：匿名管道只能用于亲子进程之间的通信，因为亲子进程在创建的过程中，具有相同的内核区文件描述符表
> */
> int pipe(int pipefd[2]);
> ```
>
> <font color = green>查看管道大小的命令</font>
>
> `ulimit -a`
>
> <font color = green>查看管道缓冲区大小函数</font>
>
> ```c
> #include<unistd.h>
> 
> /*
> 	函数功能：获取与文件或文件系统相关的配置参数（这种参数通常是一种限制，比如缓冲区大小等等）
> 	函数参数
> 	- fd：对应的文件描述符
> 	- name：需要查询的相关配置参数
> 	返回值
> 	- 成功：返回对应的配置参数限制
> 	- 失败：返回 -1，如果配置参数没有限制，也返回 -1
> */
> long fpathconf(int fd, int name);
> ```

### 3.10.4 匿名管道通信案例1

```c
/*
		案例1：通过匿名管道，子进程向匿名管道写数据，父进程从匿名管道读数据
*/
#include<unistd.h>
#include<sys/types.h>
#include<stdio.h>
#include<stdlib.h>
#include<string.h>

int main(int argc, char* argv[]) {
    int pipefd[2];
  	// 1. 创建管道
    int ret = pipe(pipefd);
    if (ret == -1) {
        perror("pipe");    
        exit(0);
    }
  
    pid_t pid = fork();
    if (pid > 0) {
      // 2. 父进程从管道的读端读数据
      char buf_read[1024] = { 0 };
 
      read(pipefd[0], buf_read, sizeof(buf_read));
      printf("parent process receive: %s, pid = %d\n", buf_read, getpid());
      printf("parent process, pipefd[0] = %d, pipefd[1] = %d\n", pipefd[0], pipefd[1]);
      //读完数据打印后，重置
      bzero(buf_read, 1024);
    }
    else if (pid == 0) {
      // 3. 子进程从管道的写端写数据
      char buf_write[] = "child to parent";
      
      write(pipefd[1], buf_write, sizeof(buf_write));
      printf("child process write to parent: %s, pid = %d\n", buf_write, getpid());
      sleep(1);
    }
    else {
        perror("fork");
    }
  
    return 0;
}
```

### 3.10.5 匿名管道通信案例2

```c
/*
    案例2：子进程执行 ps aux，将得到的内容写入匿名管道，父进程输出
    
    子进程：执行 ps aux 可执行文件, 将标准输出 stdout_fileno 重定向到管道的写端
    - execlp("ps", "ps", "aux", NULL);
    - dup2(pipefd[1], STDOUT_FILENO);
    父进程：从管道获取数据
    
*/
#include<unistd.h>
#include<sys/types.h>
#include<stdio.h>
#include<stdlib.h>
#include<wait.h>
#include<string.h>


int main(int argc, char* argv[]) {
    // 创建管道
    int pipefd[2];
    int ret = pipe(pipefd);
    if (ret == -1) {
        perror("pipe");
        exit(-1);
    }
    // 创建子进程
    pid_t pid = fork();
    if (pid > 0) {
        // 关闭写端
        close(pipefd[1]);

        // 从管道中读数据，一次只能读取 1024 字节
        char buf_read[1024] = { '\0' };
        int len = -1;
        printf("parent process receive:\n");
        while (len = read(pipefd[0], buf_read, sizeof(buf_read) - 1) > 0) {
            printf("%s", buf_read);     //ps aux产生的内容存在 '\n'  
            memset(buf_read, 0, sizeof(buf_read));      //清空字符串里面的内容
        }
        wait(NULL);
    }
    else if (pid == 0) {
        // 关闭读端
        close(pipefd[0]);

        // 文件描述符重定向，将终端打印的字符重定向到管道的写端，stdout_fileno -> fd[1]
        dup2(pipefd[1], STDOUT_FILENO);

        //执行 ps aux 可执行文件，子进程的上下文被替换
        execlp("ps", "ps", "aux", NULL);
      	// 执行错误
        perror("execlp");       

    }
    else {
        perror("fork");
    }
  
    return 0;
}
```

### 3.10.6 匿名管道的读写特点和匿名管道设置为非阻塞

#### 3.10.6.1 管道的读写特点

> - **当所有指向管道写端的文件描述符都关闭了（管道写端引用计数等于 0）**，有进程从管道的读端**读数据**，那么管道中剩余的数据被读取完后，该进程再次调用`read()`系统调用时，**会返回 0，就像读到文件的末尾一样（读端的进程不会被阻塞，因为就算阻塞等待了，也没有用，不会有进程向管道写数据）**。
>
> - **当存在指向管道写端的文件描述符没有关闭（管道写端的引用计数大于 0）**，有进程从管道的读端**读数据**，那么管道中剩余的数据被读取完后，该进程再次调用`read()`系统调用时，**会返回-1，进程进入阻塞状态（直到写端的进程向管道中写数据）**。
>
> - **当所有指向管道读端的文件描述符都关闭了（管道读端引用计数等于 0）**，有进程从管道的写端**写数据**，那么写端的写进程**会收到一个异常信号 `SIGPIPE`**，通常会导致**写进程异常终止（对管道只有写操作，没有读操作，很容易导致内存溢出）**。
> -  **当存在指向管道读端的文件描述符没有关闭（管道读端的引用计数大于 0）**，并且持有管道读端文件描述符的进程**没有从管道中读数据**，有进程从管道的写端**写数据**，那么当管道被写满时，再次调用`write()`系统调用时，**会返回-1，进程进入阻塞状态（直到读端的进程从管道中读数据，使得管道中有空位置才能再次写入数据）**。

#### 3.10.6.2 匿名管道读写特点总结

> <font color = green>读管道：</font>
>
> - **管道中有数据**，`read()`系统调用返回实际读到的字节数。
> - **管道中无数据**，**写端文件描述符全部被关闭**，`read()`系统调用返回 0（相当于读到文件末尾），**写端文件描述符没有全部被关闭**，`read()`系统调用**使进程阻塞（等待写端的进程写数据到管道中）**。
>
> <font color = green>写管道：</font>
>
> - **管道读端的文件描述符全部被关闭**，进程异常终止（进程收到`SIGPIPE`信号）。
> - **管道读端文件描述符没有全部被关闭**，**管道没有满**，`write()`系统调用将数据写入管道，返回写入的`byte`数量，**管道已满**，`write()`系统调用**使进程阻塞（等待读端的进程从管道中读数据）。**

#### 3.10.6.3 将匿名管道设置为非阻塞

从 <font color = red>3.10.6.2</font> 中我们可以知道，**当管道写端文件描述符没有全部被关闭时**，管道为空，读端进程读管道会被阻塞，同理，**当管道读端文件描述符没有全部被关闭时**，管道满，写端进程写管道会被阻塞。UNIX OS 的内核中，可以设置**上述两种情况为非阻塞**。

```c
/*
    设置管道非阻塞
    int flag = fcntl(fd[0], F_GETFL);   // 获取原来的flag
    flag |= O_NONBLOCK;                	// 位操作修改flag的值
    fcntl(fd[0], F_SETFL, flag);       	// 设置新的flag	
*/
#include<unistd.h>
#include<sys/types.h>
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<fcntl.h>

int main(int agrc, char* argv[]) {
    // 1.fork之前创建管道
    int pipefd[2];
    int ret = pipe(pipefd);
    if (ret == -1) {
        perror("pipe");    
        exit(-1);
    }

    pid_t pid = fork();

    if (pid > 0) {
        //设置管道读端非阻塞
        int flag = fcntl(pipefd[0], F_GETFL);   
        flag |= O_NONBLOCK;                    
        fcntl(pipefd[0], F_SETFL, flag);      

        //父进程从管道的读端读数据
        char buf_read[1024] = {'\0'};
        int len = 0;
        while (1) {
            // 关闭写端
            close(pipefd[1]);
						
          	// 此处 read 系统调用非阻塞
            len = read(pipefd[0], buf_read, sizeof(buf_read));
            printf("parent process read pipe, pid = %d\n", getpid());
            printf("len = %d %s\n", len, buf_read);
            memset(buf_read, 0, sizeof(buf_read));
            sleep(5);
        }
    }
    else if (pid == 0) {
        // 设置管道写端非阻塞
        int flag = fcntl(pipefd[1], F_GETFL);   
        flag |= O_NONBLOCK;                     
        fcntl(pipefd[1], F_SETFL, flag);       

        //子进程从管道的写端写数据
        char buf_write[] = "hello pipe.";
        int len = 0;
        while (1) {
            // 关闭读端
            close(pipefd[0]);
						
          	// 此处 write 系统调用非阻塞
            len = write(pipefd[1], buf_write, sizeof(buf_write));
            printf("child process write pipe, pid = %d\n", getpid());
            printf("len = %d %s\n", len, buf_write);
            sleep(1);
        }
    }
    else {
        perror("fork");
    }

    return 0;
}

```

## 3.11 有名管道概述

匿名管道，由于没有绑定实体文件，只是通过**各自进程的文件描述符标记管道的读端和写端**，只能用于亲子进程通信，为了克服这个缺点，提出了有名管道（FIFO），也叫命名管道、FIFO文件。

> <font color = green>有名管道</font>
>
> - 有名管道（FIFO）不同于匿名管道的是，它提供了一个路径名与之关联，以 FIFO 的文件形式**存储在文件系统中**，并且打开方式和普通文件一致，这样即使与 FIFO 的创建进程**不存在亲子关系的其他进程**，只要可以访问**该路径（文件）**，就能够彼此通过 FIFO 相互通信。
> - 一旦打开了 FIFO，就能够使用，操作匿名管道和其他文件一样的系统调用，如`read()`、`write()`和`close()`等。与匿名管道一样，FIFO 也有写端和读端，并且从 FIFO 中读数据的顺序和写数据的顺序是一致的，所以我们有时候称有名管道为 FIFO。
>
> <font color = green>有名管道（FIFO）和匿名管道（pipe）**有一些特点是相同的，不一样的地方在于：**</font>
>
> - FIFO 在文件系统中，作为一个特殊的文件存在，但 FIFO 中的内容却存放在内存中。
> - 当使用 FIFO 的进程退出后，FIFO 文件将继续保存到文件系统中。
> - FIFO 有名字（即 FIFO 相关的文件在内存中被打开后，**和普通文件一样，有一个`Inode`表项**），没有亲子关系的进程可以通过`Inode`表项找到打开的 FIFO，进行通信。

### 3.11.1 有名管道的使用

> <font color = green>第一步，先创建有名管道，也是最重要的一步</font>
>
> - **在一个文件系统目录下，通过shell命令创建有名管道**
>
> ```bash
> # mkfifo 有名管道名字
> ```
>
> - **通过函数创建有名管道**
>
> ```c
> #include <sys/types.h>
> #include <sys/stat.h>
> 
> int mkfifo(const char *pathname, mode_t mode);
> /*
>     函数参数
>     - pathname：管道名称的路径
>     - mode：文件的权限，和 open 的 mode 是一样的，通过八进制代码表示，在系统调用中，mode 会和 ~umask 做 & 操作，常用的 8 进制代码如下
>     	- 0777：所有用户（所有者、组用户和其他用户）都有读、写和执行权限
>     	- 0666：所有用户都有读写权限，没有执行权限
>     	- 0644：所有者有读和写权限，组用户和其他用户只有读权限
>     	- 0755：所有者有读、写和执行权限，组用户和其他用户有读和执行权限
> 			- 0664：所有者和组用户有读写权限，其他用户只有读权限
>     返回值
>     - 创建成功：0
>     - 创建失败：-1，并且生成系统调用的错误信息
> */
> ```
>
> <font color = green>第二步，使用创建的FIFO</font>
>
> - 一旦使用`mkfifo`创建了一个 FIFO，就可以使用 open 系统调用打开它，**常见的 I/O 函数都可以用于 FIFO**，如`close(), read(), write(), unlink()`等。
> - FIFO 严格遵循先进先出，**具有管道的所有特征**，比如不支持`lseek()`文件定位操作等。
>
> <font color = green>Tips：有名管道使用注意事项</font>
>
> - **当只有一个进程以只读的方式打开有名管道**，该进程会<font color = red>阻塞，`open()`系统调用导致的阻塞</font>，**直到有其他进程以只写的方式打开有名管道**，阻塞的进程**才会被唤醒**。
> - 同理，**当只有一个进程以只写的方式打开有名管道**，该进程也会被<font color = red>阻塞，`open()`系统调用导致的阻塞</font>，**直到有其他进程以只读的方式打开有名管道**，阻塞的进程**才会被唤醒**。

### 3.11.2 有名管道的读写特点

> <font color = green>读管道：</font>
>
> - **管道中有数据**，`read()`系统调用返回实际读到的字节数。
> - **管道中无数据**，**写端文件描述符全部被关闭**，`read()`系统调用返回 0（相当于读到文件末尾），**写端文件描述符没有全部被关闭**，`read()`系统调用**使进程阻塞（等待写端的进程写数据到管道中）**。
>
> <font color = green>写管道：</font>
>
> - **管道读端的文件描述符全部被关闭**，进程异常终止（进程收到`SIGPIPE`信号）。
> - **管道读端文件描述符没有全部被关闭**，**管道没有满**，`write()`系统调用将数据写入管道，返回写入的`byte`数量，**管道已满**，`write()`系统调用**使进程阻塞（等待读端的进程从管道中读数据）。**
>

### 3.11.3 有名管道实现简单的聊天功能

> <font color = green>FIFO 实现简单版聊天功能案例基本思路：</font>
>
> 1. 编写程序 C，创建 FIFO，由于管道是半双工的，可以创建两个管道，分别负责聊天双方的数据流向
> 2. 编写程序 A，模拟聊天的其中一方，对创建的两个 FIFO 分别进行读写操作
> 3. 同理，编写程序 B，模拟聊天的另一方，对创建的两个 FIFO 分别进行读写操作
> 4. 启动程序 A 和 B，通过对 FIFO 的读写操作，模拟简单的聊天过程

**程序C，创建 FIFO 文件**

```c
#include<stdio.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<string.h>
#include<stdlib.h>
#include<unistd.h>

/*
		创建两个有名管道
		- a_to_b 有名管道，程序 A 是写端，程序 B 是读端
		- b_to_a 有名管道，程序 B 是写端，程序 A 是读端
		
		#include<unistd.h>
		int access(const char *pathname, int mode)
		函数功能：检查调用进程是否有权限访问指定的文件或路径，这个函数可以测试文件是否
		存在，以及当前进程是否有读、写或执行该文件的权限
		函数参数
		- pathname：指定要检查的文件或路径的名称
		- mode：指定要检查的权限类型，可以是以下常量之一，也可以是多个常量按位或
		（OR）组合
				- F_OK：测试文件是否存在
				- R_OK：测试文件是否可读
				- W_OK：测试文件是否可写
				- X_OK：测试文件是否可执行
		返回值
		0：成功，表示具有指定权限
		-1：失败，表示没有指定的权限或发生错误，并且设置错误号
*/

int main(int argc, char* argv[]) {
    // 判断有名管道是否存在
    int is_exist = access("./a_to_b", F_OK);
  
    if (is_exist == -1) {
        printf("a_to_b named pipe doesn't exist.\n");
      
        int ret = mkfifo("./a_to_b", 0664);

        if (ret == -1) {
            // 管道创建失败
            perror("mkfifo");
        }
    }

    // 判断有名管道是否存在
    is_exist = access("./b_to_a", F_OK);
    if (is_exist == -1) {
        // 不存在
        printf("b_to_a named pipe doesn't exist.\n");
        int ret = mkfifo("./b_to_a", 0664);

        if (ret == -1) {
            // 管道创建失败
            perror("mkfifo");
        }
    }
    return 0;
}
```

**程序A，聊天的其中一方**

```C
#include<stdio.h>
#include<string.h>
#include<stdlib.h>
#include<unistd.h>
#include<fcntl.h>
#include<string.h>

/*
    聊天 A 程序
    - 以只读的方式打开b_to_a有名管道
    - 以只写的方式打开a_to_b有名管道
*/

int main(int argc, char* argv[]) {

    // 1. 只写方式打开 a_to_b
    int a_to_b_fd = open("./a_to_b", O_WRONLY);

    // 2. 只读方式打开 b_to_a
    int b_to_a_fd = open("./b_to_a", O_RDONLY);

    // 管道文件打开失败
    if (a_to_b_fd == -1 || b_to_a_fd == -1) {
        printf("end chat_a process.\n");
        exit(-1);
    }

    printf("open a_to_b and b_to_a successful...\n");

    while (1) {
        char buf_read[1024] = { '\0' };
        char buf_write[1024] = { '\0' };

        // 3. 获取键盘写入的数据
        fgets(buf_write, 1024, stdin);

        // 4. 向 a_to_b 管道写数据
        int ret1 = write(a_to_b_fd, buf_write, sizeof(buf_write));

        if (ret1 == -1) {
            // 管道读端进程关闭，写入数据失败
            perror("write");
            break;
        }

        // 5. 向 b_to_a 管道读数据
        int ret2 = read(b_to_a_fd, buf_read, sizeof(buf_read));
        if (ret2 == -1) {
            // 管道写端进程关闭，读取数据失败
            perror("read");
            break;
        }
        printf("b_to_a info: %s", buf_read);
    }
    // 6. 关闭文件描述符对应的管道
    close(a_to_b_fd);
    close(b_to_a_fd);

    printf("end chat_a process.\n");

    return 0;
}
```

**程序B，聊天的另一方**

```c
#include<stdio.h>
#include<string.h>
#include<stdlib.h>
#include<unistd.h>
#include<fcntl.h>
#include<string.h>

/*
    聊天 B 程序
    - 以只读的方式打开a_to_b有名管道
    - 以只写的方式打开b_to_a有名管道
    - 循环写入数据
    
    注意事项：
    - A 和 B 程序必须都先打开 a_to_b 有名管道的读和写端，不然程序会阻塞
*/

int main(int argc, char* argv[]) {

    // 1. 只读方式打开 a_to_b
    int a_to_b_fd = open("./a_to_b", O_RDONLY);

    // 2. 只写方式打开 b_to_a
    int b_to_a_fd = open("./b_to_a", O_WRONLY);

    // 管道文件打开失败
    if (a_to_b_fd == -1 || b_to_a_fd == -1) {
        printf("end chat_a process.\n");
        exit(0);
    }

    printf("open a_to_b and b_to_a successful...\n");

    while (1) {
        char buf_read[1024] = { '\0' };
        char buf_write[1024] = { '\0' };

        // 3. 向 a_to_b 管道读数据
        int ret1 = read(a_to_b_fd, buf_read, sizeof(buf_read));
        if (ret1 == -1) {
            // 管道写端进程关闭，读取数据失败
            perror("read");
            break;
        }
        printf("a_to_b info: %s", buf_read);

        // 4. 获取键盘写入的数据
        fgets(buf_write, 1024, stdin);

        // 5. 向 b_to_a 管道写数据
        int ret2 = write(b_to_a_fd, buf_write, sizeof(buf_write));
        if (ret2 == -1) {
            // 管道读端进程关闭，写入数据失败
            perror("write");
            break;
        }
    }
    // 6. 关闭文件描述符对应的管道
    close(a_to_b_fd);
    close(b_to_a_fd);

    printf("end chat_a process.\n");

    return 0;
}
```

## 3.12 内存映射

> 内存映射（Memory-mapped I/O）的原理非常简单，就是将磁盘文件的数据映射到内存，用户通过内存就能修改磁盘文件。
>
> <center>
>   <img src = "./images/第三章 Linux多进程开发/3-17 内存映射1.png" width = "90%">
> </center>

### 3.12.1 内存映射相关系统调用

```c
#include <sys/mman.h>
void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);

/*
	- 函数功能：将一个文件或者设备的数据映射到内存空间中
  - 函数参数
  	- addr: 创建文件或设备内存映射的地址，一般程序员不好指定，赋值为NULL，由内核决定
    - length: 要映射数据的长度（一般为页的整数倍，大于等于文件长度）
    	- 获取文件长度的API: stat(), lseek()
    	- 可以使用`getconf PAGE_SIZE`或者`ulimit -a`查看 OS 的页大小
    - prot: 内存映射保护的权限（操作的权限，不能和打开的文件权限有冲突）
    	- PROT_NONE: 没有权限
      - PROT_EXEC: 可执行权限
      - PROT_WRITE: 写权限
      - PROT_READ: 读权限
    - flags: 确定是否将内存映射更新的内容覆盖到打开的文件中
    	- MAP_SHARED: 内存映射更新的内容会覆盖打开的文件，和打开的文件同步，如果要实现ipc，使用该参数
      - MAP_PRIVATE: 内存映射更新的内容不会覆盖打开的文件，会重新创建一个新的文件（copy on write，和子进程用户区一样，读时共享，写时拷贝）
    - fd: 进行内存映射的文件描述符，通过 open 系统调用得到打开文件的描述符
    	- 注意：文件大小必须大于0，open 时和 prot 参数权限不能有冲突，应该要高于 prot 的权限
    - offset: 内存映射的偏移量，必须是逻辑页大小的整数倍，一般使用0，不需要偏移

	- 返回值
    - 成功：返回内存映射区域的首地址
    - 失败：返回MAP_FAILED, (void *)(-1), 并且设置系统调用错误信息
*/
int munmap(void *addr, size_t length);
/*
	- 函数功能：释放内存映射
  - 函数参数
  	- void* addr: 需要释放内存映射的首地址（这个地址一般根据 mmap 的返回值得到）
    - size_t length: 需要释放内存映射的长度，和 mmap() 中的 length 一致
  - 返回值
  	- 成功：0
    - 失败：-1, 并且设置错误号
*/
```

### 3.12.2 内存映射实现进程间通信

> <font color = green>有关系的进程（父子进程）间通信</font>
>
> - 还没有子进程的时候
>   - 通过唯一的父进程，先创建内存映射区
>
> - 有了内存映射区后，创建子进程
> - 父子进程共享创建的内存映射区
>
> <font color = green>没有关系的进程间通信</font>
>
> - 准备一个大小不是 0 的磁盘文件
> - **进程1**：通过磁盘文件创建内存映射区
>   - 得到一个操作这块内存的指针
> - **进程2**：通过磁盘文件创建内存映射区
>   - 得到一个操作这块内存的指针
> - 使用内存映射区通信
>
> <font color = red>注意：使用内存映射区进行通信，是非阻塞的。</font>

### 3.12.3 内存映射使用注意事项

> - 如果对内存映射创建函数`mmap()`的返回值`ptr`做`++`操作，内存映射释放函数`munmap()`是否能够成功？
>
> ```c++
> void *ptr = mmap(...);
> ptr++;		// 可以对内存映射首地址进行++操作
> munmap(ptr,len);	//内存映射释放错误，因为内存映射首地址发生了变化
> ```
>
> - 如果内存映射对应的文件`open`时，权限是`O_RDONLY`，创建内存映射区`mmap()`的`prot`参数指定`PROT_READ | PROT_WRITE`会怎么样？
>
> ```c++
> /*
> 		此时，内存映射区创建失败，mmap()返回MAP_FAILED
> 		open()函数中的权限建议和mmap()函数中的prot参数保持一致，
> 		或者open()的权限大于mmap()
> */
> 
> // 1. 打开文件
> int fd = open("./test_mmap.txt", O_RDONLY);
> if (fd == -1) {
> printf("open file failure...\n");
> exit(0);
> }
> 
> // 2. 创建内存映射区，会失败，返回MAP_FAILED
> int length = lseek(fd, 0, SEEK_END);
> void* ptr = mmap(NULL, length, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
> ```
>
> - 如果文件偏移量为1000会怎么样？
>
> ```c++
> /*
> 		创建内存映射区
> 		- 偏移量 length 是以 byte 为单位的，必须是 OS 页大小 4k 的整数倍
> 		- 如果不是，返回MAP_FAILED
> */
> int length = lseek(fd, 0, SEEK_END);
> void* ptr = mmap(NULL, length, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
> ```
>
> - `mmap()`什么情况下会调用失败？
>
> ```c++
> /*
> 		调用失败的情况
> 		- 第二个参数 length == 0 或者 length 不是 4KBytes 的整数倍
> 		- 第三个参数 prot
> 			- 只指定了写权限
> 			- PROT_READ | PROT_WRITE, 要创建内存映射区，必须有 PROT_READ
> 			- 通过open()函数打开的fd，权限小于第三个参数
> */
> void* ptr = mmap(NULL, length, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
> ```
>
> - 可以`open()`的时候，`O_CREAT`一个新文件来创建映射区吗？
>
> ```c++
> /*
> 		可以的，但是创建文件的大小如果为0的话，肯定不行，可以创建文件之后，使用 truncate() 系统调用，对新的文件进行拓展
> 		
>     #include <unistd.h>
>     #include <sys/types.h>
> 
>     int truncate(const char *path, off_t length);		
>     函数功能：用于截断或者拓展文件的长度，通过 length 指定，如果文件长度大于 length，截断，如果文件长度小于 length，拓展
>     函数参数
>     - path：操作的文件路径名
>     - length：指定截断或者拓展的文件长度
>     返回值
>     - 成功：0
>     - 失败：-1，并且设置错误号
> */
> 
> ```
>
> - `mmap()`后关闭文件描述符，对`mmap()`创建的内存映射区有没有影响？
>
> ```c++
> /*
> 		关闭内存映射对应的文件描述符，其内存映射还是存在
> */
> int fd = open("",ORDWR);
> mmap(,,,,fd,0);
> close(fd);
> ```
>
> - 对内存映射区的`ptr`指针所指向的内存，进行越界访问会怎么样？
>
> ```c++
> // 会分配 4k 的内存，页大小
> void* ptr = (NULL, 100, , , fd, 0);
> 
> // 进行越界操作，是非法的，造成段错误
> char buf[1024] = ptr + 1025;
> ```
>

### 3.12.4 内存映射实现进程间通信案例

```c
/*
		案例1：内存映射实现父子进程通信
		- 第一步：内存映射需要磁盘文件映射到内存中，所以要先有一个磁盘文件，
		可以使用`touch test_mmap.txt`创建磁盘文件，并且向磁盘文件中写入内容，保证其非空。
		- 第二步：打开非空的磁盘文件，获得文件描述符
		- 第三步：在创建子进程之前，创建内存映射区，该内存映射区父子进程共享
		- 第四步：创建子进程
		- 第五步：根据需求，进行父子进程通信
		- 第六步：关闭文件描述符，释放共享内存
*/
#include<stdio.h>
#include<sys/mman.h>
#include<sys/types.h>
#include<sys/wait.h>
#include<unistd.h>
#include<fcntl.h>
#include<string.h>
#include<stdlib.h>

int main() {
    // 1. 打开文件
    int fd = open("./test_mmap.txt", O_RDWR);
    if (fd == -1) {
        printf("open file failure...\n");
        exit(-1);
    }

    // 2. 创建内存映射区
    int length = lseek(fd, 0, SEEK_END);
    printf("length = %d\n", length);
    void* ptr = mmap(NULL, length, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);


    if (ptr == MAP_FAILED) {
        perror("mmap");
        exit(-1);
    }

    // 3. 创建子进程
    pid_t pid = fork();

    // 4. 父进程读取内存映射区数据，子进程向内存映射区写数据
    if (pid > 0) {
        int status_code = wait(NULL);     //等待子进程运行完毕，回收子进程，获取子进程退出时的状态码
        char buf_read[1024] = { '\0' };
        strcpy(buf_read, (char*)ptr);
        printf("read data: %s \nchild exit status_code: %d\n", buf_read, status_code);
    }
    else if (pid == 0) {
        strcpy((char*)ptr, "hello test...");
    }
    else {
        perror("fork");
        exit(-1);
    }

    // 5. 父进程关闭内存映射区，子进程共享文件描述符和内存映射区
    if (pid > 0) {
        close(fd);
        munmap(ptr, length);
        // getchar();
    }

    return 0;
}
```

```c
/*
	案例2：内存映射实现不同进程间的通信
	- 第一步：创建内存映射对应的磁盘文件，并且文件的长度不能为0
	- 第二步：实现通信程序 A，创建对应的内存映射区，程序 A 向内存映射中写数据
	- 第三步：实现通信程序 B，创建对应的内存映射区，程序 B 向内存映射中读数据
	- 第四步：关闭文件描述符，释放对应的内存映射
*/

// 通信程序 A 代码实现如下：
#include<stdio.h>
#include<fcntl.h>
#include<stdlib.h>
#include<unistd.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<sys/mman.h>
#include<string.h>

int main(int argc, char* argv[]) {
    // 1. 打开文件
    int fd = open("./test_mmap.txt", O_RDWR);
    if (fd == -1) {
        perror("open");
        exit(-1);
    }

    // 2. 创建内存映射区
    off_t length = (fd, 0, SEEK_END);
    void* ptr = mmap(NULL, (size_t)length, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if (ptr == MAP_FAILED) {
        perror("mmap");
        exit(-1);
    }
    printf("create mmap success, length = %d\n", (int)length);

    // 3. 通过内存映射区进行通信
    while (1) {
        char write_buf[1024] = { '\0' };
        fgets(write_buf, sizeof(write_buf), stdin);

        // 向内存映射区写数据
        strcpy((char*)ptr, write_buf);
    }

    // 4. 关闭文件描述符，释放内存映射区
    close(fd);
    int ret = munmap(ptr, (size_t)length);
    if (ret == -1) {
        perror("munmap");
        exit(-1);
    }
    return 0;
}

// 通信程序 B 代码实现如下：
#include<stdio.h>
#include<fcntl.h>
#include<stdlib.h>
#include<unistd.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<sys/mman.h>
#include<string.h>

int main(int argc, char* argv[]) {
    // 1. 打开文件
    int fd = open("./test_mmap.txt", O_RDWR);
    if (fd == -1) {
        perror("open");
        exit(-1);
    }

    // 2. 创建内存映射区
    off_t length = (fd, 0, SEEK_END);
    void* ptr = mmap(NULL, (size_t)length, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if (ptr == MAP_FAILED) {
        perror("mmap");
        exit(-1);
    }
    printf("create mmap success, length = %d\n", (int)length);

    // 3. 通过内存映射区进行通信
    while (1) {
        sleep(3);
        char read_buf[1024] = { '\0' };

        // 读取内存映射区的内容
        strcpy(read_buf, (char*)ptr);
        printf("receive data: %s\n", read_buf);
        if (strcmp(read_buf, "break\n") == 0) {
            printf("ending mmap communication.\n");
            break;
        }
    }

    // 4. 关闭文件描述符，释放内存映射区
    close(fd);
    int ret2 = munmap(ptr, (size_t)length);
    if (ret2 == -1) {
        perror("munmap");
        exit(-1);
    }

    return 0;
}
```

```c
/*
		案例3：使用内存映射实现文件拷贝
		基本思路
		- 打开需要拷贝的文件，获取文件长度
		- 创建新的文件，并且使用 truncate 系统调用拓展文件长度
		- 对需要拷贝的文件和新的文件分别进行内存映射
		- 将需要拷贝的文件，对应内存映射中的内容，拷贝到新的文件，对应的内存映射
		- 释放内存映射
		- 关闭打开的文件描述符
*/
#include<stdio.h>
#include<sys/mman.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<unistd.h>
#include<fcntl.h>
#include<string.h>
#include<stdlib.h>

int main(int argc, char* argv[]) {

    // 打开原始文件
    int fd1 = open("./english.txt", O_RDWR);
    if (fd1 == -1) {
        perror("open file failure...");
        exit(0);
    }
    size_t length = lseek(fd1, 0, SEEK_END);

    // 创建一个新的文件，并且拓展该文件
    int fd2 = open("./copy.txt", O_RDWR | O_CREAT);
    if (fd2 == -1) {
        perror("create file failure...");
        exit(0);
    }
    truncate("./copy.txt", length);


    // 分别进行内存映射
    void* ptr1 = mmap(NULL, length, PROT_READ | PROT_WRITE, MAP_SHARED, fd1, 0);
    void* ptr2 = mmap(NULL, length, PROT_READ | PROT_WRITE, MAP_SHARED, fd2, 0);

    // 内存拷贝
    strcpy((char*)ptr2, (char*)ptr1);

    // 释放资源，先申请的资源后释放
    close(fd2);
    close(fd1);
    munmap(ptr1, length);
    munmap(ptr2, length);

    return 0;
}
```

### 3.12.5 内存映射之匿名映射

> <font color = green>匿名映射</font>
>
> - 匿名映射，是不需要文件实体的内存映射
> - **和匿名管道一样，由于没有文件实体，匿名映射只适用于亲子进程间通信**，因为亲子进程间**共享**文件描述符表，可以找到同一块内存映射区

```c
#include<stdio.h>
#include<sys/mman.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<unistd.h>
#include<fcntl.h>
#include<string.h>
#include<stdlib.h>
#include<sys/wait.h>

/*
    匿名内存映射（不依靠文件实体的内存映射）
    - 创建匿名内存映射区
    - 父进程向匿名内存映射区写内容
    - 子进程从匿名内存映射区读内容
    - 释放内存映射资源
*/
int main(int argc, char* argv[]) {
    int len = 4096;

    // 由于没有文件实体，倒数第二个参数 fd 指定 -1，表示匿名映射
    void* ptr = mmap(NULL, len, PROT_READ | PROT_WRITE, MAP_SHARED, -1, 0);

    if (ptr == MAP_FAILED) {
        perror("mmap");
        exit(0);
    }

    pid_t pid = fork();
    if (pid > 0) {
        strcpy((char*)ptr, "hello world...");
        wait(NULL);
    }
    if (pid == 0) {
        sleep(1);
        char buf[1024] = { '\0' };
        strcpy(buf, (char*)ptr);
        printf("%s", buf);
    }

    if (pid > 0) {
        int status = munmap(ptr, len);
        if (status == -1) {
            perror("munmap");
            exit(0);
        }
    }
    return 0;
}
```

## 3.13 信号概述

> <font color = green>信号的概念</font>
>
> - 信号是 Linux 进程间通信的最古老的方式之一，是事件发生时，对进程的**通知机制**，也称之为**软中断**，它是在软件层次上，对中断机制的一种模拟，属于**异步通信**方式，信号可以导致一个正在运行的进程，被另一个正在运行的**异步进程**中断，转而处理某一个突发事件。
> - 发往进程的诸多信号，通常都是**源于内核**。引发内核为进程产生信号的各类事件如下：
>   - 对于前台进程，用户可以通过输入特殊的终端字符，向它发送信号，比如输入`ctrl  + c`会给进程发送一个中断信号。
>   - 硬件检测到一个错误条件（由进程运行时产生），并且发生了异常，会通知内核。然后内核发送相关信号给进程。比如**执行一条异常的机器语言指令（被 0 除等）**，或者**引用了无法访问的内存区域**。
>   - 系统状态变化，比如 alarm 定时器到期，将引起 `SIGALARM`信号，发送给 CPU 执行时间到期的进程。
>   - 运行`kill`指令或者调用`kill`函数。
>
> - 使用信号的主要两个目的
>   - **让进程知道**已经发生了一个特定的事情
>   - **强迫进程执行它自己程序中的信号处理相关代码**
> - 信号的特点
>   - 简单
>   - 不能携带大量信息
>   - 满足某个特定条件才发送
>   - 优先级比较高
> - 查看系统定义的信号列表：`kill -l`，其中前 31 个信号为常规信号，其余信号为实时信号。
>
> <center>
>   <img src = "./images/第三章 Linux多进程开发/3-19 Linux的62个信号.png" width = "90%">
> </center>

### 3.13.1 Linux 信号一览表

| 编号    | 信号名称                           | 对应事件                                                     | 默认动作                   |
| :------ | ---------------------------------- | ------------------------------------------------------------ | -------------------------- |
| 1       | `SIGHUP`                           | 用户退出 shell 时，由该 shell 启动的所有进程将收到这个信号   | 终止进程                   |
| 2       | <font color = red>`SIGINT`</font>  | 当用户按下了 `Ctrl + C` 组合键时，用户终端向正在运行中的由该终端启动的进程发出该信号 | 终止进程                   |
| 3       | <font color = red>`SIGQUIT`</font> | 用户按下 `Ctrl + \ ` 组合键时产生该信号，用户终端向正在运行中的，由该终端启动的进程发出该信号 | 终止进程                   |
| 4       | `SIGILL`                           | CPU 检测到某进程执行了非法指令                               | 终止进程并产生 core 文件   |
| 5       | `SIGTRAP`                          | 该信号由断点指令或其他 trap 指令产生                         | 终止进程并产生 core 文件   |
| 6       | `SIGABRT`                          | 调用 abort 函数时产生该信号                                  | 终止进程并产生 core 文件   |
| 7       | `SIGBUS`                           | 非法访问内存地址，包括内存对齐出错                           | 终止进程并产生 core 文件   |
| 8       | `SIGFPE`                           | 发生运算错误时发出，包括浮点运算错误、溢出及除数为 0 等所有的算法错误 | 终止进程并产生 core 文件   |
| 9       | <font color = red>`SIGKILL`</font> | 无条件终止进程，该信号不能被忽略、处理和阻塞                 | 终止进程，可以杀死任何进程 |
| 10      | `SIGUSE1`                          | 用户定义的信号，即程序员可以在程序中定义并使用该信号         | 终止进程                   |
| 11      | <font color = red>`SIGSEGV`</font> | 指示进程进行了无效内存访问（段错误）                         | 终止进程并产生 core 文件   |
| 12      | `SIGUSR2`                          | 另外一个用户自定义信号，程序员可以在程序中定义并使用该信号   | 终止进程                   |
| 13      | <font color = red>`SIGPIPE`</font> | Broken pipe 向一个没有读端的管道写数据                       | 终止进程                   |
| 14      | `SIGALRM`                          | 定时器超时，超时的时间由系统调用 alarm 设置                  | 终止进程                   |
| 15      | <font color = red>`SIGTERM`</font> | 程序结束信号，与 `SIGKILL` 不同的是，该信号可以被阻塞和终止。通常用来要求程序正常退出，执行 shell 命令 kill 时，缺省产生这个信号 | 终止进程                   |
| 16      | `SIGSTKFLT`                        | Linux早期版本出现的信号，现仍保留向后兼容                    | 终止进程                   |
| 17      | <font color = red>`SIGCHLD`</font> | 子进程结束时，父进程会收到这个信号                           | 忽略这个信号               |
| 18      | <font color = red>`SIGCONT`</font> | 如果进程已停止，则使其继续运行                               | 继续 / 忽略                |
| 19      | <font color = red>`SIGSTOP`</font> | 停止进程的执行，信号不能被忽略，处理和阻塞，收到 `SIGCONT` 后进程继续运行 | 暂停进程                   |
| 20      | `SIGTSTP`                          | 停止终端交互进程的运行，按下 `Ctrl+Z` 组合键时发出这个信号   | 暂停进程                   |
| 21      | `SIGTTIN`                          | 后台进程读终端控制台                                         | 暂停进程                   |
| 22      | `SIGTTOU`                          | 该信号类似于 `SIGTTIN`，在后台进程要向终端输出数据时发生     | 暂停进程                   |
| 23      | `SIGURG`                           | 套接字上有紧急数据时，向当前正在运行的进程发出些信号，报告有紧急数据到达，如网络带外数据到达 | 忽略该信号                 |
| 24      | `SIGXCPU`                          | 进程执行时间超过了分配给该进程的 CPU 时间，系统产生该信号并发送给该进程 | 终止进程                   |
| 25      | `SIGXFSZ`                          | 超过文件的最大长度设置                                       | 终止进程                   |
| 26      | `SIGVTALRM`                        | 虚拟时钟超时时产生该信号，类似于 `SIGALRM`，但是该信号只计算该进程占用 CPU 的使用时间 | 终止进程                   |
| 27      | `SIGPROF`                          | 类似于 `SIGVTALRM`，它不仅包括该进程占用CPU时间还包括执行系统调用时间 | 终止进程                   |
| 28      | `SIGWINCH`                         | 当前终端窗口大小发生变化时发出                               | 忽略该信号                 |
| 29      | `SIGIO`                            | 此信号向进程指示发出了一个异步IO事件                         | 忽略该信号                 |
| 30      | `SIGPWR`                           | 关机                                                         | 终止进程                   |
| 31      | `SIGSYS`                           | 无效的系统调用                                               | 终止进程并产生 core 文件   |
| 34 ~ 64 | `SIGRTMIN` ~ `SIGRTMAX`            | LINUX 的实时信号，它们没有固定的含义（可以由用户自定义）     | 终止进程                   |

> <font color= red>注意：</font>向进程发送编号为 0 的信号，可以
>
> - **检查进程是否存在**：使用 `int ret = kill(pid, 0);` 函数实现，如果 `ret == -1` 表示进程不存在，否则，进程号为 `pid` 的进程存在；
> - **权限检查**：确定当前进程是否有权限影响目标进程。

### 3.13.2 信号的 5 种默认处理动作

> - 查看信号的详细信息：`man 7 signal`
> - 信号的 5 种默认处理动作
>   - `Term`：终止进程
>   - `Ign`：当前进程忽略掉这个信号
>   - `Core`：终止进程，并且生成一个 Core 文件
>   - `Stop`：暂停当前进程
>   - `Cont`：继续执行当前被暂停的进程
>
> - 信号的几种状态：产生，未决（即产生了，还没有发送给进程的信号），递达
> - `SIGKILL`和`SIGSTOP`信号不能被捕捉、阻塞或者忽略，只能执行默认动作

### 3.13.3 信号相关的函数

- `kill()`系统调用

```c
#include <sys/types.h>
#include <signal.h>
int kill(pid_t pid, int sig);
/*
    函数功能：给任何进程或者进程组pid, 发送任何的信号 sig
    函数参数
    - pid：
        - 大于0：信号发送给指定进程
        - 等于0：信号发送给当前进程组
        - 等于-1：信号发送给每一个有权限接收该信号的进程
        - 小于-1：信号发送给进程组ID = -pid 的进程
    - sig：需要发送的信号编号或者是宏值
        - 等于0：不发送任何信号
    返回值
    - 0：成功
    - -1：失败
*/

/*
		使用例子：父进程通过 kill() 向子进程发送 SIGINT 信号
*/
#include<stdio.h>
#include<stdlib.h>
#include <sys/types.h>
#include <signal.h>
#include<string.h>
#include<unistd.h>

int main(int argc, char* argv[]) {
    pid_t pid = fork();

    if (pid == 0) {
        // 子进程
        int i;
        for (i = 0;i < 5;++i) {
            printf("child process\n");
            sleep(1);
        }
    }
    else if (pid > 0) {
        // 父进程
        printf("parent process\n");
        sleep(2);
        printf("kill child process now\n");
        kill(pid, SIGINT);
    }
    return 0;
}
```

- `raise()`系统调用

```c
#include<signal.h>
int raise(int sig);
/*
    函数功能：给当前进程发送信号
    函数参数
    - sig：需要发送的信号编号或者是宏值
        - 等于0：不发送任何信号
    返回值
    - 0：成功
    - 非0：失败
    
    等价于 kill(getpid(),sig);
*/

```

- `abort()`系统调用

```c
#include <stdlib.h>
void abort(void);
/*
    函数功能：发送 SIGABRT 信号给当前的进程，杀死当前进程

    等价于 kill(getpid(), SIGABRT);
*/
```

- `alarm()`系统调用

```c
#include <unistd.h>
unsigned int alarm(unsigned int seconds);
/*
    函数功能：设置定时器，函数调用，开始倒计时，当倒计时为 0 的时候，函数会给当前的进程发送一个信号：SIGALRM
    函数参数
    - seconds：定时器时长，单位：秒，如果参数为0，定时器无效（不进行倒计时）
        - 可以通过alarm(0)取消一个定时器
    返回值
    - alarm()之前没有定时器，返回0
    - alarm()之前有定时器，返回之前的定时器剩余的时间

    alarm()不阻塞进程，定时结束之后，给调用的进程 SIGALRM 信号，默认终止进程（定时的进行和进程状态没关系）
*/

/*
		使用例子：给当前进程设置两个定时器，当一个定时器结束，发送 SIGALRM 信号，终止进程
*/
#include<unistd.h>
#include<stdio.h>

int main(int argc, char* argv[]) {
    int seconds = alarm(5);
    printf("seconds = %d\n", seconds);

    sleep(1);

    seconds = alarm(2);
    printf("seconds = %d\n", seconds);

    while (1);
    return 0;
}
```

- `setitimer()`系统调用

```c
#include <sys/time.h>
int setitimer(int which, const struct itimerval *new_value, struct itimerval *old_value);
/*
    函数功能：间隔定时器，定时器时间到，向调用定时器的进程发送终止信号，可以替代alarm函数，可以实现周期定时
    函数参数
    - which：定时器以什么时间计时
        - ITIMER_REAL：定时器记录进程的真实时间（进程运行的所有阶段时间，包括IO，用户态，核心态等），时间到达，发送 SIGALRM 信号
        - ITIMER_VIRTUAL：定时器记录进程占用CPU的使用时间，时间到达，发送 SIGVTALRM
        - ITIMER_PROF：定时器记录进程占用CPU的时间和系统调用的时间，时间到达，发送 SIGPROF
    - new_value：设置定时器的属性
        struct itimerval {      // 定时器结构体
            struct timeval it_interval;     // 每个阶段的间隔时间，即每个定时器执行的间隔时间
            struct timeval it_value;        // 延迟多长时间执行定时器
        };
        struct timeval {        // 时间结构体
            time_t      tv_sec;     // 秒
            suseconds_t tv_usec;    // 微秒
        };
    - old_value：记录上一次的定时器时间参数，一般不使用，赋值NULL即可
    返回值
    - 定时器设置成功，返回 0
    - 失败，返回 -1
*/

/*
		使用例子：通过 setitmer 实现 2s 定时，间隔 3s 执行
		- setitimer 和 alarm 一样，是非阻塞的
*/
#include <sys/time.h>
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>

int main(int argc, char* argv[]) {

    // 创建定时器参数
    struct itimerval new_value;

    // 设置定时器参数
    new_value.it_interval.tv_sec = 3;       // 定时器执行间隔时间
    new_value.it_interval.tv_usec = 0;

    new_value.it_value.tv_sec = 2;          // 设置每一次定时的有效时间
    new_value.it_value.tv_usec = 0;

    // 创建定时器，非阻塞
    int ret = setitimer(ITIMER_REAL, &new_value, NULL);
    printf("setitimer is starting...\n");

    if (ret == -1) {
        perror("setitimer");
        exit(0);
    }

    while (1) {
        sleep(1);
        printf("setitimer is working.....\n");
    }
    return 0;
}
```

### 3.13.4 `signal()`系统调用

> `signal()`系统调用的主要功能是，**对某一个信号进行捕捉，并且可以自定义捕捉到信号的行为**。

```c
#include <signal.h>
typedef void (*sighandler_t)(int);      // 定义了一个名为sighandler_t，返回值为void，参数列表为int的函数指针
sighandler_t signal(int signum, sighandler_t handler);
/*
    函数功能：设置某一个信号的捕捉之后的行为
    函数参数
    - signum：要捕捉的信号
    - handler：对捕捉到的信号要如何处理
        - SIG_IGN：忽略捕捉到的信号
        - SIG_DEL：使用信号默认的行为
        - 自定义回调函数
            - 内核调用，开发人员自定义函数规则
            - 函数指针是实现回调的手段，函数名表示的就是函数所在内存空间中的地址
    返回值
    - 成功：返回上一次信号处理函数的地址，第一次调用，返回NULL
    - 失败：返回SIG_ERR，设置错误号
*/

/*
		使用例子
		- 使用 signal() 捕捉 SIGALRM 信号
		
*/
#include <sys/time.h>
#include<stdio.h>
#include<stdlib.h>
#include<signal.h>
#include<unistd.h>

void my_alarm(int sig_num) {
    printf("The number of catching signal is %d.\n", sig_num);
  	exit(0);
}

int main(int argc, char* argv[]) {

    // 注册信号捕捉
    //signal(SIGALRM, SIG_DFL);   // 信号默认行为
    //signal(SIGALRM, SIG_IGN);   // 忽略信号的默认行为
    signal(SIGALRM, my_alarm);        // 自定义回调函数处理信号

    // 创建定时器参数
    struct itimerval new_value;

    // 设置定时器参数
    new_value.it_interval.tv_sec = 0;       // 定时器执行间隔时间，设置为0，定时器setitimer只执行一次
    new_value.it_interval.tv_usec = 0;

    new_value.it_value.tv_sec = 3;          // 设置每一次定时的有效时间
    new_value.it_value.tv_usec = 0;

    // 创建定时器，非阻塞
    int ret = setitimer(ITIMER_REAL, &new_value, NULL);
    printf("setitimer is starting...\n");

    if (ret == -1) {
        perror("setitimer");
        exit(0);
    }

    while (1) {
        sleep(1);
    }

    return 0;
}
```

- `sigaction()`系统调用，涉及到**信号集**的知识

```c
/*
    #include <signal.h>
    int sigaction(int signum, const struct sigaction *act,
                    struct sigaction *oldact);

    函数功能：和 signal() 函数一样，进行信号捕捉，并且可以修改信号对应的行为

    函数参数
    - signum：捕捉到的信号编号，建议使用宏值来表示
    - act：捕捉到信号之后的处理动作
    - oldact：上一次对信号捕捉相关的设置，一般不使用，传递NULL

    返回值
    - 成功：0
    - 失败：-1

    struct sigaction {
        // 函数指针，指向的函数就是信号捕捉到之后的行为
        void     (*sa_handler)(int);

        // 不常用
        void     (*sa_sigaction)(int, siginfo_t *, void *);

        // 临时阻塞信号集，信号捕捉函数执行过程中，临时阻塞某些信号
        sigset_t   sa_mask;

        // 指定捕捉到的信号行为
        // 值为 0，表示使用 sa_handler，值为 SA_SIGINFO 表示使用 sa_sigaction
        int        sa_flags;

        // 被废弃，不需要指定
        void     (*sa_restorer)(void);
    };

*/
```

### 3.13.5 信号集概述

> - 许多信号相关的系统调用，都需要能表示一组不同的信号，**多个信号可以使用一个称之为信号集的数据结构来表示**，信号集的数据类型为`sigset_t`。
>
> - 在 PCB 中，有两个非常重要的信号集。一个称之为**阻塞信号集**，另一个称之为**未决信号集**。这两个信号集都是内核**使用位图机制来实现的**。但 OS 不允许我们直接对这两个信号集进行位操作。而**需要自定义另外一个集合，借助信号集操作函数**，来对 PCB 中的这两个信号集进行修改。
>
> - 信号的<font color = red>未决</font>是一种状态，指的是**信号的产生到信号被处理前的这一段时间**。
>
> - 信号的<font color = red>阻塞</font>是一个开关动作，指的是**阻止信号被处理，而不是阻止信号产生**。
>
> - 信号的<font color =red>阻塞</font>是让系统**暂时保留信号，待以后发送**。由于另外有办法让系统忽略信号，所以一般情况下，**信号的阻塞只是暂时的**，只是为了防止信号打断敏感的操作。
>
> - 阻塞信号集和未决信号集
>
>   - 用户通过键盘`Ctrl + C`，产生2号信号`SIGINT`，此时信号被创建
>
>   - 信号产生但是没有被处理，**属于未决状态**
>
>     - 在内核中，**将所有没有被处理的信号**存储在一个集合中，**这个集合叫未决信号集**
>
>     - `SIGNINT`信号的状态存储在**未决信号集第二个标志位上**，标志位为 0，说明信号**不是未决状态**，标志位为 1，说明信号**处于未决状态**
>
>   - 未决状态的信号，需要被处理，处理之前需要和另一个信号集（阻塞信号集），进行比较
>
>     - 阻塞信号集默认不阻塞任何的信号
>     - 如果想要阻塞某个信号，需要用户**调用系统的 API **
>
>   - 在处理未决信号集的时候，会查询**阻塞信号集中的标志位**，看是不是将**对应的信号标志位设置了阻塞（也就是阻塞信号集中，对应信号标志位是否为 1）**
>
>     - 如果**没有阻塞**，这个信号就被处理
>     - 如果**阻塞**了，这个信号就**继续处于未决状态**，直到阻塞信号集中，**对应的标志位为0**，阻塞解除，这个未决信号就被处理
>
> <center>
>   <img src = "./images/第三章 Linux多进程开发/3-24 阻塞信号集和未决信号集1.png" width = "90%">
> </center>

### 3.13.6 信号集相关函数

```c
#include <signal.h>

/*
	函数功能：清空阻塞信号集中的数据，将信号集中所有的标志位置为 0
  函数参数
	  - set：传出参数，需要清空的信号集地址
  返回值：成功返回0，失败返回-1
*/
int sigemptyset(sigset_t *set);

/*
	函数功能：将阻塞信号集中所有的标志位置1
	函数参数
		- set：传出参数，需要填充的信号集地址
	返回值：成功返回0，失败返回-1
*/
int sigfillset(sigset_t *set);

/*
	函数功能：设置阻塞信号集中的某一位对应的标志位为 1，表示阻塞这个信号
	函数参数
		- set：传出参数，需要操作的信号集
		- signum：需要设置阻塞的信号编号
	返回值：成功返回0，失败返回-1
*/
int sigaddset(sigset_t *set, int signum);

/*
	函数功能：设置阻塞信号集中的某一位对应的标志位为 0，表示不阻塞这个信号
	函数参数
		- set：传出参数，需要操作的信号集
		- signum：需要设置不阻塞的信号编号
	返回值：成功返回0，失败返回-1
*/
int sigdelset(sigset_t *set, int signum);

/*
	函数功能：判断某个信号是否阻塞
	函数参数
		- set：需要操作的信号集
		- signum：需要判断的那个信号编号
	返回值
		1：signum被阻塞
		0：signum不阻塞
		-1：函数调用失败
*/
int sigismember(const sigset_t *set, int signum);

/*
		信号集相关函数使用例子
*/
#include<stdio.h>
#include<signal.h>

int main(int argc, char* argv[]) {
    // 自定义阻塞信号集，阻塞的意思是，是否处理进程收到的信号，不处理是阻塞，处理是非阻塞
    sigset_t set;

    // 清空阻塞信号集中的内容
    sigemptyset(&set);

    // 判断 SIGINT 是否在阻塞信号集 set 里
    int ret1 = sigismember(&set, SIGINT);
    if (ret1 == 0) {
        printf("SIGINT 不阻塞\n");
    }
    else {
        printf("SIGINT 阻塞\n");
    }

    // 添加几个信号到阻塞信号集中
    sigaddset(&set, SIGINT);
    sigaddset(&set, SIGQUIT);

    // 判断 SIGINT 是否在阻塞信号集 set 里
    int ret2 = sigismember(&set, SIGINT);
    if (ret2 == 0) {
        printf("SIGINT 不阻塞\n");
    }
    else {
        printf("SIGINT 阻塞\n");
    }

    // 判断 SIGQUIT 是否在阻塞信号集 set 里
    int ret3 = sigismember(&set, SIGQUIT);
    if (ret3 == 0) {
        printf("SIGQUIT 不阻塞\n");
    }
    else {
        printf("SIGQUIT 阻塞\n");
    }

    // 从阻塞信号集中删除一个信号
    sigdelset(&set, SIGQUIT);

    // 判断 SIGQUIT 是否在阻塞信号集 set 里
    int ret4 = sigismember(&set, SIGQUIT);
    if (ret4 == 0) {
        printf("SIGQUIT 不阻塞\n");
    }
    else {
        printf("SIGQUIT 阻塞\n");
    }

    return 0;
}
```

### 3.13.7 内核实现信号捕捉的过程

UNIX OS 中，**内核实现信号捕捉的过程**，可以用下图直观的表示。

<center>
<img src = "./images/第三章 Linux多进程开发/3-26 内核实现信号捕捉的过程.png" width = "90%">
</center>

### 3.13.8 `sigprocmask()`系统调用

> `sigpromask()`系统调用，主要的功能是将自定义的阻塞信号集**所有标志位**，设置到**内核阻塞信号集中**。
>
> ```c
> #include <signal.h>
> 
> /*
> 	函数功能：将自定义信号集中的数据设置到内核中（设置阻塞，解除阻塞，替换）
> 	函数参数
> 		- how：如何对内核阻塞信号集进行处理
> 			- SIG_BLOCK：将用户设置的阻塞信号集添加到内核中，内核中原来的数据不变，假设用户自定义信号集是 set，内核中阻塞信号集是 mask， mask = mask | set
> 			- SIG_UNBLOCK：根据用户设置的数据，对内核中的数据进行解除阻塞，假设用户自定义信号集是 set，内核中阻塞信号集是 mask，mask = mask & (~set)
> 			- SIG_SETMASK：覆盖内核中原来的值
> 
> 		- set：用户自定义的阻塞信号集
> 		- oldset：备份（存储）修改之前的内核中阻塞信号集内容
> 	返回值
> 		- 成功：0
> 		- 失败：-1，设置错误号，EFAULT 或 EINVAL
> */
> int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);
> 
> /*
> 	函数功能：获取内核中未决信号集，pending -> adj.未决定的
> 	函数参数
> 		- set：传出参数，保存的是内核中未决信号集中的信息
> 	返回值
> 		- 成功：0
> 		- 失败：-1，设置错误号，EFAULT
> */
> int sigpending(sigset_t *set);
> 
> /*
> 		使用例子：编写一个程序，把所有常规信号（1 ~ 31）的未决状态打印出来
> 		- 要求1：设置某些通过键盘产生的信号是阻塞的
> 				- 比如 SIGINT 和 SIGQUIT 信号，终止进程
> 		- 要求2：对设置阻塞的信号，其中一个解除阻塞
> 				- 如果信号在内核阻塞信号集中，值为 0，内核未决
> 				信号集中，值为 1，那么该信号会被处理
> */
> #include<stdio.h>
> #include<signal.h>
> #include<stdlib.h>
> #include<unistd.h>
> 
> int main(int argc, char* argv[]) {
>     // 自定义阻塞信号集
>     sigset_t my_set;
> 
>     sigemptyset(&my_set);
> 
>     // 设置由键盘产生的2、3号信号阻塞
>     sigaddset(&my_set, SIGINT);
>     sigaddset(&my_set, SIGQUIT);
> 
>     // 将自定义信号集设置到内核阻塞信号集中
>     sigprocmask(SIG_BLOCK, &my_set, NULL);
> 
>     int num = 0;
>     while (1) {
>         ++num;
> 
>         // 获取内核中未决状态信号集的内容
>         int ret = sigpending(&my_set);
> 
>         if (ret == -1) {
>             perror("sigpending");
>             exit(0);
>         }
> 
>         sleep(1);
> 
>         // 访问未决信号集中的内容
>         for (int i = 1;i <= 31;++i) {
>             printf("%d", sigismember(&my_set, i));
>         }
>         printf("\n");
> 
>         if (num >= 10) {
>             // 解除阻塞信号
>             sigdelset(&my_set, SIGINT);     // 对 SIGINT 信号还是阻塞
>           	// 这里要知道，当参数是 SIG_UNBLOCK，内核阻塞信号集是如何被复制的
>             sigprocmask(SIG_UNBLOCK, &my_set, NULL);
>         }
>     }
> 
>     return 0;
> }
> ```

### 3.13.9 `sigaction()`系统调用

> `sigaction()`系统调用的主要功能是：**和`signal()`系统调用一样，进行信号捕捉，并且可以自定义捕捉到信号的行为**。
>
> - 与`signal()`系统调用不同的是，`sigaction()`系统调用可以**通过对内核阻塞信号集的赋值，自定义需要阻塞的信号**。

```c
#include <signal.h>

/*
	函数功能：和 signal() 函数一样，进行信号捕捉，并且可以修改信号对应的行为
	函数参数
		- signum：捕捉到的信号编号，建议使用宏值来表示
    - act：捕捉到信号之后的处理动作
    - oldact：上一次对信号捕捉相关的设置，一般不使用，传递NULL
	返回值
    - 成功：0
    - 失败：-1
    
  struct sigaction {
    // 函数指针，指向的函数就是信号捕捉到之后的行为
    void(*sa_handler)(int);

    // 不常用
    void(*sa_sigaction)(int, siginfo_t *, void *);

    // 临时阻塞信号集，信号捕捉函数执行过程中，临时阻塞某些信号
    sigset_t sa_mask;

    // 指定捕捉到的信号行为
    // 值为 0，表示使用 sa_handler，值为 SA_SIGINFO 表示使用 sa_sigaction
    int sa_flags;

    // 被废弃，不需要指定
    void(*sa_restorer)(void);
  };
  
*/
int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);

#include <sys/time.h>
#include<stdio.h>
#include<stdlib.h>
#include<signal.h>
#include<unistd.h>

/*
    通过 setitimer 实现 2s 的定时，间隔 3s 执行
    通过 sigaction 捕捉信号，实现周期定时
*/
void my_alarm(int sig_num) {
    printf("The number of catching signal is %d, the process will exit.\n", sig_num);
    exit(-1);
}

int main(int argc, char* argv[]) {

    // 注册信号捕捉
    struct sigaction act;
    act.sa_flags = 0;
    act.sa_handler = my_alarm;
    sigemptyset(&act.sa_mask);
    sigaction(SIGALRM, &act, NULL);

    // 创建定时器参数
    struct itimerval new_value;

    // 设置定时器参数
    new_value.it_interval.tv_sec = 3;       // 定时器执行间隔时间，设置为0，定时器setitimer只执行一次
    new_value.it_interval.tv_usec = 0;

    new_value.it_value.tv_sec = 2;          // 设置每一次定时的有效时间
    new_value.it_value.tv_usec = 0;

    // 创建定时器，非阻塞
    int ret = setitimer(ITIMER_REAL, &new_value, NULL);
    printf("setitimer is starting...\n");

    if (ret == -1) {
        perror("setitimer");
        exit(0);
    }

    while (1);

    return 0;
}
```

### 3.13.10 `SIGCHLD`信号

> <font color = green>`SIGCHLD`信号产生的条件是</font>：**子进程的运行状态发生了三种变化之一，向父进程发送的信号**，这三种状态分别是
>
> - 子进程<font color = green>结束</font>
> - 子进程<font color = green>暂停</font>
> - 子进程<font color = green>从暂停到继续运行</font>
>
> 然而，在 Linux 内核中，父进程接收到了子进程发来的`SIGCHLD`信号，**会选择默认忽略该信号**。

```c
/*
    SIGCHILD 信号产生的3个条件
    - 子进程结束
    - 子进程暂停
    - 子进程从暂停到继续运行

    都会给父进程发送该信号，父进程默认忽略该信号
*/

#include<stdio.h>
#include<signal.h>
#include<unistd.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<sys/wait.h>

/*
    子进程回收方式演变
    - 方式1：wait(NULL)
        - 出现的问题：由于信号捕捉机制，如果有多个子进程运行结束，都给父进程发送 SIGCHLD 信号，但是内核未决信号集中，只能记录一个 SIGCHLD 信号，会遗漏其他的子进程发送的信号

    - 方式2：while(1){ wait(NULL);}
        - 出现的问题：可以回收所有子进程资源，但是没有子进程结束运行的时候，会阻塞
    - 方式3：while(1){waitpid(-1, NULL, WNOHANG);}
        - 不但可以回收所有子进程资源，还是非阻塞

*/
void my_func(int num) {
    printf("SIGCHLD num is %d\n", num);
    // 方式 1
    //wait(NULL);

    // 方式 2
    // while (1) {
    //     int ret = wait(NULL);
    //     if (ret == -1) {
    //         // 没有可以回收的子进程资源
    //         break;
    //     }
    // }

    // 方式 3
    while (1) {
        int ret = waitpid(-1, NULL, WNOHANG);
        if (ret > 0) {
            // 回收一个子进程
            printf("child process die, pid = %d\n", ret);
        }
        else if (ret == 0) {
            // 还存在子进程没有运行结束
            break;
        }
        else if (ret == -1) {
            // 所有子进程都运行结束，且没有可以回收的子进程资源
            break;
        }
    }
}

int main(int argc, char* argv[]) {

    // 提前设置好阻塞信号集，阻塞SIGCHLD，因为有可能子进程很快结束，父进程没有注册完信号捕捉
    sigset_t my_set;
    sigemptyset(&my_set);
    sigaddset(&my_set, SIGCHLD);

    // 自定义阻塞信号集赋值给内核阻塞信号集
    sigprocmask(SIG_BLOCK, &my_set, NULL);

    pid_t pid;

    // 创建一些子进程
    for (int i = 0;i < 20;++i) {
        pid = fork();
        if (pid == 0) {
            break;      //子进程退出循环，防止嵌套创建子进程
        }
    }

    if (pid > 0) {

        // 捕捉子进程运行结束发送的 SIGCHLD 信号
        struct sigaction act;
        act.sa_flags = 0;
        act.sa_handler = my_func;
        sigaction(SIGCHLD, &act, NULL);

        // 父进程
        while (1) {
            // 注册完信号捕捉以后，解除阻塞
            sigprocmask(SIG_UNBLOCK, &my_set, NULL);
            sleep(1);
            printf("parent process pid: %d\n", getpid());
        }
    }
    else if (pid == 0) {
        // 子进程
        printf("child process pid: %d\n", getpid());
    }
    return 0;
}
```

## 3.14 共享内存

> - <font color = green>共享内存</font>允许两个或者多个进程**共享物理内存的同一块区域（通常被称为段）**。由于共享内存段是进程用户空间的一部分，因此这种 IPC 机制**无需内核介入**。所有需要做的就是，让一个进程将数据**复制进共享内存中**，并且这部分数据，对于其他共享同一个段的进程，也可以访问。
> - 与管道等要求发送进程将数据从用户空间的缓冲区复制进内核内存，接收进程将数据从内核内存复制进用户空间的缓冲区这种做法相比，**内存共享这种 IPC 技术速度更快**。

### 3.14.1 共享内存操作命令

> <font color = green>ipcs 用法</font>
>
> ```bash
> ipcs -a				# 打印当前系统中，所有进程间通信方式的信息
> ipcs -m				# 打印出使用共享内存进行进程间通信的信息
> ipcs -q				# 打印出使用消息队列进行进程间通信的信息
> ipcs -s				# 打印出使用信号进行进程间通信的信息
> ```
>
> <font color = green>ipcrm 用法</font>
>
> ```bash
> ipcrm -M shmkey				# 移除用 shmkey 创建的共享内存段
> ipcrm -m shmid				# 移除用 shmid 标识的共享内存段
> ipcrm -Q msqkey				# 移除用 msqkey 创建的消息队列
> ipcrm -q msqid				# 移除用 msqid 标识的消息队列
> ipcrm -S semkey				# 移除用 semkey 创建的信号
> ipcrm -s semid 				# 移除用 semid 标识的信号
> ```

### 3.14.2 共享内存使用步骤

> - 调用`shmget()`**创建**一个新的共享内存段，或者取得一个**既有共享内存段的标识符（其他进程创建的共享内存段）**。`shmget()`返回**共享内存的标识符**。
> - 调用`shmat()`**关联共享内存段**，使共享内存段成为**该进程虚拟内存的一部分**。`shmat()`返回一个指针，指向**进程虚拟地址空间中，共享内存段的起始地址**。
> - 通过`shmat()`返回的指针，可以实现对共享内存的读写操作
> - 调用`shmdt()`来**分离共享内存段**。分离共享内存段之后，**进程就无法再引用这块共享内存了**。这一步是可选，**因为进程终止时，会自动完成这一步**。
> - 调用`shmctl()`来**删除共享内存段**。只有当所有与共享内存段关联的进程，**都与共享内存段分离完成**，`shmctl()`才会真正的删除共享内存段，释放内存资源。否则，`shmctl()`只是标记共享内存段，待后续删除。

### 3.14.3 共享内存系统调用

> Linux 中，提供了**一系列共享内存相关的系统调用**。

```c
#include <sys/ipc.h>
#include <sys/shm.h>

/*
	函数功能：创建一个新的共享内存段，或者获取一个既有的共享内存段的标识，新创建内存段中的数据都会被初始化为 0
	函数参数
    - key：整型数据，通过 key 找到或者创建一个共享内存，一般使用16进制
    - size：共享内存的大小
    - shmflg：属性
        - 访问权限
        - 附加属性：创建/判断共享内存是不是存在
            - 创建：IPC_CREATE
            - 判断共享内存是否存在：IPC_EXCL，需要和 IPC_CREATE 一起使用
            - IPC_EXCL | IPC_CREATE
	返回值
    - 成功：>0，返回共享内存的引用ID，操作共享内存通过这个值
    - 失败：-1，设置错误号
*/
int shmget(key_t key, size_t size, int shmflg);

/*
	函数功能：将共享内存和当前的进程进行关联
	函数参数
    - shmid：共享内存的标识（ID），由 shmget 返回值获取
    - shmaddr：申请的共享内存起始地址，一般指定NULL，由内核决定
    - shmflg：对共享内存的操作
        - 读权限：SHM_RDONLY，必须要有读权限
        - 读写权限：0
	返回值
    - 成功：返回共享内存的起始地址
    - 失败：返回(void*)-1，并且设置错误号
*/
void *shmat(int shmid, const void *shmaddr, int shmflg);

/*
	函数功能：解除当前进程和共享内存的关联
	函数参数
		- shmaddr：共享内存的首地址
	返回值
		- 成功：返回0
    - 失败：返回-1，并且设置错误号
*/
int shmdt(const void *shmaddr);

/*
	函数功能：删除共享内存，共享内存要删除才会消失，创建共享内存的进程被销毁，对共享内存没有任何影响
	函数参数
    - shmid：共享内存的ID
    - cmd：要做的操作
        - IPC_STAT：获取共享内存的当前状态
        - IPC_SET：设置共享内存的状态
        - IPC_RMID：标记当前进程，关联共享内存被解除（只有等到所有的进程都与共享内存解除关联了，共享内存才能被销毁）
        - IPC_INFO：返回当前 OS 的共享内存限制和 buf 所指向的共享内存结构体信息
    - buf：需要设置或者获取共享内存中的属性信息
        - cmd = IPC_STAT：传出参数，获取共享内存的属性信息
        - cmd = IPC_SET：设置共享内存中的属性信息到内核中
        - cmd = IPC_RMID：设置NULL
	返回值
		- 成功：返回0
    - 失败：返回-1，并且设置错误号    
*/
int shmctl(int shmid, int cmd, struct shmid_ds *buf);

/*
	函数功能：根据指定的路径名和 int 值，生成一个共享内存的 key
	函数参数
  	- pathname：指定一个存在的路径
  	- proj_id：int类型的值，但是这个系统调用只会使用其中的 8bits
  		- 范围：0~255，一般指定一个字符'a'
	返回值
  	- 成功：返回生成的共享内存 key
  	- 失败：返回-1，并且设置错误号
*/
key_t ftok(const char* pathname, int proj_id);
```

### 3.14.4 共享内存使用案例

<font color = green>创建两个进程</font>，一个进程向共享内存中写数据，另一个进程从共享内存中读数据。

```c
/*
	进程1：向共享内存中写数据
*/
#include<sys/ipc.h>
#include<sys/shm.h>
#include<sys/types.h>
#include<stdio.h>
#include<string.h>

int main(int argc, char* argv[]) {
 // 创建共享内存
 int shm_id = shmget(100, 4096, IPC_CREAT | IPC_EXCL);
 printf("write shm_id = %d\n", shm_id);

 // 关联共享内存
 void* shm_ptr = shmat(shm_id, NULL, 0);

 // 向共享内存中写数据
 char* buf = "hello world\n";
 strcpy((char*)shm_ptr, buf);

 printf("press any key to exit write process\n");
 getchar();

 // 解除关联
 shmdt(shm_ptr);

 // 删除共享内存
 shmctl(shm_id, IPC_RMID, NULL);

 return 0;
}
```

```c
/*
	进程2：从共享内存中读数据
*/
#include<sys/ipc.h>
#include<sys/shm.h>
#include<sys/types.h>
#include<stdio.h>
#include<string.h>

int main(int argc, char* argv[]) {
 // 创建共享内存
 int shm_id = shmget(100, 0, IPC_CREAT);
 printf("read shm_id = %d\n", shm_id);

 // 关联共享内存
 void* shm_ptr = shmat(shm_id, NULL, 0);

 // 读取共享内存中的信息
 char buf[4096] = { '\0' };
 strcpy(buf, (char*)shm_ptr);
 printf("read shm: %s", buf);

 printf("press any key to exit read process\n");
 getchar();

 // 解除关联
 shmdt(shm_ptr);

 // 删除共享内存
 shmctl(shm_id, IPC_RMID, NULL);

 return 0;
}
```

### 3.14.5 共享内存问题总结

> <font color = green>问题1：</font>操作系统如何知道一块共享内存**被多少个进程关联**？
>
> - 共享内存维护了一个结构体`struct shmid_ds`，这个结构体中有一个成员变量`shm_nattach`，记录了共享内存所关联的进程数。
>
> <font color = green>问题2：</font>可不可以使用`shmctl()`对共享内存进行**多次删除**？
>
> - 结论是可以的，**因为`shmctl()`是标记删除共享内存**，不是物理意义上的删除，只有所有关联共享内存的进程**执行`shmdt()`系统调用（解除关联，即共享内存的关联数为0）**，执行`shmctl()`才是物理意义上的删除共享内存。
> - **当共享内存的`key`值被标记为 0 时**，一个进程执行`shmdt()`和该共享内存**取消关联后**，**不能够再次执行`shmat()`和该共享内存进行关联**。
>
> <font color = green>问题3：</font>共享内存和内存映射的区别？
>
> - **共享内存**可以直接创建，**内存映射**需要磁盘文件（匿名映射除外）。
> - **共享内存**效率更高，不需要和磁盘进行通信。
> - **对于共享内存**，所有进程都是操作**内存的同一块区域，对于内存映射**，每一个进程都会在自己的**虚拟地址空间**中创建一个独立的内存，和磁盘建立映射关系。
> - 当进程突然退出，**共享内存还存在，但是内存映射会随着进程的退出而消失**，OS突然终止运行，**共享内存中的数据会消失，内存映射中的数据在磁盘中会有备份**。
> - 生命周期，**对于共享内存**，进程退出，会自动和共享内存取消关联，但是共享内存还在，**需要标记删除（只有共享内存的进程关联数为0时，才会删除）**，**对于内存映射**，进程退出，内存映射的生命周期也结束。
>

## 3.15 守护进程相关知识                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        

> 在介绍守护进程前，我们先提前了解一下**什么是终端、进程组、会话**。
>
> <font color = green>终端</font>
>
> - 在 UNIX 系统中，用户通过终端登录系统后，会得到一个 shell 进程，这个终端成为 shell 进程的**控制终端（Controlling Terminal）**。进程中，控制终端是保存在 PCB 中的信息，而`fork()`会复制 PCB 中的信息，因此由 shell 进程启动的其他进程的控制终端也是这个终端。
> - **默认情况下（没有重定向，比如将标准输出重定向到文件中）**，每个进程的标准输入、标准输出和标准错误输出都指向控制终端。进程从标准输入读，也就是读用户的键盘输入。进程往标准输出或标准错误输出写，也就是输出到显示器上。
> - 在控制终端中**输入一些特殊的控制键**，可以给**前台进程（正在运行的进程）**发信号，例如`ctrl + c`会产生`SIGINT`信号，`ctrl + \`会产生`SIGQUIT`信号，**终止**正在运行的前台进程。
>
> <font color = green>进程组</font>
>
> - 进程组和会话之间形成了一种**两级层次关系**：进程组是一组**相关进程的集合**，会话是一组**相关进程组的集合**。进程组和会话是**为了支持 shell 作业控制**而定义的抽象概念，用户通过 shell 能够交互式地在前台或后台运行命令。
> - 进程组由一个或多个，共享同一进程组标识符（PGID）的进程组成。一个进程组**拥有一个进程组首进程**，该进程是**创建该组的进程**，其进程 ID 为该进程组的 ID，新进程会**继承其父进程所属的进程组 ID**。
> - 进程组拥有一个生命周期，**其开始时刻为，首进程创建组的时刻，结束时刻为，最后一个成员进程退出组的时刻**。一个进程可能会**因为终止而退出进程组**，也可能会因为**加入了另外一个进程组而退出进程组**。进程组中，**首进程无需是最后一个离开进程组的成员**。
>
> <font color = green>会话</font>
>
> - 会话是**多个进程组的集合**。会话首进程是**创建该新会话的进程**，其进程 ID 会成为会话 ID，**新进程会继承其父进程的会话 ID**。
> - 一个会话中的所有进程**共享单个控制终端**。控制终端会在会话首进程**首次打开一个终端设备时被建立**。一个终端最多**可能成为一个会话的控制终端**。
> - 在任意时刻，会话中的其中一个进程组**会成为终端的前台进程组**，其他进程组会成为后台进程组。只有**前台进程组**中的进程，**才能从控制终端中读取输入**。当用户在控制终端中，输入终端字符生成信号后，该信号会被发送到**前台进程组中的所有成员**。
> - 当控制终端的连接建立起来后，**会话首进程会成为该终端的控制进程**。
>
> <font color = green>进程组、会话和控制终端之间的关系</font>
>
> <center>
>   <img src = "./images/第三章 Linux多进程开发/3-30 进程组、会话、控制终端之间的关系1.png" width = "90%">
> </center>
>
>
> <font color = green>进程组、会话操作函数</font>
>
> ```c
> #include<unistd.h>
> 
> pid_t getpgrp(void);
> 
> pid_t getpgid(pid_t pid);
> 
> int setpgid(pid_t pid, pid_t pgid);
> 
> pid_t getsid(pid_t pid);
> 
> pid_t setsid(void);
> ```
>
> <font color = green>守护进程</font>
>
> - 守护进程（Daemon Process），也就是通常说的 Daemon 进程（精灵进程），**是 Linux 中的后台服务进程**。它是一个生存期较长的进程，通常**独立于控制终端**，并且**周期性的执行某种任务或等待处理某些发生的事件**。一般采用以 d 结尾的名字。
> - 守护进程具备以下特征：
>   - 生命周期很长，守护进程会在**系统启动的时候被创建，并且一直运行直至系统被关闭**。
>   - 它在后台运行，并且**不拥有控制终端**。没有控制终端确保了**内核永远不会为守护进程，自动生成任何控制信号以及终端相关的信号（`SIGINT`, `SIGQUIT` etc）**。
> - Linux 的**大多数服务器就是用守护进程实现的**。比如， Internet 服务器`inetd`， Web 服务器 `httpd` 等。

## 3.16 一些零零碎碎的基础知识

> - `mv`命令，不仅**可以移动文件，还可以重命名文件**，重命名文件的原理是**将待移动的文件移动到当前目录下**。
>
> ```bash
> # mv 待移动文件 文件目录/文件名
> ```
>
> - `kill -9 pid`强制杀死进程号为`pid`的进程，僵尸进程除外。
> - `kill -l`查看系统定义的信号列表。
> - `ulimit`命令，Linux种的常用命令，主要用于**查询和设置系统对进程施加的各种资源限制**。
>
> ```bash
> # -a：显示所有的资源限制。
> # -h：显示帮助信息。
> # -S：显示硬限制。
> # -H：显示软限制。
> # -s <value>：设置堆栈的最大大小。
> # -n <value>：设置最大打开文件描述符的数量。
> # -u <value>：设置可以创建的最大用户进程数。
> # -c <value>：设置core转储的最大大小（单位为512字节）。
> # -f <value>：设置单个文件的最大大小（单位为512字节）。
> # -t <value>：设置CPU时间（单位为秒）。
> # -v <value>：设置虚拟内存的最大大小（单位为千字节）。
> ```
>
> - `./可执行文件名称 &`命令，将进程挂起到终端并且**后台运行**。
> - `fg`命令，将终端中后台运行的进程转换到**前台运行。**
> - `echo $$`命令，打印当前 Shell 进程的`PID。`
> - `tty`命令，显示当前终端设备的名称。
>
> - `lseek`系统调用，支持对应文件描述符的**随机访问。**
>
> ```c
> #include <sys/types.h>
> #include <unistd.h>
> 
> /*
> 		函数功能：移动对应文件描述符的读写位置，通过 lseek，可以在文件中任意位置进行读写，主要用于文件的随机访问
> 		- 也可以指定参数，获取文件的长度(bytes)
> 		函数参数
> 		- fd：需要进行随机访问的文件描述符
> 		- offset：偏移量
> 		- whence：从哪个位置开始偏移
> 			- SEEK_SET：文件开始位置
> 			- SEEK_CUR：文件当前位置
> 			- SEEK_END：文件末尾
> 		返回值
> 		- 成功：返回新的文件偏移量
> 		- 失败：返回-1，并且设置错误号
> 		
> 		当 offset = 0，whence = SEEK_END 时，返回值是文件的长度
> */
> off_t lseek(int fd, off_t offset, int whence);
> ```
>

