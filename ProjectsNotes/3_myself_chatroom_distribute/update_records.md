# 单机到分布式优化记录

## 七、优化过程

项目最开始是单机版的，在不断了解项目的过程中，不断优化前端和后端，使得项目可玩性增加。

### 7.1 前端

> - 第一部分主要是登录、注册和聊天界面的优化显示，使得前端页面看起来更加现代化一点，包括页面布局，按钮无边框设计等；
>
> - 第二部分主要是对聊天界面的细节优化
>   - 增加聊天室分类展示，包括三类：所有房间、用户创建的房间和其它人创建的房间，不过多人聊天，大部分时间都在所有房间下；
>   - 聊天框的优化，包括显示当前用户的头像，显示当前聊天房间的名字，增加了查找和获取30天之前（需要操作MySQL）历史消息的按钮（后端业务逻辑还没有实现，后续拓展很方便）；
>   - 输入消息框和输入按钮的优化；
> - 第三部分主要是对网页顶部的优化，包括布局等，参考主流互联网公司的主页设计。

### 7.2 后端

后端层的优化，大部分的工作是在服务迁移上面，单机版的聊天室，所有的业务逻辑都在 comet 层，显然是不合理的，所以需要将其迁移到 logic 中去，comet 层只需要提供访问 logic 层的请求转发业务逻辑即可。

#### 6.2.1 comet 层



#### 6.2.2 job 层



#### 6.2.3 logic 层



#### 6.2.4 存储层

在业务的不断迁移过程中，存储层的优化主要工作是两部分，表结构及其字段的优化和新增表 CRUD 业务的完善，在核心业务迁移的过程中，也思考了一些问题。



MySQL作为消息的持久化存储是否合理？

合理，针对前30天历史消息，使用 redis 存储，也就是用户通过上拉获取历史消息的上限是30天前；30天之后的历史消息使用 MySQL 存储。



哪些数据应该一次加载之后，常驻内存？



## 八、一些神奇BUG记录

### 8.1 注册与登录

初步的数据传输实现

> **注册**
>
> 前端读取用户输入数字 ==> 前端页面 `MD5(password)` ==> 后端通过 `MD5(MD5(password) + salt)` 存储到 MySQL中。
>
> **登录**
>
> 通过用户输入的 `password`，前端页面发送`MD5(password)`给后端，登录接口进行 `MD5(MD5(password)+salt)` 进行验证，与MySQL的`password`字段进行对比，如果一致，登录成功，存储 websocket 连接和对应的 cookie。

 `MD5(password + salt)` ==> 好像还是很大概率会被破解？

是不是后端自定义了密码和盐值的组合规则，以此生成新的 MD5 码，这套组合规则不一定是简单的字符串拼接，规则定义好之后，运维人员是不知道的，所以密码很难猜到。

### 8.2 websocket连接建立与通信

**关于登录之后建立websocket连接，将websocket连接对象与用户id组合成键值对`<userid, websocketconn>`存储到全局变量中的问题**

**基本实现：**使用的是多态实现的，每一次登录进行websocket连接都会调用其构造函数，退出登录时，会执行其对应的析构函数。

**业务简单剖析：**每一次退出登录，会执行 `s_user_ws_conn_map.erase()` 操作，但是不会从`RoomTopic::user_ids{}` 数组中删除该用户所在聊天室的订阅，当用户重新登录时， websocket连接建立之后，服务器会向发送websocket连接的客户端发送一个hello帧，并且为该用户订阅到所有的默认聊天室中，其中，hello帧包括该用户的个人信息（userid和username），以及每一个房间的历史聊天记录（服务器默认每个房间显示 n 条聊天记录）。

#### 8.2.1 退出登录的问题

**场景1：**用户登录聊天室之后，退出登录，然后再继续登录，这两个操作时间间隔很短。

代码问题定位：

```cpp
/*
	websocket_conn.cc ==> 210th lines
*/
s_mtx_user_ws_conn_map.lock();
// insert current userid and websocket
// shared_from_this() ==> ensure one ControlBlock, one WebSocketConn
if (s_user_ws_conn_map.find(this->user_id) == s_user_ws_conn_map.end()) {
  s_user_ws_conn_map.insert({ this->user_id, this->shared_from_this() });
}
else {
  // the same userid maybe exists, because websocket conn maybe don't close
  // force insert(update)
  s_user_ws_conn_map[this->user_id] = this->shared_from_this();
}
s_mtx_user_ws_conn_map.unlock();
```

代码执行逻辑如下：

第一步：首次登录，执行 `OnRead()` 函数，加互斥锁并发访问全局变量 `s_user_ws_conn_map`，将 `userid` 和 `WebSocketConn` 对象组合成键值对，插入到全局变量`s_user_ws_conn_map`中；

第二步：退出登录，执行 `DisConnection()` 函数，关闭 tcp 连接（也就是关闭 websocket 通信），然后加互斥锁并发访问全局变量 `s_user_ws_conn_map`，移除退出登录用户的键值对；

