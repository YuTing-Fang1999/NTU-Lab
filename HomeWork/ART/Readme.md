### 目的
為了檢察PCB瑕疵，我們需要用X-ray重建三維的錫球以檢察內部構造  

### ART技術
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
