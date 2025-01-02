## 一、OpenCV项目配置步骤

- 视图 -> 其他窗口 -> 属性管理器 -> `Release|x64`下添加`Microsoft.Cpp.x64.user`

- 配置包含目录

​	打开`Microsoft.Cpp.x64.user`下的属性页面 -> `VC++`目录下的**包含目录**，加入如下两个文件夹：

​	`D:\CodeEnvironment\opencv\build\include`

​	`D:\CodeEnvironment\opencv\build\include\opencv2`

- 配置库目录

​	属性页找到`VC++`目录下的**库目录**，加入如下文件夹：

​	`D:\CodeEnvironment\opencv\build\x64\vc16\lib`

- 配置链接器

​	属性页链接器下输入的**附加依赖项**，加入如下指令：`opencv_world480.lib`

- 配置环境变量并重启VS

​	将如下文件夹加入到系统的环境变量：

​	`D:\CodeEnvironment\opencv\build\x64\vc16\lib`

## 二、OpenCV图像的一些基本操作

1. ##### 图像读取操作

```c++
//第一个参数为图像所在磁盘位置，第二个参数图像读取的方式，默认彩色读取
bool imread( const String& filename, int flags = IMREAD_COLOR );
//例子
Mat image = imread("D:/Photo/神童2.png", IMREAD_COLOR);
```

2. ##### 图像写入操作

```C++
//第一个参数为写入位置，第二个参数为Mat图像矩阵对象，第三个参数默认
bool imwrite( const String& filename, InputArray img,
              const std::vector<int>& params = std::vector<int>());
//例子
imwrite("D:/" + im_name, image);
```

3. ##### 图像显示操作

```c++
//第一个参数为图像显示的窗口名字，第二个参数为Mat图像矩阵对象
void imshow(const String& winname, InputArray mat);
//例子
imshow("src",src);
```

4. ##### 图像色彩空间转换，主要针对的是图像通道格式，比如从BGR转换到HSV，从BGR转换到GRAY

```c++
//第一个参数为Mat对象，表示原图，第二个参数为Mat对象，表示原图，第三个参数为色彩空间转换方式，第四个参数为输出图像的通道个数，一般会根据所转换的色彩空间自动推理
void cvtColor( InputArray src, OutputArray dst, int code, int dstCn = 0 );
//例子
cvtColor(this->src, hsv, COLOR_BGR2HSV);	//H 0~180 S,V 0~255
cvtColor(this->src, gray, COLOR_BGR2GRAY);	//灰度图像为单通道
```

5. ##### Mat图像矩阵对象的创建与赋值

```c++
//创建空白图像,第一个参数为图像大小，第二个参数为图像的通道类型，8UC3表示三通道，每个通道8位深度
Mat m3 = Mat::zeros(Size(512, 512), CV_8UC3);
//Mat对象赋值方式1：克隆，深拷贝
m1 = this->src.clone();

//Mat对象赋值方式2：拷贝，深拷贝 
this->src.copyTo(m2);

//Mat对象赋值方式3，等号，浅拷贝
Mat m3 = src;
```

6. ##### 图像像素点（通道）访问的两种方式

```c++
//访问图像的每一个通道前，需要先获取图像的基本属性
Mat img;
int w = img.cols;
int h = img.rows;
int dims = img.channels();	//每个像素通道数

//方式一，数组方式遍历（不推荐，还得分通道数的情况）
for (int i = 0; i < h ++i) {
    for (int j = 0; j < w; ++j) {
        if (dims == 1) {	//灰度图像，单通道
            int pv = this->src.at<uchar>(i, j);		//获取8位通道值
            this->src.at<uchar>(i, j) = 255 - pv;	//对通道值进行反转
        }
        if (dims == 3) {	//彩色图像，多通道
            Vec3b bgr= this->src.at<Vec3b>(i, j);	//数组返回

            this->src.at<Vec3b>(i, j)[0] = 255 - bgr[0];
            this->src.at<Vec3b>(i, j)[1] = 255 - bgr[1];
            this->src.at<Vec3b>(i, j)[2] = 255 - bgr[2];
        }
    }
}


//方式二，数组指针遍历（推荐，代码简洁高效）
	for (int i = 0; i < h; ++i) {
		uchar* current_row = this->src.ptr<uchar>(i);	//获取当前行指针
		for (int j = 0; j < w * dims; ++j) {
			*current_row++ = 255 - *current_row;		
		}
	}

```

7. ##### 图像像素点（通道）算术运算，主要是加减乘除

