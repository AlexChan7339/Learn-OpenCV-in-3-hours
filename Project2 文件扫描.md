# Project2 文件扫描

<img src="https://typora3.oss-cn-shanghai.aliyuncs.com/202511221848833.png" alt="image-20251122184806521" style="zoom:25%;" />

在这个项目中，我们仍然使用网络摄像头`Chapter1.py`作为图像输入源

```python
# Chapter1.py
import cv2
frameWidth = 640
frameHeight = 480
cap = cv2.VideoCapture(1)
cap.set(3, frameWidth)
cap.set(4, frameHeight)
cap.set(10,150)
while True:
    success, img = cap.read()
    cv2.imshow("Result", img)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

<img src="https://typora3.oss-cn-shanghai.aliyuncs.com/202511221852434.png" alt="image-20251122185224276" style="zoom:25%;" />

我们需要对文件进行投影变换，获取其平面图形

首先我们需要对图像进行预处理，来正确检测图形中的边缘，并获取轮廓`Chapter8.py`，需要找到最大且能被我们使用的轮廓



```python
import cv2
import os
widthImg = 640
heightImg = 480
root_path = r"D:\CALB\CV\opencv\Learn-OpenCV-in-3-hours\turtorial\Learn-OpenCV-in-3-hours\Resources"

def preProcessing(img):
    imgGray = cv2.cvtColor(img,cv2.COLOR_BGR2GRAY)
    imgBlur = cv2.GaussianBlur(imgGray,(5,5),1)
    imgCanny = cv2.Canny(imgBlur,200,200)
    kernel = np.ones((5,5))
    # 有时候边缘会非常薄,需要使用膨胀功能使它们变厚
    # 再使用腐蚀函数让边缘再稍微薄一点
    imgDial = cv2.dilate(imgCanny,kernel,iterations=2)
    imgThres = cv2.erode(imgDial,kernel,iterations=1)
    return imgThres

def getContours(img):
    biggest = np.array([])
    maxArea = 0
    contours,hierarchy = cv2.findContours(img,cv2.RETR_EXTERNAL,cv2.CHAIN_APPROX_NONE)
    x,y,w,h = 0,0,0,0 # 放置判断失效，xywh的值会是上一个判断有效的矩形框的值
    for cnt in contours:
        area = cv2.contourArea(cnt)
        if area>5000:
            cv2.drawContours(imgContour, cnt, -1, (255, 0, 0), 3)
            peri = cv2.arcLength(cnt,True)
            approx = cv2.approxPolyDP(cnt, 0.02*peri, True)
            if area > maxArea and len(approx) == 4:
                biggest = approx# 记录4个点坐标
                maxArea = area
    return biggest

    
while True:
    simg = cv2.imread(os.path.join(root_path, '1.jpg'))
    imgContour = img.copy()
    imgThres = preProcessing(img)
    getContours(imgThres)
    cv2.imshow("Result", imgContour)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

<img src="https://typora3.oss-cn-shanghai.aliyuncs.com/202511221940830.png" alt="image-20251122194044663" style="zoom:25%;" />

接下来需要获取文件的俯视图中的四个点

