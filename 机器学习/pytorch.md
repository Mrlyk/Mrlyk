# PyTorch

PyTorch 是一个针对深度学习, 并且使用 GPU 和 CPU 来优化的 tensor library (张量库)

> 张量如同数组和矩阵一样, 是一种特殊的数据结构。在`PyTorch`中, 神经网络的输入、输出以及网络的参数等数据, 都是使用张量来进行描述。
>
> 张量的使用和`Numpy`中的`ndarrays`很类似, 区别在于张量可以在`GPU`或其它专用硬件上运行, 这样可以得到更快的加速效果。





## 共享模型

就像 docker 有 docker hub 共享镜像仓库一样。pytorch 也有一个  hub，开源用户在上面上传已经训练好的模型。我们也可以从上面下载我们需要的模型

```python
import torch

# 加载模型 接收一个仓库有地址和要加载的模型，从远程仓库加载模型（下载路径：C:/Users/.cache\torch\hub\master.zip）
# 这个模型是已经预训练过的
model = torch.hub.load('ultralytics/yolov5', 'yolov5s')
```





## 参考文章

中文文档：https://pytorch.apachecn.org/#/docs/1.7/03

