## RHEED 图像自定义二值化方法

​		RHEED图像自定义二值化方法的优化是一个**长期的过程**，**迭代过程中会不断地优化亮斑目标检测的精确度**。

## 一、版本1 

​		初步确定使用**自适应阈值**是最好的RHEED图像二值化方法。

```c++
static void gray_and_binary(Mat& src, Mat& gray, Mat& binary) {
	// 高斯模糊
	GaussianBlur(src, src, Size(5, 5), 20, 5);

	// 灰度化
	cvtColor(src, gray, COLOR_BGR2GRAY);

	// 自定义阈值
	//threshold(gray, binary, 95, 255, THRESH_BINARY);

	// 全局阈值二值化
	//threshold(gray, binary, 0, 255, THRESH_BINARY | THRESH_OTSU);

	// 自适应阈值二值化
	adaptiveThreshold(gray, binary, 255, ADAPTIVE_THRESH_GAUSSIAN_C, THRESH_BINARY, 21, 0);

	// 三角阈值法
	//threshold(gray, binary, 0, 255, THRESH_BINARY | THRESH_TRIANGLE);

	// Canny基于轮廓的二值化
	//Canny(gray, binary, 40, 160);
  
  // 形态学操作
  Mat kernel = getStructuringElement(MORPH_RECT, Size(3, 3), Point(-1, -1));
	morphologyEx(binary, binary, MORPH_OPEN, kernel, Point(-1, -1), 3);
	morphologyEx(binary, binary, MORPH_CLOSE, kernel, Point(-1, -1), 3);
}

```

## 二、版本2 

​		与 version 1 不同的是，version 2 利用了`adaptiveThreshold()`函数的`double C`参数，`double C`参数是**局部阈值减去的一个值**。

```c++
void RHEED::gray_and_binary(Mat& image, Mat& gray, Mat& binary) {
	GaussianBlur(image, image, Size(3, 3), 0);

	cvtColor(image, gray, COLOR_BGR2GRAY);

	// 最后一个参数为每一个局部阈值减去常数
	adaptiveThreshold(gray, binary, 255, ADAPTIVE_THRESH_MEAN_C, THRESH_BINARY, 5, -1.9);

	// 形态学操作
	Mat kernel = getStructuringElement(MORPH_RECT, Size(3, 3));
	//erode(binary, binary, kernel, Point(-1, -1), 1);
	//dilate(binary, binary, kernel, Point(-1, -1), 2);
	//morphologyEx(binary, binary, MORPH_OPEN, kernel);
	morphologyEx(binary, binary, MORPH_OPEN, kernel, Point(-1, -1), 1);
	morphologyEx(binary, binary, MORPH_CLOSE, kernel, Point(-1, -1), 2);
}
```

## 三、版本3

​		本版本尝试通过**对图像进行加权融合**，这也带来了一个坏处，就是**噪声的干扰更多了**，初步的解决方案是**利用图像锐化操作（加大特征的显示，噪声可以使用高斯模糊解决）**，每对两张图像加权之后，进行一次锐化。

### 3.1 图像融合

```c++
/*
		图像融合函数原型
		函数参数
		- src1：输入的第一个叠加图像
		- alpha：第一个叠加图像的权重，这个值貌似可以超过1，如果低于1，图像的特征会被弱化
		- src2：输入的第二个叠加图像
		- beta：第二个叠加图像的权重
		- gama：是一个标量值，可正可负，将被加到每一个求和的结果上
		- dst：图像融合的结果，大小和深度与 src1 相同
		- dtype：可选参数，指定输出图像的深度，默认 -1 即可，即输出图像的深度与 src1 一样
*/
CV_EXPORTS_W void addWeighted(InputArray src1, double alpha, InputArray src2, double beta, double gamma, OutputArray dst, int dtype = -1);

/*
		图像融合使用例子
*/
int main(int argc, char* argv[]) {
	RHEED rheed;

	Mat src1 = imread("D:\\Photo\\rheed_image5\\137.png");
	Mat src2 = imread("D:\\Photo\\rheed_image5\\158.png");

	Mat src3, src4;
	
  // 图像融合
	addWeighted(src1, 1, src2, 1, -60, src3);

	// 8 领域 Laplace 算子
	Mat kernel2 = (Mat_<double>(3, 3) <<
		0, -1, 0,
		-1, 5, -1,
		0, -1, 0
		);

	// laplace锐化
	cv::filter2D(src3, src4, -1, kernel2, Point(-1, -1));


	imshow("src1", src1);
	imshow("src2", src2);
	imshow("src3", src3);
	imshow("src4", src4);

	waitKey(0);
	destroyAllWindows();
	return 0;
}
```

