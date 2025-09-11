## 基本概念
图片的基本数据类型:
> Mat：矩阵类型，能够保存图像。
Point：一个像素点，或者任何类型的“点”。
Scalar：一个四维点类，是许多函数的参数，常用于描述颜色。
Size：同样是一对数据构成的组，一般表示一块区域或图像的宽高，有些时候可以和point互换。
Rect：rectangle，矩形类，拥有Point和Size成员，用于表示一块矩形的区域。
RotatedRect：同上，不过有额外的成员angle用于表示角度。注意这个类的角度系统有些独特，**务必阅读**：[OpenCV中旋转矩形的角度](https://blog.csdn.net/heroacool/article/details/105410202)。
## 常用基本操作(实例代码)
```python
# 读取图片  
image = cv2.imread("opencv_logo.jpg")
# 打印图片的形状（高度，宽度，通道数）  
print(image.shape)
# 显示图片
cv2.imshow("image", image)  
cv2.waitKey() # 让窗口暂停
```
> [!Tips]
> 文件名,路径名避免使用中文

```python
# 颜色通道顺序：BGR  
cv2.imshow("blue", image[:, :, 0])  
cv2.imshow("green", image[:, :, 1])  
cv2.imshow("red", image[:, :, 2])  

# 彩色图片灰度化  
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)  
cv2.imshow("gray", gray)
```
```python
# 裁切图像
crop = image[10:170, 40:200]
```
```python
# 绘制图案
import numpy as np # 需要引入numpy来创建画布
# 创建黑色画布  
image = np.zeros([300, 300, 3], dtype=np.uint8)

# 绘制线段（对象， 起点， 终点， 颜色， 粗细）  
cv2.line(image, (100, 200), (250, 250), (255, 0, 0), 2)  
# 绘制矩形（~，起点， 对角点， 颜色， 粗细）  
cv2.rectangle(image, (30, 100), (60, 150), (0, 255, 0), 2)  
# 绘制圆形（~，圆心， 半径， 颜色， 粗细）  
cv2.circle(image, (150, 100), 20, (0, 0, 255), 3)  
# 绘制字符串（~， 内容， 坐标， 字体格式序号， 缩放系数， 颜色， 粗细， 线条类型序号）  
cv2.putText(image, "hello", (100, 50), 0, 1, (255, 255, 255), 2, 1)
```
```python
# 选择相机
# 获取摄像头设备的指针(设备管理器 -> 照相机)  
capture = cv2.VideoCapture(0)  
ret = True  

# 摄像头的读取是连续不断的，需要循环读取  
while ret:  
ret, frame = capture.read()  
'''  
ret：这是一个布尔值，表示读取操作是否成功。  
如果 ret 为 True，表示成功读取了一帧图像；如果为 False，则表示读取失败，可能是因为视频流结束或者其他错误。  
在处理视频流时，这个返回值通常用于控制循环，直到视频流结束。  
frame：这是一个NumPy数组，代表了从视频捕获对象读取的当前帧。  
这个数组通常是一个三维的，其形状为 (高度, 宽度, 通道数)，其中通道数可以是1（灰度图像）或3（彩色图像，分别对应红、绿、蓝通道）。  
'''  
cv2.imshow("camera", frame)  
key = cv2.waitKey(1) # 等待键盘输入1ms  
if key != -1: # 按任意键跳出循环  
break  
  
capture.release() # 释放指针
```
## 识别算法
```python
# 图片特征提取
# 获取特征点 （对象， 最多的点数， 质量优度水平， 特征点之间的最小距离）  
corners = cv2.goodFeaturesToTrack(gray, 500, 0.1, 10)  
# 标记出每个点  
for corner in corners:  
x, y = corner.ravel()  
cv2.circle(image, (int(x), int(y)), 3, (255, 0, 255), -1)
```
```python
# 模板匹配
# 选取匹配模板  
template = gray[75:105, 235:265]  
  
# 使用标准相关匹配算法——将待检测对象和模板都标准化再来计算匹配度  
match = cv2.matchTemplate(gray, template, cv2.TM_CCOEFF_NORMED)  
locations = np.where(match >= 0.9) # 找出匹配系数大于0.9的匹配点  
  
w, h = template.shape[0:2]  
for p in zip(*locations[::-1]): # 循环遍历每一个匹配点并画出矩形框标记  
x1, y1 = p[0], p[1]  
x2, y2 = x1 + w, y1 + h  
cv2.rectangle(image, (x1, y1), (x2, y2), (0, 255, 0), 2)
```
```python
# 梯度检测
# 使用拉普拉斯算子（检测边缘——梯度剧烈变化处）  
laplacian = cv2.Laplacian(gray, cv2.CV_64F)  
# canny边缘检测（定义边缘为梯度区间）  
# 梯度大于200 -> 变化足够强烈，确定是边缘  
# 梯度小于100 -> 变化较为平缓，确定非边缘  
# 梯度介于二者之间 -> 待定，看其是否与已知的边缘像素相邻  
canny = cv2.Canny(gray, 100, 200)
```
```python
# 三种阈值算法
# 图片灰度二值化  
gray = cv2.imread("ball4.png", cv2.IMREAD_GRAYSCALE)  
ret, binary = cv2.threshold(gray, 10, 255, cv2.THRESH_BINARY)  
# 图片自适应二值化（划分区块二值化，效果更好）  
binary_adaptive = cv2.adaptiveThreshold(  
gray, 255, cv2.ADAPTIVE_THRESH_GAUSSIAN_C, cv2.THRESH_BINARY, 115, 1)  
# 大津算法（基于图片灰度聚类分析，自定义阈值）  
ret1, binary_otsu = cv2.threshold(gray, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)
```
```python
# 腐蚀和膨胀(原图像需要提前二值化)
# 腐蚀和膨胀操作  
erosion = cv2.erode(binary, kernel)  
dilation = cv2.dilate(binary, kernel)
```
## 在树莓派上配置OpenCV
### 虚拟环境及OpenCV安装
https://www.bilibili.com/video/BV1FL4y1K7UB
```shell
# 安装venv
sudo apt install python3-venv
# 创建虚拟环境
python3 -m venv myenv
# 进入虚拟环境sourse 
source ./myenv/bin/activate
```