第三步：用户退出登录后快速点击再次登录，服务器新建一个线程，处理 websocket 连接，此时又会执行 `OnRead()` 函数，并且加互斥锁并发访问全局变量 `s_user_ws_conn_map`，将 `userid` 和 `WebSocketConn` 对象组合成键值对，插入到全局变量`s_user_ws_conn_map`中。

**问题本质：**考虑这样一种情况，由于用户退出登录之后，快速点击再次登录，第二步执行退出登录的线程，来不及关闭 websocket 连接和移除键值对，但是执行第三步线程又获取了互斥锁，全局变量`s_user_ws_conn_map`中存在原用户旧的websocket连接，导致插入新的websocket连接失败，最后第二步的线程后续将旧的 websocket 连接删除，导致该用户再次登录后，新的websocket连接没有记录到全局变量中。当该用户在新的websocket连接上发送聊天信息时，由于全局变量没有记录该websocket连接，<font color = red>所以自己的聊天窗口无法收到刚刚发送的消息</font>。

### 8.3 MySQL插入

**批量插入聊天记录到数据库时，变量生命周期问题**

`SetParam()` 第二个值接收的是引用，不能在循环内部定义拷贝变量，不然在`ExecuteUpdate()`执行时，变量生命周期结束，导致SQL执行异常。

```cpp
/*
	代码定位：/server/application/logic/api/api_message.cc ==> storeMessagesToDB
	
*/

std::vector<string> msg_ids, room_ids;
std::vector<uint32_t> user_ids;

// 预先分配空间
msg_ids.reserve(messages.size());
room_ids.reserve(messages.size());
user_ids.reserve(messages.size());

for (const auto& msg : messages) {
  msg_ids.emplace_back(msg.id.empty() ? generateMessageId() : msg.id);
  room_ids.emplace_back(room_id);  // 来自函数参数，需要为每条消息复制
  user_ids.emplace_back(static_cast<uint32_t>(msg.user_id));  // 需要类型转换
}

int param_index = 0;
for (size_t i = 0; i < messages.size(); ++i) {
  stmt.SetParam(param_index++, msg_ids[i]);
  stmt.SetParam(param_index++, room_ids[i]);
  stmt.SetParam(param_index++, user_ids[i]);
  stmt.SetParam(param_index++, messages[i].username);
  stmt.SetParam(param_index++, messages[i].content);
}

if (!stmt.ExecuteUpdate()) {
  LOG_ERROR << "Failed to execute batch SQL statement";
  return false;
}
```

### 8.4 redis 语句拼接问题

**redis中不能存储特殊字符的聊天信息，譬如空格等**

修改了 `cache_pool.cc` 源码，将redis命令从简单拼接的形式改为了调用`redisCommandArgv()`组合redis命令的形式，后者只需要提供插入的键值对即可，避免了拼接造成的特殊字符无法识别问题。



## 九、项目中一些值得深思的问题

在做项目的各个模块时，应该优先考虑先把功能做出来，还是在做某一个功能模块的时候，优先考虑性能，这涉及到项目上线的效率问题？

关于Kafka在项目中的真正优势，comet层对请求进行转发到logic层后，为什么不直接接收logic层的响应数据？而是logic层将消息发送到Kafka中，再由job层消费Kafka中的数据，通过gRPC调用comet层，对响应消息具体处理的业务逻辑。

什么是单例模式，在项目中频繁使用单例模式代替函数式接口有什么风险？

项目核心功能目前有三大块，处理用户发送的聊天信息（使用Kafka的消息推送形式），获取房间历史消息（不属于消息推送原则，请求转发直接响应模式），创建房间（不属于消息推送原则，请求转发直接响应模式）。==> 深入思考为什么要引入Kafka和gRPC，而不是直接响应的形式。



分布式下，创建房间的广播消息具体是怎样实现的。



设置操作码，不用解析json数据来验证。



前端优化大致路径：

> UI 的优化，我的房间和其它房间分开显示；
>
> 一些布局调整；
>
> 新增所有房间显示；
>
> 所有房间、我的房间、其它房间



docker镜像遇到的一些问题

第三方依赖从本地安装还是git clone的形式

需要的中间件也要单独做成镜像

一旦镜像生成，镜像中包含了第三方依赖，那么每次启动容器都需要重新安装依赖，如何解决？



在制作docker镜像的过程中，前端项目集成了nginx服务，但是前端项目的资源都是动态资源，不合理，需要拆分。



关于logic层，可以将各个服务进行进一步拆分。



base库是从哪里来的，读取配置文件，工具库等，从github开源的吗？



服务端的 Dockerfile 将comet层，logic层和job层构建成了一个docker镜像。



项目拆分：重建CMakeList，出现muduo网络库重复引用问题。



问题：前端开放80端口，websocket连接的处理逻辑需要清晰，不然项目部署会有问题。



查询历史消息的bug问题。


添加nginx代理之后，用户长时间不活跃，nginx主动断开与后端的tcp连接，导致前端触发`onClose()`事件，收到websocket关闭码1006，导致用户主动退出聊天界面。

Redis6.0.16容器环境下，不支持开区间语法问题。