```python
import cv2

import cv2
import os
widthImg = 640
heightImg = 480
root_path = r"D:\CALB\CV\opencv\Learn-OpenCV-in-3-hours\turtorial\Learn-OpenCV-in-3-hours\Resources"

def preProcessing(img):
    imgGray = cv2.cvtColor(img,cv2.COLOR_BGR2GRAY)
    imgBlur = cv2.GaussianBlur(imgGray,(5,5),1)
    imgCanny = cv2.Canny(imgBlur,200,200)
    kernel = np.ones((5,5))
    # 有时候边缘会非常薄,需要使用膨胀功能使它们变厚
    # 再使用腐蚀函数让边缘再稍微薄一点
    imgDial = cv2.dilate(imgCanny,kernel,iterations=2)
    imgThres = cv2.erode(imgDial,kernel,iterations=1)
    return imgThres

def getContours(img):
    biggest = np.array([])
    maxArea = 0
    contours,hierarchy = cv2.findContours(img,cv2.RETR_EXTERNAL,cv2.CHAIN_APPROX_NONE)
    x,y,w,h = 0,0,0,0 # 放置判断失效，xywh的值会是上一个判断有效的矩形框的值
    for cnt in contours:
        area = cv2.contourArea(cnt)
        if area>5000:
            # cv2.drawContours(imgContour, cnt, -1, (255, 0, 0), 3)
            peri = cv2.arcLength(cnt,True)
            approx = cv2.approxPolyDP(cnt, 0.02*peri, True)
            if area > maxArea and len(approx) == 4:
                biggest = approx# 记录4个点坐标
                maxArea = area
    cv2.drawContours(imgContour, biggest, -1, (255, 0, 0), 20)
    return biggest

def getWarp(img,biggest):
    pass


while True:
    img = cv2.imread(os.path.join(root_path, '1.jpg'))
    imgContour = img.copy()
    imgThres = preProcessing(img)
    biggest = getContours(imgThres)
    print(biggest)
    getWarp(img, biggest)
    cv2.imshow("Result", imgContour)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

<img src="https://typora3.oss-cn-shanghai.aliyuncs.com/202511221949177.png" alt="image-20251122194952044" style="zoom:25%;" />

接下来需要获取文件的俯视图(`Charter5.py`)

```python
import cv2

import cv2
import os
widthImg = 640
heightImg = 480
root_path = r"D:\CALB\CV\opencv\Learn-OpenCV-in-3-hours\turtorial\Learn-OpenCV-in-3-hours\Resources"

def preProcessing(img):
    imgGray = cv2.cvtColor(img,cv2.COLOR_BGR2GRAY)
    imgBlur = cv2.GaussianBlur(imgGray,(5,5),1)
    imgCanny = cv2.Canny(imgBlur,200,200)
    kernel = np.ones((5,5))
    # 有时候边缘会非常薄,需要使用膨胀功能使它们变厚
    # 再使用腐蚀函数让边缘再稍微薄一点
    imgDial = cv2.dilate(imgCanny,kernel,iterations=2)
    imgThres = cv2.erode(imgDial,kernel,iterations=1)
    return imgThres

def getContours(img):
    biggest = np.array([])
    maxArea = 0
    contours,hierarchy = cv2.findContours(img,cv2.RETR_EXTERNAL,cv2.CHAIN_APPROX_NONE)
    x,y,w,h = 0,0,0,0 # 放置判断失效，xywh的值会是上一个判断有效的矩形框的值
    for cnt in contours:
        area = cv2.contourArea(cnt)
        if area>5000:
            # cv2.drawContours(imgContour, cnt, -1, (255, 0, 0), 3)
            peri = cv2.arcLength(cnt,True)
            approx = cv2.approxPolyDP(cnt, 0.02*peri, True)
            if area > maxArea and len(approx) == 4:
                biggest = approx# 记录4个点坐标
                maxArea = area
    cv2.drawContours(imgContour, biggest, -1, (255, 0, 0), 20)
    return biggest

def getWarp(img,biggest):

    pts1 = np.float32(biggest)
    pts2 = np.float32([[0, 0], [widthImg, 0], [0, heightImg], [widthImg, heightImg]])
    matrix = cv2.getPerspectiveTransform(pts1, pts2)
    imgOutput = cv2.warpPerspective(img, matrix, (widthImg, heightImg))
    return imgOutput

while True:
    img = cv2.imread(os.path.join(root_path, '1.jpg'))
    imgContour = img.copy()
    imgThres = preProcessing(img)
    biggest = getContours(imgThres)
    print(biggest)
    imgWarped = getWarp(img, biggest)
    cv2.imshow("Result", imgContour)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

<img src="https://typora3.oss-cn-shanghai.aliyuncs.com/202511221959514.png" alt="image-20251122195900412" style="zoom:25%;" />

