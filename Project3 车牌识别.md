# Project3 车牌识别



<img src="https://typora3.oss-cn-shanghai.aliyuncs.com/202511222149726.png" alt="image-20251122214948416" style="zoom:25%;" />

这个项目，我们仍然是将网络摄像头作为图像的输入源，使用**级联方法** `Chapter9.py`进行检测车牌

```python
import cv2
import os
# 参数
root_path = r'D:\CALB\CV\opencv\Learn-OpenCV-in-3-hours\turtorial\Learn-OpenCV-in-3-hours\Resources'
nPlateCascade = cv2.CascadeClassifier(os.path.join(root_path, 'haarcascade_russian_plate_number.xml'))
imglist = [os.path.join(root_path, img+'.jpg') for img in ['p1', 'p2', 'p3']]
color = (255, 0, 0)
count = 0

for imgFile in imglist:
    img = cv2.imread(imgFile)
    imgGray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    numberPlates = nPlateCascade.detectMultiScale(imgGray, 1.1, 10)
    for (x, y, w, h) in numberPlates:
        cv2.rectangle(img, (x, y), (x+w, y+h), (0, 255, 0), 2)
        cv2.putText(img,"Number Plate", (x,y-5),
                        cv2.FONT_HERSHEY_COMPLEX_SMALL, 1, color, 2)
        imgRoi = img[y:y + h, x:x + w]
        cv2.imshow("ROI", imgRoi)
    cv2.imshow("Result", img)
    if cv2.waitKey(0) & 0xFF == ord('s'):
        cv2.imwrite(os.path.join(root_path, str(count)+".jpg"), imgRoi)
        # 存储ROI区域图像s
        cv2.rectangle(img, (0, 200), (640, 300), (0, 255, 0), cv2.FILLED)
        cv2.putText(img, "Scan Saved", (150, 265), cv2.FONT_HERSHEY_DUPLEX,
                    2, (0, 0, 255), 2)
        # 表示存储成功
        cv2.imshow("Result", img)
        cv2.waitKey(500)
        count += 1


```

<img src="https://typora3.oss-cn-shanghai.aliyuncs.com/202511222207967.png" alt="image-20251122220711778" style="zoom:25%;" />

按下S键后，会进行保存图像

<img src="https://typora3.oss-cn-shanghai.aliyuncs.com/202511222210990.png" alt="image-20251122221055891" style="zoom:25%;" />