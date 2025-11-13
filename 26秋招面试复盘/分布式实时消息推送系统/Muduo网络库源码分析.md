# Muduo网络库源码深度剖析

## 目录
- [一、Muduo整体架构](#一muduo整体架构)
- [二、核心组件详解](#二核心组件详解)
- [三、Reactor模式实现原理](#三reactor模式实现原理)
- [四、多线程模型](#四多线程模型)
- [五、核心技术栈总结](#五核心技术栈总结)
- [六、设计思想与最佳实践](#六设计思想与最佳实践)

---

## 一、Muduo整体架构

### 1.1 分层架构

```
┌─────────────────────────────────────────────────────┐
│         应用层 (User Application)                    │
│  TcpServer / TcpClient / HttpServer                 │
└───────────────────┬─────────────────────────────────┘
                    │
┌───────────────────▼─────────────────────────────────┐
│        网络层 (Network Layer)                        │
│  ├─ TcpConnection (管理单个TCP连接)                 │
│  ├─ Acceptor (监听新连接)                            │
│  ├─ Connector (发起连接)                             │
│  └─ Buffer (应用层缓冲区)                            │
└───────────────────┬─────────────────────────────────┘
                    │
┌───────────────────▼─────────────────────────────────┐
│        事件层 (Event Layer)                          │
│  ├─ EventLoop (事件循环，Reactor核心)               │
│  ├─ Channel (fd事件分发器)                          │
│  ├─ Poller (I/O多路复用封装：epoll/poll)            │
│  └─ TimerQueue (定时器队列)                          │
└───────────────────┬─────────────────────────────────┘
                    │
┌───────────────────▼─────────────────────────────────┐
│        线程层 (Thread Layer)                         │
│  ├─ EventLoopThread (I/O线程)                       │
│  ├─ EventLoopThreadPool (I/O线程池)                 │
│  ├─ ThreadPool (计算线程池)                          │
│  └─ Thread / Mutex / Condition                       │
└───────────────────┬─────────────────────────────────┘
                    │
┌───────────────────▼─────────────────────────────────┐
│        系统调用层 (System Call)                      │
│  socket / epoll_wait / read / write / close         │
└─────────────────────────────────────────────────────┘
```

---

### 1.2 Reactor模式类图

```
┌─────────────────────────────────────────────────────┐
│                   TcpServer                          │
├─────────────────────────────────────────────────────┤
│ - EventLoop* loop_                                   │
│ - Acceptor* acceptor_                                │
│ - EventLoopThreadPool* threadPool_                   │
│ - map<string, TcpConnectionPtr> connections_        │
├─────────────────────────────────────────────────────┤
│ + setThreadNum(int)                                  │
│ + start()                                            │
│ + setConnectionCallback(cb)                          │
│ + setMessageCallback(cb)                             │
│ - newConnection(sockfd, peerAddr)                    │
└───────────────────┬─────────────────────────────────┘
                    │ 拥有
                    ▼
┌─────────────────────────────────────────────────────┐
│                   Acceptor                           │
├─────────────────────────────────────────────────────┤
│ - EventLoop* loop_                                   │
│ - Socket acceptSocket_   (listenfd)                  │
│ - Channel acceptChannel_                             │
│ - NewConnectionCallback newConnectionCallback_       │
├─────────────────────────────────────────────────────┤
│ + listen()                                           │
│ - handleRead()   // accept新连接                     │
└───────────────────┬─────────────────────────────────┘
                    │ 拥有
                    ▼
┌─────────────────────────────────────────────────────┐
│                 TcpConnection                        │
├─────────────────────────────────────────────────────┤
│ - EventLoop* loop_                                   │
│ - Socket* socket_       (connfd)                     │
│ - Channel* channel_                                  │
│ - Buffer inputBuffer_                                │
│ - Buffer outputBuffer_                               │
│ - ConnectionCallback connectionCallback_             │
│ - MessageCallback messageCallback_                   │
├─────────────────────────────────────────────────────┤
│ + send(data)                                         │
│ + shutdown()                                         │
│ - handleRead()   // 读取数据                         │
│ - handleWrite()  // 发送数据                         │
└───────────────────┬─────────────────────────────────┘
                    │ 拥有
                    ▼
┌─────────────────────────────────────────────────────┐
│                   Channel                            │
├─────────────────────────────────────────────────────┤
│ - EventLoop* loop_                                   │
│ - const int fd_                                      │
│ - int events_        (关注的事件：EPOLLIN/EPOLLOUT)  │
│ - int revents_       (实际发生的事件)                │
│ - ReadEventCallback readCallback_                    │
│ - EventCallback writeCallback_                       │
├─────────────────────────────────────────────────────┤
│ + handleEvent()   // 事件分发                        │
│ + enableReading() / enableWriting()                  │
└───────────────────┬─────────────────────────────────┘
                    │ 注册到
                    ▼
┌─────────────────────────────────────────────────────┐
│                  EventLoop                           │
├─────────────────────────────────────────────────────┤
│ - Poller* poller_                                    │
│ - vector<Channel*> activeChannels_                   │
│ - vector<Functor> pendingFunctors_                   │
│ - mutex_t mutex_                                     │
│ - int wakeupFd_     (eventfd用于唤醒)                │
├─────────────────────────────────────────────────────┤
│ + loop()          // 事件循环                        │
│ + runInLoop(cb)   // 在I/O线程执行回调               │
│ + queueInLoop(cb) // 加入队列稍后执行                │
└───────────────────┬─────────────────────────────────┘
                    │ 拥有
                    ▼
┌─────────────────────────────────────────────────────┐
│              Poller (抽象基类)                       │
├─────────────────────────────────────────────────────┤
│ # map<int, Channel*> channels_                       │
├─────────────────────────────────────────────────────┤
│ + poll(timeout, activeChannels) = 0                  │
│ + updateChannel(channel) = 0                         │
│ + removeChannel(channel) = 0                         │
└───────────────────┬─────────────────────────────────┘
                    │ 继承
          ┌─────────┴─────────┐
          ▼                   ▼
┌─────────────────┐   ┌──────────────────┐
│   EPollPoller   │   │   PollPoller     │
├─────────────────┤   ├──────────────────┤
│ - int epollfd_  │   │ - pollfd* pollfds│
│ - vector<       │   │                  │
│   epoll_event>  │   │                  │
│   events_       │   │                  │
├─────────────────┤   ├──────────────────┤
│ + poll()        │   │ + poll()         │
└─────────────────┘   └──────────────────┘
```

---

## 二、核心组件详解

### 2.1 EventLoop - 事件循环核心

**代码位置**: `muduo/net/EventLoop.h` 和 `EventLoop.cc`

#### 核心数据结构

```cpp
class EventLoop : noncopyable {
private:
    // ===== 线程相关 =====
    const pid_t threadId_;              // 创建EventLoop的线程ID
    std::atomic<bool> looping_;         // 是否正在循环

    // ===== I/O多路复用 =====
    std::unique_ptr<Poller> poller_;    // epoll/poll封装
    std::vector<Channel*> activeChannels_;  // 活跃的Channel列表
    Channel* currentActiveChannel_;     // 当前正在处理的Channel

    // ===== 定时器 =====
    std::unique_ptr<TimerQueue> timerQueue_;

    // ===== 跨线程任务队列 =====
    mutable MutexLock mutex_;
    std::vector<Functor> pendingFunctors_ GUARDED_BY(mutex_);

    // ===== 跨线程唤醒 =====
    int wakeupFd_;                      // eventfd，用于唤醒epoll_wait
    std::unique_ptr<Channel> wakeupChannel_;

    // ===== 状态标志 =====
    std::atomic<bool> quit_;
    bool eventHandling_;
    bool callingPendingFunctors_;

    Timestamp pollReturnTime_;
    int64_t iteration_;

public:
    void loop();
    void quit();
    void runInLoop(Functor cb);
    void queueInLoop(Functor cb);

    // 定时器接口
    TimerId runAt(Timestamp time, TimerCallback cb);
    TimerId runAfter(double delay, TimerCallback cb);
    TimerId runEvery(double interval, TimerCallback cb);
    void cancel(TimerId timerId);

    // Channel管理
    void updateChannel(Channel* channel);
    void removeChannel(Channel* channel);
    bool hasChannel(Channel* channel);

    // 线程安全检查
    void assertInLoopThread() {
        if (!isInLoopThread()) {
            abortNotInLoopThread();
        }
    }

    bool isInLoopThread() const {
        return threadId_ == CurrentThread::tid();
    }
};
```

---

#### EventLoop::loop()主循环逻辑

**源码位置**: `muduo/net/EventLoop.cc` (推测实现)

```cpp
void EventLoop::loop() {
    assert(!looping_);
    assertInLoopThread();  // 必须在创建线程调用
    looping_ = true;
    quit_ = false;

    LOG_TRACE << "EventLoop " << this << " start looping";

    while (!quit_) {
        // ========== 阶段1: 等待I/O事件 ==========
        activeChannels_.clear();
        pollReturnTime_ = poller_->poll(kPollTimeMs, &activeChannels_);
        ++iteration_;

        if (Logger::logLevel() <= Logger::TRACE) {
            printActiveChannels();
        }

        // ========== 阶段2: 处理I/O事件 ==========
        eventHandling_ = true;
        for (Channel* channel : activeChannels_) {
            currentActiveChannel_ = channel;
            currentActiveChannel_->handleEvent(pollReturnTime_);
        }
        currentActiveChannel_ = NULL;
        eventHandling_ = false;

        // ========== 阶段3: 处理跨线程任务 ==========
        doPendingFunctors();
    }

    LOG_TRACE << "EventLoop " << this << " stop looping";
    looping_ = false;
}
```

**核心流程图**:

```
┌──────────────────────────────────────────────────┐
│  EventLoop::loop()                                │
└───────────┬──────────────────────────────────────┘
            │
            ▼
┌───────────────────────────────────────────────────┐
│  while (!quit_) {                                  │
│                                                    │
│    ┌─────────────────────────────────────┐       │
│    │  Step 1: epoll_wait()               │       │
│    │  等待I/O事件（可能阻塞）             │       │
│    │  activeChannels = poller_->poll()   │       │
│    └─────────────────────────────────────┘       │
│            │                                       │
│            ▼                                       │
│    ┌─────────────────────────────────────┐       │
│    │  Step 2: 处理I/O事件                │       │
│    │  for (Channel* ch : activeChannels) │       │
│    │      ch->handleEvent()               │       │
│    │          ├─ readCallback_()          │       │
│    │          ├─ writeCallback_()         │       │
│    │          └─ errorCallback_()         │       │
│    └─────────────────────────────────────┘       │
│            │                                       │
│            ▼                                       │
│    ┌─────────────────────────────────────┐       │
│    │  Step 3: 处理跨线程任务              │       │
│    │  doPendingFunctors()                 │       │
│    │  执行pendingFunctors_中的回调       │       │
│    └─────────────────────────────────────┘       │
│  }                                                 │
└────────────────────────────────────────────────────┘
```

---

#### 跨线程通信机制

**问题**: 其他线程如何唤醒EventLoop？

**解决方案**: `eventfd` + `wakeupChannel_`

```cpp
// 初始化（EventLoop构造函数）
wakeupFd_ = ::eventfd(0, EFD_NONBLOCK | EFD_CLOEXEC);
wakeupChannel_ = new Channel(this, wakeupFd_);
wakeupChannel_->setReadCallback(
    std::bind(&EventLoop::handleRead, this));
wakeupChannel_->enableReading();  // 注册到epoll

// 其他线程调用
void EventLoop::queueInLoop(Functor cb) {
    {
        MutexLockGuard lock(mutex_);
        pendingFunctors_.push_back(std::move(cb));
    }

    // 需要唤醒的情况:
    // 1. 调用线程不是I/O线程
    // 2. 当前正在执行pendingFunctors（避免下次loop时才执行）
    if (!isInLoopThread() || callingPendingFunctors_) {
        wakeup();
    }
}

void EventLoop::wakeup() {
    uint64_t one = 1;
    ssize_t n = ::write(wakeupFd_, &one, sizeof one);
    if (n != sizeof one) {
        LOG_ERROR << "EventLoop::wakeup() writes " << n << " bytes";
    }
}

void EventLoop::handleRead() {
    uint64_t one = 1;
    ssize_t n = ::read(wakeupFd_, &one, sizeof one);
    if (n != sizeof one) {
        LOG_ERROR << "EventLoop::handleRead() reads " << n << " bytes";
    }
}
```

**工作流程**:

```
线程A (EventLoop线程)            线程B (业务线程)
    │                               │
    ├─ loop()                       │
    │   └─ epoll_wait()  🔴阻塞    │
    │       监听: connfd, listenfd, │
    │            wakeupFd           │
    │                               ├─ runInLoop(callback)
    │                               │   └─ queueInLoop(callback)
    │                               │       ├─ 加锁，添加到pendingFunctors
    │                               │       └─ wakeup()
    │                               │           └─ write(wakeupFd, 1) ⚡
    │                               │
    ├─ epoll_wait() 🟢 返回        │
    │   (wakeupFd可读)              │
    ├─ wakeupChannel_->handleEvent() │
    │   └─ handleRead()             │
    │       └─ read(wakeupFd, &one) │
    ├─ doPendingFunctors()          │
    │   └─ 执行callback             │
    └─ 继续epoll_wait()             │
```

**关键设计**:
1. **为什么用eventfd而不是pipe？**
   - eventfd只用一个fd，pipe需要两个
   - eventfd语义更清晰（计数器），pipe是字节流
   - eventfd性能更好

2. **为什么需要callingPendingFunctors标志？**
   ```cpp
   void EventLoop::doPendingFunctors() {
       std::vector<Functor> functors;
       callingPendingFunctors_ = true;

       {
           MutexLockGuard lock(mutex_);
           functors.swap(pendingFunctors_);  // 🔑 swap减少锁持有时间
       }

       for (const Functor& functor : functors) {
           functor();  // 可能又调用queueInLoop()
       }

       callingPendingFunctors_ = false;
   }
   ```
   如果在执行回调时又有新任务加入，需要再次wakeup，否则要等到下一次epoll返回才能执行。

---

### 2.2 Channel - fd事件分发器

**代码位置**: `muduo/net/Channel.h`

#### 核心职责

Channel是fd和EventLoop之间的桥梁，负责：
1. 保存fd及其关注的事件（EPOLLIN/EPOLLOUT）
2. 保存事件的回调函数
3. 将事件分发给对应的回调函数

```cpp
class Channel : noncopyable {
private:
    EventLoop* loop_;        // 所属的EventLoop
    const int fd_;           // 文件描述符（不负责close）
    int events_;             // 关注的事件（EPOLLIN/EPOLLOUT等）
    int revents_;            // epoll返回的就绪事件
    int index_;              // 在Poller中的状态索引

    // 回调函数
    ReadEventCallback readCallback_;
    EventCallback writeCallback_;
    EventCallback closeCallback_;
    EventCallback errorCallback_;

    std::weak_ptr<void> tie_;
    bool tied_;
    bool eventHandling_;
    bool addedToLoop_;

public:
    // 事件常量
    static const int kNoneEvent;   // 0
    static const int kReadEvent;   // EPOLLIN | EPOLLPRI
    static const int kWriteEvent;  // EPOLLOUT

    // 设置回调
    void setReadCallback(ReadEventCallback cb) { readCallback_ = std::move(cb); }
    void setWriteCallback(EventCallback cb) { writeCallback_ = std::move(cb); }
    void setCloseCallback(EventCallback cb) { closeCallback_ = std::move(cb); }
    void setErrorCallback(EventCallback cb) { errorCallback_ = std::move(cb); }

    // 修改关注的事件
    void enableReading() { events_ |= kReadEvent; update(); }
    void disableReading() { events_ &= ~kReadEvent; update(); }
    void enableWriting() { events_ |= kWriteEvent; update(); }
    void disableWriting() { events_ &= ~kWriteEvent; update(); }
    void disableAll() { events_ = kNoneEvent; update(); }

    // 事件分发
    void handleEvent(Timestamp receiveTime);

private:
    void update() {
        addedToLoop_ = true;
        loop_->updateChannel(this);  // 通知EventLoop更新epoll
    }
};
```

---

#### Channel::handleEvent()事件分发逻辑

```cpp
void Channel::handleEvent(Timestamp receiveTime) {
    std::shared_ptr<void> guard;
    if (tied_) {
        guard = tie_.lock();  // 提升为shared_ptr，防止对象被销毁
        if (guard) {
            handleEventWithGuard(receiveTime);
        }
    } else {
        handleEventWithGuard(receiveTime);
    }
}

void Channel::handleEventWithGuard(Timestamp receiveTime) {
    eventHandling_ = true;

    LOG_TRACE << reventsToString();

    // ===== 错误事件 =====
    if ((revents_ & EPOLLHUP) && !(revents_ & EPOLLIN)) {
        if (logHup_) {
            LOG_WARN << "fd = " << fd_ << " Channel::handle_event() EPOLLHUP";
        }
        if (closeCallback_) closeCallback_();
    }

    if (revents_ & EPOLLERR) {
        if (errorCallback_) errorCallback_();
    }

    // ===== 读事件 =====
    if (revents_ & (EPOLLIN | EPOLLPRI | EPOLLRDHUP)) {
        if (readCallback_) readCallback_(receiveTime);
    }

    // ===== 写事件 =====
    if (revents_ & EPOLLOUT) {
        if (writeCallback_) writeCallback_();
    }

    eventHandling_ = false;
}
```

**事件处理优先级**:
1. 错误事件（EPOLLERR, EPOLLHUP）
2. 读事件（EPOLLIN）
3. 写事件（EPOLLOUT）

---

### 2.3 Poller - I/O多路复用封装

**代码位置**: `muduo/net/Poller.h` 和 `muduo/net/poller/EPollPoller.h`

#### Poller抽象基类

```cpp
class Poller : noncopyable {
protected:
    typedef std::map<int, Channel*> ChannelMap;
    ChannelMap channels_;  // fd -> Channel* 映射表

public:
    typedef std::vector<Channel*> ChannelList;

    // 核心接口（纯虚函数）
    virtual Timestamp poll(int timeoutMs, ChannelList* activeChannels) = 0;
    virtual void updateChannel(Channel* channel) = 0;
    virtual void removeChannel(Channel* channel) = 0;

    virtual bool hasChannel(Channel* channel) const;

    // 工厂方法（根据环境变量选择epoll或poll）
    static Poller* newDefaultPoller(EventLoop* loop);

    void assertInLoopThread() const {
        ownerLoop_->assertInLoopThread();
    }

private:
    EventLoop* ownerLoop_;
};
```

---

#### EPollPoller实现

**代码位置**: `muduo/net/poller/EPollPoller.h`

```cpp
class EPollPoller : public Poller {
private:
    int epollfd_;                              // epoll_create返回的fd
    std::vector<struct epoll_event> events_;   // epoll_wait返回的事件数组

    static const int kInitEventListSize = 16; // 初始事件数组大小

public:
    EPollPoller(EventLoop* loop)
        : Poller(loop),
          epollfd_(::epoll_create1(EPOLL_CLOEXEC)),
          events_(kInitEventListSize) {
        if (epollfd_ < 0) {
            LOG_SYSFATAL << "EPollPoller::EPollPoller";
        }
    }

    ~EPollPoller() override {
        ::close(epollfd_);
    }

    // ===== 核心方法 =====

    Timestamp poll(int timeoutMs, ChannelList* activeChannels) override {
        LOG_TRACE << "fd total count " << channels_.size();

        // 调用epoll_wait
        int numEvents = ::epoll_wait(epollfd_,
                                     &*events_.begin(),
                                     static_cast<int>(events_.size()),
                                     timeoutMs);
        int savedErrno = errno;
        Timestamp now(Timestamp::now());

        if (numEvents > 0) {
            LOG_TRACE << numEvents << " events happened";
            fillActiveChannels(numEvents, activeChannels);

            // 事件数组满了，扩容（类似vector的动态扩容）
            if (static_cast<size_t>(numEvents) == events_.size()) {
                events_.resize(events_.size() * 2);
            }
        } else if (numEvents == 0) {
            LOG_TRACE << "nothing happened";
        } else {
            if (savedErrno != EINTR) {
                errno = savedErrno;
                LOG_SYSERR << "EPollPoller::poll()";
            }
        }

        return now;
    }

    void updateChannel(Channel* channel) override {
        Poller::assertInLoopThread();
        const int index = channel->index();

        LOG_TRACE << "fd = " << channel->fd()
                  << " events = " << channel->events()
                  << " index = " << index;

        if (index == kNew || index == kDeleted) {
            // 新的Channel或已删除的Channel，添加到epoll
            int fd = channel->fd();
            if (index == kNew) {
                assert(channels_.find(fd) == channels_.end());
                channels_[fd] = channel;
            } else { // index == kDeleted
                assert(channels_.find(fd) != channels_.end());
                assert(channels_[fd] == channel);
            }

            channel->set_index(kAdded);
            update(EPOLL_CTL_ADD, channel);
        } else {
            // 已存在的Channel，修改或删除
            int fd = channel->fd();
            assert(channels_.find(fd) != channels_.end());
            assert(channels_[fd] == channel);
            assert(index == kAdded);

            if (channel->isNoneEvent()) {
                update(EPOLL_CTL_DEL, channel);
                channel->set_index(kDeleted);
            } else {
                update(EPOLL_CTL_MOD, channel);
            }
        }
    }

    void removeChannel(Channel* channel) override {
        Poller::assertInLoopThread();
        int fd = channel->fd();

        LOG_TRACE << "fd = " << fd;
        assert(channels_.find(fd) != channels_.end());
        assert(channels_[fd] == channel);
        assert(channel->isNoneEvent());

        int index = channel->index();
        assert(index == kAdded || index == kDeleted);

        size_t n = channels_.erase(fd);
        (void)n;
        assert(n == 1);

        if (index == kAdded) {
            update(EPOLL_CTL_DEL, channel);
        }
        channel->set_index(kNew);
    }

private:
    // Channel的三种状态
    static const int kNew = -1;      // 未添加到epoll
    static const int kAdded = 1;     // 已添加到epoll
    static const int kDeleted = 2;   // 从epoll删除但仍在channels_中

    void fillActiveChannels(int numEvents, ChannelList* activeChannels) const {
        assert(static_cast<size_t>(numEvents) <= events_.size());

        for (int i = 0; i < numEvents; ++i) {
            Channel* channel = static_cast<Channel*>(events_[i].data.ptr);

#ifndef NDEBUG
            int fd = channel->fd();
            ChannelMap::const_iterator it = channels_.find(fd);
            assert(it != channels_.end());
            assert(it->second == channel);
#endif

            channel->set_revents(events_[i].events);
            activeChannels->push_back(channel);
        }
    }

    void update(int operation, Channel* channel) {
        struct epoll_event event;
        memset(&event, 0, sizeof event);
        event.events = channel->events();
        event.data.ptr = channel;  // 🔑 关键：将Channel指针存入epoll_event

        int fd = channel->fd();
        LOG_TRACE << "epoll_ctl op = " << operationToString(operation)
                  << " fd = " << fd << " event = { " << channel->eventsToString() << " }";

        if (::epoll_ctl(epollfd_, operation, fd, &event) < 0) {
            if (operation == EPOLL_CTL_DEL) {
                LOG_SYSERR << "epoll_ctl op =" << operationToString(operation) << " fd =" << fd;
            } else {
                LOG_SYSFATAL << "epoll_ctl op =" << operationToString(operation) << " fd =" << fd;
            }
        }
    }
};
```

**关键设计**:

1. **Channel状态管理**:
   ```
   kNew (初始状态)
     │
     ├─ updateChannel() → EPOLL_CTL_ADD
     │
     ▼
   kAdded (已添加到epoll)
     │
     ├─ events_ == kNoneEvent → EPOLL_CTL_DEL
     │
     ▼
   kDeleted (从epoll删除，但仍在channels_)
     │
     ├─ updateChannel() → EPOLL_CTL_ADD
     │
     └─ removeChannel() → 从channels_删除
   ```

2. **为什么event.data.ptr存Channel指针？**
   - epoll返回的是`epoll_event`数组，通过`data.ptr`可以直接获取Channel
   - 避免通过fd在map中查找Channel，提高性能

3. **事件数组动态扩容**:
   ```cpp
   if (numEvents == events_.size()) {
       events_.resize(events_.size() * 2);  // 类似vector
   }
   ```

---

### 2.4 TcpServer - TCP服务器封装

**代码位置**: `muduo/net/TcpServer.h` 和 `TcpServer.cc`

#### 核心数据结构

```cpp
class TcpServer : noncopyable {
private:
    EventLoop* loop_;  // Acceptor所在的EventLoop（主线程）
    const string ipPort_;
    const string name_;

    std::unique_ptr<Acceptor> acceptor_;  // 监听新连接
    std::shared_ptr<EventLoopThreadPool> threadPool_;  // I/O线程池

    ConnectionCallback connectionCallback_;
    MessageCallback messageCallback_;
    WriteCompleteCallback writeCompleteCallback_;
    ThreadInitCallback threadInitCallback_;

    AtomicInt32 started_;
    int nextConnId_;  // 连接ID生成器
    ConnectionMap connections_;  // 连接名 -> TcpConnection

public:
    typedef std::map<string, TcpConnectionPtr> ConnectionMap;

    TcpServer(EventLoop* loop,
              const InetAddress& listenAddr,
              const string& nameArg,
              Option option = kNoReusePort);

    ~TcpServer();

    // ===== 配置接口 =====
    void setThreadNum(int numThreads);
    void setThreadInitCallback(const ThreadInitCallback& cb) {
        threadInitCallback_ = cb;
    }

    // ===== 回调设置 =====
    void setConnectionCallback(const ConnectionCallback& cb) {
        connectionCallback_ = cb;
    }
    void setMessageCallback(const MessageCallback& cb) {
        messageCallback_ = cb;
    }
    void setWriteCompleteCallback(const WriteCompleteCallback& cb) {
        writeCompleteCallback_ = cb;
    }

    // ===== 启动服务 =====
    void start();  // 线程安全，可以多次调用

private:
    // Acceptor的回调
    void newConnection(int sockfd, const InetAddress& peerAddr);

    // TcpConnection的回调
    void removeConnection(const TcpConnectionPtr& conn);
    void removeConnectionInLoop(const TcpConnectionPtr& conn);
};
```

---

#### TcpServer::start()启动流程

**源码位置**: `muduo/net/TcpServer.cc:59`

```cpp
void TcpServer::start() {
    if (started_.getAndSet(1) == 0) {  // 🔑 原子操作，保证只启动一次
        // 1. 启动线程池
        threadPool_->start(threadInitCallback_);

        assert(!acceptor_->listening());

        // 2. 在主线程的EventLoop中启动Acceptor
        loop_->runInLoop(
            std::bind(&Acceptor::listen, get_pointer(acceptor_)));
    }
}
```

**流程图**:

```
TcpServer::start()
    │
    ├─ threadPool_->start(threadInitCallback)
    │   └─ 创建N个EventLoopThread，每个线程运行EventLoop::loop()
    │
    └─ loop_->runInLoop(Acceptor::listen)
        └─ acceptSocket_.listen()  // socket API: listen()
        └─ acceptChannel_.enableReading()  // 注册listenfd到epoll
            └─ acceptChannel_.setReadCallback(&Acceptor::handleRead)
```

---

#### TcpServer::newConnection()新连接处理

**源码位置**: `muduo/net/TcpServer.cc:71`

```cpp
void TcpServer::newConnection(int sockfd, const InetAddress& peerAddr) {
    loop_->assertInLoopThread();  // 必须在主线程

    // 1. 从线程池选择一个EventLoop（Round-Robin）
    EventLoop* ioLoop = threadPool_->getNextLoop();

    // 2. 生成连接名
    char buf[64];
    snprintf(buf, sizeof buf, "-%s#%d", ipPort_.c_str(), nextConnId_);
    ++nextConnId_;
    string connName = name_ + buf;

    LOG_INFO << "TcpServer::newConnection [" << name_
             << "] - new connection [" << connName
             << "] from " << peerAddr.toIpPort();

    InetAddress localAddr(sockets::getLocalAddr(sockfd));

    // 3. 创建TcpConnection对象
    TcpConnectionPtr conn(new TcpConnection(ioLoop,
                                            connName,
                                            sockfd,
                                            localAddr,
                                            peerAddr));

    // 4. 加入连接表
    connections_[connName] = conn;

    // 5. 设置回调
    conn->setConnectionCallback(connectionCallback_);
    conn->setMessageCallback(messageCallback_);
    conn->setWriteCompleteCallback(writeCompleteCallback_);
    conn->setCloseCallback(
        std::bind(&TcpServer::removeConnection, this, _1));

    // 6. 在ioLoop线程中初始化连接（跨线程调用）
    ioLoop->runInLoop(std::bind(&TcpConnection::connectEstablished, conn));
}
```

**关键点**:

1. **线程调度**: `threadPool_->getNextLoop()` Round-Robin
2. **跨线程初始化**: `ioLoop->runInLoop(TcpConnection::connectEstablished)`
3. **生命周期管理**: 使用`shared_ptr`管理TcpConnection

---

#### 新连接处理流程图

```
主线程EventLoop                      子线程EventLoop (ioLoop)
    │                                      │
    ├─ epoll_wait()                        ├─ epoll_wait()
    │   └─ listenfd可读                    │   (等待新连接的connfd)
    │                                      │
    ├─ Acceptor::handleRead()              │
    │   ├─ accept() → connfd               │
    │   └─ TcpServer::newConnection()      │
    │       ├─ ioLoop = getNextLoop() ──┐  │
    │       ├─ new TcpConnection(ioLoop) │  │
    │       └─ ioLoop->runInLoop(       │  │
    │            TcpConnection::        │  │
    │            connectEstablished)    │  │
    │                                   │  │
    │                                   │  │
    │  queueInLoop添加到ioLoop的       │  │
    │  pendingFunctors队列              │  │
    │  write(wakeupFd, 1) ⚡ 唤醒       │  │
    │                                   │  │
    │                                   └──┼─► 跨线程任务
    │                                      │
    │                                      ├─ epoll_wait() 返回
    │                                      │   (wakeupFd可读)
    │                                      │
    │                                      ├─ doPendingFunctors()
    │                                      │   └─ TcpConnection::connectEstablished()
    │                                      │       ├─ channel_->enableReading()
    │                                      │       │   (注册connfd到epoll)
    │                                      │       └─ connectionCallback_()
    │                                      │           (用户的连接建立回调)
    │                                      │
    │                                      ├─ epoll_wait()
    │                                      │   (现在监听connfd的读写事件)
    │                                      │
    │                                      └─ ... 后续数据收发 ...
```

---

### 2.5 TcpConnection - TCP连接管理

**代码位置**: `muduo/net/TcpConnection.h` 和 `TcpConnection.cc`

#### 核心数据结构

```cpp
class TcpConnection : noncopyable,
                      public std::enable_shared_from_this<TcpConnection> {
private:
    enum StateE { kDisconnected, kConnecting, kConnected, kDisconnecting };

    EventLoop* loop_;                    // 所属的EventLoop
    const string name_;
    StateE state_;  // FIXME: use atomic variable
    bool reading_;

    // Socket封装
    std::unique_ptr<Socket> socket_;
    std::unique_ptr<Channel> channel_;

    // 地址
    const InetAddress localAddr_;
    const InetAddress peerAddr_;

    // 回调函数
    ConnectionCallback connectionCallback_;
    MessageCallback messageCallback_;
    WriteCompleteCallback writeCompleteCallback_;
    HighWaterMarkCallback highWaterMarkCallback_;
    CloseCallback closeCallback_;

    // 缓冲区
    size_t highWaterMark_;
    Buffer inputBuffer_;   // 应用层读缓冲区
    Buffer outputBuffer_;  // 应用层写缓冲区

    std::any context_;  // 用户自定义上下文

public:
    TcpConnection(EventLoop* loop,
                  const string& name,
                  int sockfd,
                  const InetAddress& localAddr,
                  const InetAddress& peerAddr);

    ~TcpConnection();

    // ===== 发送数据 =====
    void send(const void* data, int len);
    void send(const StringPiece& message);
    void send(Buffer* buf);  // this one will swap data

    // ===== 关闭连接 =====
    void shutdown();  // NOT thread safe
    void forceClose();
    void forceCloseWithDelay(double seconds);

    // ===== TCP选项 =====
    void setTcpNoDelay(bool on);

    // ===== 生命周期管理 =====
    void connectEstablished();   // 连接建立时调用（只调用一次）
    void connectDestroyed();     // 连接销毁时调用（只调用一次）

private:
    // Channel的回调
    void handleRead(Timestamp receiveTime);
    void handleWrite();
    void handleClose();
    void handleError();

    // 发送数据的内部实现
    void sendInLoop(const StringPiece& message);
    void sendInLoop(const void* data, size_t len);

    // 关闭连接的内部实现
    void shutdownInLoop();
    void forceCloseInLoop();

    void setState(StateE s) { state_ = s; }

    const char* stateToString() const;
    void startReadInLoop();
    void stopReadInLoop();
};
```

---

#### TcpConnection::connectEstablished()连接建立

```cpp
void TcpConnection::connectEstablished() {
    loop_->assertInLoopThread();
    assert(state_ == kConnecting);
    setState(kConnected);

    // 🔑 tie：将TcpConnection的生命周期绑定到Channel
    // 防止在handleEvent中对象被销毁
    channel_->tie(shared_from_this());

    // 注册读事件到epoll
    channel_->enableReading();  // TcpConnection所在线程的epoll

    // 回调用户的连接建立回调
    connectionCallback_(shared_from_this());
}
```

---

#### TcpConnection::handleRead()读数据

```cpp
void TcpConnection::handleRead(Timestamp receiveTime) {
    loop_->assertInLoopThread();
    int savedErrno = 0;

    // 从socket读取数据到inputBuffer_
    ssize_t n = inputBuffer_.readFd(channel_->fd(), &savedErrno);

    if (n > 0) {
        // 有数据，回调用户的消息回调
        messageCallback_(shared_from_this(), &inputBuffer_, receiveTime);
    } else if (n == 0) {
        // 对端关闭连接（FIN）
        handleClose();
    } else {
        // 错误
        errno = savedErrno;
        LOG_SYSERR << "TcpConnection::handleRead";
        handleError();
    }
}
```

---

#### TcpConnection::send()发送数据

```cpp
void TcpConnection::send(const void* data, int len) {
    send(StringPiece(static_cast<const char*>(data), len));
}

void TcpConnection::send(const StringPiece& message) {
    if (state_ == kConnected) {
        if (loop_->isInLoopThread()) {
            sendInLoop(message);
        } else {
            // 跨线程调用，转发到I/O线程
            void (TcpConnection::*fp)(const StringPiece& message) = &TcpConnection::sendInLoop;
            loop_->runInLoop(
                std::bind(fp,
                          this,     // FIXME
                          message.as_string()));
        }
    }
}

void TcpConnection::sendInLoop(const StringPiece& message) {
    sendInLoop(message.data(), message.size());
}

void TcpConnection::sendInLoop(const void* data, size_t len) {
    loop_->assertInLoopThread();
    ssize_t nwrote = 0;
    size_t remaining = len;
    bool faultError = false;

    if (state_ == kDisconnected) {
        LOG_WARN << "disconnected, give up writing";
        return;
    }

    // ===== 情况1: outputBuffer_为空，且Channel未关注EPOLLOUT =====
    // 尝试直接write，避免拷贝到outputBuffer_
    if (!channel_->isWriting() && outputBuffer_.readableBytes() == 0) {
        nwrote = sockets::write(channel_->fd(), data, len);
        if (nwrote >= 0) {
            remaining = len - nwrote;

            // 全部写完
            if (remaining == 0 && writeCompleteCallback_) {
                loop_->queueInLoop(std::bind(writeCompleteCallback_, shared_from_this()));
            }
        } else { // nwrote < 0
            nwrote = 0;
            if (errno != EWOULDBLOCK) {
                LOG_SYSERR << "TcpConnection::sendInLoop";
                if (errno == EPIPE || errno == ECONNRESET) { // FIXME: any others?
                    faultError = true;
                }
            }
        }
    }

    // ===== 情况2: 未写完，加入outputBuffer_ =====
    assert(remaining <= len);
    if (!faultError && remaining > 0) {
        size_t oldLen = outputBuffer_.readableBytes();

        // 检查高水位标记
        if (oldLen + remaining >= highWaterMark_
            && oldLen < highWaterMark_
            && highWaterMarkCallback_) {
            loop_->queueInLoop(std::bind(highWaterMarkCallback_, shared_from_this(), oldLen + remaining));
        }

        // 将剩余数据加入outputBuffer_
        outputBuffer_.append(static_cast<const char*>(data)+nwrote, remaining);

        if (!channel_->isWriting()) {
            // 关注EPOLLOUT事件，等待socket可写
            channel_->enableWriting();
        }
    }
}
```

**发送数据流程**:

```
用户调用 TcpConnection::send(data, len)
    │
    ├─ 是否在I/O线程？
    │   ├─ 是 → sendInLoop(data, len)
    │   └─ 否 → runInLoop(sendInLoop(data, len))  // 跨线程
    │
    ▼
sendInLoop(data, len)
    │
    ├─ outputBuffer_为空 && 未关注EPOLLOUT？
    │   │
    │   ├─ 是 → 尝试直接write()
    │   │   │
    │   │   ├─ 全部写完？
    │   │   │   └─ 是 → writeCompleteCallback_()
    │   │   │       返回
    │   │   │
    │   │   └─ 未写完 → remaining = len - nwrote
    │   │
    │   └─ 否 → remaining = len
    │
    ├─ remaining > 0？
    │   │
    │   ├─ 是 → outputBuffer_.append(data + nwrote, remaining)
    │   │       channel_->enableWriting()  // 关注EPOLLOUT
    │   │
    │   └─ 否 → 返回
    │
    └─ 等待socket可写...
        │
        ├─ epoll_wait() 返回EPOLLOUT
        │
        └─ handleWrite()
            ├─ write(outputBuffer_.peek(), outputBuffer_.readableBytes())
            ├─ outputBuffer_.retrieve(nwrote)  // 移动readerIndex
            │
            ├─ outputBuffer_为空？
            │   ├─ 是 → channel_->disableWriting()  // 取消EPOLLOUT
            │   │       writeCompleteCallback_()
            │   │
            │   └─ 否 → 等待下次EPOLLOUT
            │
            └─ ...
```

**关键设计**:

1. **避免不必要的拷贝**: 如果outputBuffer_为空，先尝试直接write
2. **LT模式**: 只要outputBuffer_有数据，就关注EPOLLOUT；写完后取消关注
3. **高水位回调**: 防止发送速度跟不上生产速度导致内存暴涨

---

### 2.6 Buffer - 应用层缓冲区

**代码位置**: `muduo/net/Buffer.h`

#### 内存布局

```
+-------------------+------------------+------------------+
| prependable bytes |  readable bytes  |  writable bytes  |
|                   |     (CONTENT)    |                  |
+-------------------+------------------+------------------+
|                   |                  |                  |
0      <=      readerIndex   <=   writerIndex    <=     size
```

#### 核心数据结构

```cpp
class Buffer : public muduo::copyable {
public:
    static const size_t kCheapPrepend = 8;     // 预留头部空间（用于添加协议头）
    static const size_t kInitialSize = 1024;   // 初始大小

    explicit Buffer(size_t initialSize = kInitialSize)
        : buffer_(kCheapPrepend + initialSize),
          readerIndex_(kCheapPrepend),
          writerIndex_(kCheapPrepend) {
        assert(readableBytes() == 0);
        assert(writableBytes() == initialSize);
        assert(prependableBytes() == kCheapPrepend);
    }

    // ===== 容量查询 =====
    size_t readableBytes() const { return writerIndex_ - readerIndex_; }
    size_t writableBytes() const { return buffer_.size() - writerIndex_; }
    size_t prependableBytes() const { return readerIndex_; }

    // ===== 读取数据 =====
    const char* peek() const { return begin() + readerIndex_; }
    void retrieve(size_t len);
    void retrieveAll();
    string retrieveAllAsString();

    // ===== 写入数据 =====
    void append(const char* data, size_t len);
    void append(const void* data, size_t len);
    void append(const StringPiece& str);

    // ===== 预留空间（用于协议头）=====
    void prepend(const void* data, size_t len);

    // ===== 扩容 =====
    void ensureWritableBytes(size_t len) {
        if (writableBytes() < len) {
            makeSpace(len);
        }
        assert(writableBytes() >= len);
    }

    // ===== 从fd读取数据 =====
    ssize_t readFd(int fd, int* savedErrno);

private:
    std::vector<char> buffer_;
    size_t readerIndex_;
    size_t writerIndex_;

    static const char kCRLF[];  // "\r\n"

    char* begin() { return &*buffer_.begin(); }
    const char* begin() const { return &*buffer_.begin(); }

    char* beginWrite() { return begin() + writerIndex_; }
    const char* beginWrite() const { return begin() + writerIndex_; }

    void makeSpace(size_t len);
};
```

---

#### Buffer::readFd()从socket读取数据

**关键技术**: **readv + 栈上临时缓冲区**

```cpp
ssize_t Buffer::readFd(int fd, int* savedErrno) {
    // 🔑 栈上临时缓冲区（避免频繁扩容）
    char extrabuf[65536];

    struct iovec vec[2];
    const size_t writable = writableBytes();

    // ===== 第一块：Buffer的writable区域 =====
    vec[0].iov_base = begin() + writerIndex_;
    vec[0].iov_len = writable;

    // ===== 第二块：栈上临时缓冲区 =====
    vec[1].iov_base = extrabuf;
    vec[1].iov_len = sizeof extrabuf;

    // when there is enough space in this buffer, don't read into extrabuf.
    // when extrabuf is used, we read 128k-1 bytes at most.
    const int iovcnt = (writable < sizeof extrabuf) ? 2 : 1;

    // ===== 调用readv，一次系统调用读取到两块内存 =====
    const ssize_t n = sockets::readv(fd, vec, iovcnt);

    if (n < 0) {
        *savedErrno = errno;
    } else if (static_cast<size_t>(n) <= writable) {
        // Buffer空间足够
        writerIndex_ += n;
    } else {
        // Buffer空间不够，extrabuf也用上了
        writerIndex_ = buffer_.size();
        append(extrabuf, n - writable);  // 将extrabuf的数据追加到Buffer
    }

    return n;
}
```

**为什么这样设计？**

| 方案 | 优点 | 缺点 |
|------|------|------|
| **每次read前resize到足够大** | 简单 | 浪费内存，HTTP短连接每次都申请大buffer |
| **read多次直到EAGAIN** | 节省内存 | 多次系统调用 |
| **readv + 栈上临时缓冲区 ✅** | 一次系统调用读取大量数据，按需扩容 | 略复杂 |

**原理图**:

```
Buffer (heap)                栈上临时缓冲区 (stack)
┌────────────┬────────────┐  ┌──────────────────────┐
│  readable  │  writable  │  │      extrabuf        │
│            │   (1KB)    │  │       (64KB)         │
└────────────┴────────────┘  └──────────────────────┘
             ▲                 ▲
             │                 │
             └─── vec[0]       └─── vec[1]
                    │                 │
                    └─────┬───────────┘
                          │
                    readv(fd, vec, 2)
                          │
        ┌─────────────────┴──────────────────┐
        │                                     │
  n <= writable                       n > writable
        │                                     │
        ▼                                     ▼
writerIndex_ += n           writerIndex_ = buffer_.size()
                            append(extrabuf, n - writable)
```

---

#### Buffer::makeSpace()扩容或内部搬移

```cpp
void Buffer::makeSpace(size_t len) {
    if (writableBytes() + prependableBytes() < len + kCheapPrepend) {
        // ===== 情况1: 空间真的不够，需要扩容 =====
        buffer_.resize(writerIndex_ + len);
    } else {
        // ===== 情况2: 空间够，但是分散了，内部搬移 =====
        // move readable data to the front, make space inside buffer
        assert(kCheapPrepend < readerIndex_);
        size_t readable = readableBytes();
        std::copy(begin() + readerIndex_,
                  begin() + writerIndex_,
                  begin() + kCheapPrepend);
        readerIndex_ = kCheapPrepend;
        writerIndex_ = readerIndex_ + readable;
        assert(readable == readableBytes());
    }
}
```

**优化思想**: 优先内部搬移，避免频繁扩容

```
扩容前:
┌────────────┬──────────┬────────────┐
│ prependable│ readable │  writable  │
│   (10KB)   │  (2KB)   │   (1KB)    │
└────────────┴──────────┴────────────┘

内部搬移后:
┌────┬──────────┬─────────────────────┐
│prep│ readable │     writable        │
│(8B)│  (2KB)   │      (11KB)         │
└────┴──────────┴─────────────────────┘
```

---

## 三、Reactor模式实现原理

### 3.1 经典Reactor模式

#### 标准Reactor模式的角色

```
┌─────────────────────────────────────────────────────┐
│                Reactor (事件分发器)                  │
│  while (true) {                                      │
│      events = demultiplexer.select()  // 等待事件   │
│      for (event in events) {                         │
│          handler = get_handler(event)                │
│          handler.handle_event()                      │
│      }                                                │
│  }                                                    │
└─────────────────────────────────────────────────────┘
            │                           ▲
            ▼                           │
┌───────────────────┐       ┌───────────────────────┐
│   Demultiplexer   │       │   Event Handlers      │
│   (epoll/select)  │       │  (AcceptHandler,      │
│                   │       │   ReadHandler,        │
│   监听多个fd      │       │   WriteHandler)       │
└───────────────────┘       └───────────────────────┘
```

---

#### Muduo的Reactor映射

| Reactor角色 | Muduo实现 | 职责 |
|------------|-----------|------|
| **Reactor** | `EventLoop` | 事件循环，调用epoll_wait |
| **Demultiplexer** | `Poller (EPollPoller)` | I/O多路复用 |
| **Event Handler** | `Channel` | 事件处理器，保存回调函数 |
| **Concrete Handler** | `Acceptor`, `TcpConnection` | 具体的处理逻辑 |

---

### 3.2 Muduo的Reactor工作流程

```
┌──────────────────────────────────────────────────────┐
│ EventLoop::loop() - Reactor主循环                     │
└───────────────┬──────────────────────────────────────┘
                │
                ▼
┌──────────────────────────────────────────────────────┐
│ 1. poller_->poll(timeout, &activeChannels)           │
│    └─ EPollPoller::poll()                            │
│        └─ epoll_wait(epollfd_, events, size, timeout)│
│            │                                          │
│            ├─ listenfd可读 → Acceptor::handleRead()  │
│            ├─ connfd可读   → TcpConnection::handleRead() │
│            ├─ connfd可写   → TcpConnection::handleWrite() │
│            └─ wakeupFd可读 → EventLoop::handleRead()  │
└──────────────────────────────────────────────────────┘
                │
                ▼
┌──────────────────────────────────────────────────────┐
│ 2. for (Channel* ch : activeChannels) {              │
│        ch->handleEvent(pollReturnTime)               │
│    }                                                  │
│    └─ Channel::handleEvent()                         │
│        ├─ EPOLLIN  → readCallback_()                 │
│        ├─ EPOLLOUT → writeCallback_()                │
│        └─ EPOLLERR → errorCallback_()                │
└──────────────────────────────────────────────────────┘
                │
                ▼
┌──────────────────────────────────────────────────────┐
│ 3. doPendingFunctors()                               │
│    └─ 执行其他线程投递的任务                         │
└──────────────────────────────────────────────────────┘
```

---

### 3.3 新连接到来的完整流程

```
Step 1: 客户端发起连接
    │
    ▼
Step 2: 主线程EventLoop检测到listenfd可读
    │
    ├─ epoll_wait() 返回
    ├─ activeChannels包含acceptChannel_
    └─ acceptChannel_->handleEvent()
        └─ Acceptor::handleRead()
            │
            ├─ connfd = ::accept(listenfd, ...)  🔑 系统调用
            │
            └─ newConnectionCallback_(connfd, peerAddr)
                └─ TcpServer::newConnection(connfd, peerAddr)
                    │
                    ├─ ioLoop = threadPool_->getNextLoop()  🔑 负载均衡
                    ├─ conn = new TcpConnection(ioLoop, connfd, ...)
                    ├─ connections_[connName] = conn
                    │
                    └─ ioLoop->runInLoop(TcpConnection::connectEstablished)
                        │
                        └─ queueInLoop() → wakeup() ⚡
                            └─ write(wakeupFd, 1)
    │
    ▼
Step 3: 子线程EventLoop被唤醒
    │
    ├─ epoll_wait() 返回 (wakeupFd可读)
    ├─ wakeupChannel_->handleEvent()
    │   └─ EventLoop::handleRead()
    │       └─ read(wakeupFd, &one)
    │
    └─ doPendingFunctors()
        └─ TcpConnection::connectEstablished()
            │
            ├─ channel_->tie(shared_from_this())  🔑 生命周期管理
            ├─ channel_->enableReading()
            │   └─ epoll_ctl(epollfd, EPOLL_CTL_ADD, connfd, ...)
            │
            └─ connectionCallback_(conn)  🔑 用户回调
    │
    ▼
Step 4: 子线程EventLoop继续监听connfd的读写事件
```

---

## 四、多线程模型

### 4.1 One Loop Per Thread模型

**核心思想**: 每个线程最多有一个EventLoop，每个EventLoop只属于一个线程

```cpp
// EventLoop构造函数
EventLoop::EventLoop()
    : looping_(false),
      quit_(false),
      eventHandling_(false),
      callingPendingFunctors_(false),
      iteration_(0),
      threadId_(CurrentThread::tid()),  // 🔑 记录创建线程的ID
      poller_(Poller::newDefaultPoller(this)),
      timerQueue_(new TimerQueue(this)),
      wakeupFd_(createEventfd()),
      wakeupChannel_(new Channel(this, wakeupFd_)),
      currentActiveChannel_(NULL) {
    LOG_DEBUG << "EventLoop created " << this << " in thread " << threadId_;

    // 检查当前线程是否已有EventLoop
    if (t_loopInThisThread) {
        LOG_FATAL << "Another EventLoop " << t_loopInThisThread
                  << " exists in this thread " << threadId_;
    } else {
        t_loopInThisThread = this;  // thread_local变量
    }

    wakeupChannel_->setReadCallback(
        std::bind(&EventLoop::handleRead, this));
    wakeupChannel_->enableReading();
}
```

**线程安全检查**:

```cpp
void EventLoop::assertInLoopThread() {
    if (!isInLoopThread()) {
        abortNotInLoopThread();
    }
}

bool EventLoop::isInLoopThread() const {
    return threadId_ == CurrentThread::tid();
}
```

---

### 4.2 EventLoopThread - I/O线程封装

**代码位置**: `muduo/net/EventLoopThread.h` (推测实现)

```cpp
class EventLoopThread : noncopyable {
public:
    typedef std::function<void(EventLoop*)> ThreadInitCallback;

    EventLoopThread(const ThreadInitCallback& cb = ThreadInitCallback(),
                    const string& name = string());
    ~EventLoopThread();

    EventLoop* startLoop();  // 🔑 启动线程，返回EventLoop指针

private:
    void threadFunc();  // 线程主函数

    EventLoop* loop_ GUARDED_BY(mutex_);
    bool exiting_;
    Thread thread_;
    MutexLock mutex_;
    Condition cond_ GUARDED_BY(mutex_);
    ThreadInitCallback callback_;
};
```

**实现**:

```cpp
EventLoop* EventLoopThread::startLoop() {
    assert(!thread_.started());
    thread_.start();  // 启动线程，执行threadFunc

    EventLoop* loop = NULL;
    {
        MutexLockGuard lock(mutex_);
        while (loop_ == NULL) {
            cond_.wait();  // 🔑 等待线程创建EventLoop
        }
        loop = loop_;
    }

    return loop;
}

void EventLoopThread::threadFunc() {
    EventLoop loop;  // 🔑 栈上对象，线程退出时自动析构

    if (callback_) {
        callback_(&loop);
    }

    {
        MutexLockGuard lock(mutex_);
        loop_ = &loop;
        cond_.notify();  // 🔑 通知startLoop()
    }

    loop.loop();  // 🔑 进入事件循环

    // 循环结束
    MutexLockGuard lock(mutex_);
    loop_ = NULL;
}
```

**同步机制**:

```
主线程                       I/O线程
    │                           │
    ├─ startLoop()              │
    │   ├─ thread_.start() ───► │
    │   │                       ├─ threadFunc()
    │   │                       │   ├─ EventLoop loop
    │   │                       │   ├─ loop_ = &loop
    │   │                       │   └─ cond_.notify() ⚡
    │   │                       │
    │   ├─ cond_.wait() 🔴      │
    │   ├─ 被唤醒 🟢            │
    │   └─ return loop_         │
    │                           │
    │                           ├─ loop.loop()
    │                           │   (进入事件循环)
    │                           │
    └─ 继续...                  └─ ...
```

---

### 4.3 EventLoopThreadPool - I/O线程池

**代码位置**: `muduo/net/EventLoopThreadPool.h` 和 `.cc`

```cpp
class EventLoopThreadPool : noncopyable {
public:
    EventLoopThreadPool(EventLoop* baseLoop, const string& nameArg);
    ~EventLoopThreadPool();

    void setThreadNum(int numThreads) { numThreads_ = numThreads; }
    void start(const ThreadInitCallback& cb = ThreadInitCallback());

    // ===== 负载均衡 =====
    EventLoop* getNextLoop();  // Round-Robin
    EventLoop* getLoopForHash(size_t hashCode);  // 哈希

    std::vector<EventLoop*> getAllLoops();

    bool started() const { return started_; }
    const string& name() const { return name_; }

private:
    EventLoop* baseLoop_;  // 主线程的EventLoop
    string name_;
    bool started_;
    int numThreads_;
    int next_;  // Round-Robin的索引
    std::vector<std::unique_ptr<EventLoopThread>> threads_;
    std::vector<EventLoop*> loops_;
};
```

**实现**:

```cpp
void EventLoopThreadPool::start(const ThreadInitCallback& cb) {
    assert(!started_);
    baseLoop_->assertInLoopThread();

    started_ = true;

    for (int i = 0; i < numThreads_; ++i) {
        char buf[name_.size() + 32];
        snprintf(buf, sizeof buf, "%s%d", name_.c_str(), i);

        EventLoopThread* t = new EventLoopThread(cb, buf);
        threads_.push_back(std::unique_ptr<EventLoopThread>(t));
        loops_.push_back(t->startLoop());  // 🔑 启动线程
    }

    if (numThreads_ == 0 && cb) {
        cb(baseLoop_);  // 单线程模式，回调baseLoop
    }
}

EventLoop* EventLoopThreadPool::getNextLoop() {
    baseLoop_->assertInLoopThread();
    assert(started_);

    EventLoop* loop = baseLoop_;

    if (!loops_.empty()) {
        // Round-Robin
        loop = loops_[next_];
        ++next_;
        if (implicit_cast<size_t>(next_) >= loops_.size()) {
            next_ = 0;
        }
    }

    return loop;
}

EventLoop* EventLoopThreadPool::getLoopForHash(size_t hashCode) {
    baseLoop_->assertInLoopThread();
    EventLoop* loop = baseLoop_;

    if (!loops_.empty()) {
        loop = loops_[hashCode % loops_.size()];
    }

    return loop;
}
```

---

### 4.4 多线程模型总结

#### 线程模型图

```
┌─────────────────────────────────────────────────────┐
│  主线程 (Main Thread)                                │
│                                                      │
│  EventLoop (baseLoop)                               │
│     │                                                │
│     ├─ Acceptor (listenfd)                          │
│     │   └─ accept() → connfd                        │
│     │                                                │
│     └─ 分派connfd到I/O线程 ──┐                      │
│                               │                      │
└───────────────────────────────┼──────────────────────┘
                                │
              ┌─────────────────┼─────────────────┐
              │                 │                 │
              ▼                 ▼                 ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  I/O线程1       │  │  I/O线程2       │  │  I/O线程N       │
│                 │  │                 │  │                 │
│  EventLoop1     │  │  EventLoop2     │  │  EventLoopN     │
│    ├─ conn1     │  │    ├─ conn2     │  │    ├─ connN     │
│    ├─ conn4     │  │    ├─ conn5     │  │    ├─ connN+1   │
│    └─ conn7     │  │    └─ conn8     │  │    └─ connN+2   │
│                 │  │                 │  │                 │
│  处理读写事件   │  │  处理读写事件   │  │  处理读写事件   │
└─────────────────┘  └─────────────────┘  └─────────────────┘
```

#### 线程数量配置建议

| 场景 | numThreads | 说明 |
|------|-----------|------|
| 单线程 | 0 | 适合简单服务，或连接数少于1000 |
| I/O密集 | CPU核心数 | 充分利用多核，减少线程切换 |
| 计算密集 | CPU核心数 - 1 | 留一个核心给主线程 |
| 混合场景 | CPU核心数 * 2 | 平衡I/O等待和计算 |

---

## 五、核心技术栈总结

### 5.1 C++技术

| 技术 | 使用场景 | 代码位置 |
|------|---------|---------|
| **智能指针** | `shared_ptr<TcpConnection>` 管理连接生命周期 | `TcpServer.h:100` |
| **std::function** | 回调函数封装 | `Callbacks.h` |
| **std::bind** | 绑定成员函数 | `TcpServer.cc:35` |
| **std::enable_shared_from_this** | 在成员函数中获取`shared_ptr` | `TcpConnection.h:42` |
| **std::weak_ptr** | `Channel::tie_`防止循环引用 | `Channel.h:100` |
| **std::any** | `TcpConnection::context_`存储用户自定义数据 | `TcpConnection.h:80` |
| **std::atomic** | `EventLoop::looping_`等无锁标志 | `EventLoop.h:14` |
| **std::vector** | `Buffer::buffer_`动态数组 | `Buffer.h:42` |
| **std::map** | `TcpServer::connections_`连接表 | `TcpServer.h:114` |
| **std::unique_ptr** | `Acceptor`, `Poller`等独占所有权 | `TcpServer.h:105` |
| **noncopyable** | 禁止拷贝的基类 | `noncopyable.h` |

---

### 5.2 Linux系统编程

| 技术 | 系统调用 | 用途 |
|------|---------|------|
| **epoll** | `epoll_create`, `epoll_ctl`, `epoll_wait` | I/O多路复用 |
| **socket** | `socket`, `bind`, `listen`, `accept`, `connect` | TCP网络编程 |
| **非阻塞I/O** | `O_NONBLOCK`, `fcntl` | 避免阻塞 |
| **eventfd** | `eventfd`, `read`, `write` | 线程间唤醒 |
| **timerfd** | `timerfd_create`, `timerfd_settime` | 定时器 |
| **signalfd** | `signalfd` | 信号处理 |
| **readv/writev** | `readv`, `writev` | 分散/聚集I/O |
| **SO_REUSEADDR** | `setsockopt` | 地址复用 |
| **SO_REUSEPORT** | `setsockopt` | 端口复用（负载均衡） |
| **TCP_NODELAY** | `setsockopt` | 禁用Nagle算法 |

---

### 5.3 设计模式

| 模式 | 实现 | 说明 |
|------|------|------|
| **Reactor模式** | `EventLoop` + `Poller` + `Channel` | 事件驱动 |
| **单例模式** | `EventLoop`的`thread_local` | 每线程一个实例 |
| **工厂模式** | `Poller::newDefaultPoller()` | 选择epoll或poll |
| **RAII** | `MutexLockGuard`, `Buffer` | 资源自动管理 |
| **观察者模式** | `Channel`的回调函数 | 事件通知 |
| **策略模式** | `ConnectionCallback`, `MessageCallback` | 行为可配置 |
| **对象池** | `EventLoopThreadPool` | 线程复用 |

---

### 5.4 性能优化技术

| 技术 | 实现 | 效果 |
|------|------|------|
| **LT触发模式** | `epoll` LT | 编程简单，不易丢事件 |
| **非阻塞I/O** | `O_NONBLOCK` | 避免阻塞 |
| **分散读取** | `readv` + 栈上缓冲区 | 减少系统调用 |
| **延迟写** | `outputBuffer_` | 批量发送 |
| **对象池** | `EventLoopThreadPool` | 减少线程创建开销 |
| **Round-Robin** | `getNextLoop()` | 负载均衡 |
| **智能指针** | `shared_ptr`, `weak_ptr` | 自动内存管理 |
| **减少锁竞争** | One Loop Per Thread | 每个连接只在一个线程处理 |

---

## 六、设计思想与最佳实践

### 6.1 核心设计原则

#### 1. One Loop Per Thread

**好处**:
- 无需加锁：每个连接的读写都在同一个线程
- 缓存友好：数据局部性好
- 简化编程模型

#### 2. 用户代码在I/O线程执行

**好处**:
- 避免锁：回调函数在I/O线程执行，访问连接数据无需加锁
- 低延迟：不需要跨线程通信

**代价**:
- 用户回调不能阻塞：否则影响其他连接
- 耗时操作需要放到线程池：如数据库查询

---

### 6.2 线程安全的保证

#### EventLoop的线程安全

```cpp
// ❌ 错误：在其他线程直接操作EventLoop
void Thread1() {
    loop->updateChannel(channel);  // 危险！
}

// ✅ 正确：通过runInLoop转发到I/O线程
void Thread1() {
    loop->runInLoop([channel]() {
        loop->updateChannel(channel);
    });
}
```

#### TcpConnection的线程安全

```cpp
// TcpConnection::send()可以跨线程调用
void TcpConnection::send(const StringPiece& message) {
    if (state_ == kConnected) {
        if (loop_->isInLoopThread()) {
            sendInLoop(message);
        } else {
            // 🔑 跨线程，转发到I/O线程
            loop_->runInLoop(
                std::bind(&TcpConnection::sendInLoop,
                          this,
                          message.as_string()));
        }
    }
}
```

---

### 6.3 资源管理

#### TcpConnection的生命周期

**问题**: 如何在回调执行期间保证TcpConnection不被销毁？

**解决**: `Channel::tie()`

```cpp
void TcpConnection::connectEstablished() {
    loop_->assertInLoopThread();
    assert(state_ == kConnecting);
    setState(kConnected);

    // 🔑 将TcpConnection的生命周期绑定到Channel
    channel_->tie(shared_from_this());

    channel_->enableReading();
    connectionCallback_(shared_from_this());
}

void Channel::handleEvent(Timestamp receiveTime) {
    std::shared_ptr<void> guard;
    if (tied_) {
        guard = tie_.lock();  // 🔑 提升为shared_ptr
        if (guard) {
            handleEventWithGuard(receiveTime);
        }
    } else {
        handleEventWithGuard(receiveTime);
    }
}
```

**原理**: 在`handleEvent`期间，`guard`持有`TcpConnection`的`shared_ptr`，保证对象不被销毁。

---

### 6.4 面试常考点

#### Q1: Muduo如何实现跨线程唤醒？

**答案**: 使用`eventfd` + `wakeupChannel_`

1. 每个EventLoop有一个`wakeupFd_`（eventfd）
2. `wakeupFd_`注册到epoll，关注EPOLLIN
3. 其他线程调用`queueInLoop()`时，写`wakeupFd_`唤醒epoll_wait
4. EventLoop被唤醒后，执行`pendingFunctors_`中的任务

---

#### Q2: Muduo的Buffer为什么用readv？

**答案**: **一次系统调用 + 按需扩容**

- `readv`可以一次读取到两块内存（Buffer + 栈上临时缓冲区）
- 如果数据量小，只用Buffer，不扩容
- 如果数据量大，临时缓冲区接收溢出数据，然后`append`到Buffer

---

#### Q3: Muduo的TcpConnection为什么不直接write？

**答案**: **延迟写 + 避免阻塞**

1. 如果`outputBuffer_`为空且未关注EPOLLOUT，先尝试直接write
2. 如果写不完，剩余数据加入`outputBuffer_`，关注EPOLLOUT
3. socket可写时，再从`outputBuffer_`发送

**好处**: 批量发送，减少系统调用

---

#### Q4: Muduo如何保证线程安全？

**答案**: **One Loop Per Thread + runInLoop**

- 每个TcpConnection只在一个EventLoop中处理，避免锁
- 跨线程操作通过`runInLoop()`转发到I/O线程执行

---

**文档生成时间**: 2025-11-03
**项目路径**: `/home/utopianyouth/zerovoice/unit11/myself-chatroom-distribute`
**Muduo版本**: 基于陈硕的muduo网络库