```c++
//方式一，通过Mat类提供的API计算，需要创建一个Mat图像矩阵，属于矩阵层面的运算
Mat m = Mat::zeros(this->src.size(), this->src.type());
m = Scalar(2, 2, 2);	//创建一个新的矩阵，并且对通道初始化

//第一个参数和第二个参数为操作数，第三个参数需要新的Mat对象接收运算结果
add(this->src, m, image1);
subtract(this->src, m, image2);
divide(this->src, m, image3);
multiply(this->src, m, image4);

//方式二，利用指针遍历图像每一个通道，操作每一个通道，属于普通数据类型的运算
for (int i = 0; i < h; ++i) {
    uchar* current_row1 = image1.ptr<uchar>(i);
    uchar* current_row2 = image2.ptr<uchar>(i);
    uchar* current_row3 = image3.ptr<uchar>(i);
    uchar* current_row4 = image4.ptr<uchar>(i);
    for (int j = 0; j < w * channels; ++j) {
        //saturate_cast<uchar>()函数保证数据不会溢出uchar的表示范围，无符号八位0~255
        *current_row1++ = saturate_cast<uchar>(*current_row1 + 100);
        *current_row2++ = saturate_cast<uchar>(*current_row2 - 100);
        *current_row3++ = saturate_cast<uchar>(*current_row3 * 2);
        *current_row4++ = saturate_cast<uchar>(*current_row4 / 2);
    }
}
```

8. ##### 利用OpenCV中GUI库提供的createTrackbar()创建滚动条，实现图像的对比度调整

```c++
/*
	第一个参数和第二个参数为字面意思，第二个参数为滚动条绑定的窗口名
	第三个参数定义滚动条位置
	第四个参数定义滚动条最大值
	第五个参数为滚动条绑定的函数事件指针
	第六个参数为用户数据，可以理解为滚动条需要操作的对象
*/
int createTrackbar(const String& trackbarname, const String& winname,
                              int* value, int count,
                              TrackbarCallback onChange = 0,
                              void* userdata = 0);

//例子
createTrackbar("light_value", "imageLight_contrast_operator", nullptr, 100, OnTrack1, (void *)(&this->src));
createTrackbar("contrast_value", "imageLight_contrast_operator", nullptr, 200, OnTrack2, (void*)(&this->src));

//其中，OnTrack1和OnTrack2为绑定在滚动条的函数事件，函数的定义需要满足TrackbarCallback所规定的形式
```

9. ##### OpenCV中`int waitKey(int delay = 0)`函数对应的键盘事件解析

```c++
/*
	int waitKey(int delay = 0)函数解析：该函数的功能是，通过参数delay阻塞程序的运行，delay的单位为ms。
	当delay != 0时，这段时间内，等待用户按下一个键，当检测到用户按下的键，就会返回对应的ASCII码，若用户没有按下键，则等待delay毫秒后，返回-1。
	当delay = 0时，等待用户按下一个键，否则程序一直阻塞，和system("pause")函数功能雷同，不同的是，waitKey()响应的键盘事件是OpenCV库创建的窗口，而system("pause")响应的键盘事件是控制台。
*/

//例子，通过键盘事件改变图像的色彩空间
	Mat dst = this->src.clone();
	while (true) {
		//控制用户与图形界面的交互，返回值为按下键盘的ASCII码，如果在形参指定时间内没有按下键，返回-1
		int c = waitKey(100);	
		if (c == 27) {	//Esc的ASCII码
			break;
		}
		if (c == 49) {
			cout << "Entering key 1" << endl;
			cvtColor(QuickDemo::src, dst, COLOR_BGR2HSV);
		}
		if (c == 50) {
			cout << "Entering key 2" << endl;
			cvtColor(QuickDemo::src, dst, COLOR_BGR2GRAY);
		}
		if (c == 51) {
			cout << "Entering key 3" << endl;
			Mat tmp = Mat::zeros(QuickDemo::src.size(), QuickDemo::src.type());
			tmp = Scalar(50, 50, 50);
			add(QuickDemo::src, tmp, dst);		//图像矩阵做加法操作
		}
		imshow("Key Event Demo", dst);			//实时更新窗口里面的图像内容
	}

```

10. ##### OpenCV中21种自带颜色转换的图像操作

