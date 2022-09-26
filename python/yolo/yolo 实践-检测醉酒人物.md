# yolo 实践-检测醉酒人物

首先来看一个简单的检测🌰：

其中模型使用的是 yolo 官方预训练好的，可以检测画面中人、领带、也可以检测汽车、卡车！

```python
import torch  # 用来加载 yolo 模型和做检测c
import matplotlib
from matplotlib import pyplot as plt  # 图形绘制库 用于图片可视化绘制
import numpy as np  # 用于数组转换
import cv2  # 用来访问摄像头和图片，可以在视频流上实时绘制



# 加载模型 接收一个仓库有地址和要加载的模型，从远程仓库加载模型（下载路径：C:/Users/.cache\torch\hub\master.zip）
# 这个模型是已经预训练过的，所以我们在使用 yolov5 的时候实际只要获得其训练后的模型就可以了
model = torch.hub.load('ultralytics/yolov5', 'yolov5s')

# print('查看当前渲染使用的是哪个 gui 引擎', matplotlib.get_backend())
# 默认是 TkAgg 使用 Tkinter（即 tk interface） 是 Python 标准 GUI 库，简称 “Tk”，上面加载模型会把这里重置为 agg（不自带 gui），所以这里重新声明了一遍
matplotlib.use('TkAgg')


# 检测图片
# img = 'https://ultralytics.com/images/zidane.jpg' # 可以远程加载
img = './drowsiness-detection/data/images/zidane.jpg' # 为了速度还是用本地的

# 给模型传入我们要检测图片，只要字符串路径即可
results = model(img)  

# 输出检测结果（这里用 yolov5s 检测出 2 个人，用 yolov5m 确定检测出 3 个，推测是后者识别更强，识别了背景中的人？这点值得注意，有时候我们只需要主画面的）
results.print()
print(results.xyxy) # 可以输出模型检测到的张量坐标


plt.imshow(np.squeeze(results.render())) # 绘制检测结果

plt.show()
```

## 实时检测

上面的🌰检测是禁止的画面，下面通过 opencv 来获取摄像头画面进行实时检测。当然也可以使用 opencv 捕获屏幕对屏幕上的画面进行实时检测！





## 自定义检测目标



- `cv2.waitKey(1000)`：在1000ms内根据键盘输入返回一个值。waitKey() 函数的功能是不断刷新图像 , 频率时间为delay , 单位为ms 。返回值为当前键盘按键值
- `0xFF` ：一个十六进制数
- `ord('q')` ：返回q的ascii码

0xFF是一个十六进制数，转换为二进制是11111111。waitKey返回值的范围为（0-255），刚好也是8个二进制位。那么我们将 cv2.waitKey(1) & 0xFF计算一下，发现结果仍然是 waitKey 的返回值，那为何要多次一举呢？直接 cv2.waitKey(1) == ord('q')不就好了吗。

**实际上在linux上使用waitkey有时会出现waitkey返回值超过了（0-255）的范围的现象**。通过cv2.waitKey(1) & 0xFF运算，当waitkey返回值正常时 cv2.waitKey(1) = cv2.waitKey(1000) & 0xFF,当返回值不正常时，cv2.waitKey(1000) & 0xFF的范围仍不超过（0-255），就避免了一些奇奇怪怪的BUG。

## 模型检测到的张量

上面我们输出了张量`print(results.xyxy) `，结果如下图

```text
[tensor([[7.43291e+02, 4.83438e+01, 1.14176e+03, 7.20000e+02, 8.79861e-01, 0.00000e+00],
        [4.41990e+02, 4.37337e+02, 4.96585e+02, 7.10036e+02, 6.75119e-01, 2.70000e+01],
        [1.23051e+02, 1.93238e+02, 7.14690e+02, 7.19771e+02, 6.66693e-01, 0.00000e+00],
        [9.78990e+02, 3.13579e+02, 1.02530e+03, 4.15526e+02, 2.61517e-01, 2.70000e+01]], device='cuda:0')]
```

其中一个列表的结果从前往后依次指 (e+02 -> * 10^2)

- xmin
- ymin
- xmax
- ymax
- confidence
- class