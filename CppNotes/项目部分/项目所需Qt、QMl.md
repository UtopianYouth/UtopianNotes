[toc]

# 一、Qt

## 1.简介

Qt常用于开发图形界面应用程序，但它除了绘制优秀的界面外还内置了很多其他的功能，多线程、访问数据库、图库处理、音频视频处理、网络通信、文件操作等。其主要用于桌面级和嵌入式的开发。

## 2.优点

Qt相对于其他GUI库的优点：

* 简单易学：封装的很好，不需要了解windows API
* 资料丰富：其用户数量庞大，无论是官方还是非官方的资料都很齐全
* 界面优秀：可以较为容易的做出漂亮的界面和炫酷的动画
* 独立安装：全部编译为本地代码，不需要其他库的支撑
* 跨平台：跨平台支持很好，成本很低

## 3.Qt开发工具列表：

| 工具          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| **qmake**     | 核心的项目构建工具，可以生成跨平台的 **.pro 项目文件**，并能依据不同操作系统和编译工具生成相应的 Makefile，用于构建可执行程序或链接库。 |
| **uic**       | User Interface Compiler，用户界面编译器，Qt 使用 XML 语法格式的 .ui 文件定义用户界面，uic 根据 .ui 文件生成用于创建用户界面的 C++ 代码头文件，比如 ui_xxxx.h 。 |
| moc           | Meta-Object Compiler，元对象编译器，moc 处理 C++ 头文件的类定义里面的 Q_OBJECT 宏，它会生成源代码文件，比如 moc_*****.cpp ，其中包含相应类的元对象代码，元对象代码主要用于实现 Qt 信号/槽机制、运行时类型定义、动态属性系统。 |
| rcc           | Resource Compiler，资源文件编译器，负责在项目构建过程中编译 .qrc 资源文件，将资源嵌入到最终的 Qt 程序里。 |
| **qtcreator** | 集成开发环境，包含项目生成管理、代码编辑、图形界面可视化编辑、 编译生成、程序调试、上下文帮助、版本控制系统集成等众多功能， 还支持手机和嵌入式设备的程序生成部署。 |
| assistant     | Qt 助手，帮助文档浏览查询工具，Qt 库所有模块和开发工具的帮助文档、示例代码等都可以检索到，是 Qt 开发必备神器，也可用于自学 Qt。 |
| **designer**  | Qt 设计师，专门用于可视化编辑图形用户界面（所见即所得），生成 .ui 文件用于 Qt 项目，或者用于设计QML界面。 |
| **linguist**  | Qt 语言家，代码里用 tr() 宏包裹的就是可翻译的字符串，开发人员可用 lupdate 命令生成项目的待翻译字符串文件 .ts，用 linguist 翻译多国语言 .ts ，翻译完成后用 lrelease 命令生成 .qm 文件，然后就可用于多国语言界面显示。 |
| qmlscene      | 在 Qt 4.x 里是用 qmlviewer 进行 QML 程序的原型设计和测试，Qt 5 用 qmlscene 取代了旧的 qmlviewer。新的 qmlscene 另外还支持 Qt 5 中的新特性 scenegraph 。 |

## 4.项目组成和管理

在Qt Creator左侧工具栏中单击“编辑”按钮，即可进入项目管理窗口，Qt Creator可以打开多个项目，但只有一个活动项目，活动项目名称加粗显示。

在项目名称节点下，分组管理着各种源文件，文件和分组如下：

* xxx.pro文件是项目管理文件，包括一些对项目的设置项。
*  Headers 分组，该节点下是项目内的所有头文件（.h） 
*  Sources 分组：该节点下是项目内的所有 C++源文件（.cpp） 
* Resources分组：该节点下是项目内的各种资源文件，如QML文件，图标（图片）文件，翻译文件等
  * 使用qrc文件加载资源
*  Forms 分组：该节点下是项目内的所有界面文件（.ui） 用QML编写界面则没有（.ui）文件

## 5.pro文件详解

在此文件中#为注释

