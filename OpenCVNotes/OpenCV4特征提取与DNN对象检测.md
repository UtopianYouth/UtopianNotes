## 第一部分 图像特征提取

### 1. 图像特征提取相关知识

​		在计算机视觉中，**图像特征是该图像的唯一表述，是图像的DNA**，一般用向量数据表示，图像特征数据具有<font color = red>尺度空间不变性、像素迁移不变性、关照一致性原则和旋转不变性原则</font>。图像特征提取**有两种方式，传统图像特征提取和`DNN/CNN`特征提取**。

- **传统图像特征提取**：基于纹理、角点、颜色分布、梯度、边缘等。
- **`DNN/CNN`特征提取**：基于监督学习，进行自动特征提取。  

#### 1.1 角点检测

​		从图像特征的概念来看，**角点（角点一般是一个区域，而不是特指一个像素点）也是属于图像特征的一部分**，快速回顾OpenCV4图像与视频分析的**角点部分**即可。

#### 1.2 关键点检测

​		<font color = red>关键点检测</font>，是在图像或视频中识别**出具有代表性的关键点**，关键点可以用来**实现物体的识别和行为的跟踪**，关键点对理解图像内容很重要，如人脸上的鼻子、眼睛等。OpenCV4提供了**两种关键点检测方法**，分别是**ORB关键点检测**和**SIFT关键点检测**，其中，ORB关键点检测算法比SIFT关键点检测算法速度更快。

```c++
/*
		创建ORB关键点检测对象
		函数参数	
    - nfeatures: 关键点检测的上限个数
		返回值
		- Ptr<ORB>: 关键点检测对象指针
*/
static Ptr<ORB> create(int nfeatures=500, float scaleFactor=1.2f, int nlevels=8, int edgeThreshold=31, int firstLevel=0, int WTA_K=2, ORB::ScoreType scoreType=ORB::HARRIS_SCORE, int patchSize=31, int fastThreshold=20);

/*
		ORB关键点检测函数原型
		函数参数
		- image: 关键点检测的图像
		- keypoints: 存储关键点检测的结果
		- mask: 关键点检测的ROI区域
		
*/
virtual void detect( InputArray image, CV_OUT std::vector<KeyPoint>& keypoints, InputArray mask=noArray() );


/*
		使用例子
		- 对需要进行关键点检测的图像进行灰度化和二值化
		- 创建ORB关键点检测对象
		- 调用detect函数进行关键点检测
		- optional：绘制关键点检测结果到原图像中
		
*/
void QuickDemo::ORB_key_point_detection() {
	Mat src, gray, bianry;
	src = imread("D:\\Photo\\rheed_image2\\25.png", IMREAD_COLOR);
	gray_and_binary1(src, gray, bianry);

	Ptr<ORB> key_point_pts = ORB::create(10);	//定义关键点检测指针，检测个数上限为300

	//关键点检测并绘制
	vector<KeyPoint> pts;
	key_point_pts->detect(src, pts, Mat());
  
  //绘制检测到的关键点
	drawKeypoints(src, pts, src, Scalar(100, 200, 255), DrawMatchesFlags::DEFAULT);
	imshow("src", src);
}
```

```c++
/*
		创建SIFT关键点检测对象
		函数参数
		- nfeatures: 关键点检测的上限个数
		返回值
		- Ptr<SIFT>: SIFT关键点检测对象指针
*/
static Ptr<SIFT> create(int nfeatures = 0, int nOctaveLayers = 3, double contrastThreshold = 0.04, double edgeThreshold = 10, double sigma = 1.6, bool enable_precise_upscale = false);

/*
		SIFT关键点检测函数原型
		函数参数
		- image: 关键点检测的图像
		- keypoints: 存储关键点检测的结果
		- mask: 关键点检测的ROI区域
		
*/
virtual void detect( InputArray image, CV_OUT std::vector<KeyPoint>& keypoints, InputArray mask=noArray() );

/*
		使用例子
    - 对需要进行关键点检测的图像进行灰度化和二值化
		- 创建SIFT关键点检测对象
		- 调用detect函数进行关键点检测
		- optional：绘制关键点检测结果到原图像中
*/
void QuickDemo::SIFT_key_point_detection() {
	Mat src, gray, bianry;
	src = imread("D:\\Photo\\rheed_image2\\25.png", IMREAD_COLOR);
	gray_and_binary1(src, gray, bianry);

	Ptr<SIFT> key_point_pts = SIFT::create(300);	//定义关键点检测指针，检测个数上限为300

	//关键点检测
	vector<KeyPoint> pts;
	key_point_pts->detect(src, pts, Mat());
  
  //绘制检测到的关键点
	drawKeypoints(src, pts, src, Scalar(100, 200, 255), DrawMatchesFlags::DEFAULT);

	imshow("src", src);
}
```