此时会发现视角扭曲，原因在于pts1中点的排列不是根据左上--右上--左下--右下的顺序进行排列，因此我们需要对pts1中的值进行排列

```python
import cv2

import cv2
import os
widthImg = 640
heightImg = 480
root_path = r"D:\CALB\CV\opencv\Learn-OpenCV-in-3-hours\turtorial\Learn-OpenCV-in-3-hours\Resources"

def preProcessing(img):
    imgGray = cv2.cvtColor(img,cv2.COLOR_BGR2GRAY)
    imgBlur = cv2.GaussianBlur(imgGray,(5,5),1)
    imgCanny = cv2.Canny(imgBlur,200,200)
    kernel = np.ones((5,5))
    # 有时候边缘会非常薄,需要使用膨胀功能使它们变厚
    # 再使用腐蚀函数让边缘再稍微薄一点
    imgDial = cv2.dilate(imgCanny,kernel,iterations=2)
    imgThres = cv2.erode(imgDial,kernel,iterations=1)
    return imgThres

def getContours(img):
    biggest = np.array([])
    maxArea = 0
    contours,hierarchy = cv2.findContours(img,cv2.RETR_EXTERNAL,cv2.CHAIN_APPROX_NONE)
    x,y,w,h = 0,0,0,0 # 放置判断失效，xywh的值会是上一个判断有效的矩形框的值
    for cnt in contours:
        area = cv2.contourArea(cnt)
        if area>5000:
            # cv2.drawContours(imgContour, cnt, -1, (255, 0, 0), 3)
            peri = cv2.arcLength(cnt,True)
            approx = cv2.approxPolyDP(cnt, 0.02*peri, True)
            if area > maxArea and len(approx) == 4:
                biggest = approx# 记录4个点坐标
                maxArea = area
    cv2.drawContours(imgContour, biggest, -1, (255, 0, 0), 20)
    return biggest

def reorder(myPoints):
    # pts2中[0,0]加起来是四个中最小的，[width,height]加起来是四个中最大的，[width,0]相减是正数，[0,height]相减是负数
    myPoints = myPoints.reshape((4,2))
    myPointsNew = np.zeros((4,1,2),np.int32)# 初始化新矩阵
    add = myPoints.sum(1)# 每个点的x坐标+y坐标，从axis=1方向上进行累加
    # print("add", add)
    myPointsNew[0] = myPoints[np.argmin(add)]
    myPointsNew[3] = myPoints[np.argmax(add)]
    diff = np.diff(myPoints,axis=1)
    # 后一列减去千亿列
    myPointsNew[1]= myPoints[np.argmin(diff)]
    myPointsNew[2] = myPoints[np.argmax(diff)]
    # print("NewPoints",myPointsNew)
    return myPointsNew


def getWarp(img,biggest):
    # print(biggest.shape)
    # (4, 1, 2),需要将1进行去除
    biggest = reorder(biggest)
    pts1 = np.float32(biggest)
    pts1 = pts1.reshape((4, 2))
    pts2 = np.float32([[0, 0], [widthImg, 0], [0, heightImg], [widthImg, heightImg]])
    matrix = cv2.getPerspectiveTransform(pts1, pts2)
    imgOutput = cv2.warpPerspective(img, matrix, (widthImg, heightImg))
    return imgOutput

while True:
    img = cv2.imread(os.path.join(root_path, '1.jpg'))
    imgContour = img.copy()
    imgThres = preProcessing(img)
    biggest = getContours(imgThres)
    print(biggest)
    imgWarped = getWarp(img, biggest)
    cv2.imshow("Result", imgWarped)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

<img src="https://typora3.oss-cn-shanghai.aliyuncs.com/202511222046926.png" alt="image-20251122204628823" style="zoom:25%;" />

我们能看到文件按的上面和坐标存在灰色的边缘，我们可以用resize对其进行裁剪(4个角各裁剪 20个像素)；再将处理的图和原图stack（`Chapter6.py`）在一起进行输出

```python
import cv2

