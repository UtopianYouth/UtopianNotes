## 一、图像部分

### 1.1 图像的五种阈值分割方法

​		OpenCV提供了**5种阈值分割的方法**，一般的，我们对图像进行二值分割比较多，所以前面两种阈值分割的方法用的比较多，需要注意的是，自定义阈值T的分割法支持多通道，以通道为单位操作。

```c++
//阈值法1，二值化，if > T 255 else 0
threshold(gray, binary1, 127, 255, THRESH_BINARY);

//阈值法2，二值化，if < T 255 else 0 
threshold(gray, binary2, 127, 255, THRESH_BINARY_INV);	//invert 反转

//阈值法3，if < T 原值 else T
threshold(gray, binary3, 127, 255, THRESH_TRUNC);		//truncate 截断

//阈值法4，if < T 0 else 原值
threshold(gray, binary4, 127, 255, THRESH_TOZERO);

//阈值法5，if < T 原值 else 0
threshold(gray, binary5, 127, 255, THRESH_TOZERO_INV);
```

###  1.2 图像的全局阈值分割法

​		利用OpenCV提供的API**确定阈值T**，然后基于T进行二值分割，常见的全局阈值分割法有：**均值法，OTSU阈值法，三角阈值法**。

```C++
//均值法，自己求灰度图像的像素均值
Scalar s = mean(gray);		
double t1 = threshold(gray, binary1, s[0], 255,THRESH_BINARY);	//返回阈值T

//OTSU阈值，求出每个像素点的最小类内（以像素点为基准，将图像像素点分为两类）方差，作为阈值
double t2 = threshold(gray, binary2, 0, 255, THRESH_BINARY | THRESH_OTSU);


//三角阈值，原理基于最多像素值（只对单峰图像效果较好，如医学生物类图像）和最大像素值
double t3 = threshold(gray, binary3, 0, 255, THRESH_BINARY | THRESH_TRIANGLE);

```

### 1.3 自适应阈值（局部阈值）分割法

​		**自适应阈值分割法**特适用于**光照不均匀或背景复杂的图像**，在这些情况下，全局阈值分割法**可能无法有效地**分割目标和背景。

```c++
/*
	自适应阈值分割法函数原型
	
	src: 单通道图像
	dst: 输出的目标图像
	maxValue: 大于0的最大像素值
	adaptiveMethod: 使用的自适应阈值方法，常见的有 ADAPTIVE_THRESH_GAUSSIAN_C和ADAPTIVE_THRESH_MEAN_C
	thresholdType: 所使用阈值的方法
	blockSize: 像素邻居的块大小，可以理解为局部块的大小
	C: 常数C，为整数，计算出局部块的阈值之后，所减去的值
*/
void adaptiveThreshold(InputArray src, OutputArray dst, double maxValue, int adaptiveMethod, int thresholdType, int blockSize, double C);

/*
		自适应阈值使用例子
*/
void QuickDemo::SelfAdaptionThresholdDemo() {
	Mat src = imread("D:\\Photo\\images\\text1.png"); 
	Mat gray;
	Mat binary1, binary2,binary3;
	cvtColor(src, gray, COLOR_BGR2GRAY);

	//OTSU全局阈值，对某些图像进行二值分割效果不好
	double t2 = threshold(gray, binary3, 0, 255, THRESH_BINARY | THRESH_OTSU);
	cout << t2 << endl;

	//自适应阈值二值分割法，通过图像卷积操作，每个局部的阈值都是不一样的
	adaptiveThreshold(gray, binary1, 255, ADAPTIVE_THRESH_GAUSSIAN_C, THRESH_BINARY, 33, 10 );
	adaptiveThreshold(gray, binary2, 255, ADAPTIVE_THRESH_MEAN_C, THRESH_BINARY, 21, 8);


	imshow("gray", gray);
	imshow("adaptive_gaussian_threshold", binary1);
	imshow("adaptive_mean_threshold", binary2);
	imshow("OTSU_threshold", binary3);
}


```

###  1.4 图像中联通组件的发现

​		在OpenCV的图像中，联通组件可以简单的理解为是**图像中每一个元素（比如一个风景图像，花，树木等都属于一个联通组件）**，<font color = red>一般图像的背景也是一个联通组件，在检测联通组件的时候，背景一般是第一个联通组件，通常被赋值为0</font>。

```c++
/*
	联通组件发现函数原型1（第2个原型在第五部分介绍）

	param image: input image.
    param labels: output result, storage the label value of connected components.
    param connectivity: can be understood as the method of finding connected components, normal value is 8 or 4.
    param ltype: output image label type. Currently CV_32S and CV_16U are supported.
    param ccltype: connected components algorithm type, normal value is CCL_DEFAULT.
    
    return: the number of connected components.
*/
int connectedComponents(InputArray image, OutputArray labels, int connectivity, int ltype, int ccltype);

/*
	使用例子
	binary1：一个二值图像，调用联通组件API前，应该先将图像进行二值化
	labels：存储了联通组件信息，同一个联通组件在图像中覆盖的位置，值相同，其中，背景的值为0
*/
Mat labels = Mat::zeros(binary1.size(), CV_32S);
int num_labels = connectedComponents(binary1, labels, 8, CV_32S, CCL_DEFAULT);	//返回联通组件个数
```

### 1.5 续 1.4

​		在OpenCV中，提供的另一个联通组件发现的API，`connectedComponentsWithStats`，该API不仅可以实现联通组件的发现，还可以存储**每一个联通组件的状态信息（联通组件的中心位置，联通组件的矩形范围）**。

```C++
/*
	联通组件发现函数原型2
	
    param image: input image.
    param labels: output result, storage the label value of connected components.
    param stats: the status of connected component, including backgroud label.
    param centroids: the central position of connected component, including x and y axis
    param connectivity: can be understood as the method of finding connected components, normal value is 8 or 4.
    param ltype: output image label type. Currently CV_32S and CV_16U are supported.
    param ccltype: connected components algorithm type, normal value is CCL_DEFAULT.
    
    return: the number of connected components.
	
*/

int connectedComponentsWithStats(InputArray image, OutputArray labels, OutputArray stats, OutputArray centroids, int connectivity, int ltype, int ccltype);

/*
	使用例子
	labels：每一个联通组件的取值
	stats：每一个联通组件的矩形范围信息，比如最左上角点的信息，矩形的宽和高，矩形面积等
	centroids：每一个联通组件的中心位置
*/
Mat labels = Mat::zeros(binary.size(), CV_32S);		//存储联通组件标签，同一个联通组件的像素标签相同
Mat stats, centroids;	//stats存储了每个联通组件的矩形范围，centroids存储了每个联通组件中心位置

int num_labels = connectedComponentsWithStats(binary, labels, stats, centroids, 8, CV_32S, CCL_DEFAULT);	//CV_32S: 32位有符号整数

```

### 1.6 轮廓发现和绘制

​		在OpenCV中，图像轮廓的定义很好理解，**就是每一个图像元素的边缘（比如风景图，花的边缘和树的边缘等）**，轮廓发现和绘制就是通过OpenCV提供的API，**将图像所有的元素轮廓进行检测，并且绘制出来，其中，轮廓发现是基于二值图的**。

```c++
/*
	轮廓发现函数原型
	
	param image: 输入的二值图像
	param contours: 存储检测到的每一个轮廓点集合(vector<vector<cv::Point>>).
	param heirarchy: 可选参数vector(vector<cv::Vec4i>), 存储内外轮廓之间的嵌套关系，hierarchy[i][0]~hierarchy[i][3]分别存储了后一个轮廓、前一个轮廓、父轮廓和内嵌轮廓的索引编号。
	mode: 轮廓检测的模式, 常见的两种模式为RETR_TREE（检测所有轮廓，并且构建所有轮廓的拓扑关系）和RETR_EXTERNAL（只检测外部轮廓）.
	method: 轮廓逼近的方法，常见的取值为CHAIN_APPROX_SIMPLE（只取能体现轮廓的少部分点）和CHAIN_APPROX_NONE（取轮廓所有的点，效率较慢）
	param offset: 默认即可.
*/
void findContours( InputArray image, OutputArrayOfArrays contours, OutputArray hierarchy, int mode, int method, Point offset = Point());

/*
	使用例子
	binary：二值图
*/
vector<vector<Point>> contours;		//存储每一个轮廓所构成的点集合
vector<Vec4i> hierachy;				//存储图像的层次结构
findContours(binary, contours, hierachy, RETR_TREE, CHAIN_APPROX_SIMPLE, Point());

```

```c++
/*
	轮廓绘制函数原型，解释一些比较关键的参数

	image：与上面findContours()不同的是，这里image是多通道图像，很好理解，我们绘制的轮廓一般是带颜色的
	contourIdx：轮廓数组contours的索引，大于0时绘制一个轮廓，小于零时绘制轮廓数组contours的所有轮廓
	hierarchy：存储图像轮廓层次的数组，默认即可
*/
void drawContours( InputOutputArray image, InputArrayOfArrays contours,int contourIdx, const Scalar& color, int thickness = 1, int lineType = LINE_8, InputArray hierarchy = noArray(), int maxLevel = INT_MAX, Point offset = Point());

/*
	使用例子
	src：需要绘制轮廓的多通道图像
	第三个参数为-1表示绘制所有轮廓，也可以使用循环一个一个轮廓的绘制
*/
drawContours(src, contours, -1, Scalar(0, 255, 255), 2, LINE_AA);	//第三个参数为负数，绘制所有轮廓

for (size_t i = 0; i < contours.size(); ++i) {
	drawContours(src, contours, i, Scalar(0, 0, 255), 2, LINE_AA);	//第三个参数为正数，绘制一个轮廓
}

```

### 1.7 轮廓面积与周长，最大 or 最小外接矩形

​		如6所示，当我们进行轮廓发现之后，**会得到轮廓数组contours**，通过该轮廓数组，利用OpenCV提供的一些**轮廓相关的API**，可以输出轮廓相关的信息。

```c++
/*
	计算轮廓面积函数原型
	contour：轮廓的点集合
	oriented：是否返回带有方向的值（矢量），默认false返回标量即可
*/
double contourArea( InputArray contour, bool oriented = false);

/*
	计算轮廓周长函数原型
	curve：轮廓点集合
	closed：曲线（轮廓）是否闭合，findContours返回的轮廓一般是闭合的，这里true即可
*/
double arcLength( InputArray curve, bool closed);

/*
	轮廓最大外接矩形函数原型
	array：轮廓的点集合
	
	返回值为矩形，可以绘制
*/
Rect boundingRect( InputArray array);

/*
	轮廓最小外接矩形函数原型
	points：轮廓的点集合
	
	返回值为旋转矩形，可以绘制
*/
RotatedRect minAreaRect( InputArray points);


/*
	续6，例子，当我们进行轮廓发现之后，得到contours数组，对该数组进行遍历
*/
for (size_t i = 0; i < contours.size(); ++i) {
    double area = contourArea(contours[i]);		//计算轮廓面积
    double size = arcLength(contours[i], true);	//计算轮廓周长
    cout << area << ", " << size << endl;

    //绘制最大ROI矩形区域，矩形规则摆放
    Rect box = boundingRect(contours[i]);		
    rectangle(src, box, Scalar(0, 0, 255), 2, 8);

    //绘制最小ROI矩形区域，矩形不是规则摆放的
    RotatedRect rotatedRect = minAreaRect(contours[i]);
    vector<Point2f> pts(4);		
    rotatedRect.points(pts);	//旋转矩形的四个点，提供了绘制不规则矩形的方法
    for (int i = 0; i < 4; ++i) {
        line(src, pts[i], pts[(i + 1) % 4], Scalar(150, 75, 0), 2, 8);
    }
    cout << rotatedRect.angle << endl;

    //绘制椭圆形状的ROI区域
    ellipse(src, rotatedRect, Scalar(0, 150, 75), 2, 8);
}
```

