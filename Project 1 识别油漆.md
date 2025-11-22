# Project 1 识别油漆

在这个项目中，我们 需要使用网络摄像头（使用`Chapter1` 的代码）找到油漆颜色

```python
# Chapter1.py
import cv2
frameWidth = 640
frameHeight = 480
# 定义视频帧的宽度（frameWidth=640）和高度（frameHeight=480），后续用于设置摄像头采集画面的分辨率。
cap = cv2.VideoCapture(1)# 设置设备号
# 0：默认摄像头，如笔记本自带的
# 1：外接摄像头（如 USB 摄像头）
cap.set(3, frameWidth)
cap.set(4, frameHeight)
cap.set(10,150)# 设置亮度
while True:
    # 获取图像
    success, img = cap.read()
    cv2.imshow("Result", img)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

- `cap.set(propId, value)`：用于设置摄像头的属性，`propId`是属性编号(property ID)
  - `3`：对应 `cv2.CAP_PROP_FRAME_WIDTH`，设置帧宽度为 `frameWidth=640`；
  - `4`：对应 `cv2.CAP_PROP_FRAME_HEIGHT`，设置帧高度为 `frameHeight=480`；
  - `10`：对应 `cv2.CAP_PROP_BRIGHTNESS`，设置摄像头亮度为 `150`（亮度值范围通常为 0~255）。

要找到颜色，就需要引入颜色检测的代码（转化为HSV空间）



```python
import cv2
import numpy as np
frameWidth = 640
frameHeight = 480
# 设置框架的宽度和高度
cap = cv2.VideoCapture(1)# 设置设备号
# 0：默认摄像头，笔记本自带的
cap.set(3, frameWidth)
cap.set(4, frameHeight)
cap.set(10,150)# 设置亮度
myColors = [[5,107,0,19,255,255],	
            # orage 对应的HSV lower& upper:[Hus Min, Sat Min, Val Min, Hue Max,  Sat Max, Val Max]
            [133,56,0,159,156,255],
            [57,76,0,100,255,255],
            ]		# 要检测的颜色列表


def findColor(img, myColors):
    imgHSV = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
    for color in myColors:
        # 检测列表中的每个颜色
        lower = np.array(myColors[0:3])
        upper = np.array(myColors[3:6])
        mask = cv2.inRange(imgHSV,lower,upper)
        cv2.imshow("str(color[0])", mask)
        # 弹出3个mask窗口，对应3个颜色的检测窗口
    


while True:
    # 获取图像
    success, img = cap.read()
    findColor(img, myColors)
    cv2.imshow("Result", img)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

<img src="https://typora3.oss-cn-shanghai.aliyuncs.com/202511221609514.png" alt="image-20251122160910301" style="zoom:25%;" />

接下来需要找到物体的轮廓（`Chapter8.py`），并确定他的边界框，即确定他的位置

