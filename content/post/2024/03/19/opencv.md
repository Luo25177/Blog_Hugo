---
title: "opencv图像处理"
date: 2024-03-19 09:53:24
categories: "图像处理"
tags: ["python", "图像处理"]
summary: "opencv图像处理（python版）"
---

## 程序说明

### 图片转灰度图

这个操作可以在一开始读取照片时就完成，也可以自己根据每个像素点，把每个像素点的r,b,g三个参数调成一样的，也就能形成灰度图

```python
def imread(filename: str, flags: int = ...)

```

这里的flags能选择读入图片的形式：0是灰度图，1是彩色图

```python
for i in range(img.shape[0]):
    for j in range(img.shape[1]):
        point = [i,j]
        (b,g,r) = img[i,j]
        b = int(b)
        g = int(g)
        r = int(r)
        # gray = 255
        gray = (b + g + r)/3  # 第一种灰度化方法
        # gray = r*0.299 + g*0.587 + b*0.114   # 第二种灰度化方法
        img[i,j] = np.array([gray,gray,gray],dtype = np.uint8)

```

这个就是把每个像素点上的r,g,b改为相同的数值
在图片读入后，每个图片的shape数据可能含有不同的参数数量：
比如在彩色图中就是（height，width，3）表示高度和宽度和3个色彩通道

### HSV图

```python
cvtColor(src: Mat, code: int, dts: Mat = ..., dstCn: int = ...)

```

在这个函数中，src是传入的图片，数据类型为int，dst是输出的图片，dstCn就是输出图片的数据类型

## 噪声

### 高斯噪声

```python
def gaussian_noise(img):
    for i in range(0,img.shape[0]):
        for j in range(0,img.shape[1]):
            s = np.random.normal(0,24,3)
            (b,g,r) = img[i,j]
            img[i,j] = (clamp(b+s[0]),clamp(g+s[1]),clamp(r+s[2]))
    return img

```

```python
def gaussian_noise(img):
    for i in range(0,img.shape[0]):
        for j in range(0,img.shape[1]):
            s = np.random.normal(0,24,3)
            (b,g,r) = img[i,j]
            img[i,j] = (clamp(b+s[0]),clamp(g+s[1]),clamp(r+s[2]))
    return img

```

其实高斯噪声就是图片上的每个像素点都不同程度的发白，也就是每个像素点上的r，b，g都会不同程度的增大，增大的程度满足于正态分布

### 椒盐噪声

```python
def sp_noise(img, prog):
    ran = prog * img.shape[0] *img.shape[1]
    for n in range(0,int(ran)):
        i = np.random.randint(0,img.shape[0])
        j = np.random.randint(0,img.shape[1])
        if np.random.randint(0 ,2):
            img[i,j] = 255
        else:
            img[i,j] = 0
    return img

```

椒盐噪声就是图片上一些随机的点随机的变为黑色或者白色，这样也使得照片看起来就像被撒上了一把椒盐
函数中的prog就是生成椒盐噪声的点的比率

## 去噪

### 高斯去噪

```python
GaussianBlur(src: Mat, # 要处理的图片
ksize,     # 卷积核的大小
sigmaX,
dts: Mat )  # 输出图像

```

去噪原理就是在卷积核矩阵内运用权重

### 中值去噪

```python
medianBlur(src: Mat, ksize, d: Mat = ...)  # kesize是中值检测的范围大小

```

中值去噪就是在某一像素点一定范围内对所有的像素点的rgb值排序，然后取中值

### 均值去噪

```python
blur(src: Mat, ksize, dts: Mat = ) # kesize是检测的范围大小

```

均值去噪就是在某一像素点一定范围内对像素点取平均值，然后赋值给该像素点的rbg值

## 图像的轮廓提取

### 膨胀

```python
erode(imggray,(3,3).dst:Mat =)

```

函数中第一个参数是输入的图片,最后一个参数表示输出的图片，中间的(3,3)表示每个像素点膨胀的大小

### 腐蚀

```python
dilate(src: Mat, kernel, dts: Mat)

```

函数的第一个参数为输入的图片，第二个参数kernel是腐蚀的大小，最后一个参数是输出的图片

### 轮廓提取

```python
morphologyEx(img, cv2.MORPH_GRADIENT, (3,3))

```

这是第一种方法，第一个参数是传入的图片，第二个参数是选择模式，选择gradient就是梯度运算，也就是轮廓提取，如果选择0的话就是腐蚀，选择1就是膨胀，其他的就是原图

```python
imggray_2=cv2.Laplacian(imggray,cv2.CV_64F)
imggray_l=cv2.convertScaleAbs(imggray_2)

```

这是第二种方法，也是梯度运算来提取图像轮廓的

## 图像检测

### 圆形检测

```python
circles = c.HoughCircles(img,  # 输入图像
    c.HOUGH_GRADIENT,  # 使用的检测方法
    1,  # 原图与输出的分辨率之比
    50,  # 圆心之间最小间距
    param1=50,  # 高阈值
    param2=40,  # 越大检测到的圆越完美
    minRadius=50, # 最小的的圆的半径
    maxRadius=0)  # 最大的半径

```

圆形检测就是检测图像中的圆形，并且返回值为一个数列的数列，每个数列里包含着三个参数，前两个参数是圆形的坐标，第三个参数是圆形的半径大小

### 直线检测

```python
lines = c.HoughLinesP(img,
    0.5,     # 最小尺寸步长
    np.pi/480,     # 最小角度步长
    30,  # 阈值，越小判定出的直线越多
    minLineLength=100,  # 最小线长
    maxLineGap=10)  # 非共线直线的最大距离

```