### 1.8 轮廓匹配

​		在OpenCV中，**提供了API对两幅图像的相同轮廓进行匹配**，轮廓匹配的基本原理是基于**图像的矩实现的**，那么，什么是矩呢？其实，**矩**可以理解为**用数学公式的方式来表达图像的几何信息**。矩又分为很多种，几何矩、中心矩、归一化中心矩和Hu不变矩，通过**对矩的理解，才能知道OpenCV进行轮廓匹配的原理**。

​		<font color = red>几何矩</font>是基于图像像素强度的统计量，用于描述图像的几何特性。对于一个二值图像或灰度图像，其几何矩由下式定义：
$$
Geometric Moments_{(p,q)} = \sum_{x=0}^{X-1} \sum_{y=0}^{Y-1} x^{p} y^{q} f(x,y)
$$
其中，$(p+q)$表示图像矩的阶数，$f(x,y)$表示图像中某一像素点的灰度值（二值图像的$f(x,y) = 0 or 1$），在图像处理中，比较常用的是**0阶矩、一阶矩和二阶矩**。

- 0阶矩$(p+q=0)$对于图像处理，有什么几何特性呢？由二值图像可知，我们的**图像像素灰度值不是0就是1**，所以，将其带入到0阶矩的公式中，**为0的像素灰度值表示背景，为1的像素灰度值表示前景轮廓**，0阶矩公式即表示为**所有轮廓的像素灰度值和（轮廓的总面积）**。
- 同理，对于一阶矩$(p + q = 1)$的几何特性，$GeometricMoments_{(1,0)}$表示**图像灰度值中心点（质心）在x方向上的值**，$GeometricMoments_{(0,1)}$表示**图像质心在y方向上的值**。
- 二阶矩的几何意义是，**描述图像的惯性，也称为惯性矩**，它描述了图像的形状，方向和对称信息，二阶矩有三个$GeometricMoments_{(2,0)}$、$GeometricMoments_{(0,2)}$和$GeometricMoments_{(1,1)}$，$GeometricMoments_{(2,0)}$为x轴方向上的惯性矩，描述了图像在x轴上的扩散程度，$GeometricMoments_{(0,2)}$为y轴方向上的惯性矩，描述了图像在y轴上的扩散程度，其中，这**两个较大的值称为长轴，较小的值称为短轴（想象一下椭圆的长短轴）**。$GeometricMoments_{(1,1)}$为x轴和y轴之间的联合惯性矩，它衡量了图像在两个正交方向上的耦合程度，即图像是否倾向于某个特定的角度，**如果该值较大，就说明图像存在明显的倾斜（非对称）**。

​		<font color = red>中心矩</font>：在介绍Hu不变矩之前，先理解一下中心矩与归一化中心矩，因为**Hu不变矩是基于归一化中心矩计算得到的**。中心矩增加了图像的质心，**使矩的值不随图像轮廓位置的改变而改变**。这个好理解，举个例子，在图像的变换中，**原图像轮廓的所在位置，在新图像上发生了变化**，根据几何矩的计算公式，像素点所在位置$(x,y)$发生了改变，**矩的值也就随之发生了改变**，但是，中心矩的计算方式使**矩的值不会因轮廓位置的改变而改变**。
$$
CentralMoments_{(p,q)} = \sum_{x=0}^{X-1} \sum_{y=0}^{Y-1} (x-\bar{x}) ^{p} (y-\bar{y})^{q} f(x,y)
$$
​		<font color = red>归一化中心距</font>：就是在中心距的基础上，除以一个**适当的尺度因子（通常是零阶矩的幂次）**，对比中心矩，归一化中心矩保证了**矩的值在图像缩放的时候也保持不变**。
$$
NormalizedCentralMoments_{(p,q)}= \frac{CentralMoments_{(p,q)}}{GeometricMoments_{(0,0)}^{r}}, r= \frac{p+q+2}{2}
$$
​		<font color = red>Hu不变矩</font>：Hu不变矩通过对**归一化中心矩的线性组合**，构成了**7个值**，保证了图像**矩的值在图像旋转时也保持不变**，它是归一化中心矩的进一步优化，使得**Hu不变矩**能够成为**最好的图像几何信息数学表达方式**，在图像进行平移、缩放和旋转时，都能够保证矩的值不变，下列公式，用$N$表示归一化中心矩，$M$表示Hu不变矩。
$$
\begin{array}{left}
M_{1}=N_{20}+N_{02} \\
M_{2}=(N_{20}-N_{02})^{2}+4 N_{11}^{2} \\
M_{3}=(N_{30}-3 N_{12})^{2}+(3 N_{21}-N_{03})^{2} \\
M_{4}=(N_{30}+N_{12})^{2}+(N_{21}+N_{03})^{2} \\
M_{5}=(N_{30}-3 N_{12})(N_{30}+N_{12})[(N_{30}+N_{12})^{2}-3(N_{21}+N_{03})^{2}]+(3 N_{21}-N_{03})(N_{21}+N_{03})[3(N_{30}+N_{12})^{2}-(N_{21}+N_{03})^{2}] \\
M_{6}=(N_{20}-N_{02})[(N_{30}+N_{12})^{2}-(N_{21}+N_{03})^{2}]+4 N_{11}(N_{30}+N_{12})(N_{21}+N_{03}) \\
M_{7}=(N_{30}-3 N_{12})(N_{21}+N_{03})[(N_{30}+N_{12})^{2}-3(N_{21}+N_{03})^{2}]+(3 N_{21}-N_{03})(N_{30}+N_{12})[3(N_{30}+N_{12})^{2}-(N_{21}+N_{03})^{2}]
\end{array}
$$
​		通过对几何矩、中心矩和归一化中心矩的介绍，我们应该对**图像的矩**有一个清晰的认识。不同阶数的几何矩，它能表达图像中不同的几何信息，但是，随着图像的变化（如平移，缩放，旋转），其矩的值也会变化。所以，就出现了中心矩，**中心距在几何矩的基础上**，解决了平移导致图像距变化的问题，**归一化中心矩在中心矩的基础上**，解决了缩放导致图像矩变化的问题，**Hu不变矩在归一化中心矩的基础上**，解决了旋转导致图像矩变化的问题。

```c++
/*
		轮廓匹配函数原型
		
		contour1: 待匹配轮廓的Hu不变矩
		contour2：匹配轮廓的Hu不变矩
		method：主要的取值有CONTOURS_MATCH_I1、CONTOURS_MATCH_I2和CONTOURS_MATCH_I3，分别表示使用Hu不变矩的欧几里得距离作为相似度度量、使用Hu不变矩的曼哈顿距离作为相似度度量和使用Hu不变矩的切比雪夫距离作为相似度度量
		return: 返回值表示两个轮廓之间的形状相似度，这个值越小，表示两个轮廓越相似
*/
double matchShapes( InputArray contour1, InputArray contour2,int method, double parameter );


/*
		使用例子
*/

void contours_find_info(Mat& image, vector<vector<Point>>& contours) {
	Mat gray;
	Mat binary;
	//灰度化 & 二值化
	cvtColor(image, gray, COLOR_BGR2GRAY);
	threshold(gray, binary, 0, 255, THRESH_BINARY | THRESH_OTSU);

	findContours(binary, contours, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE, Point());
}

void QuickDemo::ContoursMatchDemo() {
	Mat src1 = imread("D:\\Photo\\images\\abc.png");
	Mat src2 = imread("D:\\Photo\\images\\a.png");
	if (src1.empty() || src2.empty()) {
		return;
	}
  
  //src1图像中有ABC三个字母，src2图像中只有A字母，需要通过src2匹配src1中的A字母轮廓，并且绘制
	imshow("src1", src1);
	imshow("src2", src2);

	//轮廓发现
	vector<vector<Point>> contours1;		//存储src1图像轮廓
	vector<vector<Point>> contours2;		//存储src2图像轮廓

	contours_find_info(src1, contours1);	
	contours_find_info(src2, contours2);

	//计算src2图像中A字母轮廓的几何距
	Moments mm2 = moments(contours2[0]);

	
	//计算src2图像中A字母轮廓的Hu距
	Mat Hu2;
	HuMoments(mm2, Hu2);

	//遍历src1图像中的所有字母轮廓，与src2图像中的A字母轮廓进行匹配
	for (int i = 0; i < contours1.size(); ++i) {
		//计算src1图像中轮廓的几何距
		Moments mm1 = moments(contours1[i]);

		//通过质心和像素总和求出轮廓的中心位置
    cout << mm1.m00 << endl;		//得出0阶距的值就是轮廓的像素总和	
    
		double cx = mm1.m10 / mm1.m00;
		double cy = mm1.m01 / mm1.m00;

		cout << cx << " " << cy << endl;

		//通过src1图像中字母轮廓的几何距，计算Hu距
		Mat Hu1;
		HuMoments(mm1, Hu1);

		//通过Hu距进行轮廓匹配
		double dist = matchShapes(Hu1, Hu2, CONTOURS_MATCH_I1, 0);

		if (dist < 1) {
			cout << dist << endl;	
      
			//在src1图像中绘制出从src2图像匹配到的轮廓
      drawContours(src1, contours1, i, Scalar(255, 0, 255), 2, LINE_AA);
		}
	}

	imshow("contours_match_demo", src1);
}
```



### 1.9 轮廓逼近

​		在谈**轮廓逼近**之前，想先聊一下什么是**边缘检测和轮廓检测**：

​		<font color = red>边缘检测</font>主要是通过一些手段检测数字图像中**明暗变化剧烈（即梯度值比较大）的像素点**，**偏向于图像的像素点变化**。如**canny边缘检测**，结果通常保存在和源图像一样尺寸和类型的边缘图中。<font color = red>轮廓检测</font>指检测图像中的**对象边界**，更偏向于**关注上层语义对象**。如OpenCV中的`findContours()`函数， 它会得到每一个轮廓并以点向量方式存储。图像的边缘检测更加**注重于图像的像素点**，而图像的轮廓检测更加**注重于图像中的对象**。

​		<font color = red>轮廓逼近</font>，就是通过对轮廓外形无限逼近，删除非关键点、得到轮廓的关键点，不断逼近轮廓的真实形状。

```c++
/*
		轮廓逼近函数原型
		
		curve: 需要进行逼近的轮廓
		approxCurve：存储轮廓逼近的结果，源码对这个参数的要求是，类型需要和curve一致
		epsilon：逼近精度，表示原始轮廓与近似轮廓之间的最大允许距离（最大误差）
		closed：轮廓是否闭合，选择默认true即可
*/
void approxPolyDP( InputArray curve,OutputArray approxCurve,double epsilon, bool closed );

/*
		使用例子，通过轮廓逼近API找到轮廓的关键点，并且绘制出结果
*/
void QuickDemo::ContoursApproxDemo() {
	Mat src = imread("D://Photo//images//contours.png",IMREAD_COLOR);

	Mat gray;
	Mat binary;
	cvtColor(src, gray, COLOR_BGR2GRAY);	//转灰度图像
	threshold(gray, binary, 0, 255, THRESH_BINARY | THRESH_OTSU);	//转二值图像

	//轮廓发现
	vector<vector<Point>> contours;	//存储每一个轮廓的信息
	vector<Vec4i> hierarchy;		//存储轮廓层次
	findContours(binary, contours, hierarchy, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE, Point());

	//轮廓逼近
	for (size_t i = 0; i < contours.size(); ++i) {
		vector<Point> results;		
		approxPolyDP(contours[i], results, 4, true);		//轮廓逼近
    
    //绘制轮廓逼近的结果
    for (auto result: results) {
			circle(src, result, 2, Scalar(255, 200, 0), 2, LINE_AA);
		}

	imshow("src", src);
	imshow("binary", binary);
}
```