```python
import cv2
import numpy as np
frameWidth = 640
frameHeight = 480
# 设置框架的宽度和高度
cap = cv2.VideoCapture(1)# 设置设备号
# 0：默认摄像头，笔记本自带的
cap.set(3, frameWidth)
cap.set(4, frameHeight)
cap.set(10,150)# 设置亮度
myColors = [[5,107,0,19,255,255],	
            # orage 对应的HSV lower& upper:[Hus Min, Sat Min, Val Min, Hue Max,  Sat Max, Val Max]
            [133,56,0,159,156,255],
            [57,76,0,100,255,255],
            ]		# 要检测的颜色列表


def findColor(img, myColors):
    imgHSV = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
    for color in myColors:
        # 检测列表中的每个颜色
        lower = np.array(myColors[0:3])
        upper = np.array(myColors[3:6])
        mask = cv2.inRange(imgHSV,lower,upper)
        x, y = getContours(mask)# 只对颜色区域绘制Contour
        # 在点旁边绘制circle
        cv2.circle(imgResult,(x,y),15, (255, 0, 0),cv2.FILLED)
        cv2.imshow("str(color[0])", mask)
        # 弹出3个mask窗口，对应3个颜色的检测窗口
    
def getContours(img):
    contours,hierarchy = cv2.findContours(img,cv2.RETR_EXTERNAL,cv2.CHAIN_APPROX_NONE)
    x,y,w,h = 0,0,0,0 # 放置判断失效，xywh的值会是上一个判断有效的矩形框的值
    for cnt in contours:
        area = cv2.contourArea(cnt)
        if area>500:
            cv2.drawContours(imgResult, cnt, -1, (255, 0, 0), 3)
            peri = cv2.arcLength(cnt,True)
            approx = cv2.approxPolyDP(cnt, 0.02*peri, True)
            x, y, w, h = cv2.boundingRect(approx)
    return x+w//2,y
	# 返回笔尖的位置

while True:
    # 获取图像
    success, img = cap.read()
    imgResult = img.copy()
    findColor(img, myColors)
    cv2.imshow("Result", imgResult)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

<img src="https://typora3.oss-cn-shanghai.aliyuncs.com/202511221653229.png" alt="image-20251122165343090" style="zoom:25%;" />

> 由于笔倾斜角度的不同，点的位置会有所偏移（除非保证笔竖直）
>
> <img src="https://typora3.oss-cn-shanghai.aliyuncs.com/202511221655855.png" alt="image-20251122165534731" style="zoom:25%;" />

需要改变contour的颜色，将其改为我们正在检测的颜色

```python
import cv2
import numpy as np
frameWidth = 640
frameHeight = 480
# 设置框架的宽度和高度
cap = cv2.VideoCapture(1)# 设置设备号
# 0：默认摄像头，笔记本自带的
cap.set(3, frameWidth)
cap.set(4, frameHeight)
cap.set(10,150)# 设置亮度
myColors = [[5,107,0,19,255,255],	
            # orage 对应的HSV lower& upper:[Hus Min, Sat Min, Val Min, Hue Max,  Sat Max, Val Max]
            [133,56,0,159,156,255],
            [57,76,0,100,255,255],
            ]		# 要检测的颜色列表
myColorValues = [[51,153,255],          # orange BGR
                 [255,0,255],		# purple BGR
                 [0,255,0],			# green BGR
                 ]
myPoints = []  ## [x , y , colorId ]
# colorId=1 代表orange color

def findColor(img, myColors, myColorValues):
    imgHSV = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
    count = 0 # 作为myColorValues中的值： 从list中依次获取颜色
    newPoints = []
    for color in myColors:
        # 检测列表中的每个颜色
        lower = np.array(myColors[0:3])
        upper = np.array(myColors[3:6])
        mask = cv2.inRange(imgHSV,lower,upper)
        x, y = getContours(mask)# 只对颜色区域绘制Contour
        # 在点旁边绘制circle
        cv2.circle(imgResult,(x,y),15,myColorValues[count],cv2.FILLED)
        if x != 0 and y != 0:
            # 排除被错误地检测（x,y,w,h = 0,0,0,0）的情况
            newpoints.append([x, y, count])
        cv2.imshow("str(color[0])", mask)
        # 弹出3个mask窗口，对应3个颜色的检测窗口
    return newPoints
    
def getContours(img):
    contours,hierarchy = cv2.findContours(img,cv2.RETR_EXTERNAL,cv2.CHAIN_APPROX_NONE)
    x,y,w,h = 0,0,0,0 # 放置判断失效，xywh的值会是上一个判断有效的矩形框的值
    for cnt in contours:
        area = cv2.contourArea(cnt)
        if area>500:
            # cv2.drawContours(imgResult, cnt, -1, (255, 0, 0), 3)
            peri = cv2.arcLength(cnt,True)
            approx = cv2.approxPolyDP(cnt, 0.02*peri, True)
            x, y, w, h = cv2.boundingRect(approx)
    return x+w//2,y
	# 返回笔尖的位置

def drawOnCanvas(myPoints,myColorValues):
    for point in myPoints:
        cv2.circle(imgResult, (point[0], point[1]), 10, myColorValues[point[2]], cv2.FILLED)
        
while True:
    # 获取图像
    success, img = cap.read()
    imgResult = img.copy()
    newPoints = findColor(img, myColors, myColorValues)
    if len(newPoints)!=0:
        for newP in newPoints:
            # 不能在一个列表里面放另一个列表,否则drawOnCanvas中的point[0]就不是个数，而是tuple，此时无法绘制点
            myPoints.append(newP)
    if len(myPoints)!=0:
        drawOnCanvas(myPoints, myColorValues)
    cv2.imshow("Result", imgResult)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

> 可以在 `rapidtables.com/web/color/RGB_Color.html`  找到`myColorValues`中的BGR值
>

![image-20251122180657207](https://typora3.oss-cn-shanghai.aliyuncs.com/202511221807937.png)