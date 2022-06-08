## 第一课
```
opencv--图像压缩
pyautogui--模拟鼠标键盘
socket--图像传输
tkinter--界面
```
## 第二课
```python
from PIL import ImgeGrab
im=ImageGrab.grab()
im.show()
im.save("E/work")--有损压缩
完成截图
```

压缩方法：png
```
numpy ==1.19.1--实现两张图片做减法
opencv--把压缩后的图像压缩成ppt传输
Pillow==7.2.0
PyAutoGUI==0.9.50
```

```python
import numpy as np
from cv2 import cv2
from PIL import ImageGrab
import time

im1 = ImageGrab.grab()
time.sleep(1)
print( "test")
im2 = ImageGrab.grab()
in1 = np.asarray (im1)
in2 =np.asarray (im2)
in3 = in2 - in1
cv2.imwrite("./1.png", in1)//无损压缩
cv2.imwrite("./2.png", in2)
cv2.imwrite("./sub.png", in3)//差异化传输
```
## 第三课
```python
实现一个远程屏幕共享的功能
socket
opencv
传输步骤
1.截屏
2.定义传输协议
    1.数据长度
    2.数据
#接收步骤
1.协议解析
    1.读4字节->uint32 -数据长度
    2.读取第一步的数据长度的数据
2.图像加法
3.图像显示
```

```
>import keyboard
>>> keyboard. press_and_release(C)
c
>>> import mouse
>>>mouse.get position()
(1502, 9)
>>> x
>>>mouse.move(x[0], x[1])

```


## 第五课
GitHub 中 keyboard 中的_winkeyboard.py

