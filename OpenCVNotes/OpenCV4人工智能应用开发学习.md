# QT5 & OpenCV4.8 从入门到实战

### 一、环境搭建

#### 1.1 OpenCV环境集成到 QT 项目中

第一步，右击项目文件，添加外部库文件与库目录。

<center>
  <img src = ".\note_images\1.png" width = "80%">
</center>



```bash
"D:\CodeEnvironment\opencv\build\x64\vc16\lib\opencv_world480.lib"
"D:\CodeEnvironment\opencv\build\include"
```

<center>
  <img src = ".\note_images\2.png" width = "90%">
</center>


第二步，将 QT 项目的 `DEBUG` 模式改为 `RELEASE` 模式。

#### 1.2 一些开发 Bug 提示

> QT 中，解决对 `QString` 字符串赋值中文乱码问题，在 `.pro` 文件中，加入如下内容即可：
>
> ```bash
> # pro文件中，解决中文乱码
> msvc {
>    QMAKE_CFLAGS += /utf-8
>   QMAKE_CXXFLAGS += /utf-8
> }
>```