#### 1.3 基于关键点检测的描述子生成

​		<font color = red>描述子</font>，就是在**关键点检测的基础上**，用来描述**关键点周围区域的信息**，**描述子是一个向量，向量的维度就是描述子的长度**。描述子的生成如下：

- **局部特征提取**：首先检测出关键点，然后在这些关键点周围选取一定大小的区域
- **特征描述**：计算该区域内的**统计信息**，如**像素变化梯度方向的直方图、像素强度对比等**。
- **特征描述向量化**：将计算出的统计信息**编码成一个固定长度的向量，这就是描述子（同一辐图像的所有描述子长度相等，不同图像的描述子可能长度不相等）。**

通常，描述子所描述的图像特征，**具有不变性和鲁棒性（不同条件下，如图像旋转，缩放和亮度变化，描述子都保持一致的描述能力）**，描述子主要用于**特征匹配**。

```c++
/*
		基于ORB关键点检测描述子生成
		函数参数
		- image: 计算描述子的原始图像
		- keypoints: 计算描述子的关键点
		- descriptors: 存储输出的描述子信息
*/
virtual void compute( InputArray image,CV_OUT CV_IN_OUT std::vector<KeyPoint>& keypoints,OutputArray descriptors);
/*
		基于SIFT关键点检测描述子生成（和ORB描述子生成一样）
    函数参数
		- image: 计算描述子的原始图像
		- keypoints: 计算描述子的关键点
		- descriptors: 存储输出的描述子信息
*/
virtual void compute( InputArray image, CV_OUT CV_IN_OUT std::vector<KeyPoint>& keypoints, OutputArray descriptors );

/*
		以ORB关键点检测为前提，计算描述子
*/
void QuickDemo::ORB_key_point_detection() {
	Mat src, gray, bianry;
	src = imread("D:\\Photo\\rheed_image2\\25.png", IMREAD_COLOR);
	gray_and_binary1(src, gray, bianry);

	Ptr<ORB> key_point_pts = ORB::create(10);	//定义关键点检测指针，检测个数上限为300

	//关键点检测并绘制
	vector<KeyPoint> pts;
	key_point_pts->detect(src, pts, Mat());
	drawKeypoints(src, pts, src, Scalar(100, 200, 255), DrawMatchesFlags::DEFAULT);

	//描述子生成
	Mat descriptors;
	key_point_pts->compute(src, pts, descriptors);
	cout << descriptors.rows << " * " << descriptors.cols << endl;

	//打印描述子
	for(int i = 0; i < descriptors.rows; ++i){
		cout << descriptors.ptr<int>(i) << endl;		//描述子指针
		for (int j = 0; j < descriptors.cols; ++j) {
			cout << descriptors.at<int>(i, j) <<" ";	
		}
		cout << endl;
	}
	imshow("src", src);
}

/*
		以SIFT关键点检测为前提，计算描述子，和ORB关键点检测计算描述子方式雷同
*/
```

除了上述`API`可以实现描述子的生成，`ORB`和`SIFT`关键点检测对象还提供了`detectAndCompute()`接口，结合了`detect()`和`compute()`两个`API`的功能，先计算关键点，然后**基于关键点得出对应的描述子**。

```c++
/*
		关键点和描述子生成
		函数参数
		- image：需要计算关键点和描述子的图像
		- mask：限定图像的范围
		- keypoints：输出检测到的关键点
		- descriptors：输出计算得到的描述子
		- useProvidedKeypoints：
			- 默认false，该函数会重新计算关键点，并且利用重新计算的关键点进行描述子生成
			- 为true，该函数不会重新计算关键点，而是使用提供的关键点进行描述子生成（在需要多次描述子生成的情景下，可以节省资源，并且保证了关键点和描述子的一致性）
*/
virtual void detectAndCompute( InputArray image, InputArray mask, CV_OUT std::vector<KeyPoint>& keypoints, OutputArray descriptors, bool useProvidedKeypoints=false);

void QuickDemo::detect_and_compute() {
	Mat box;
	box = imread("D:\\Photo\\images\\book.jpg", IMREAD_COLOR);

	imshow("box", box);

	Ptr<ORB> orb = ORB::create(300);
  
  //存储关键点
	vector<KeyPoint> kypts_box;			

  //存储关键点周围的描述子信息
	Mat desc_box;	

  //检测和计算图像中的关键点以及描述子信息
	orb->detectAndCompute(box, Mat(), kypts_box, desc_box);	
}
```