```C++
//OpenCV库种存在枚举变量ColormapTypes，里面存放了21种颜色风格
enum ColormapTypes
{
    COLORMAP_AUTUMN = 0, 
    COLORMAP_BONE = 1, 
    COLORMAP_JET = 2, 
    COLORMAP_WINTER = 3,
    COLORMAP_RAINBOW = 4,
    COLORMAP_OCEAN = 5, 
    COLORMAP_SUMMER = 6, 
    COLORMAP_SPRING = 7, 
    COLORMAP_COOL = 8, 
    COLORMAP_HSV = 9, 
    COLORMAP_PINK = 10, 
    COLORMAP_HOT = 11, 
    COLORMAP_PARULA = 12, 
    COLORMAP_MAGMA = 13,
    COLORMAP_INFERNO = 14,
    COLORMAP_PLASMA = 15,
    COLORMAP_VIRIDIS = 16, 
    COLORMAP_CIVIDIS = 17,
    COLORMAP_TWILIGHT = 18, 
    COLORMAP_TWILIGHT_SHIFTED = 19,
    COLORMAP_TURBO = 20,
    COLORMAP_DEEPGREEN = 21
};
//OpenCV提供了applyColorMap()函数，转换图像的颜色风格，第三个参数为枚举类型ColormapTypes中颜色风格的编号
void applyColorMap(InputArray src, OutputArray dst, int colormap);
```

11. ##### 图像像素的逻辑操作

```c++
Mat m1, m2, dst;
//逻辑操作针对通道数值的二进制表示，主要包括与或非，异或操作
bitwise_and(m1, m2, dst);
imshow("dst1", dst);

bitwise_or(m1, m2, dst);
imshow("dst2", dst);

//异或，相同为0，相异为1
bitwise_xor(m1, m2, dst);
imshow("dst3", dst);

//取反操作在二值图目标提取应用较多
bitwise_not(this->src, dst);	//取反操作写法一
dst = ~this->src;	//取反操作写法二
```

12. ##### 图像的通道分离与合并

```c++
Mat src, dst;
vector<Mat>mv;	//Mat对象数组，接收通道分离后的图像

//对一个指定图像进行通道分离
split(src, mv);

//去除部分通道的值，然后合并
mv[2] = 0;
merge(mv, dst);

//可以输出通道分离与合并后的图像
```

13. ##### 对纯色图像进行背景更换

```c++
Mat src, hsv, redBG, mask;

//第一步，图像色彩空间转换为hsv，更容易提取颜色
cvtColor(src, hsv, COLOR_BGR2HSV);

//第二步，通过通道颜色表示范围表，将指定背景颜色通道值赋值为255，其余赋值为0
inRange(hsv, Scalar(100, 43, 46), Scalar(124, 255, 255), mask);

//第三步，创建需要更换颜色的canvas，以纯红为例
Mat redBG = Mat::zeros(src.size(), src.type());
redBG = Scalar(0, 0, 180);

//第四步，将inRange()生成的二值图进行取反，将ROI区域通道值赋值为255，非ROI（背景）区域赋值为0
bitwise_not(mask, mask);

//第五步，利用copyTO()函数，将原图像根据二值图的ROI区域复制到纯色背景图中
src.copyTo(redBG, mask);
```

14. ##### 像素统计学函数

```c++
//统计出每一个通道的最大值与最小值，并且输出其所在像素点位置
vector<Mat> mv;
Mat src;
split(src, mv);

double minValue, maxValue;	//存储通道的最大最小值
Point minLoc, maxLoc;		//存储最大最小通道值的像素点位置

//只能对单通道进行统计
for (int i = 0; i < mv.size(); ++i) {
    //
    minMaxLoc(mv[i], &minValue, &maxValue, &minLoc, &maxLoc, Mat());
    cout << "No." << i << " channels: " << endl;
    cout << "minValue: " << minValue << ", " << "maxValue: " << maxValue << endl;
    cout << "minLoc: " << minLoc << ", " << "maxLoc: " << maxLoc << endl;
}

//统计所有通道的均值与方差
Mat tmp = Mat::zeros(this->src.size(), this->src.type());
tmp = Scalar(255, 0, 0);

Mat mean, stddev; 
meanStdDev(tmp, mean, stddev);
cout << mean.at<double>(0, 0) << endl;	//输出mean图像矩阵(0, 0)位置的值
cout << "means: " << mean << endl;
cout<<"stddev: "<<stddev<<endl;		//Standard Deviation 标准差
```

15. ##### OpenCV中常见几何图形的绘制