### 1.10 轮廓拟合

​		拟合生成**最相似的圆或者椭圆**，读者可以**对轮廓拟合、轮廓的最小外接矩形（续7.）三个方法进行对比**，实现的需求是差不多的，这里对比一下**轮廓拟合和轮廓最小外接矩形（ROI）的区别**。

- <font color = red>轮廓拟合</font>：轮廓拟合的主要目标是**用一个简单的几何形状描述轮廓的形状特征**，更多的用于形状分析和识别，OpenCV提供了**三个轮廓拟合的API**，`fitEllipse();`、`fitLine();`和`approxPolyDP();`，分别使用椭圆、直线和**多边形进行拟合**（多边形拟合也叫轮廓逼近）。
- <font color = red>最小外接矩形</font>：最小外接矩形侧重于**定义轮廓的边界和位置**，主要用于ROI区域的定义和图像预处理阶段（减少图像分析的范围），OpenCV提供了**一个轮廓最小外接矩形的API**，`RotatedRect minAreaRect( InputArray points);`

```c++
/*
		椭圆轮廓拟合原函数
		
		points: 轮廓的点集合，通常是Mat对象或者是vector<Point>对象
		return: 返回椭圆对应的旋转矩阵
*/
RotatedRect fitEllipse( InputArray points );

/*
		直线轮廓拟合原函数
		
		points: 轮廓的点集合
		line: 拟合输出的的结果，是包含四个元素的向量，(vx, vy, x0, y0)，(vx, vy)表示直线对应的归一化向量，(x0, y0)表示直线通过的点
		distType: 计算点到直线距离的方式，常见的有最小二乘法、最小中值误差或最小均值误差法，详细可以见DistanceTypes类
		param: 额外参数，这里使用0即可，让fitLine()自动匹配最优的参数
		reps: 半径精度，表示轮廓点到拟合直线的距离
		aeps: 角度精度，OpenCV官方建议将reps和aeps的值设置为0.1
*/
void fitLine( InputArray points, OutputArray line, int distType,double param, double reps, double aeps);

/*
		多边形轮廓拟合原函数
    
    curve: 拟合轮廓的点集合，vector<Point>或者Mat对象
    approxCurve：轮廓拟合的结果，参数类型与curve一致
    epsilon: 拟合精度，表示原始轮廓与近似轮廓之间的最大允许距离（最大误差）
		closed: 轮廓是否闭合，默认true即可
*/
void approxPolyDP( InputArray curve,OutputArray approxCurve,double epsilon, bool closed);

/*
		这里主要演示椭圆轮廓拟合例子
*/
void QuickDemo::ContoursFittingDemo() {
	Mat src = imread("D://Photo//images//self_circle.png", IMREAD_COLOR);
	Mat gray;
	Mat binary;
	cvtColor(src, gray, COLOR_BGR2GRAY);	//转灰度图像
	threshold(gray, binary, 0, 255, THRESH_BINARY | THRESH_OTSU);	//转二值图像

	//轮廓发现
	vector<vector<Point>> contours;	
	vector<Vec4i> hierarchy;		//存储轮廓拓扑信息
	findContours(binary, contours, hierarchy, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE, Point());

	//轮廓拟合
	for (size_t i = 0; i < contours.size(); ++i) {
		RotatedRect rrt = fitEllipse(contours[i]);	//椭圆轮廓拟合，基于二值图
		Point center = rrt.center;	//旋转矩阵的中心
		circle(src, center, 2, Scalar(0, 0, 255), 2, 8);	
		ellipse(src, rrt, Scalar(255, 0, 0), 2, 8);		//根据旋转矩阵绘制椭圆
	}

	imshow("src", src);
	imshow("binary", binary);
}

```

### 1.11 霍夫直线检测`HoughLines()`

​		<font color = red>霍夫直线检测</font>，通俗的理解就是，对图像中的**点集合进行直线检测**，其基本原理是：将图像看成一个直角坐标系，像素点$(x,y)$**看作已知参数**，组成极坐标方程$\rho(\theta) = x \cdot cos(\theta) + y \cdot sin(\theta)$，此时就构成了一个，$\theta$作为自变量，$\rho$作为因变量的**三角函数曲线**。同理，对其他的$n-1$个像素点进行该变换，就组成了$n$条不同的$\rho_{i}(\theta) = x \cdot cos(\theta) + y \cdot sin(\theta),i\subseteq[1,n],\theta\subseteq[0,\pi)$三角函数曲线，其中，$\theta$的取值范围**跟图像中直线与x轴正向的夹角有关**。霍夫检测认为，如果这$n$个点作为已知参数构成的$n$条三角函数曲线**相交于一个点**，那么这$n$个点就**属于同一条直线上**，$n$越多，就说明越多的点在一条直线上，直线就越平滑，至此，霍夫对一条直线的检测就完成了。

```c++
/*
		霍夫直线检测HoughLines()函数原型
		
		image: 输入的单通道二值图像
		lines：存储检测到的直线几何信息，vector<Vec3f>或者Mat对象存储，其中，lines[i][0]存储图像原点到直线的距离；lines[i][1]存储直线的法线（方向指向lines[i]），与x轴正向的夹角；lines[i][3]存储图像中有多少个像素点在lines[i]中
		rho：直线法线的像素精度，默认取1即可
		theta：直线法线（方向指向lines[i]），与x轴正向夹角的角度精度，如果以度数为精度，默认取值CV_PI/180.0
		threshold：检测直线的阈值，只有超过threshold个像素单位才认为是一条直线
		srn：rho方向上的高斯滤波器的标准差，默认0即可
		stn：theta方向上的高斯滤波器的标准差，默认0即可
		min_theta：指定theta的最小值，可以过滤一些不需要检测的直线
		max_theta：指定theta的最大值，同上
*/
void HoughLines( InputArray image, OutputArray lines,double rho, double theta, int threshold,double srn = 0, double stn = 0,double min_theta = 0, double max_theta = CV_PI);

/*
		霍夫直线检测HoughLines()使用例子
*/
void QuickDemo::HoughLinesDemo() {
	Mat src = imread("D://Photo//images//lines1.png", IMREAD_COLOR);
	//二值化
	Mat gray;
	Mat binary;
	GaussianBlur(src, src, Size(3, 3), 0); 
	cvtColor(src, gray, COLOR_BGR2GRAY);	//转灰度图像
	threshold(gray,binary,0,255,THRESH_BINARY|THRESH_OTSU);	//转二值图像

	vector<Vec3f> lines;	
	HoughLines(binary, lines, 1, (CV_PI / 180.0), 200, 0, 0, 0);

	//绘制直线
	for (size_t i = 0; i < lines.size(); ++i) {
		float rho = lines[i][0];	//距离，左上角为起始点
		float theta = lines[i][1];	//角度
		float acc = lines[i][2];	//转换为极坐标曲线，经过同一个点的曲线个数
		double x0, y0;	//存储直角坐标系上原始的点

		x0 = rho * cos(theta);
		y0 = rho * sin(theta);
		//输出lines[i]里面的内容
		printf("rho: %.2lf, theta: %.2lf, acc: %.2lf\n", rho, theta, acc);
		printf("x0 = %.2lf, y0 = %.2lf\n", x0, y0);

		Point pt1, pt2;			//找到直线上延长的两个点
		float new_rho = CV_PI - theta;
		pt1.x = cvRound(x0 + 1000 * sin(new_rho));		
		pt1.y = cvRound(y0 + 1000 * cos(new_rho));
		pt2.x = cvRound(x0 - 1000 * sin(new_rho));
		pt2.y = cvRound(y0 - 1000 * cos(new_rho));

		line(src, pt1, pt2, Scalar(255, 150, 0), 2, LINE_AA);
	}

	imshow("src", src);
	imshow("binary", binary);
}
```



### 1.12 霍夫直线检测`HoughLinesP()`

​		与11的霍夫检测直线不同的是，OpenCV提供的**第二个霍夫直线检测API**，`HoughLinesP()`更加注重于图像中直线段的检测（从API的参数也可以看出来），**跟传统的霍夫检测`HoughLines()`不同的是**，`HoughLinesP()`只对局部的像素点进行统计，保证准确率的同时，直线检测的效率也提高了。

```c++
/*
		霍夫直线检测HoughLinesP()函数原型
		
		image：输入的单通道二值图像
		lines：存储检测到直线的几何信息，<vector<Vec4i>>或者Mat数据类型，lines[i][0]、lines[i][1]、lines[i][2]和lines[i][3]分别表示直线起始坐标(x1, y1, x2, y2)
		rho：直线法线（方向指向直线）的像素精度（也叫步长或者距离分辨率），默认取1
		theta：直线法线（方向指向直线）与x轴正向夹角的角度精度（也叫角度分辨率），默认CV_PI/180.0
		threshold：检测直线的阈值，只有超过threshold个像素单位才认为是一条直线
		minLineLength：检测直线的最小线段长度
		maxLineGap：检测到的两条直线最大允许间隙（单位像素，直线距离）
*/
void HoughLinesP( InputArray image, OutputArray lines, double rho, double theta, int threshold, double minLineLength = 0, double maxLineGap = 0)

/*
		霍夫直线检测HoughLinesP()使用例子
*/
void QuickDemo::HoughLinesPDemo() {
	Mat src = imread("D://Photo//images//morph01.png", IMREAD_COLOR);
	Mat gray;
	Mat binary;

	GaussianBlur(src, src, Size(3, 3), 0);	//高斯模糊
	cvtColor(src, gray, COLOR_BGR2GRAY);
	Canny(gray, binary, 80, 160, 3, false);	//利用边缘检测进行二值化

	vector<Vec4i> lines;		//存储四个值为整数的向量
	HoughLinesP(binary, lines, 1, CV_PI / 180.0, 80, 200, 10);
	Mat result = Mat::zeros(src.size(), src.type());
	for (size_t i = 0; i < lines.size(); ++i) {
		int x1 = lines[i][0];
		int y1 = lines[i][1];
		int x2 = lines[i][2];
		int y2 = lines[i][3];
		line(result, Point(x1, y1), Point(x2, y2), Scalar(255, 0, 0), 2, LINE_AA);	//gap 间隔
	}

	imshow("src", src);
	imshow("result", result);
	imshow("binary", binary);

}
```

### 1.13 霍夫圆检测

​	<font color = red>霍夫圆检测</font>，和霍夫直线检测原理类似，对于$X-Y$坐标系下（对应于我们的图像空间）的圆方程$(x-a)^{2}+(y-b)^{2} = r^{2}$，我们进行霍夫空间变换到$a-b$坐标系中，将$x$、$y$和$r$看作已知参数，$a-b$坐标系下，新圆的方程为$(a-x)^{2}+(b-y)^{2} = r^{2}$，同理，对$X-Y$坐标系下圆的$n$个点转换到$a-b$坐标系中，就能得到半径为$r$的$n$个圆，着$a-b$坐标系下的$n$个圆必相交于一个点，至此，霍夫圆检测就完成了。对图像中所有的边缘点，设定$r$的范围，就能检测一定条件下所有的点了。