import cv2
import os
widthImg = 640
heightImg = 480
root_path = r"D:\CALB\CV\opencv\Learn-OpenCV-in-3-hours\turtorial\Learn-OpenCV-in-3-hours\Resources"

def preProcessing(img):
    imgGray = cv2.cvtColor(img,cv2.COLOR_BGR2GRAY)
    imgBlur = cv2.GaussianBlur(imgGray,(5,5),1)
    imgCanny = cv2.Canny(imgBlur,200,200)
    kernel = np.ones((5,5))
    # 有时候边缘会非常薄,需要使用膨胀功能使它们变厚
    # 再使用腐蚀函数让边缘再稍微薄一点
    imgDial = cv2.dilate(imgCanny,kernel,iterations=2)
    imgThres = cv2.erode(imgDial,kernel,iterations=1)
    return imgThres

def getContours(img):
    biggest = np.array([])
    maxArea = 0
    contours,hierarchy = cv2.findContours(img,cv2.RETR_EXTERNAL,cv2.CHAIN_APPROX_NONE)
    x,y,w,h = 0,0,0,0 # 放置判断失效，xywh的值会是上一个判断有效的矩形框的值
    for cnt in contours:
        area = cv2.contourArea(cnt)
        if area>5000:
            # cv2.drawContours(imgContour, cnt, -1, (255, 0, 0), 3)
            peri = cv2.arcLength(cnt,True)
            approx = cv2.approxPolyDP(cnt, 0.02*peri, True)
            if area > maxArea and len(approx) == 4:
                biggest = approx# 记录4个点坐标
                maxArea = area
    cv2.drawContours(imgContour, biggest, -1, (255, 0, 0), 20)
    # 绘制出 4 个孤立的点（或零散线段），无法形成完整轮廓，它是将每个点当作一个轮廓， 但 1 个点无法构成轮廓；最终函数尝试绘制 4 个 “仅含 1 个点的轮廓”，结果就是在 4 个点的位置画粗点（或零散线段），完全无法形成矩形。
    # 如果是cv2.drawContours(imgContour, [biggest], -1, (255, 0, 0), 20)：正确绘制出四边形的完整轮廓（线宽 20 的蓝色边框）
    return biggest

def reorder(myPoints):
    # pts2中[0,0]加起来是四个中最小的，[width,height]加起来是四个中最大的，[width,0]相减是正数，[0,height]相减是负数
    myPoints = myPoints.reshape((4,2))
    myPointsNew = np.zeros((4,1,2),np.int32)# 初始化新矩阵
    add = myPoints.sum(1)# 每个点的x坐标+y坐标，从axis=1方向上进行累加
    # print("add", add)
    myPointsNew[0] = myPoints[np.argmin(add)]
    myPointsNew[3] = myPoints[np.argmax(add)]
    diff = np.diff(myPoints,axis=1)
    myPointsNew[1]= myPoints[np.argmin(diff)]
    myPointsNew[2] = myPoints[np.argmax(diff)]
    # print("NewPoints",myPointsNew)
    return myPointsNew


def getWarp(img,biggest):
    # print(biggest.shape)
    # (4, 1, 2),需要将1进行去除
    biggest = reorder(biggest)
    pts1 = np.float32(biggest)
    pts1 = pts1.reshape((4, 2))
    pts2 = np.float32([[0, 0], [widthImg, 0], [0, heightImg], [widthImg, heightImg]])
    matrix = cv2.getPerspectiveTransform(pts1, pts2)
    imgOutput = cv2.warpPerspective(img, matrix, (widthImg, heightImg))
    imgCropped = imgOutput[20:imgOutput.shape[0]-20,20:imgOutput.shape[1]-20]
    imgCropped = cv2.resize(imgCropped,(widthImg,heightImg))
    return imgCropped