```c++
Mat src;

//矩形的绘制
Rect rect;
rect.x = 50;	//初始化矩阵的基本参数
rect.y = 50;
rect.width = 100;
rect.height = 100;
rectangle(src, rect, Scalar(255, 0, 0), 2, LINE_AA, 0);	//thickness小于0表示填充


//圆的绘制
Point p1(100,100);		//初始化矩形的圆心
circle(src, p1, 50, Scalar(0, 0, 255), 2, LINE_AA, 0);

//椭圆的绘制
Point p2(100,100);		//初始化椭圆的圆心
RotatedRect rotatedRect;
rotatedRect.center = p2;		
rotatedRect.size = Size(100, 50);	//分别表示椭圆2*a和2*b的值
ellipse(src, rotatedRect, Scalar(255, 255, 0), 2, LINE_AA);	//ellipse 椭圆

//线条的绘制
line(src, Point(50, 50), Point(150, 150), Scalar(0, 255, 255), 2, LINE_AA, 0);	//需要两个点

Mat tmp = Mat::zeros(this->src.size(), this->src.type());
Mat dst;
//图像带权重的加法操作，gamma参数是偏置值，直接加到每一个通道上
addWeighted(this->src, 0.8, tmp, 0.2, 50, dst);

```

16. ##### OpenCV中提供的随机数生成

```c++
RNG rng(123);	//123表示随机数种子，随机数种子控制随机数的生成，相同种子的RNG对象生成的随机数序列是一样的

rng.uniform(0,512); 	//生成[0,512)范围的随机数
```

17. ##### 绘制和填充多边形，以五边形为例

```c++
Mat canvas = Mat::zeros(Size(512, 512), CV_8UC3);

//存储点集合
vector<Point> ps;

//初始化点
ps.push_back(Point(100, 100));
ps.push_back(Point(350, 100));
ps.push_back(Point(450, 280));
ps.push_back(Point(320, 450));
ps.push_back(Point(80, 400));

//绘制多边形，第三个参数表示是否将多边形闭合，第三个参数表示是否绘制闭合的多边形
polylines(canvas, ps, true, Scalar(0, 0, 255), 8, LINE_AA, 0);

//填充多边形
fillPoly(canvas, ps, Scalar(0, 0, 255), LINE_AA, 0);

//绘制轮廓和填充多边形
vector<vector<Point>> pss;	//创建点集合数组
pss.push_back(ps);		
drawContours(canvas, pss, -1, Scalar(0, 0, 255), -1, LINE_AA);	//第三个参数为-1表示绘制所有的点集合，第五个参数为-1表示填充轮廓，大于0绘制多边形



```

18. ##### 鼠标事件控制绘画，截取ROI区域

```c++
Point sp(-1, -1);	//记录开始位置
Point ep(1, 1);		//记录结束位置
Mat tmp;	//保存最原始的图片

//绑定鼠标事件, x, y表示鼠标在canvas里面的位置
static void OnMouse(int event, int x,int y, int flags, void* userdata) {
	if (event == EVENT_LBUTTONDOWN) {	//左键按下，初始化开始坐标位置
		sp.x = x;
		sp.y = y;
		cout << "Start point: " << sp << endl;
	}
	else if (event == EVENT_LBUTTONUP) {	//左键抬起，初始化结束坐标位置
		ep.x = x;
		ep.y = y;
		Rect rect(sp, ep);
		imshow("mouse_event", tmp);
		if (sp.x != ep.x && sp.y != ep.y) {		//当在mouse_event窗口点击且没有移动时，rect为NULL，创建图像矩阵失败
			Mat roi = tmp(rect).clone();
			imshow("ROI", roi);		//截取图像的矩形区域
			sp.x = -1;
			sp.y = -1;
		}
	}
	else if (event == EVENT_MOUSEMOVE) {
		Mat image = tmp.clone();
		ep.x = x;
		ep.y = y;
		if (sp.x > 0 && sp.y > 0) {
			image = tmp.clone();	//每次移动都用原始图像覆盖，只显示当前移动所创建的矩形
			rectangle(image, sp, ep, Scalar(0, 0, 150), 2, LINE_AA, 0);
			imshow("mouse_event", image);
		}
	}
}

//在一个普通函数中设置鼠标事件回调函数
void MouseDrawingDemo() {
    Mat src;
	imshow("mouse_event", src);	//在设置鼠标事件回调函数之前创建窗口
	tmp = src.clone();		//创建原始图像副本
	setMouseCallback("mouse_event", OnMouse, NULL);
}
```

19. ##### 图像的像素类型转换与归一化操作