```c++
/*
		霍夫圆检测函数原型
		
		image：经过灰度化和边缘检测处理的单通道八位图像
		circles：存储检测到的圆的几何信息，vector<Vec4f>，circles[i][0]表示检测到第i个圆圆心的x值，circles[i][1]表示检测到第i个圆圆心的y值，circles[i][2]表示检测到第i个圆的半径，circles[i][3]表示检测到第i个圆所拥有的像素单位个数
		method：检测圆的方法，常见的两个检测方法是HOUGH_GRADIENT和HOUGH_GRADIENT_ALT
		dp：这个参数很重要，说一下，dp = 图像分辨率 / 累计数组分辨率，图像分辨率好理解，累计数组是什么呢？霍夫圆检测算法运行的过程中，霍夫参数空间为(a, b, r)，累计数组的作用就是，记录每一对参数空间(a0, b0, r0)中有多少个曲线H(a, b, r)经过，统计累计数组的局部最大值，就能检测到一个圆了（累计数组局部最大值就是检测圆边缘点的个数）。知道了累计数组是什么，累计数组分辨率就好理解了，指的就是a，b，r的分辨率，a，b代表圆心，r代表半径。dp值越接近1，说明a，b，r的分辨率越接近图像分辨率，检测的精度就越高，dp值越大，检测精度就越低。所以，dp最好的取值范围是[1,2]，dp不可能小于1，因为图像局部的分辨率不可能超过整个图像的分辨率
		minDist：检测到的各个圆之间，其圆心的最小距离
		param1：边缘检测的高阈值，默认100即可
		param2：累加器阈值，类似于HoughLines()的threshold，只有累计数组中向量(a0, b0, r0)的累加值高于param2，才会被认为(a0, b0, r0)是检测到的一个圆
		minRadius：检测圆的最小半径，限制r，提高检测效率
		maxRadius：检测圆的最大半径，限制r，提高检测效率
*/
void HoughCircles( InputArray image, OutputArray circles,int method, double dp, double minDist, double param1 = 100, double param2 = 100, int minRadius = 0, int maxRadius = 0 );

/*
		霍夫圆检测使用例子
*/
void QuickDemo::HoughCircleDemo() {
	Mat src = imread("D://Photo//images//coins.jpg", IMREAD_COLOR);
	Mat gray;
	Mat binary;

	GaussianBlur(src, src, Size(9, 9), 2);	//高斯模糊
	cvtColor(src, gray, COLOR_BGR2GRAY);
	Canny(gray, binary, 40, 160, 3, false);	//利用边缘检测进行二值化

	vector<Vec4f> circles;

	int dp = 2;		//图像分辨率与累加器数组circles分辨率的ratio
	double minDist = 40;		//检测到的圆心最小距离，正常来说是最小半径的两倍
	double minRadius = 20;	//检测圆半径的最小值	
	double maxRadius = 70;	//检测圆半径的最大值
	HoughCircles(binary, circles, HOUGH_GRADIENT, dp, minDist, 100, 100, minRadius, maxRadius);

	//绘制检测到的圆
	for (size_t i = 0; i < circles.size(); ++i) {
		Point center(circles[i][0], circles[i][1]);		//圆心
		int radius = cvRound(circles[i][2]);				//半径
		cout << circles[i][3] << endl;						//像素单位个数
		circle(src, center, radius, Scalar(0, 0, 255), 4, LINE_AA, 0);	//绘制圆
		circle(src, center, 2, Scalar(255, 0, 0), 2, LINE_AA, 0);		//绘制圆心
	}

	imshow("src", src);
	imshow("gray", gray);
	imshow("binary", binary);
}

```

### 1.14 图像形态学操作，腐蚀（erosion）与膨胀（dilate）

​		图像的腐蚀（erosion）和膨胀（dilation）是形态学操作的基础，常用于图像处理和分析中，特别是在**二值图像（前景一般为白色，背景一般为黑色）**中用于改变图像中**对象的形状和大小**。

​		<font color = red>腐蚀</font>，一般用来**缩小**图像的前景区域（将轮廓加以腐蚀），具体的，定义一个**核（结构元素）**，然后**将核沿着图像滑动（有点类似于卷积操作）**，当核每经过**图像局部区域**，都会将**局部区域像素最小值替代核中的某一个像素值（具体位置在定义核的时候表明）**。

​		<font color = red>膨胀</font>，膨胀与腐蚀刚好相反，一般用来**放大**图像的前景区域（将轮廓加以膨胀），膨胀操作与腐蚀操作的原理几乎一致，只是**像素最大值替代核中某一个像素值**。

```c++
/*
		腐蚀操作函数原型
		
    src：任意通道数的图像，常见的BGR, GRAY, BINARY图像都可以进行腐蚀操作
		dst：腐蚀操作输出的目标图像
		kernel：腐蚀操作用到的核（结构元素）
		anchor：腐蚀操作替代核中像素值的位置，Point(-1, -1)表示默认替代核的中心位置像素值
		iterations：表示进行腐蚀操作的迭代次数，默认1即可
		borderType：处理边界的类型，默认即可，具体类型可以在BorderTypes枚举中查看
		borderValue：指定边界的填充值，默认即可
*/
void erode( InputArray src, OutputArray dst, InputArray kernel, oint anchor = Point(-1,-1), int iterations = 1, int borderType = BORDER_CONSTANT, const Scalar& borderValue = morphologyDefaultBorderValue());

/*
		膨胀操作函数原型
		
		src：任意通道数的图像，常见的BGR, GRAY, BINARY图像都可以进行膨胀操作
		dst：膨胀操作输出的目标图像
		kernel：膨胀操作用到的核（结构元素）
		anchor：膨胀操作替代核中像素值的位置，Point(-1, -1)表示默认替代核的中心位置像素值
		iterations：表示进行膨胀操作的迭代次数，默认1即可
		borderType：处理边界的类型，默认即可，具体类型可以在BorderTypes枚举中查看
		borderValue：指定边界的填充值，默认即可
*/
void dilate( InputArray src, OutputArray dst, InputArray kernel, Point anchor = Point(-1,-1), int iterations = 1, int borderType = BORDER_CONSTANT, const Scalar& borderValue = morphologyDefaultBorderValue());

/*
		通常对图像进行形态学操作的时候，我们会提前定义好核（结构元素），定义核的函数原型如下
		
		shape：核的形状，OpenCV提供了三种核的形状，分别是矩形(MORPH_RECT = 0, 八领域)，十字形(MORPH_CROSS = 1, 四领域)，椭圆形(MORPH_ELLIPSE = 2)
		ksize：size()数据类型，表示核的大小
		anchor：替代核中的像素值位置，Point(-1, -1)默认替代核的中心位置
*/
Mat getStructuringElement(int shape, Size ksize, Point anchor = Point(-1,-1));

/*
		腐蚀与膨胀操作使用例子
*/
void QuickDemo::ErodeAndDilateDemo() {
	Mat src = imread("D://Photo//images//cells.png", IMREAD_COLOR);
	Mat gray;
	Mat binary;
	//二值化
	GaussianBlur(src, src, Size(9, 9), 2);	//高斯模糊
	cvtColor(src, gray, COLOR_BGR2GRAY);
	threshold(gray, binary, 0, 255, THRESH_BINARY | THRESH_OTSU);

	//定义腐蚀与膨胀的核,最后一个参数-1表示代替中心位置的像素
	Mat kernel = getStructuringElement(MORPH_RECT, Size(5, 5), Point(-1, -1));
	Mat result1 = Mat::zeros(src.size(), src.type());
	Mat result2 = Mat::zeros(src.size(), src.type());
  
	//对二值图像的腐蚀操作
	erode(binary, result1, kernel);		//erode：腐蚀，缩小轮廓
	//对二值图像的膨胀操作	
	dilate(binary, result2, kernel);	//dilate：膨胀，扩大轮廓

	imshow("binary", binary);
	imshow("erode result1", result1);
	imshow("dilate result2", result2);
}
```

### 1.15 图像形态学操作，开操作（erosion）与闭操作（dilate）

​		<font color = red>开操作</font>，开操作是**先腐蚀，再膨胀**，开操作的目的是**用来消除小物体、在纤细点处分离物体、平滑较大物体的边界**，这个很好理解，因为先进行的腐蚀操作（缩小轮廓），**会去除轮廓的一些小凸起**，然后进行膨胀操作，**会对较大物体的轮廓进行放大**，使得看起来更加平滑。

​		<font color = red>闭操作</font>，闭操作是**先膨胀，再腐蚀**，闭操作的目的是将图像中靠近的**图块进行连通**，这个也很好理解，因为先进行的膨胀操作，**靠的较近的图块由于轮廓会被放大，这时，两个图块的轮廓就会重合**，达到图块连通的目的，然后再进行腐蚀操作，**将连通部分的突起进行缩小**。

```c++
/*
		OpenCV提供了形态学操作的API，morphologyEx，该API可以执行erode, dilate, open, close等形态学操作，函数原型如下
		src：任意通道的图像，常见的BGR, GRAY, BINARY图像都可以进行形态学操作
		dst：形态学操作输出的结果，图像大小、类型和src一致
		op: 形态学操作，MorphTypes提供了8种形态学操作，这里先知道四种，后续篇幅会有其他四种的介绍，MORPH_OPEN = 2, MORPH_CLOSE = 3, MORPH_ERODE = 0, MORPH_DILATE = 1
    kernel：形态学操作用到的核（结构元素）
		anchor：形态学操作替代核中像素值的位置，Point(-1, -1)表示默认替代核的中心位置像素值
		iterations：形态学操作迭代次数
    borderType：处理边界的类型，默认即可，具体类型可以在BorderTypes枚举中查看
		borderValue：指定边界的填充值，默认即可
*/
void morphologyEx( InputArray src, OutputArray dst, int op, InputArray kernel, Point anchor = Point(-1,-1), int iterations = 1, int borderType = BORDER_CONSTANT, const Scalar& borderValue = morphologyDefaultBorderValue());

/*
		开操作和闭操作使用例子
*/
void QuickDemo::MorphologyOpenAndCloseDemo() {
	Mat src = imread("D://Photo//images//fill.png", IMREAD_COLOR);
	Mat gray, binary;

	//灰度且二值化
	cvtColor(src, gray, COLOR_BGR2GRAY);
	threshold(gray, binary, 0, 255, THRESH_BINARY_INV | THRESH_OTSU);

	Mat dst1, dst2;

	//定义开闭操作的卷积核，第一个参数为卷积核的形状
	Mat kernel = getStructuringElement(MORPH_RECT, Size(3, 3), Point(-1, -1));

	//开操作,删除小的干扰块，一定要进行二值化，并且使背景通道值为0，因为腐蚀是用通道小的值替代
	morphologyEx(binary, dst1, MORPH_OPEN, kernel, Point(-1, -1), 1);
	
	//闭操作，填充闭合区域
	morphologyEx(binary, dst2, MORPH_CLOSE, kernel, Point(-1, -1), 1);

	imshow("src", src);
	imshow("binary", binary);
	imshow("dst1_open", dst1);
	imshow("dst2_close", dst2);
}
```

### 1.16 图像形态学梯度

​		<font color = red>形态学梯度</font>，形态学梯度的主要目的是**将图像中图块的轮廓提取出来**，为什么梯度可以提取出轮廓呢？这个很好理解，还记得我们在进行腐蚀和膨胀操作的时候，就是对**图块边缘轮廓进行缩小或放大**。这样，腐蚀的结果、膨胀的结果和原图的**像素差别就是轮廓部分的大小不一样**，又因为梯度就是在这三者之间做像素差，所以结果就**把轮廓给提取出来了**。具体的有**基本梯度，内梯度，外梯度三种**。