直线检测就是检测直线的函数，返回值是一个数列的数列，每个数列里含有四个参数，前两个是直线的起点坐标，后两个是直线的终点坐标

## 特征点检测

### harries角点检测

```python
dst = cv2.cornerHarris(gray, # 传入图像
2, # 邻域大小
3,
0.04)
dst = cv2.dilate(dst,None)
img[dst>0.01*dst.max()]=[0,0,255]  # 将角点绘制到原图上

void cvCornerHarris( const CvArr* image,  # 输入图像
CvArr* harris_responce,  # Harris检测responces的图像。与输入图像等大。
int block_size,   # 邻域大小
int aperture_size=3,   # 扩展 Sobel 核的大小
double k=0.04 )

```

角点检测就是把图像中的角点给标注出来

### sift特征点提取

```python
sift = cv2.SIFT_create()     # 创造sift算子
kp, des= sift.detectAndCompute(gray, None)   # 从灰度图例提取特征keypoint
img_2 = cv2.drawKeypoints(img,kp,(0,255,255)) # 在图像中画上特征点
cv2.imshow("as",img_2)

```

sift特征点提取需要先定义一个sift算子，然后只能对灰度图处理，从灰度图里提取特征点的数据，最后画在原图上就可以

### orb特征点提取

```python
orb = cv2.ORB_create() # 定义一个orb算子
kp = orb.detect(gray, None)   # 从灰度图中提取信息
img_3 =cv2.drawKeypoints(img,kp,(0,0,255)) # 最后把特征点画到原图上

```

orb特征点提取跟sift相似，也是需要先创建一个算子，然后再从灰度图里提取数据，最后画在原图上

### BFMATCH特征点匹配

```python
sift = cv2.SIFT_create()
kp_1, des_1= sift.detectAndCompute(gray_1, None)
kp_2, des_2 = sift.detectAndCompute(gray_2, None)
bf = cv2.BFMatcher()   # 创造一个BF匹配算子
matches = bf.knnMatch(des_1, des_2, k=2)   # 从两张图片上的des数据提取相似特征点的信息
good = []
for m,n in matches:
    if m.distance < 0.8*n.distance:
        good.append(m)                # 对提取的相似匹配特征点进行筛选，
res = cv2.drawMatches(img_1, kp_1, img_2, kp_2, good, None, flags=2)  # 最后画出特征点
cv2.imshow("as",res)
cv2.waitKey(0)
cv2.destroyAllWindows()

```

特征点匹配就是把两张图片上的特征点分别提取出来，然后再找相似的特征点，随后筛选出比较好的，匹配度高的特征点，最后在画在图上

## 学习记录

### 图像

在每张图片上，每个像素点的数据中都包括三个参数rgb，表示这个像素点的颜色

### 转灰度图

这个操作可以在一开始读取照片时就完成，也可以自己根据每个像素点，把每个像素点的r,b,g三个参数调成一样的，也就能形成灰度图
这个就是把每个像素点上的r,g,b改为相同的数值
在图片读入后，每个图片的shape数据可能含有不同的参数数量：
比如在彩色图中就是（height，width，3）表示高度和宽度和3个色彩通道

### HSV图特点

一种颜色空间, 也称六角锥体模型 (Hexcone Model)。 这个模型中颜色的参数分别是：色调（H），饱和度（S），明度（V）。

### 高斯噪声特点

其实高斯噪声就是图片上的每个像素点都不同程度的发白，也就是每个像素点上的r，b，g都会不同程度的增大，增大的程度满足于正态分布

### 椒盐噪声特点

椒盐噪声就是图片上一些随机的点随机的变为黑色或者白色，这样也使得照片看起来就像被撒上了一把椒盐

### 高斯去噪原理

去噪原理就是在卷积核矩阵内运用一定的权重计算来获得rgb的值

### 中值去噪原理

中值去噪就是在某一像素点一定范围内对所有的像素点的rgb值排序，然后取中值

### 均值去噪原理

均值去噪就是在某一像素点一定范围内对像素点取平均值，然后赋值给该像素点的rbg值

### 圆形检测原理

就是通过分析灰度图像来检测图像中的圆形
圆形检测就是检测图像中的圆形，并且返回值为一个数列的数列，每个数列里包含着三个参数，前两个参数是圆形的坐标，第三个参数是圆形的半径大小

### 直线检测原理

直线检测也是分析灰度图
直线检测就是检测直线的函数，返回值是一个数列的数列，每个数列里含有四个参数，前两个是直线的起点坐标，后两个是直线的终点坐标

### harries 角点检测

检测的是角点，再检测图像中，如果选取的是周围颜色一致的点，那么在这个点周围是没有特点的，而在直线上，沿着直线行进时也是看不出来的，只有在角点上，只要图框移动，都能很清楚看出来，所以要检测角点。

### sift 特征点提取

是一种检测局部特征算法，该算法通过求一幅图中的特征点及其有关scale和orientation的描述子来得到特征
SIFT所查找到的关键点是一些十分突出，不会因光照，仿射变换和噪音等因素而变化的点，如角点、边缘点、暗区的亮点及亮区的暗点等

### orb 特征点检测

ORB算法分为两部分，分别是特征点提取和特征点描述

### 特征点匹配

一个图像的特征点由两部分构成：关键点（Keypoint）和描述子（Descriptor）
关键点指的是该特征点在图像中的位置，有些还具有方向、尺度信息
描述子通常是一个向量，按照人为的设计的方式，描述关键点周围像素的信息，通常描述子是按照外观相似的特征应该有相似的描述子设计的
首先提取图像中的关键点，这部分是查找图像中具有某些特征（不同的算法有不同的）的像素，然后根据得到的关键点位置，计算特征点的描述子，最后根据特征点的描述子，进行匹配