def stackImages(scale,imgArray):
    rows = len(imgArray)
    cols = len(imgArray[0])
    rowsAvailable = isinstance(imgArray[0], list)
    width = imgArray[0][0].shape[1]
    height = imgArray[0][0].shape[0]
    if rowsAvailable:
        for x in range ( 0, rows):
            for y in range(0, cols):
                if imgArray[x][y].shape[:2] == imgArray[0][0].shape [:2]:
                    imgArray[x][y] = cv2.resize(imgArray[x][y], (0, 0), None, scale, scale)
                else:
                    imgArray[x][y] = cv2.resize(imgArray[x][y], (imgArray[0][0].shape[1], imgArray[0][0].shape[0]), None, scale, scale)
                if len(imgArray[x][y].shape) == 2: imgArray[x][y]= cv2.cvtColor( imgArray[x][y], cv2.COLOR_GRAY2BGR)
        imageBlank = np.zeros((height, width, 3), np.uint8)
        hor = [imageBlank]*rows
        hor_con = [imageBlank]*rows
        for x in range(0, rows):
            hor[x] = np.hstack(imgArray[x])
        ver = np.vstack(hor)
    else:
        for x in range(0, rows):
            if imgArray[x].shape[:2] == imgArray[0].shape[:2]:
                imgArray[x] = cv2.resize(imgArray[x], (0, 0), None, scale, scale)
            else:
                imgArray[x] = cv2.resize(imgArray[x], (imgArray[0].shape[1], imgArray[0].shape[0]), None,scale, scale)
            if len(imgArray[x].shape) == 2: imgArray[x] = cv2.cvtColor(imgArray[x], cv2.COLOR_GRAY2BGR)
        hor= np.hstack(imgArray)
        ver = hor
    return ver

while True:
    img = cv2.imread(os.path.join(root_path, '1.jpg'))
    imgContour = img.copy()
    imgThres = preProcessing(img)
    biggest = getContours(imgThres)
    print(biggest)
    imgWarped = getWarp(img, biggest)
    imageArray = ([img, imgThres],
                  [imgContour, imgWarped])
    stackedImages = stackImages(0.6,imageArray)
    cv2.imshow("Result", stackedImages)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

<img src="https://typora3.oss-cn-shanghai.aliyuncs.com/202511222138967.png" alt="image-20251122213804797" style="zoom:25%;" />

当摄像头里检测不到轮廓时，会报错，因此我们需要把这个情况考虑在内