- 基本梯度：膨胀减去腐蚀之后的结果，`MorphTypes`种提供了`MORPH_GRADIENT`实现
- 内梯度：原图减去腐蚀之后的结果，需要自己做像素差
- 外梯度：膨胀减去原图之后的结果，需要自己做像素差

```c++

/*
		形态学梯度原函数也是和上述15一样，用OpenCV提供的morphologyEx执行即可，将第三个参数op赋值为MORPH_GRADIENT即可实现基本梯度的需求
*/
void morphologyEx( InputArray src, OutputArray dst, int op, InputArray kernel, Point anchor = Point(-1,-1), int iterations = 1, int borderType = BORDER_CONSTANT, const Scalar& borderValue = morphologyDefaultBorderValue());


/*
		形态学梯度使用例子
		
		细心的读者会发现，我这里形态学操作跟上面不同的是，使用GRAY类型的图像，这也验证了，形态学操作支持任意类型的图像
*/
void QuickDemo::MorphologyGradientDemo() {
	Mat src = imread("D://Photo//images//LinuxLogo.jpg", IMREAD_COLOR);
	Mat gray, binary;

	//灰度且二值化
	cvtColor(src, gray, COLOR_BGR2GRAY);
	threshold(gray, binary, 0, 255, THRESH_BINARY_INV | THRESH_OTSU);

	//定义形态学操作的卷积核
	Mat kernel = getStructuringElement(MORPH_ELLIPSE, Size(3, 3), Point(-1, -1));
	Mat basic_grad, inter_grad, exter_grad;
	
	//基本梯度
	morphologyEx(gray, basic_grad, MORPH_GRADIENT, kernel, Point(-1, -1), 1);

	Mat dst1, dst2;
	erode(gray, dst1, kernel);		//腐蚀
	dilate(gray, dst2, kernel);		//膨胀
	
	//内梯度
	subtract(gray, dst1, inter_grad);
	//外梯度
	subtract(dst2, gray, exter_grad);

	imshow("src", src);
	imshow("gray", gray);
	imshow("binary", binary);
	imshow("basic_grad", basic_grad);
	imshow("inter_grad", inter_grad);
	imshow("exter_grad", exter_grad);
}
```



### 1.17 图像形态学操作，顶帽、黑帽和击中不击中

​		<font color = red>顶帽</font>，顶帽操作的主要目的是**提取图像中的噪声，突出原图像中，局部区域比周围亮的部分**，它是原图减去开操作的结果。这个好理解，我们知道，开操作的主要目的是**用来消除小物体、在纤细点处分离物体、平滑较大物体的边界**，用原图减去开操作之后，我们就能提取出那些**消除的小物体（噪声），以及大物体没有平滑的部分（局部区域比周围亮的部分）**。

​		<font color = red>黑帽</font>，黑帽操作的主要目的是**提取图像中，局部区域比周围暗的部分**，它是闭操作减去原图的结果。这个也好理解，我们知道，闭操作的主要目的是**将两个靠近的图块进行连通**，原图这两个图块之间本来是不连通的（局部区域比周围暗的部分），所以用闭操作减去原图，就能**把这个比周围暗的区域提取出来**。

​		<font color = red>击中不击中</font>，

```c++
/*
		顶帽与黑帽操作，直接通过OpenCV提供的形态学操作API执行即可，第三个参数 op = MORPH_TOPHAT为顶帽操作，op = MORPH_BLACKHAT为黑帽操作
*/
void morphologyEx( InputArray src, OutputArray dst, int op, InputArray kernel, Point anchor = Point(-1,-1), int iterations = 1, int borderType = BORDER_CONSTANT, const Scalar& borderValue = morphologyDefaultBorderValue());

/*
		顶帽与黑帽操作使用例子
*/
void QuickDemo::OtherMorphologyDemo() {
	Mat src = imread("D://Photo//images//cross.png", IMREAD_COLOR);
	Mat gray, binary;

	//灰度且二值化
	cvtColor(src, gray, COLOR_BGR2GRAY);
	threshold(gray, binary, 0, 255, THRESH_BINARY_INV | THRESH_OTSU);

	//定义形态学操作卷积核
	Mat kernel = getStructuringElement(MORPH_ELLIPSE, Size(15, 15), Point(-1, -1));

	//顶帽
	Mat dst1;
	morphologyEx(binary, dst1, MORPH_TOPHAT, kernel, Point(-1, -1), 1);

	//黑帽
	Mat dst2;
	morphologyEx(binary, dst2, MORPH_BLACKHAT, kernel, Point(-1, -1), 1);

	imshow("gray", gray);
	imshow("binary", binary);
	imshow("top_hat demo", dst1);
	imshow("black_hat demo", dst2);
}
```



## 二、视频部分

### 2.1 视频流的基本操作

- 视频流的读写操作
- 获取视频流基本信息

```c++
/*
	视频流读操作函数原型1
	
	explicit修饰的构造函数不允许隐式类型转换，只能使用构造函数，如VideoCapture capture = 0是非法的，如果没有explicit修饰，就是合法的
	
	index: 表示打开的摄像头索引号，0表示打开默认的摄像头
	apiPreference: 指定了首选的捕获API，CAP_ANY是一个宏，表示让OpenCV选择最适合的API。如果需要指定的API，则可以通过传递相应的宏来实现
	
*/
explicit VideoCapture(int index, int apiPreference = CAP_ANY);

/*
	视频流读操作函数原型2
	
	filename: 表示需要读取的视频流文件绝对路径
	其余参数和上述类似
	
*/
explicit VideoCapture(const String& filename, int apiPreference = CAP_ANY);

/*
	视频流写操作函数原型
	
	filename：视频流写的文件绝对路径
	fourcc：视频流编解码算法，four character code，四字符代码，一个标识符，比如写入MP4类型的文件，可以使用VideoWriter::fourcc('m', 'p', '4', 'v')
	fps：视频流写入的帧数
	frameSize：视频流写入的每一帧大小
	isColor：默认true表示彩色帧，false表示灰度帧
	
*/
VideoWriter(const String& filename, int fourcc, double fps, Size frameSize, bool isColor = true);

/*
	获取视频流基本信息，openCV中的VideoCapture对象提供了get函数，用来获取视频流相关的信息
*/
Videocapture capture2("D:\\Photo\\rheed_video\\1.mp4");

//获取视频流每一秒的帧数
int fps = capture2.get(CAP_PROP_FPS);
//FOURCC: four character code，四字符代码，表示视频压缩算法类型的一种
int type = capture2.get(CAP_PROP_FOURCC);	
//获取视频流每一帧的宽度
int width = capture2.get(CAP_PROP_FRAME_WIDTH);
//获取视频流每一帧的高度
int height = capture2.get(CAP_PROP_FRAME_HEIGHT);
//获取视频流的总帧数
int count_of_frames = capture2.get(CAP_PROP_FRAME_COUNT);
```

### 2.2 图像的色彩空间实操

&emsp;&emsp;该部分对图像主要的色彩空间进行介绍，然后**通过HSV色彩空间**颜色范围的过滤，实现纯色背景的替换。

&emsp;&emsp;<font color = red>BGR色彩空间</font>：是图像处理中**最常见的颜色表示模型**，一个像素点使用三通道表示（Blue, Green, Red），**每一个通道占8 bit（0~255）**，所以对于BGR色彩空间来说，通常每一个像素点大小都是24位。

&emsp;&emsp;<font color = red>HSV色彩空间</font>：也是图像处理中一种**常用的色彩表示模型**，它将颜色**分解为三个独立的部分，色调（Hue），饱和度（Saturation）和明度（Value）**，和BGR色彩空间一样，每一个像素点都是24 bit。

- **色调**：取值范围0-180，色调反映了颜色的基本类型，红色对应的色调大约是0，绿色大约是60，蓝色大约是120。
- **饱和度**：取值范围0-255，饱和度代表颜色鲜艳的取值范围，值越大，颜色越鲜艳，反之，颜色越接近灰度，0表示灰度。
- **明度**：表示颜色的亮度或者暗度，与饱和度不同的是，0表示黑色，取值范围是0到255。

&emsp;&emsp;<font color = red>GRAY色彩空间</font>，单通道灰度图像，也是图像任务处理的常用色彩空间，由于其单通道的原因，处理效率通常较高，取值范围0-255，0表示黑色，255表示白色。

&emsp;&emsp;除了上述三种常见的图像处理色彩空间，还有**诸如Lab、YCbCr等色彩空间**，这些色彩空间都是不常见的，用于特定设备上的色彩空间。

```c++

/*
	这是一个图像色彩空间转换的Demo，程序中包括了如何利用HSV色彩空间的特点，通过inRange函数对色调（Hue）进行过滤，进行纯色背景的更换。
*/

/*
	inRange()函数原型：用于创建一个二值掩码图（binary mask），该掩码图的作用是，指示输入图像中，哪些像素点的值落在指定的范围之内。通常用于颜色分割或阈值（自己设定Scalar值）处理，从而突出指定范围内的颜色或阈值像素点。
	
	src：输入的源图像，可以是单通道或者是多通道图像
	lowerb：像素点每一个通道的下界，Array类型或者Scalar类型
	upperb：像素点每一天通道的上界，Array类型或者Scalar类型
	dst：输出的目标图像，和src大小一致，一般是二值掩码图
*/
void inRange(InputArray src, InputArray lowerb,InputArray upperb, OutputArray dst);


/*
		色彩空间转换使用例子，利用HSV色彩空间提取ROI区域
*/
void QuickDemo::ImageColorConvert() {
	VideoCapture capture2("D:\\Photo\\images\\01.mp4");	
	if(!capture2.isOpened()){	
		cout << "Open failure..." << endl;
		return;
	}

	Mat frame2, hsv, lab, mask, result;
	while (true) {
		int c = waitKey(100 / 2);
		if (c == 27) {	//ESC
			break;
		}
		bool ret = capture2.read(frame2);	//读取视频帧到Mat对象
		if (!ret) {		//读取失败，一般视频到了尾部
			break;
		}

		GaussianBlur(frame2, frame2, Size(7, 7), 0);
		cvtColor(frame2, hsv, COLOR_BGR2HSV);
		cvtColor(frame2, lab, COLOR_BGR2Lab);

    //在hsv图像中过滤掉绿色背景，找到HSV对应绿色背景的H
		inRange(hsv, Scalar(25, 43, 46), Scalar(77, 255, 255), mask);	

    //取反获取ROI区域
		bitwise_not(mask, mask);	
    
    //根据二值图提取的ROI区域，对frame2进行位与操作，mask限制与操作的范围是ROI区域
		bitwise_and(frame2, frame2, result, mask);

		imshow("frame2", frame2);
		imshow("hsv", hsv);
		imshow("lab", lab);
		imshow("result", result);
	}
	capture2.release();
}
```

### 2.3  直方图反向投影

&emsp;&emsp;<font color = red>直方图反向投影</font>，主要作用是估计**图像中某个像素点的值**与**预先计算好的直方图像素分布的相似度**，它通常用于**对象检测、颜色分割**等任务。在知道直方图反向投影之前，我们首先得知道**什么是直方图**。

&emsp;&emsp;<font color = red>直方图</font>，是一个统计学的概念，它可以**体现出一段连续值的分布情况**，比如0到99之间，有100个整数，其中，小于50的整数有50个，大于等于50的整数有50个。我们可以定义横坐标为0表示小于50的整数这一类，为1表示大于等于50的整数这一类。这样，我们就完成了直方图的定义。