### 3.2 图像锐化

主要有两个方法，分别是**基于`Laplace`算子的锐化和`USM(Unsharp Mask sharpening)`锐化**。

- Laplace锐化（初步锐化）的主要原理是基于`Laplace`算子，对图像进行卷积，Laplace 锐化可以增强图像的边缘和细节，**但也会放大图像中的噪声**，所以，在使用 Laplace 算子之前，可以用**高斯模糊等进行去噪**。

```c++
/*
		Laplace锐化函数原型，OpenCV提供的该函数原型不仅可以进行图像锐化，还可以实现模糊，边缘检测等
		函数参数
		- src：输入的图像
		- dst：输出的目标图像，和src同大小
		- ddepth：输出图像dst的深度，默认值为-1，输出的图像深度和src一致
		- kernel：用于卷积操作的内核矩阵
		- anchor：锚点，表示内核相对于图像像素的位置，默认值为Point(-1, -1)，意味着锚点位于内核中心
		- delta：值加到每个结果像素上，默认为0
		- borderType：边界类型，定义了卷积操作如何处理图像边界外的像素，BORDER_DEAULT通常意味着使用复制边界处理方法（这样的话，卷积操作可以覆盖到图像的每一个像素点）
		
*/
CV_EXPORTS_W void filter2D(InputArray src, OutputArray dst, int ddepth, InputArray kernel, Point anchor = Point(-1,-1), double delta = 0, int borderType = BORDER_DEFAULT);

/*
		Laplace锐化使用例子，两种方式，推荐第二种，需要自定义锐化卷积核
		- 第一种方式：手动遍历图像每一个像素点，进行锐化
		- 第二种方式：定义锐化卷积核，使用filter2D()函数对图像锐化
*/
void laplace_demo() {
	Mat src = imread("D:\\Photo\\rheed_image4\\54.png", IMREAD_COLOR);
	if (src.empty()) {
		printf("could not load image...\n");
		return;
	}
	imshow("src", src);

	GaussianBlur(src, src, Size(3, 3), 0);

	Mat dst1 = Mat::zeros(src.size(), src.type());
	Mat dst2 = Mat::zeros(src.size(), src.type());
	Mat dst3 = Mat::zeros(src.size(), src.type());

	// 手动 8 邻域 laplace 算子对图像进行锐化
	for (int i = 0; i < src.rows; ++i) {
		for (int j = 0; j < src.cols; ++j) {
			for (int k = 0; k < 3; ++k) {
				int sum = 9 * src.at<Vec3b>(i, j)[k] - src.at<Vec3b>(i - 1, j - 1)[k] - src.at<Vec3b>(i - 1, j)[k]
					- src.at<Vec3b>(i - 1, j + 1)[k] - src.at<Vec3b>(i, j - 1)[k] - src.at<Vec3b>(i, j + 1)[k]
					- src.at<Vec3b>(i + 1, j - 1)[k] - src.at<Vec3b>(i + 1, j)[k] - src.at<Vec3b>(i + 1, j + 1)[k];
				dst1.at<Vec3b>(i, j)[k] = saturate_cast<uchar>(sum);		//范围内强制类型转换
			}
		}
	}

	// 利用 opencv 的API实现laplace算子图像锐化，关键是如何定义 Laplace 算子
	// 4 领域 Laplace 算子
	Mat kernel1 = (Mat_<double>(3, 3) <<
		0, -1, 0,
		-1, 5, -1,
		0, -1, 0
		);
	// 8 领域 Laplace 算子
	Mat kernel2 = (Mat_<double>(3, 3) <<
		-1, -1, -1,
		-1, 9, -1,
		-1, -1, -1
		);

	cv::filter2D(src, dst2, -1, kernel1, Point(-1, -1));
	cv::filter2D(src, dst3, -1, kernel2, Point(-1, -1));

	imshow("operate laplace result", dst1);
	imshow("auto 4 domain laplace result", dst2);
	imshow("auto 8 domain laplace result", dst3);
}

```