```python
import cv2

import cv2
import os
widthImg = 640
heightImg = 480
root_path = r"D:\CALB\CV\opencv\Learn-OpenCV-in-3-hours\turtorial\Learn-OpenCV-in-3-hours\Resources"

def preProcessing(img):
    imgGray = cv2.cvtColor(img,cv2.COLOR_BGR2GRAY)
    imgBlur = cv2.GaussianBlur(imgGray,(5,5),1)
    imgCanny = cv2.Canny(imgBlur,200,200)
    kernel = np.ones((5,5))
    # 有时候边缘会非常薄,需要使用膨胀功能使它们变厚
    # 再使用腐蚀函数让边缘再稍微薄一点
    imgDial = cv2.dilate(imgCanny,kernel,iterations=2)
    imgThres = cv2.erode(imgDial,kernel,iterations=1)
    return imgThres

def getContours(img):
    biggest = np.array([])
    maxArea = 0
    contours,hierarchy = cv2.findContours(img,cv2.RETR_EXTERNAL,cv2.CHAIN_APPROX_NONE)
    x,y,w,h = 0,0,0,0 # 放置判断失效，xywh的值会是上一个判断有效的矩形框的值
    for cnt in contours:
        area = cv2.contourArea(cnt)
        if area>5000:
            # cv2.drawContours(imgContour, cnt, -1, (255, 0, 0), 3)
            peri = cv2.arcLength(cnt,True)
            approx = cv2.approxPolyDP(cnt, 0.02*peri, True)
            if area > maxArea and len(approx) == 4:
                biggest = approx# 记录4个点坐标
                maxArea = area
    cv2.drawContours(imgContour, biggest, -1, (255, 0, 0), 20)
    return biggest

def reorder(myPoints):
    # pts2中[0,0]加起来是四个中最小的，[width,height]加起来是四个中最大的，[width,0]相减是正数，[0,height]相减是负数
    myPoints = myPoints.reshape((4,2))
    myPointsNew = np.zeros((4,1,2),np.int32)# 初始化新矩阵
    add = myPoints.sum(1)# 每个点的x坐标+y坐标，从axis=1方向上进行累加
    # print("add", add)
    myPointsNew[0] = myPoints[np.argmin(add)]
    myPointsNew[3] = myPoints[np.argmax(add)]
    diff = np.diff(myPoints,axis=1)
    myPointsNew[1]= myPoints[np.argmin(diff)]
    myPointsNew[2] = myPoints[np.argmax(diff)]
    # print("NewPoints",myPointsNew)
    return myPointsNew


def getWarp(img,biggest):
    # print(biggest.shape)
    # (4, 1, 2),需要将1进行去除
    biggest = reorder(biggest)
    pts1 = np.float32(biggest)
    pts1 = pts1.reshape((4, 2))
    pts2 = np.float32([[0, 0], [widthImg, 0], [0, heightImg], [widthImg, heightImg]])
    matrix = cv2.getPerspectiveTransform(pts1, pts2)
    imgOutput = cv2.warpPerspective(img, matrix, (widthImg, heightImg))
    imgCropped = imgOutput[20:imgOutput.shape[0]-20,20:imgOutput.shape[1]-20]
    imgCropped = cv2.resize(imgCropped,(widthImg,heightImg))
    return imgCropped

def stackImages(scale,imgArray):
    rows = len(imgArray)
    cols = len(imgArray[0])
    rowsAvailable = isinstance(imgArray[0], list)
    width = imgArray[0][0].shape[1]
    height = imgArray[0][0].shape[0]
    if rowsAvailable:
        for x in range ( 0, rows):
            for y in range(0, cols):
                if imgArray[x][y].shape[:2] == imgArray[0][0].shape [:2]:
                    imgArray[x][y] = cv2.resize(imgArray[x][y], (0, 0), None, scale, scale)
                else:
                    imgArray[x][y] = cv2.resize(imgArray[x][y], (imgArray[0][0].shape[1], imgArray[0][0].shape[0]), None, scale, scale)
                if len(imgArray[x][y].shape) == 2: imgArray[x][y]= cv2.cvtColor( imgArray[x][y], cv2.COLOR_GRAY2BGR)
        imageBlank = np.zeros((height, width, 3), np.uint8)
        hor = [imageBlank]*rows
        hor_con = [imageBlank]*rows
        for x in range(0, rows):
            hor[x] = np.hstack(imgArray[x])
        ver = np.vstack(hor)
    else:
        for x in range(0, rows):
            if imgArray[x].shape[:2] == imgArray[0].shape[:2]:
                imgArray[x] = cv2.resize(imgArray[x], (0, 0), None, scale, scale)
            else:
                imgArray[x] = cv2.resize(imgArray[x], (imgArray[0].shape[1], imgArray[0].shape[0]), None,scale, scale)
            if len(imgArray[x].shape) == 2: imgArray[x] = cv2.cvtColor(imgArray[x], cv2.COLOR_GRAY2BGR)
        hor= np.hstack(imgArray)
        ver = hor
    return ver

while True:
    img = cv2.imread(os.path.join(root_path, '1.jpg'))
    imgContour = img.copy()
    imgThres = preProcessing(img)
    biggest = getContours(imgThres)
    # print(biggest)
    if biggest.size != 0:
        imgWarped=getWarp(img,biggest)
        imageArray = ([img, imgThres],
                  [imgContour, imgWarped])
    else:
        imageArray = ([img, imgThres],
                  [img, img])

    stackedImages = stackImages(0.6,imageArray)
    cv2.imshow("Result", stackedImages)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

<img src="https://typora3.oss-cn-shanghai.aliyuncs.com/202511222144586.png" alt="image-20251122214448393" style="zoom:25%;" />