&emsp;&emsp;<font color = red>图像直方图</font>，就是通过对图像中所有的像素点进行统计，得到像素点通道取值的分布，比如对于HSV色彩空间，可以通过**图像直方图**，描述出色调（Hue）的统计分布情况。

```c++
/*
		直方图反向投影函数原型
		
		images：Mat对象数组，也是进行直方图反向投影的目标
		nimages：Mat对象数组中的图像个数
		channels：进行反向投影的通道数组首地址，比如我们是基于HSV色彩空间的前两个通道进行反向投影，那么channels的结构为int channels[] = { 0,1 }
		hist：进行直方图反向投影所需的直方图对象（Mat类）
		backProject：直方图反向投影输出的结果
		ranges：图像进行直方图统计的通道范围
		scale：输出反向投影的缩放因子，默认1即可，值越大，可以提高输出反向投影结果的对比度，使得匹配的区域更加明显
		uniform：表明是否使用均值（均衡化）直方图，默认true即可
*/
void calcBackProject( const Mat* images, int nimages, const int* channels, InputArray hist, OutputArray backProject, const float** ranges, double scale = 1, bool uniform = true);

/*
		直方图函数原型
		
    images：Mat对象数组，也是进行直方图反向投影的目标
		nimages：Mat对象数组中的图像个数
		channels：进行反向投影的通道数组首地址，比如我们是基于HSV色彩空间的前两个通道进行反向投影，那么channels的结构为int channels[] = { 0,1 }
		mask：通过掩码图限制直方图统计的ROI区域，默认Mat()即可
		hist：输出的直方图统计结果，Mat类型
		dims：直方图的维度，大部分情况2即可
		histSize：直方图的大小，int类型的数组
		ranges：图像进行直方图统计的通道范围
		uniform：表明是否使用均值（均衡化）直方图，默认true即可
		accumulate：表示新计算的直方图是否加在旧的直方图结果上，默认false即可
*/
void calcHist( const Mat* images, int nimages, const int* channels, InputArray mask, OutputArray hist, int dims, const int* histSize, const float** ranges, bool uniform = true, bool accumulate = false);


/*
		直方图反向投影使用例子
		
		1.确定需要进行直方图计算的匹配图像
		2.计算匹配图像的直方图
		3.基于已知图像进行直方图反向投影
		4.输出反向投影的结果
*/
void QuickDemo::HistogramInvertProjection() {
	Mat src = imread("D:\\Photo\\images\\hand.jpg", IMREAD_COLOR);
	Mat model = imread("D:\\Photo\\images\\hand_section.png", IMREAD_COLOR);

	Mat model_hsv, src_hsv;
	cvtColor(model, model_hsv, COLOR_BGR2HSV);
	cvtColor(src, src_hsv, COLOR_BGR2HSV);

	int h_bins = 512, s_bins = 512;		//直方图的大小
	int histSize[] = { h_bins,s_bins };
	int channels[] = { 0,1 };			//统计的通道个数
	Mat roiHist;						//存储直方图结果
	float h_range[] = { 0,180 };		//需要统计的每一个通道取值范围
	float s_range[] = { 0,255 };
	const float* ranges[] = { h_range,s_range };	//取值范围数组
	//获取直方图
	calcHist(&model_hsv, 1, channels, Mat(), roiHist, 2, histSize, ranges, true, false);
	//对直方图进行通道归一化，归一化的范围是[0,255]
	normalize(roiHist, roiHist, 0, 255, NORM_MINMAX, -1,Mat());

	Mat back_projection;		//存储反向投影结果
	calcBackProject(&src_hsv, 1, channels, roiHist, back_projection, ranges, 1.0);

	imshow("model_hsv", model_hsv);
	imshow("src_hsv", src_hsv);       

	imshow("roiHist", roiHist);
	imshow("back projection", back_projection);

}
```

### 2.4 Harris角点检测

&emsp;&emsp;<font color = red>角点检测</font>是计算机视觉中获取图像特征的一种方式，那么，在图像中，什么是角点呢？**角点通常由两个边缘相交而产生（注意：角点是一片像素区域，而不是一个像素点）**，‌对于同一场景，即使视角发生变化，角点都具有稳定性，**角点无论在梯度方向上，还是梯度幅值（梯度向量的模）上都有着较大的变化**。OpenCV中，提供了多种角点检测的API，这里先介绍最简单的**Harris角点检测**。

```c++
/*
		Harris角点检测函数原型
	
		src：单通道图像：CV_8UC1或者CV_32FC1，通常我们传入灰度图像CV_8UC1
		dst：存储角点检测的图像，图像类型为CV_32FC1
		blockSize：角点检测的窗口大小，这个窗口的作用是计算角点响应，通过对角点检测窗口在图像上移动，通过每一个像素点的梯度矩阵，得出角点响应值
		ksize：Sobel导数核的大小，通常设置为3，导数核的作用是计算角点里面每一个像素点的梯度，为计算角点响应值提供参数
		k：Harris角点检测自由参数，通常取值为0到0.04之间，较小的值会导致更多的角点被检测出来（可能会包含噪声），较大的值会导致较少的角点被检测出来（但是结果会更加可靠）。
		borderType：如何处理图像边界上的像素，默认缺省参数即可
*/
void cornerHarris( InputArray src, OutputArray dst, int blockSize,int ksize, double k,int borderType = BORDER_DEFAULT);

/*
		Mat对象归一化函数原型
		
		src：输入的图像
		dst：存储和src大小一致的Mat对象
		alpha：进行归一化的最小值边界
		beta：进行归一化的最大值边界
		norm_type：归一化的类型，如果设置了alpha和beta的值，那么常用的类型是NORM_MINMAX
		dtype：depth type，默认负值，规定dst类型和src类型一致，若dtype大于0，则dst只有颜色通道和src一致，图像深度depth = CV_MAT_DEPTH(dtype)
		mask：规定进行归一化感兴趣的区域，如果是图像的全部范围，Mat()即可
*/
void normalize( InputArray src, InputOutputArray dst, double alpha = 1, double beta = 0, nt norm_type = NORM_L2, int dtype = -1, InputArray mask = noArray());

/*
		图像缩放，取绝对值和转换为无符号8位整数的函数原型
		
		从OpenCV官方的函数解释，该函数对图像进行了三种操作
		1. 缩放 + 偏置值
		2. 在第一步的基础上，对所有像素值取绝对值
		3. 在第二步的基础上，将所有的像素值转换位CV_8UC1类型
		
		src：输入需要处理的图像
		dst：存储处理后的图像
		alpha：缩放因子，如果为默认值1，不进行缩放
		beta：偏置值，如果为默认值0，不进行偏置
		
		很明显，如果调用该API，最后两个参数使用默认值，则只会对图像的像素值取绝对值，然后转换为CV_8UC1
*/
void convertScaleAbs(InputArray src, OutputArray dst, double alpha = 1, double beta = 0);

/*
		使用例子，一般的，对图像进行角点检测的步骤如下：
		1. 获取灰度图像
		2. 确定角点检测的大小（计算角点响应值 corner response value），Sobel导数核的大小（计算像素点灰度值变化的梯度）
		3. 调用cornerHarris进行Harris角点检测
		4. 角点检测得出来的Mat对象是CV_32FC1类型
		5. 对4得到的Mat对象调用normalize()，进行归一化得到新的Mat对象
		6. 对5得到的Mat对象调用convertScaleAbs()，进行比例因子缩放，加上相应的偏置值，最后取绝对值得到新的Mat对象，该Mat对象就能比较直观的显示角点了
		7(Optional). 可以根据6得到的角点检测结果，绘制到原图中
*/
void QuickDemo::HarrisCornerDemo() {
	Mat src = imread("D://Photo//images//abc.png");
	Mat gray, binary;
	cvtColor(src, gray, COLOR_BGR2GRAY);
	Mat dst;
	int block_size = 5;
	int k_size = 3;
	double k = 0.04;
	cornerHarris(gray, dst, block_size, k_size, k);		//进行角点检测之后，得到的目标图像是单通道的FLOAT32类型
	Mat dst_norm = Mat::zeros(dst.size(), dst.type());
	normalize(dst, dst_norm, 0, 255, NORM_MINMAX, -1, Mat());

	convertScaleAbs(dst_norm, dst_norm);	//对图像的每一个通道进行比例因子缩放，加上偏置值，最后取绝对值得到目标图像

	//draw corners
	for (int row = 0; row < src.rows; ++row) {
		for (int col = 0; col < src.cols; ++col) {
      //corner response value
			int rsp = dst_norm.at<uchar>(row, col);
			if (rsp > 100) {	//阈值设置为100
				//一般地，角点检测窗口的中心像素点会被看作是角点的代表像素点，Point输入的是图像坐标的值，图像遍历是按行列的，注意转换
        circle(src, Point(col, row), 3, Scalar(0, 0, 255), 2);	
			}
		}
	}
	imshow("src", src);
	imshow("gray", gray);
}


```

### 2.5 Shi-Tomas角点检测

&emsp;&emsp;<font color = red>Shi-Tomas角点检测</font>的原理和Harris角点检测一样，只是到最后计算角点响应值的方式不一样，**Shi-Tomas角点检测角点响应值为矩阵$M$的特征值较小的那个**，即$R = min(\lambda_{1}, \lambda_{2})$，最后，设置一个阈值，**只有角点响应值$R$大于阈值时，才认为该区域是角点**。

```c++
/*
		Shi-Tomas角点检测函数原型1，OpenCV提供了两种重载的API
		
		image：输入需要进行角点检测的图像，8位或者32位的单通道图像
		corners：输出检测到的角点（注意，这是一个点集合，但是角点是一个区域，这里的点指的是角点中的代表性点）
		maxCorners：检测最大的角点个数
		qualityLevel：角点质量的下限，这个值介于0到1之间，该参数决定了角点检测的阈值，具体的，thresh = qualityLevel * max_value，max_value为整个图像中最大的角点响应值
		minDistance：检测角点之间的最小欧几里得距离	
		mask：掩码值，Mat类型，指定角点检测的图像区域，mask对应的非零像素区域是可以进行角点检测的区域，默认Mat()即可
		blockSize：角点检测的窗口大小，默认3*3
		useHarrisDetector：是否使用Harris角点检测，默认false即可
		k：如果使用Harris角点检测，k为计算角点响应值所需的参数，介于0到0.04之间
		
		从Harris角点检测的原理可以知道，在进行角点响应值计算的时候，需要用到像素点的梯度值，Shi-Tomas角点检测提供的第一个API自己实现了梯度计算这一步骤
*/
void goodFeaturesToTrack( InputArray image, OutputArray corners, int maxCorners, double qualityLevel, double minDistance, InputArray mask = noArray(), int blockSize = 3, bool useHarrisDetector = false, double k = 0.04);


/*
		Shi-Tomas角点检测函数原型2，与第一个API不同的是，该API多了一个gradientSize参数，该参数决定了计算像素点梯度时，使用Sobel导数核的大小
*/
void goodFeaturesToTrack( InputArray image, OutputArray corners, int maxCorners, double qualityLevel, double minDistance, InputArray mask, int blockSize, int gradientSize, bool useHarrisDetector = false, double k = 0.04);

/*
	Shi_Tomas角点检测
*/
void QuickDemo::ShiTomasCornerDemo() {
	VideoCapture capture("D://Photo//images//bike.avi");
	if (!capture.isOpened()) {
		cout << "Open failure........" << endl;
		return;
	}
	int fps = capture.get(CAP_PROP_FPS);
	while (1) {
		int c = waitKey(1000 / fps);
		if (c == 48) {		// 0键退出
			break;
		}
		Mat src;
		bool tmp = capture.read(src);
		if (!tmp) {			//读取视频最后一帧，退出
			break;
		}
		Mat gray, dst;
		dst = src.clone();
		cvtColor(src, gray, COLOR_BGR2GRAY);
		vector<Point2f> corners;		//存储ShiTomas角点检测的结果
		double quality_level = 0.3;
		int blockSize = 3;
		int gradientSize = 3;
		goodFeaturesToTrack(gray, corners, 200, quality_level, 3, Mat(), blockSize, gradientSize, false);//ShiTomas角点检测接口

    //与cornerHarris()不同的是，Shi-Tomas角点检测API直接返回角点的点集合数组
		for (auto corner : corners) {
			circle(dst, corner, 4, Scalar(0, 0, 255), 2, LINE_AA);
		}
		imshow("src", src);
		imshow("dst", dst);
	}
	capture.release();
}
```