```pro
TEMPLATE = app                  #表示项目使用的模板是app，是一般的应用程序
QT += qml quick gui widgets     #表示项目中加入qml quick gui widgets模块
TARGET = MYPlayer               #表示生成的目标可执行文件的名称
RESOURCES += resource/MYPlayer.qrc   #资源项目文件
TRANSLATIONS += ./resource/ui/translation/MYPlayer_zh_CN.ts  #翻译文件
SOURCES += \            
        main.cpp \
        MainApp.cpp
HEADERS += MainApp.h
```

## 6.Qt元对象系统

 Qt 的元对象系统（Meta-Object System）提供了对象之间通信的信号与槽机制、运行时类型信息和动态属性系统。 

元对象系统由以下三个基础组成：

1. QObject 类是所有使用元对象系统的类的基类。
2. 在一个类的 private 部分声明 Q_OBJECT宏，使得类可以使用元对象的特性，如动态属性、信号与槽。
3. MOC（元对象编译器）为每个 QObject 的子类提供必要的代码来实现元对象系统的特性。

 构建项目时，MOC 工具读取 C++ 源文件，当它发现类的定义里有 Q_OBJECT 宏时，它就会为这个类生成另外一个包含有元对象支持代码的 C++ 源文件，这个生成的源文件连同类的实现文件一起被编译和连接。 

## 7.Qt基本模块