#### 1.4 基于描述子的特征匹配

​		通过上一小节得到的**描述子**，我们知道**描述子（向量）是用来描述图像关键点周围区域的信息**，而这些关键点周围区域，描述了一个图像的**特征**。因此，我们可以基于不同图像的描述子，进行描述子信息对比，**实现不同图像的特征匹配**。

​		`OpenCV4`中，提供了**两种特征匹配的类**，**分别是暴力匹配`BFMatcher`和FLANN匹配`FlannBasedMatcher`**。

- 暴力特征匹配：全局搜索，计算最小距离，返回相似描述子集合
- `FLANN(Fast Library for Approximate Nearest Neighbors, 快速库近似最近邻居)`特征匹配，2009年发布的**开源高维数据匹配算法库**，**支持KMeans、 KDTree、 KNN、多探针LSH等搜索与匹配算法  **

```c++
/*
 		暴力特征匹配BFMatcher对象指针的创建
 		函数参数
 		- normType：距离度量类型，指定如何计算两个描述子（向量）之间的相似度，默认NORM_L2
 			- NORM_L1：使用哈曼顿距离作为相似度度量
 			- NORM_L2：使用欧几里得距离作为相似度度量
 			- NORM_HAMMING：使用汉明距离作为相似度度量，通常用于二进制描述子（ORB, BRISK等）
			- NORM_HAMMING2：类似于NORM_HAMMING，适用于更多二进制位的描述子 
 		- crossCheck：是否使用交叉检查
 			- true，查询描述子最接近训练描述子（匹配与被匹配的关系），训练描述子最接近查询描述子，才认为查询描述子和训练描述子是匹配的，有利于减少错误匹配，但是会增加计算量
 			- false，不使用交叉检查，即查询描述子最接近训练描述子即可
    返回值
    - Ptr<BFMatcher>, BFMatcher对象指针
*/
CV_WRAP static Ptr<BFMatcher> create(int normType=NORM_L2, bool crossCheck=false) ;

/*
		暴力特征匹配函数原型
		函数参数
		- queryDescriptors：查询描述子，即主动进行特征匹配图像的描述子
		- trainDescriptors：训练描述子，即被进行特征匹配图像的描述子
		- matches：存储特征匹配的结果，为描述子索引对
		- mask：进行特征匹配的两幅图像范围（查询描述子）
*/
CV_WRAP void match(InputArray queryDescriptors, InputArray trainDescriptors, CV_OUT std::vector<DMatch>& matches, InputArray mask=noArray()) const;

/*
		暴力特征匹配使用例子
*/
void QuickDemo::feature_match() {
	Mat box_in_scene, box;
	box = imread("D:\\Photo\\images\\book.jpg", IMREAD_COLOR);
	box_in_scene = imread("D:\\Photo\\images\\book_on_desk.jpg", IMREAD_COLOR);

	imshow("box", box);
	imshow("box_in_scene", box_in_scene);

	Ptr<ORB> orb = ORB::create(300);
  //存储关键点
	vector<KeyPoint> kypts_box;			
	vector<KeyPoint> kypts_box_in_scene;

  //存储关键点周围的描述子信息
	Mat desc_box, desc_box_in_scene;

  //检测和计算图像中的关键点以及描述子信息
	orb->detectAndCompute(box, Mat(), kypts_box, desc_box);	
	orb->detectAndCompute(box_in_scene, Mat(), kypts_box_in_scene, desc_box_in_scene);

  //特征匹配的结果
	Mat result1;

	//基于ORB关键点检测的暴力特征匹配
	vector<DMatch> matches1;
	Ptr<BFMatcher> bf_matcher = BFMatcher::create(NORM_HAMMING, false);
	bf_matcher->match(desc_box, desc_box_in_scene,matches1, Mat());

	/*
			绘制特征匹配的结果
			主要参数
			- 特征匹配的图像，特征匹配图像的关键点，特征匹配的结果
	*/
	drawMatches(box, kypts_box, box_in_scene, kypts_box_in_scene, matches1, result1);

	imshow("ORB关键点检测 + 暴力特征匹配结果", result1);
}
```