### 2.6 利用Image Watch调试程序，进行视频帧分析

&emsp;&emsp;本部分主要是针对**Visual Studio 2019**插件**Image Watch**的使用，通过Image Watch，在程序的调试过程中，我们可以直观的看到Mat对象的变化情况，在Image Watch中，我们可以查看图像中**任意一点的像素位置和像素的取值**，为了解程序逻辑提供了很大的帮助。

```c++

void QuickDemo::VideoFrameColorAnalysisAndExtract() {
	VideoCapture capture("D://Photo//images//balltest.mp4");
	if (!capture.isOpened()) {
		cout << "Open failure........" << endl;
		return;
	}
	int fps = capture.get(CAP_PROP_FPS);
	while (1) {
		Mat src, hsv, dst, mask;
		int c = waitKey(1000 / fps);
		if (c == 27) {		// ESC键退出
			break;
		}
		bool tmp = capture.read(src);
		if (!tmp) {			//读取视频最后一帧，退出
			cout << "提取视频帧失败" << endl;
			break;
		}

		GaussianBlur(src, src, Size(5, 5), 0);
		
		//去除纯色背景，提取出ROI区域，以二值图的形式表示
		cvtColor(src, hsv, COLOR_BGR2HSV);
		inRange(hsv, Scalar(26, 43, 46), Scalar(34, 255, 255), mask);
		mask = ~mask;
		Mat blueBG = Mat::zeros(src.size(), src.type());
		blueBG = Scalar(255, 180, 0);
		src.copyTo(blueBG, mask);		//以mask的规则将src图片的内容复制到blueBG中去

		//进行轮廓发现
		vector<vector<Point>> contours;
		vector<Vec4i> hierarchy;
		findContours(mask, contours, hierarchy, RETR_TREE, CHAIN_APPROX_SIMPLE, Point());
		int index = -1;
		double max_area = -1;
		for (size_t i = 1; i < contours.size(); ++i) {			//一般的，第一个轮廓存储的是整个图像的背景
			if (contourArea(contours[i]) > max_area) {
				index = i;
				max_area = contourArea(contours[i]);
			}
		}

		//轮廓拟合，跟踪目标物体
		if (index >= 0) {
			RotatedRect rrt = minAreaRect(contours[index]);
			ellipse(src, rrt, Scalar(180, 160, 255), 2, LINE_AA);
			circle(src, rrt.center, 2, Scalar(0, 0, 255), 2, LINE_AA);
		}

		imshow("src", src);
		imshow("hsv", hsv);
		imshow("mask", mask);
		imshow("blueBG", blueBG);
	}
	capture.release();
}
```

### 2.7 视频帧背景分析（通过背景减去器，提取前景的ROI区域）

&emsp;&emsp;在OpenCV中，**提供了三种视频背景分析的方法**，分别是：**K最近邻算法**（K Nearest Neighbors，KNN），**高斯混合模型**（Gaussian Mixture Model，GMM），**模糊积分**（Fuzzy Integral）。

- KNN算法的基本工作原理是，**使用像素的历史值来估计背景模型**。它通过维护一个历史帧队列，对于每个像素点，算法**存储最近K帧内的值**。当新的帧到来时，计算**新帧像素点值与历史帧队列中K最近邻像素值的距离**，如果距离大于某个阈值，则认为**该像素属于前景**，否则，属于背景。
- GMM算法的基本工作原理是，对于每一个像素，算法使用一个高斯混合模型估计背景分布，模型中**包含几个高斯分布，每一个高斯分布代表一个不同的背景状态**，高斯分布的参数（均值和方差）是根据像素历史值动态更新的。当新帧到来时，**GMM算法会计算该像素与背景模型中各个高斯分布概率密度函数的似然**，如果**似然**低于某个阈值，则认为**该像素属于前景**，否则，属于背景。
- 模糊积分是一种**基于模糊逻辑的方法**，它结合**多个背景减除的结果**来提高准确性，如使用KNN和GMM方法，**生成多个背景模型**。对于新帧的每一个像素，**模糊积分**算法会计算其在不同背景模型中的前景可能性，最后，使用模糊逻辑运算符来**计算最终的前景掩码**。

下面通过使用**混合高斯模型**（GMM）的背景减除器来实现背景的消除。

```c++
/*
		基于混合高斯模型的背景减除器函数原型
		
		history：该参数决定，算法学习历史帧的长度，较大的值可以提供更稳定和更准确的背景模型，但也会增加处理时间
		varThreshold：该参数控制像素之间的方差阈值（历史帧），用于判断该像素点是否属于背景，如果一个像素方差低于阈值，则认为是背景，否则认为是前景，较高的阈值会检测更多的背景，减少误报，但可能会错过一些前景对象。不难理解，对于静态的背景，一般像素方差会较小，而对于移动的前景，像素方差会较大
		detectShadows：是否尝试检测阴影部分，默认为true即可，有助于提高检测的准确性
		
		return：返回值为BackgroundSubtractorMOG2类的智能指针类型，通过该智能指针，可以获取前景和背景的相关信息
*/
Ptr<BackgroundSubtractorMOG2> createBackgroundSubtractorMOG2(int history=500, double varThreshold=16, bool detectShadows=true);

/*
		使用例子
*/
void QuickDemo::VideoBackgroundAnalysis() {
	VideoCapture capture("D://Photo//images//opencv_demo//vtest.avi");
	if (!capture.isOpened()) {
		cout << "Open failure........" << endl;
		return;
	}

	auto pMOG2 = createBackgroundSubtractorMOG2(1000, 200, true);
	int fps = capture.get(CAP_PROP_FPS);
	while (1) {
		Mat src, mask, bg_image;
		int c = waitKey(1000 / fps);
		if (c == 27) {		// ESC 键退出
			break;
		}
		bool tmp = capture.read(src);
		if (!tmp) {			//读取视频最后一帧，退出
			cout << "提取视频帧失败" << endl;
			break;
		}

		GaussianBlur(src, src, Size(5, 5), 0);

		//通过背景减去API返回的智能指针，获取前景和背景相关信息
		pMOG2->apply(src, mask);
		pMOG2->getBackgroundImage(bg_image);

		//形态学开操作，先腐蚀，再膨胀
		Mat kernel = getStructuringElement(MORPH_RECT,Size(1,5),Point(-1,-1));
		morphologyEx(mask, mask, MORPH_OPEN, kernel, Point(-1, -1));

		//基于掩码图所获得的轮廓信息，在原图中标记移动对象
		vector<vector<Point>> contours;
		vector<Vec4i> hierarchy;
		findContours(mask, contours, hierarchy, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE, Point());
		for (size_t i = 0; i < contours.size(); ++i) {
			double area = contourArea(contours[i]);
			if (area < 200) continue;
			Rect rect = boundingRect(contours[i]);
			//绘制矩形边框
			rectangle(src, rect, Scalar(0, 200, 255), 2, LINE_AA);

			//绘制椭圆边框
			RotatedRect rrt = minAreaRect(contours[i]);
			ellipse(src, rrt, Scalar(255, 200, 0), 2, LINE_AA);  //最小外接矩形
			circle(src, rrt.center, 2, Scalar(0, 0, 255), 2, LINE_AA);
		}
		imshow("src", src);
		imshow("bg_image", bg_image);
		imshow("mask", mask);
	}
	capture.release();
}
```

### 2.8 基于光流法的视频分析 - 上

&emsp;&emsp;光流可以看成是图像结构光的变化，或者是**图像亮度模式明显的移动**。简单的理解就是，光流描述了连续两帧之间，**像素点的位移向量**。光流法是基于**视频分析**所提出的概念，OpenCV提供了两种光流分析方法，**稀疏光流分析和稠密光流分析**。