- USM(Unsharp Mask sharpening)锐化的基本原理如下：

  - 第一步，对图像进行**滤波（模糊）操作**

  - 第二步，原图像和模糊图像进行像素**差值**（也有残差的概念，即实际值和观测值之间的差异）

  - 第三步，**将原图像和第二步得到的差值图像**进行**求和操作**，该操作涉及到一个**变量`amount`**，也叫**锐化因子**，控制**差值图像**的比例

​	总体操作可以用**如下公式表示**，USM方法可以实现比Laplace方法**更加细致的锐化**，可以使用`threshold`阈值来**动态控制锐化区域**：

$$
dst[i][j] = \left\{\begin{matrix} 
        src[i][j]+\left ( src[i][j] -  GuassianBlur[i][j] \right )\times amount,\space if>=threshold \\  
        src[i][j],\space if < threshold
      \end{matrix}\right.
$$

```c++
/*
		USM锐化使用例子
*/
void USM_demo() {
	Mat src = imread("D:\\Photo\\rheed_image4\\50.png", IMREAD_COLOR);
	if (src.empty()) {
		printf("could not load image...\n");
		return;
	}

	Mat blur1 = Mat::zeros(src.size(), src.type());
	Mat blur2 = Mat::zeros(src.size(), src.type());

	Mat dst1 = Mat::zeros(src.size(), src.type());
	Mat dst2 = Mat::zeros(src.size(), src.type());
	
	// 二维高斯卷积核，对核内的每一个像素值进行高斯概率密度计算
	GaussianBlur(src, blur1, Size(99, 99), 0);
	GaussianBlur(src, blur2, Size(99, 99), 0);

	float amount = 0.90;
	int threshold = 5;

	// 方式1，逐像素操作，可以控制阈值
	for (int row = 0; row < src.rows; ++row) {
		for (int col = 0; col < src.cols; ++col) {
			int b = src.at<Vec3b>(row, col)[0] - blur1.at<Vec3b>(row, col)[0];
			int g = src.at<Vec3b>(row, col)[1] - blur1.at<Vec3b>(row, col)[1];
			int r = src.at<Vec3b>(row, col)[2] - blur1.at<Vec3b>(row, col)[2];
			if (abs(b) >= threshold) {
				b = src.at<Vec3b>(row, col)[0] + b * amount;
				dst1.at<Vec3b>(row, col)[0] = saturate_cast<uchar>(b);
			}

			if (abs(g) >= threshold) {
				g = src.at<Vec3b>(row, col)[1] + g * amount;
				dst1.at<Vec3b>(row, col)[1] = saturate_cast<uchar>(g);
			}

			if (abs(r) >= threshold) {
				r = src.at<Vec3b>(row, col)[2] + r * amount;
				dst1.at<Vec3b>(row, col)[2] = saturate_cast<uchar>(r);
			}
		}
	}



	// 方式2，整图操作
	// 转换为浮点数类型，防止做差操作溢出
	Mat src_float, blur_float;
	src.convertTo(src_float, CV_32F);
	blur2.convertTo(blur_float, CV_32F);

	// 计算残差图 mask
	Mat subtract_mask = src_float - blur_float;

	// 将残差图按比例加回到原始图像中
	Mat sharpened = src_float + amount * subtract_mask;

	// 1.0 缩放系数，0 偏置值 
	sharpened.convertTo(dst2, CV_8UC3, 1.0, 0);
	imshow("src", src);
	imshow("USM 逐像素 + threshold", dst1);
	imshow("USM 整图操作", dst2);
}
```
