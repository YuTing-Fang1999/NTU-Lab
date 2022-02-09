### 目的
為了檢察PCB瑕疵，我們需要用X-ray重建三維的錫球以檢察內部構造  

### ART技術
ART(Algebraic Reconstruction Technique)   
此例子是模擬重建過程  
將1D重建為2D  

### 問題
* 重建後如何決定原本的heigh?  
* 展示圖的時候要  
```cpp
cv::normalize(results, results, 0, 255, NORM_MINMAX, CV_8UC1);
```
* 計算圖的時候要
```cpp
cv::normalize(newArray, newArray, 0, 1, NORM_MINMAX, CV_32FC1);
```

### 程式環境
* 使用C++
* 使用vs2022與vcpkg

### 原理
<img src="https://user-images.githubusercontent.com/66452317/153203523-7612f902-4ffa-425c-bf6f-de702aa79237.png" width=600/>

![image](https://github.com/YuTing-Fang1999/NTU-Lab/blob/main/HomeWork/ART/resource/rotate.gif)
![image](https://github.com/YuTing-Fang1999/NTU-Lab/blob/main/HomeWork/ART/resource/addCol.gif)

在旋轉時，超出的範圍會被切掉  
![image](https://github.com/YuTing-Fang1999/NTU-Lab/blob/main/HomeWork/ART/resource/oriImg2.png)
![image](https://github.com/YuTing-Fang1999/NTU-Lab/blob/main/HomeWork/ART/resource/rotate2.gif)  
復原也只能復原沒被切掉的部分    
![image](https://github.com/YuTing-Fang1999/NTU-Lab/blob/main/HomeWork/ART/resource/result2.png)

### 公式

![image](https://user-images.githubusercontent.com/66452317/153180249-1c422d2a-2f52-443b-a226-9ff1c2e620c6.png)

### code
```cpp
#include <opencv2/core.hpp>
#include <opencv2/imgcodecs.hpp>
#include <opencv2/highgui.hpp>
#include <opencv2/opencv.hpp>

#include <iostream>
#include <stdio.h>

using namespace cv;

using namespace std;


Mat rotate(Mat src, double angle) {
	Mat dst;
	Point2f pt(src.cols / 2., src.rows / 2.);
	Mat r = getRotationMatrix2D(pt, angle, 1.0);
	warpAffine(src, dst, r, Size(src.cols, src.rows));
	return dst;
}

int main() {
	string image_path = samples::findFile("oriImg.png");
	Mat oriImg = imread(image_path, IMREAD_GRAYSCALE);
	int oriRows = oriImg.rows;

	Mat rotatedImg;
	Mat colAddImg(180, oriImg.cols, CV_32FC1); //0~179度
	

	if (oriImg.empty()) {
		cout << "Could not read the image: " << image_path << endl;
		return 1;
	}

	for (int angle = 0; angle < 180; angle++) {
		//旋轉圖片(0 ~ 179度)
		rotatedImg = rotate(oriImg, angle);
		
		float originalColSum;
		//遍歷每個col
		for (int col = 0; col < rotatedImg.cols; col++) {
			//計算每個colSum
			originalColSum = 0;
			for (int row = 0; row < rotatedImg.rows; row++) {
				originalColSum += rotatedImg.at<uchar>(row, col);
			}
			//算完後要與row平均(防止溢位)
			colAddImg.at<float>(angle, col) = originalColSum / oriRows;
		}
	}

	cv::normalize(colAddImg, colAddImg, 0, 255, NORM_MINMAX, CV_8UC1);
	namedWindow("colAddImg", WINDOW_AUTOSIZE);
	imshow("colAddImg", colAddImg);
	imwrite("colAddImg.png", colAddImg);
	//waitKey(0);
	//注意這裡要使用CV_32FC1才能用at<float>
	cv::normalize(colAddImg, colAddImg, 0, 1, NORM_MINMAX, CV_32FC1);
	
	//ART
	Mat results(oriRows, colAddImg.cols, CV_32FC1, Scalar(0));

	double anglePerPhoto = 11.25;
	int totIter = 100;
	for (int iter = 0; iter < totIter; iter++) {
		for (double angle = 0; angle < 180; angle += anglePerPhoto) {
			for (int col = 0; col < results.cols; col++) {
				float colSum = colAddImg.at<float>(angle, col);
				float newColSum = 0;
				for (int row = 0; row < results.rows; row++) {
					newColSum += results.at<float>(row, col);
				}
				for (int row = 0; row < results.rows; row++) {
					results.at<float>(row, col) += ((colSum - newColSum) / oriRows);
				}
			}
			results = rotate(results, anglePerPhoto);
		}
		results = rotate(results, 180);
	}

	cv::normalize(results, results, 0, 255, NORM_MINMAX, CV_8UC1);

	//OUTPUT RESULT HERE
	namedWindow("result", WINDOW_AUTOSIZE);
	imshow("result", results);
	imwrite("result.png", results);
	waitKey(0);

	return 0;
}

```