```c++
/*
		FLANN特征匹配对象的创建
		函数参数
		- indexParams：指向flann::IndexParams对象的智能指针，用于指定构建描述子时使用的参数，默认情况下makePtr<flann::KDTreeIndexParams>()，表示使用KD树作为索引结构（将图像描述子在向量中的索引用KD树存储）
		- searchParams：指向flann::SearchParams对象的智能指针，用于指定搜索最近邻时使用的参数，默认情况下makePtr<flann::SearchParams>()
*/
CV_WRAP FlannBasedMatcher(const Ptr<flann::IndexParams>& indexParams=makePtr<flann::KDTreeIndexParams>(), const Ptr<flann::SearchParams>& searchParams=makePtr<flann::SearchParams>());

/*
		FLANN特征匹配函数原型
		函数参数
    - queryDescriptors：查询描述子，即主动进行特征匹配图像的描述子
		- trainDescriptors：训练描述子，即被进行特征匹配图像的描述子
		- matches：存储特征匹配的结果，为描述子索引对
		- mask：进行特征匹配的两幅图像范围（查询描述子）
*/
CV_WRAP void match(InputArray queryDescriptors, InputArray trainDescriptors, CV_OUT std::vector<DMatch>& matches, InputArray mask=noArray()) const;

/*
		FLANN特征匹配使用例子
*/
void QuickDemo::feature_match() {
	Mat box_in_scene, box;
	box = imread("D:\\Photo\\images\\book.jpg", IMREAD_COLOR);
	box_in_scene = imread("D:\\Photo\\images\\book_on_desk.jpg", IMREAD_COLOR);

	imshow("box", box);
	imshow("box_in_scene", box_in_scene);

	Ptr<ORB> orb = ORB::create(300);
	vector<KeyPoint> kypts_box;			//存储关键点
	vector<KeyPoint> kypts_box_in_scene;

	Mat desc_box, desc_box_in_scene;	//存储关键点周围的描述子信息

	orb->detectAndCompute(box, Mat(), kypts_box, desc_box);		//检测和计算图像中的关键点以及描述子信息
	orb->detectAndCompute(box_in_scene, Mat(), kypts_box_in_scene, desc_box_in_scene);

	Mat result2;

	//基于ORB关键点检测的FLANN特征匹配
	vector<DMatch> matches2;
	FlannBasedMatcher flann_matcher = FlannBasedMatcher(new flann::LshIndexParams(6, 12, 2));
	flann_matcher.match(desc_box, desc_box_in_scene, matches2, Mat());

	//绘制特征匹配的结果
	drawMatches(box, kypts_box, box_in_scene, kypts_box_in_scene, matches2, result2);
  
	imshow("ORB关键点检测 + flann特征匹配结果", result2);
}
```

#### 1.5 图像的单应性变换/透视变换

​		图像的单应性变换和透视变换，描述的是同一件事情，**即从一个图像到另一个图像之间的一种映射**，单应性变换是在数学上的概念，而透视变换是计算机视觉上的概念。

​		在进行图像的透视变换时，最重要的是**求出原图像到新图像的相对位置，对应的透视变换矩阵**，然后，**用原图像矩阵上的像素点乘上透视变换矩阵，得到新图像上的像素点位置**。

​		图像透视变换主要应用在<font color = red>图像旋转，缩放，倾斜和平移</font>。