```c++
Mat src, dst;

cout << src.type() << endl;				//输出图像的像素类型 
this->src.convertTo(dst, CV_32FC3);		//像素类型转换
cout << dst.type() << endl;

/*
	param third：归一化的上限
	param fourth：归一化的下限
	param fifth：归一化的类型
*/
normalize(dst, dst, 1.0, 0, NORM_MINMAX);	//转换为float类型的像素后，进行归一化操作


imshow("src", src);
imshow("normalize_dst", dst);
```

20. ##### 图像的放缩与插值

图像的放缩：即放大或缩小图像的尺寸

图像的插值：在图像进行放大或者缩小的过程中，图像的**像素点会变少或者变多**，这时图像的信息就需要通过某种方法来进行获取，插值法就是旨在 <font color = Blue>通过某些规则/规范/约束，获取这些多出坐标点的像素值</font>，常见的线性插值包括：**一阶线性插值（最近邻近插值）、双线性插值核双三次插值**。

```c++
Mat src;
Mat zoomin, zoomout;
int width = src.cols;
int height = src.rows;
imshow("src",src);

//线性插值法
resize(src, zoomin, Size(width / 2, height / 2), 0, 0, INTER_LINEAR);
imshow("zoomin", zoomin);
resize(src, zoomout, Size(1.5 * width, 1.5 * height), 0, 0, INTER_LINEAR);
imshow("zoomout", zoomout);
```

21. ##### 图像的翻转（flip，镜像操作）

OpenCV提供flip函数，可以实现三种翻转（镜像）操作：x轴、y轴和y = x对称。

```c++
Mat src, dst;

imshow("src", src);

flip(src, dst, 0);	//x轴对称翻转
imshow("flip(0)", dst);

flip(src, dst, 1);	//y轴对称翻转
imshow("flip(1)", dst);

flip(src, dst, -1);	//y=x对称翻转 
imshow("flip(-1)", dst);
```

22. ##### 图像旋转，根据仿射变换矩阵 **M** 控制图像的旋转方式，仿射变换矩阵**M**中每个元素如下，其中，**x和y表示图像的中心点位置，a = scale\*cos(angle)，b = scale\*sin(angle)，angle表示放大系数，angle表示角度**。

$$
\begin{bmatrix}
  a& b& (1-a)\cdot x-b\cdot y\\
  -b& a& b\cdot x+(1-a)\cdot y
\end{bmatrix}
$$

**重点关注第三列的元素：第一行第三列和第二行第三列元素分别表示图像经过旋转和缩放后，图像中心点在x轴和y轴的平移量，这个平移量保证了图像旋转之后，中心点x和y的值不变**。

```c++
	Mat src, dst, M;
	int w = src.cols;
	int h = src.rows;
	/*
		第一步，获取仿射变换矩阵
		param first: 中心点位置
		param second: 旋转角度
		param third: 放大系数
	*/
	M = getRotationMatrix2D(Point(w / 2, h / 2), 45, 1.0);

	double cos = M.at<double>(0, 0);
	double sin = M.at<double>(0, 1);
	
	//求出旋转后图像的宽度和高度
	double nw = w * cos + h * sin;
	double nh = w * sin + h * cos;

	//第二步，修改仿射变换矩阵的偏置值
	M.at<double>(0, 2) += (nw / 2 - w / 2);
	M.at<double>(1, 2) += (nh / 2 - h / 2);

	//第三步，进行仿射变换
	warpAffine(this->src, dst, M, Size(nw, nh));
	imshow("src", this->src);
	imshow("rotate 45°", dst);
```

23. ##### 读取写入视频流文件

```c++
VideoCapture capture(0);		//参数为0，打开默认摄像头
Mat frame;

//获取视频属性
int frame_width = this->video.get(CAP_PROP_FRAME_WIDTH);
int frame_height = this->video.get(CAP_PROP_FRAME_HEIGHT);
int total_count = this->video.get(CAP_PROP_FRAME_COUNT);	//视频总帧数
double fps = this->video.get(CAP_PROP_FPS);	


/*
	创建视频写入对象
	param second: 视频编解码格式
	param last: 控制视频颜色
*/
VideoWriter write_object("D:\\test.mp4", this->video.get(CAP_PROP_FOURCC), fps, Size(frame_width, frame_height), true);	//最后一个参数视频颜色

while (1) {
    int flag = waitKey(100/6);		//程序阻塞可以控制帧率		
    if (flag == 27) break;
    this->video.read(frame);		//将视频中一帧图像提取到frame图像矩阵中
    flip(frame, frame, 1);			//图像沿y轴翻转
    write_object.write(frame);		//以帧的形式写入视频流文件
    if (frame.empty()) break;
    imshow("frame", frame);
}

//release视频资源，最好主动释放，因为摄像头属于硬件资源，便于以后多线程控制访问
capture.release();
write_object.release();
```