```C++
/*
		基于稀疏光流法（KLT）的视频分析函数原型
		该函数原型执行的是Lucas-Kanade光流算法，是一种经典的稀疏光流计算方法，用于追踪一组预选特征点（基于角点检测获取预选特征点），在连续图像帧中的运动。
		
		preImg：8位单通道灰度图像，上一帧图像，光流计算的起点
		nextImg：和preImg相同大小和类型的图像，当前帧图像，光流计算的目标帧
		prePts：preImg中，通过角点检测出来的旧特征点
		nextPts：nextImg中，通过API利用角点检测计算出来的新特征点
		status：指示每一个特征点的追踪状态，如果特征点追踪成功，对应的status值为1，否则，status值为0，vector<uchar>类型
		err：指示每一个特征点的追踪误差，vector<float>
		winSize：用于计算光流的邻域窗口大小，更大的窗口可以提供更好的稳定性，但是可能牺牲精度，默认是Size(21, 21)
		maxLevel：表示光流计算中，使用金字塔结构的最深层级，金字塔用于多尺度分析（因为Lucas-Kanade算法在假设像素点移动距离很小时才有效，当移动距离很大时，考虑将图像分辨率缩小，比如底层图像是原始分辨率100*100，倒数第二层图像是50*50的分辨率），可以提高追踪的鲁棒性，默认值为3
		criteria：光流搜索迭代算法的终止条件，包括迭代次数和两次迭代间变化的最小阈值
		flags：标志位，指定额外的行为选项，默认为0即可
		minEigThreshshold：最小特征值阈值，用于确定光流矩阵的奇异性和稳定性，当像素点对应的光流矩阵，其最小特征值小于该阈值时，那么该像素点的光流估计会被认为是不可靠的（无效的特征点），反之，该像素点光流估计可靠。但是，并不是minEigThreshold设置的越小越好，太小的话，会导致本就不可靠的特征点被误判为可靠，默认为1e-4即可。
*/
void calcOpticalFlowPyrLK( InputArray prevImg, InputArray nextImg, InputArray prevPts, InputOutputArray nextPts, OutputArray status, OutputArray err, Size winSize = Size(21,21), int maxLevel = 3, TermCriteria criteria = TermCriteria(TermCriteria::COUNT+TermCriteria::EPS, 30, 0.01), int flags = 0, double minEigThreshold = 1e-4);


/*
		基于稀疏光流法（KLT）的视频分析使用例子
*/
RNG rng(1234);
void DrawLines(Mat& frame, const vector<Point2f>& pts1, const vector<Point2f>& pts2) {
	for (size_t i = 0; i < pts1.size(); ++i) {
		line(frame, pts1[i], pts2[i], Scalar(rng.uniform(0,256), rng.uniform(0, 256), rng.uniform(0, 256)), 2, 8, 0);
	}
}
void QuickDemo::VideoOpticalFlowAnalysis1() {
	VideoCapture capture("D:\\Photo\\images\\balltest.mp4");
	if (!capture.isOpened()) {
		cout << "Open video failure...." << endl;
		return;
	}
	namedWindow("frame", WINDOW_AUTOSIZE);
	
	//对视频第一帧进行角点检测
	Mat old_frame, old_gray;
	capture.read(old_frame);
	cvtColor(old_frame, old_gray, COLOR_BGR2GRAY);
	
	vector<Point2f> initPoints;
	vector<Point2f> feature_pts;
	double quality_level = 0.01;
	int minDistance = 10;
	
	//shi-Tomas角点检测
	goodFeaturesToTrack(old_gray, feature_pts, 100, quality_level, minDistance, Mat(), 3, false);

	Mat frame, gray;
	vector<Point2f> pts[2];		//定义了存储两个vector<Point2f>的数组，存储新帧和旧帧的特征点集合
	pts[0].insert(pts[0].end(), feature_pts.begin(), feature_pts.end());	//旧帧特征点集合
	initPoints.insert(initPoints.end(), feature_pts.begin(), feature_pts.end());

	vector<uchar> status;
	vector<float> error;
	TermCriteria criteria = TermCriteria(TermCriteria::COUNT + TermCriteria::EPS, 10, 0.01);	//停止标准

	while (true) {
		bool isGrabbed = capture.read(frame);
		if (!isGrabbed)  break;
		int c = waitKey(1000 / capture.get(CAP_PROP_FPS));
		if (c == 27)  break;

		cvtColor(frame, gray, COLOR_BGR2GRAY);

		//calculate optical flow, based on gray images, new frame's cotners will be calculated.
		calcOpticalFlowPyrLK(old_gray, gray, pts[0], pts[1], status, error, Size(31, 31), 3, criteria, 0);
		//遍历计算光流得出来的新帧点集合
		int i=0, k = 0;
		for (i = 0; i < pts[1].size(); ++i) {
			//距离与状态检测
			double dist = static_cast<double>(abs(pts[1][i].x - pts[0][i].x)) + static_cast<double>(abs(pts[1][i].y - pts[0][i].y));
			if (dist > 2.0 && status[i]) {
				//更新点集合状态，将status无效的点覆盖
				pts[0][k] = pts[0][i];		
				pts[1][k] = pts[1][i];
				initPoints[k] = initPoints[i];
				++k;

				int b = rng.uniform(0, 256);
				int g = rng.uniform(0, 256);
				int r = rng.uniform(0, 256);
				circle(frame, pts[1][i], 2, Scalar(b, g, r), 2, 8);		//在新帧中绘制特征点
				line(frame, pts[0][i], pts[1][i], Scalar(b, g, r), 2, 8, 0);	//新帧中绘制光流变化直线
			}
		}

		//update key points
		pts[0].resize(k);	//只有前k个点的status有效，pts[0]随着循环的进行，会不断缩小
		pts[1].resize(k);
		initPoints.resize(k);

		//基于初始点绘制跟踪线
		DrawLines(frame, initPoints, pts[1]);

		//update to old
		std::swap(old_gray, gray);
		cv::swap(pts[0], pts[1]);

		//re-init
		if (pts[0].size() < 40) {
			//shi-Tomas角点检测
			goodFeaturesToTrack(old_gray, feature_pts, 200, quality_level, minDistance, Mat(), 3, false);
			pts[0].insert(pts[0].end(), feature_pts.begin(), feature_pts.end());
			initPoints.insert(initPoints.end(), feature_pts.begin(), feature_pts.end());
		}
		imshow("KLT_Demo", frame);
	}

	capture.release();
}
```

### 2.9 基于光流法的视频分析 - 下

&emsp;&emsp;<font color = red>稠密光流分析法</font>，与稀疏光流分析法不同的是，**稠密光流算法估计每一像素点的光流向量（运动矢量）**，这意味着整个图像中的**所有像素都会被考虑**，从而产生**一个稠密的位移向量场**。**OpenCV中，稠密光流分析法是基于$ Gunnar Farneback$算法**。

```c++
/*
		稠密光流视频分析法函数原型
		
		prev：单通道8位灰度图像，光流计算的起点
		next：和prev同大小、同类型的图像，光流计算的目标帧
		flow：存储每个像素点的光流向量，Mat<Point2f>类型
		pyr_scale：金字塔缩放比例，用于创建多尺度金字塔，例如，0.5表示下一层金字塔图像尺寸是上一层的一半，默认值0.5即可
		levels：金字塔的深度，更大的深度意味着更加精细的尺度分析，默认值3即可
		winsize：计算光流时使用邻域的窗口大小，窗口越大，光流场越平滑，但是可能会丢失细节，默认值15即可
		iterations：在每一个金字塔层中迭代优化光流的次数，更多的迭代次数可以提高光流的精度，默认值3即可
		poly_n：用于进行多项式扩展的邻域大小，在计算光流时，会对每个像素点的邻域进行多项式拟合，以提高估计的准确性，默认值5即可
		poly_sigma：多项式拟合时的高斯权重函数的标准差，它决定了邻域内哪些点对中心点的影响更大默认值1.1即可
		flags：用于指定附加行为的标志位，默认值0即可
		
*/
void calcOpticalFlowFarneback(InputArray prev, InputArray next, InputOutputArray flow, double pyr_scale, int levels, int winsize, int iterations, int poly_n, double poly_sigma, int flags);

/*
		将笛卡尔系的二维坐标向量转换为极坐标系
		x：输入的x坐标，通常是单通道的Mat类对象，CV_32F或CV_64F
		y：输入的y坐标，类型和x的类型一致
		magnitude：存储极坐标系下向量的幅度（向量的模）
		angle：存储极坐标系下向量的角度，即与x轴正向的夹角
		angleDegrees：角度的单位，默认false即弧度制，true即角度制
*/
void cartToPolar(InputArray x, InputArray y, OutputArray magnitude, OutputArray angle, bool angleInDegrees = false);


/*
		稠密光流分析法使用例子，基本步骤
		
		1. 对视频流第一帧图像进行灰度化，作为第一个旧帧
		2. 在循环中，提取第一个新帧，灰度化
		3. 调用稠密光流分析法函数
		4. 提取光流向量在x轴和y轴方向上的偏移量
		5. 将偏移量转换为极坐标表示
		6. 将转换为极坐标的角度映射到0~180，极坐标向量的幅度映射到0~255（利用归一化），然后将角度的映射作为H，设置mv[1]为S，幅度的映射作为V，进行通道合并成HSV图像
		7. 最后，将HSV图像转换为BGR图像，将稠密光流分析的结果展示出来
		
*/
void QuickDemo::VideoOpticalFlowAnalysis2() {
	VideoCapture capture("D://Photo//images//opencv_demo//vtest.avi");
	if (!capture.isOpened()) {
		cout << "Open failure........" << endl;
		return;
	}
	Mat frame, pre_frame;
	Mat gray, pre_gray;
	capture.read(pre_frame);
	cvtColor(pre_frame, pre_gray, COLOR_BGR2GRAY);
	Mat hsv = Mat::zeros(pre_frame.size(), pre_frame.type());
	Mat mag = Mat::zeros(pre_frame.size(), CV_32FC1);		//32位单通道浮点数
	Mat ang = Mat::zeros(pre_frame.size(), CV_32FC1);		
	Mat xpts = Mat::zeros(pre_frame.size(), CV_32FC1);		
	Mat ypts = Mat::zeros(pre_frame.size(), CV_32FC1);		

	vector<Mat> mv;
	split(hsv, mv);		//通道分离

	Mat result;

	double fps = capture.get(CAP_PROP_FPS);

	while (1) {
		int c = waitKey(1000.0 / fps);
		if (c == 27) break;
		bool tmp = capture.read(frame);
		if (!tmp) {			//读取视频最后一帧，退出
			break;
		}	

		cvtColor(frame, gray, COLOR_BGR2GRAY);

		Mat flow;		//稠密光流分析法输出检测的像素点，也就是每个像素点的光流向量
		calcOpticalFlowFarneback(pre_gray, gray, flow, 0.5, 3, 15, 3, 5, 1.2, 0);

    //提取光流向量的x和y轴偏移量
		for (int row = 0; row < flow.rows; ++row) {
			for (int col = 0; col < flow.cols; ++col) {
				const Point2f& flow_xy = flow.at<Point2f>(row, col);
				xpts.at<float>(row, col) = flow_xy.x;
				ypts.at<float>(row, col) = flow_xy.y;
			}
		}
    
    //将光流向量转换为极坐标表示
		cartToPolar(xpts, ypts, mag, ang);
    
    //向量的角度映射到0到180度表示
		ang = ang * 180 / CV_PI / 2;
    
    //将向量的模归一化，映射到0到255
		normalize(mag, mag, 0, 255, NORM_MINMAX);
    
    //对极坐标系下向量的幅度和角度取绝对值，转换为CV_8UC1类型
		convertScaleAbs(mag, mag);		
		convertScaleAbs(ang, ang);
    
    //将类型转换后的ang和mag进行通道合并，hsv色彩空间
		mv[0] = ang;
		mv[1] = Scalar(255);
		mv[2] = mag;
		merge(mv, hsv);
		cvtColor(hsv, result, COLOR_HSV2BGR); 

		imshow("frame", frame);
		imshow("result", result);
	}
	capture.release();
}

```

### 2.10 均值迁移分析法

```c++
/*
		均值迁移分析法函数原型
*/


/*
		均值迁移分析法使用例子
*/
void QuickDemo::MeanValueShiftAnalysis() {
	VideoCapture capture("D://Photo//images//balltest.mp4");
	if (!capture.isOpened()) {
		cout << "Open failure........" << endl;
		return;
	}
	double fps = capture.get(CAP_PROP_FPS);


	Mat frame, hsv, hue, mask, hist, backproj;
	capture.read(frame);

	bool init = true;
	Rect trackWindow;

	int hsize = 16;
	float hranges[] = { 0,180 };
	const float* ranges[] = { hranges };

	//从frame中提取ROI区域
	Rect selection = selectROI("MeanShift Demo", frame, true, false);

	while (1) {
		int c = waitKey(1000.0 / fps);
		if (c == 27) break;

		bool isCrabbed = capture.read(frame);
		if (!isCrabbed)	break;
		cvtColor(frame, hsv, COLOR_BGR2HSV);
		inRange(hsv, Scalar(26, 43, 46), Scalar(34, 255, 255), mask);
		int ch[] = { 0,0 };
		hue.create(hsv.size(), hsv.depth());
		mixChannels(&hsv, 1, &hue, 1, ch, 1);	//通道混合

		//初始化
		if (init) {		
			Mat roi(hue, selection);
			Mat mask_roi(mask, selection);
			//计算直方图
			calcHist(&roi, 1, 0, mask_roi, hist, 1, &hsize, ranges);
			normalize(hist, hist, 0, 255, NORM_MINMAX);
			trackWindow = selection;
			init = false;
		}

		//计算直方图反向投影
		calcBackProject(&hue, 1, 0, hist, backproj, ranges);
		//直方图反向投影的结果与掩码图进行与操作
		backproj &= mask;

		//迭代次数和两次迭代间变化的最小阈值
		TermCriteria criteria = TermCriteria(TermCriteria::COUNT | TermCriteria::EPS, 30, 1);

		meanShift(backproj, trackWindow, criteria);
		
		//均值迁移分析
		RotatedRect rrt = CamShift(backproj, trackWindow, criteria);

		
		//绘制矩形
		rectangle(frame, trackWindow, Scalar(0, 0, 255), 2, LINE_AA);

		//绘制椭圆
		ellipse(frame, rrt, Scalar(255, 200, 0), 2, LINE_AA);

		imshow("MeanShift Demo", frame);
	}
	capture.release();
}
```