```c++
/*
		求单应性变换矩阵函数原型
		函数参数
		- srcPoints：原图像上需要进行单应性变换的像素点集合
		- dstPoints：新图像上进行单应性变换后的像素点集合
		
		返回值
		- Mat：计算出来的单应性变换矩阵
		
		后面的参数默认即可
*/
CV_EXPORTS_W Mat findHomography(InputArray srcPoints, InputArray dstPoints, int method = 0, double ransacReprojThreshold = 3, OutputArray mask=noArray(), const int maxIters = 2000, const double confidence = 0.995);

/*
		单应性变换/透视变换函数原型
		函数参数
		- src：输入需要进行透视变换的原图像
		- dst：透视变换之后的目标图像
		- M：透视变换矩阵，规格是 3*3
		- dsize：目标图像的大小
		
		后面参数默认即可
*/
CV_EXPORTS_W void warpPerspective( InputArray src, OutputArray dst, InputArray M, Size dsize, int flags = INTER_LINEAR, int borderMode = BORDER_CONSTANT, const Scalar& borderValue = Scalar());

/*
		透视变换在图像中轮廓校正的应用
		- 首先进行轮廓发现
		- 找到需要进行透视变换的轮廓
		- 通过轮廓逼近，找到轮廓边缘的关键像素点集合
		- 确定边缘的关键像素点，到新图像位置的像素点集合
		- 求出透视变换矩阵
		- 根据轮廓大小确定新图像的尺寸
		- 进行透视变换
*/
void QuickDemo::image_homography() {
	Mat src = imread("D:\\Photo\\images\\mytest.jpg", IMREAD_COLOR);
	Mat gray, binary;
	
	//二值化
	gray_and_binary1(src, gray, binary);

	//轮廓发现，绘制最大轮廓
	vector<vector<Point>> contours;
	vector<Vec4i> hierarchy;
	findContours(binary, contours, hierarchy, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE);

	int index = -1;
	double max = -1;
	for (int i = 0; i < contours.size(); ++i) {
		double area = contourArea(contours[i]);		//获取轮廓面积
		if (area > max) {
			max = area;
			index = i;
		}
	}
	//绘制最大轮廓
	drawContours(src, contours, index, Scalar(0, 0, 255), 2, LINE_AA);

	//利用多边形轮廓逼近，找到最大轮廓的边缘点（进行轮廓逼近前，先进行轮廓发现）
	vector<Point2f> pts;		//存储轮廓逼近的点集合结果
	approxPolyDP(contours[index], pts, 50, true);
	for (auto pt : pts) {
		circle(src, pt, 8, Scalar(200, 100, 100), 2, LINE_AA);
		cout << pt << endl;
	}

	//获取最小外接矩形的宽和高
	RotatedRect min_rect = minAreaRect(contours[index]);

	//进行了单应性变换和透视变换后，设定目标图像的大小
	double w = min_rect.size.width;		
	double h = min_rect.size.height;

	cout << w << endl << h << endl;

	//单应性变换，这里注意坐标变换时，目标图点集合的位置要与源图点集合的位置一一对应

	//变换的目标点集合
	vector<Point2f> dst_pts;	
	dst_pts.push_back(Point2f(0, 0));
	dst_pts.push_back(Point2f(0, h));
	dst_pts.push_back(Point2f(w, h));
	dst_pts.push_back(Point2f(w, 0));

	Mat tmp = findHomography(pts, dst_pts);		//返回单应性变换矩阵

	Mat result;
	warpPerspective(src, result, tmp, Size(w, h));		//根据计算出来的单应性变换矩阵，进行透视变换，第四个参数为透视变换后的图像大小		

	imshow("src", src);
	imshow("result", result);
}
```

## 第二部分 DNN对象检测

#### 2.1 OpenVINO在Visual Studio下的开发环境搭建，修改项目属性表（四部分）

- ###### 包含目录

```
D:\CodeEnvironment\openvino_toolkit_windows_2023.3\runtime\include

D:\CodeEnvironment\openvino_toolkit_windows_2023.3\runtime\include\openvino

D:\CodeEnvironment\openvino_toolkit_windows_2023.3\runtime\include\ngraph

D:\CodeEnvironment\openvino_toolkit_windows_2023.3\runtime\include\ie
```

- ###### 库目录

```
D:\CodeEnvironment\openvino_toolkit_windows_2023.3\runtime\lib\intel64\Release
```

- ###### 链接器

将库目录下的文件加入到链接器中

```
openvino.lib
openvino_c.lib
openvino_onnx_frontend.lib
openvino_paddle_frontend.lib
openvino_pytorch_frontend.lib
openvino_tensorflow_frontend.lib
openvino_tensorflow_lite_frontend.lib
```

- ###### 环境变量

```
D:\CodeEnvironment\openvino_toolkit_windows_2023.3\runtime\bin\intel64\Release
D:\CodeEnvironment\openvino_toolkit_windows_2023.3\runtime\3rdparty\tbb\bin
```