24. ##### 图像通道统计，输出直方图

```c++
//存储通道分离
vector<Mat> bgr;
split(this->src, bgr);
//定义绘制直方图参数变量
const int channels[1] = { 0 };	
const int bins[1] = { 256 };	//通道取值的个数
float hranges[2] = { 0,255 };	//通道值的取值范围
const float* ranges[1] = { hranges };	//指针变量数组


Mat b_hist;
Mat g_hist;
Mat r_hist;

//计算三个通道的直方图
calcHist(&bgr[0], 1, 0, Mat(), b_hist, 1, bins, ranges);
calcHist(&bgr[1], 1, 0, Mat(), g_hist, 1, bins, ranges);
calcHist(&bgr[2], 1, 0, Mat(), r_hist, 1, bins, ranges);



```

25. ##### 续24，绘制直方图

```c++
//直方图的基本参数
int hist_w = 512;
int hist_h = 400;
//cvRound函数对浮点数进行四舍五入
int bin_w = cvRound((double)hist_w / bins[0]);		//直方图横轴每个元素所占像素点大小
Mat HistImage = Mat::zeros(Size(hist_w, hist_h), CV_8UC3);
HistImage = Scalar(255, 255, 255);	//直方图纯白背景

//归一化直方图数据
normalize(b_hist, b_hist, 0, HistImage.rows, NORM_MINMAX, -1, Mat());
normalize(g_hist, g_hist, 0, HistImage.rows, NORM_MINMAX, -1, Mat());
normalize(r_hist, r_hist, 0, HistImage.rows, NORM_MINMAX, -1, Mat());

//绘制直方图曲线
for (int i = 1; i < bins[0]; ++i) {
    Point p1(bin_w * (i - 1), hist_h - cvRound(b_hist.at<float>(i - 1, 0)));
    Point p2(bin_w * (i), hist_h - cvRound(b_hist.at<float>(i, 0)));
    line(HistImage,  p1, p2, Scalar(255, 0, 0), 2, LINE_AA, 0);

    Point p3(bin_w * (i - 1), hist_h - cvRound(g_hist.at<float>(i - 1, 0)));
    Point p4(bin_w * (i), hist_h - cvRound(g_hist.at<float>(i, 0)));
    line(HistImage, p3, p4, Scalar(0, 255, 0), 2, LINE_AA, 0);

    Point p5(bin_w * (i - 1), hist_h - cvRound(r_hist.at<float>(i - 1, 0)));
    Point p6(bin_w * (i), hist_h - cvRound(r_hist.at<float>(i, 0)));
    line(HistImage, p5, p6, Scalar(0, 0, 255), 2, LINE_AA, 0);
}

//显示直方图
imshow("HistImage", HistImage);
```

26. ##### 图像卷积操作之均值模糊

OpenCV中，对图像进行模糊操作的主要目的是降低图像的噪声，通常采用卷积核的方式。

均值卷积：对卷积核内的元素**求平均值**代替卷积核覆盖图像范围的**中心位置**

```c++
Mat dst;
/*
	均值卷积（卷积核系数都是同值）操作函数，自定义卷积操作为3*3大小
	param third: 卷积核的大小
	param fourth: -1表示默认改变像素点位置是卷积核的中心位置
*/
blur(this->src, dst, Size(3, 3), Point(-1, -1));
imshow("src", this->src);
imshow("Convolution Blur", dst);
```

27. ##### 图像卷积操作之高斯模糊

对需要改变卷积核中心位置的像素点进行高斯函数计算，具体的，计算整个核覆盖的像素点高斯函数值（由于高斯函数是**概率密度**，卷积核的每一个像素点新值和为1），得出核的新像素点大小。

```c++
Mat	dst;
GaussianBlur(this->src, dst, Size(2, 2), 10);
imshow("src", this->src);
imshow("Gaussian Blur", dst);
```

28. ##### 图像卷积之高斯双边模糊

对于一张图像，非轮廓的部分进行模糊，轮廓部分进行原有信息保存（像素值差异大的进行保留）

```c++
Mat dst;
bilateralFilter(this->src, dst, 0, 100, 10);
imshow("src",src);
imshow("bilateralFilter", dst);
```