| 模块                  | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| Qt Core               | 其他模块都用到的核心非图形类                                 |
| **Qt GUI**            | 设计 GUI 界面的基础类，包括 OpenGL                           |
| **Qt Multimedia**     | 音频、视频、摄像头和广播功能的类                             |
| Qt Multimedia Widgets | 实现多媒体功能的界面组件类                                   |
| Qt Network            | 使网络编程更简单和轻便的类                                   |
| Qt QML                | 用于 QML 和 [JavaScript](http://c.biancheng.net/js/) 语言的类 |
| **Qt Quick**          | 用于构建具有定制用户界面的动态应用程序的声明框架             |
| **Qt Quick Controls** | 创建桌面样式用户界面，基于 Qt Quick 的用户界面控件           |
| **Qt Quick Dialogs**  | 用于 Qt Quick 的系统对话框类型                               |
| Qt Quick Layouts      | 用于 Qt Quick 2 界面元素的布局项                             |
| Qt SQL                | 使用 SQL 用于数据库操作的类                                  |
| Qt Test               | 用于应用程序和库进行单元测试的类                             |
| Qt Widgets            | 用于构建 GUI 界面的 [C++](http://c.biancheng.net/cplus/) 图形组件类 |

## 8.Qt定时器

QTimer类： 如果Qt无法交付所请求的计时器事件数量，它将静默地丢弃一些计时器事件。所以有时候用QTimer可能不能正常刷新，此时可以使用startTimer。

1. 用 startTimer启动一个定时器，并返回定时器id。如果启动失败，返回0.
2. 定时器每隔interval 毫秒就会启动一次，直到调用killTimer(). 如果interval=0,当没有其他系统时间发生时，会调用一次。
3. 当定时器发生时，会调用timerEvent(QTimerEvent *event).如果多个定时器在运行，可用通过timeId()来区分。

如：startTimer(40);   //一秒25帧

## 9.QThread

线程等待

void msleep(unsigned long msecs)

强制当前线程睡眠msecs毫秒

## 10.QAudioFormat、QAudioOutput

需要在qmake中添加 QT += multimedia 

QAudioFormat：

| 参数               | 描述                                                   |
| ------------------ | ------------------------------------------------------ |
| Sample Rate        | 每秒音频数据采样数（以赫兹为单位）。                   |
| Number of channels | 音频通道数（通常为单声道一个，为立体声两个）           |
| Sample size        | 每个样本中存储了多少数据（通常为8或16位）              |
| Sample type        | 样本的数值表示（通常为有符号整数，无符号整数或浮点数） |
| Byte order         | 样本的字节顺序（通常为小端，大端）                     |

QAudioOutput：

| 函数        | 作用                                                         |
| ----------- | ------------------------------------------------------------ |
| start()     | 返回一个指向内部QIODevice的指针，该指针用于将数据传输到系统的音频输出。write()可以直接向其中写入数据。 |
| stop()      | 停止音频输出，与系统资源分离。                               |
| suspend()   | 停止处理音频数据，保留缓冲的音频数据。                       |
| resume()    | 在suspend()后恢复处理音频数据                                |
| bytesFree() | 返回音频缓冲区中可用的可用字节数。                           |
| ...         | ...                                                          |



# 二、QML

1. QML是一种标记语言，类似于网页设计的HTML，主要用于描述程序界面，其用于定义用户界面元素的外观和行为，可以借助CSS对它进行美化，也可以借助JavaScript进行交互。其注释方式与C++相同，//和/* */
2. 使用QML开发界面优点
   * QML很灵活，可以做出炫酷的效果
   * QML是标记语言，容易编写和理解，也易于学习
   * QML界面简洁且美观，自带很多动画
   * 不同平台的QML使用相同的渲染机制，界面效果一致，不会随操作系统的同而变化

## QML基础语法

### 1.示例

一个QML文档分为import和对象生命两个部分

* import：用于指定该文档所需要引入的模块。通常这是一个模块名和版本号，比如QtQuick 2.0，也可以引入自己的模块或者其他文件
* QML元素：一个QML文档有且只有一个根元素，类似XML文档的规定。QML文档中的元素同样类似XML文档

### 2.元素关系：

* 层次关系
  * 子元素处于相对于父元素的坐标系统中，即子元素的x和y的坐标值始终相对于父元素。

### 3.基础属性

（1）属性的值由其类型决定，如果一个属性没有给值，则会使用属性的默认值,自定义属性会给定系统类型的默认值。

（2）属性可以依赖于其他属性，这种行为被称为绑定，其类似于信号槽机制，所依赖属性发生变化，该属性会得到通知，自动更新自己的值。int类型的属性会自动转换成字符串，在值发生变化时，绑定依然成立

（3）我们可以声明默认属性

* id                                                          //标识符，[^必须唯一]，相当于该元素的指针，可以直接用名字指代该元素。id不是字符串，而是一个特殊的标识符类型。一旦指定不可更改，命名规则与C++指针一致，不允许反查id
* x、y坐标                                              //相对于父元素的坐标
* width、height                                       //宽高
* property int times:24                               //自定义属性，格式为property <type> <name>:<value>

  * property alias anotherTimes:times   //属性别名，“别名即引用”
* text:"Greetings" + times                        //文本和值
* onHeightChanged:console.log('height:',height)   //属性值改变的信号处理回调，后面为控制台输出信息
* focus:true                                             //接受键盘事件需要设置focus
* color:focus?"red":"black"                      //根据focus值改变颜色

所有属性都有对应数值改变的槽函数，格式为on+属性名+Changed，属性名首字母大写，属性改变会发出相应的信号比如height属性改变，发出heightChanged信号

### 4.所用组件

#### 可视元素

1. Item           //一般用作容器
   * Item是所有可视元素中最基本的一个，其是所有其它可视元素的父元素
2. Image       //图片组件
   * smooth  :设置平滑
   * source：图面路径设置，资源中选取文件，相对路径前需要加**qrc:///**,绝对路径前需要加**file:///**
3. Rectanle    //默认的长方形组件
   * radius：半径，可以绘制圆
4. Text          //文本组件
   * text
5. MouseArea      //鼠标事件处理组件，不可见
   * onDoubleClicked
   * onEntered
   * onExited

可视元素通用属性：

| 分组     | 属性                                                         |
| -------- | ------------------------------------------------------------ |
| 几何     | x和y 用于定义元素左上角的坐标，width 和height 则定义了元素的范围。z定义了元素上下的层叠关系。 |
| 布局     | anchors（具有 left、right、top、bottom、vertical 和 horizontal center 等属性）用于定位元素相对于其它元素的margins的位置。 |
| 键盘处理 | Key 和 KeyNavigation 属性用于控制键盘；focus 属性则用于启用键盘处理，也就是获取焦点。 |
| 变形     | 提供 scale 和 rotate 变形以及更一般的针对 x、y、z 坐标值变换以及transformOrigin点的 transform 属性列表。 |
| 可视化   | opacity 属性用于控制透明度；visible 属性用于控制显示/隐藏元素；clip 属性用于剪切元素；smooth 属性用于增强渲染质量。 |
| 状态定义 | 提供一个由状态组成的列表 states 和当前状态 state 属性；同时还有一个transitions列表，用于设置状态切换时的动画效果。 |

#### 其它元素

1. Slider      //进度条
   * stepSize     //精度
2. Button      //按钮
   * onClicked
3. FileDialog      //选择文件框，需要执行open函数
   * title：标题
   * folder：默认选择路径
   * nameFilters：过滤器，过滤出可选择文件，
     * ["xxxx(\*.xx \*.xx)","xxxx(\*)"],如：["Video files(\*.mp4 \*.rmvb \*.flv)","All files(*)"]
   * onAccepted：选择了文件
   * onRejected：取消选择
4. Timer       //计时器，需要调用start开启计时，也可以用stop结束计时
   * interval：时间
   * onTriggered            //开始计时后经过相应时间后触发

# 三、C++补充

## 1.List

Lists将元素按顺序储存在链表中. 与 向量(vectors)相比, 它允许快速的插入和删除，但是随机访问却比较慢。

|      函数       |              作用              |
| :-------------: | :----------------------------: |
|    assign()     |           给list赋值           |
|     back()      |        返回最后一个元素        |
|     begin()     |   返回指向第一个元素的迭代器   |
|     clear()     |          删除所有元素          |
|   **empty()**   |  **如果list是空的则返回true**  |
|      end()      |        返回末尾的迭代器        |
|     erase()     |          删除一个元素          |
|   **front()**   |       **返回第一个元素**       |
| get_allocator() |        返回list的配置器        |
|    insert()     |      插入一个元素到list中      |
|   max_size()    |  返回list能容纳的最大元素数量  |
|     merge()     |          合并两个list          |
|   pop_back()    |        删除最后一个元素        |
| **pop_front()** |       **删除第一个元素**       |
| **push_back()** |  **在list的末尾添加一个元素**  |
|  push_front()   |    在list的头部添加一个元素    |
|    rbegin()     | 返回指向第一个元素的逆向迭代器 |
|    remove()     |         从list删除元素         |
|   remove_if()   |       按指定条件删除元素       |
|     rend()      |    指向list末尾的逆向迭代器    |
|    resize()     |         改变list的大小         |
|    reverse()    |        把list的元素倒转        |
|   **size()**    |    **返回list中的元素个数**    |
|     sort()      |           给list排序           |
|    splice()     |          合并两个list          |
|     swap()      |          交换两个list          |
|    unique()     |      删除list中重复的元素      |

## 2.mutex(互斥锁)

多线程访问同一资源时，为了保证数据的一致性，最简单的方式就是使用 mutex（互斥锁）。 

创建实例，直接调用mutex的lock/unlock函数即可

```c++
mutex mux;
mux.lock();
/////////////////////////////
mux.unlock();
```



# 四、操作

## 1.语言翻译更新

1. 在pro文件中添加 TRANSLATIONS = xxxxxx.ts
2. 编译程序后，于Qt Creator中工具——外部——Qt语言家——更新翻译   //此时生成.ts文件
3. 使用Qt Linguist打开.ts文件选择对应语句填写对应译文并勾选翻译内容
4. Qt Creator——工具——外部——Qt语言家——发布更新      //此时生成.qm文件
5. 在项目中加载.qm文件

```QML
//rectangle.qml
import QtQuick 2.0
//根元素：Rectangle
Rectangle{
	//命名根元素
	id:root //声明属性：<name>:<value>，id必须唯一
	width:120;height:240
	color:"#D8D8D8" //颜色属性
	//声明一个嵌套元素（根元素的子元素）
	Image{
		id:rocket
			x:(parent.width - width)/2;y:40//使用parent引用父元素
			source:'assets/rocket.png'
	}
	//根元素的另一个子元素
	Text{
		//该元素未命名
		y:rocket.y + rocket.height + 20 //使用id引用元素
		width:root.width   //使用id引用元素
		horizontalAlignment:Text.AlignHCenter
		text:'Rocket'
	}
}
```





[^必须唯一]:QML有一种动态作用域机制，后加载的文档会覆盖掉前面加载的文档的相同id，但这样极其难以维护，所以请保证id唯一