# yolo 入门

> yolo：You Only Look Once。是一个基于深度学习目标检测与识别方法库！

这里由个人兴趣入门一下该工具！

官方文档：https://github.com/ultralytics/yolov5/blob/master/.github/README_cn.md

## 安装

首先需要把库拉下来

```
git clone https://github.com/ultralytics/yolov5  # 克隆
cd yolov5
pip install -r requirements.txt  # 安装
```

**环境要求**

除了本地项目还要求安装 [PyTorch](https://pytorch.org/get-started/locally/) 。由于网络问题等原因，默认下载的都是 cpu 运行版本的库。而 Nvidia 的 GPU 对神经学习是有强大的优化的。所以可以去官网手动下载符合版本的 GPU 库，然后手动安装下载的 .whl 文件。

- 对应版本：

  （1）打开 Nvidia 控制面板 - 帮助 - 系统信息 - 组件，查看 CUDA 的版本号，我现在的是 11.6。所以下载的时候也要下载 CU116 版本的

  （2）版本号后面是 cp310 ，这里指的是 python 版本，310即 3.10 的 python 版本

安装后看到项目目录主要有这几个文件：

- detect.py 检测
- train.py 训练

## 配置检测参数

在 detect.py 的 `parse_opt()`函数中可以看到有很多默认的检测参数。下面对其中的参数做说明记录。

```python
def parse_opt():
    parser = argparse.ArgumentParser()
    # 配置模型，模型可以从下面模型中选
    parser.add_argument('--weights', nargs='+', type=str, default=ROOT / 'yolov5s.pt', help='model path(s)')
    parser.add_argument('--source', type=str, default=ROOT / 'data/images', help='file/dir/URL/glob, 0 for webcam')
    parser.add_argument('--data', type=str, default=ROOT / 'data/coco128.yaml', help='(optional) dataset.yaml path')
    parser.add_argument('--imgsz', '--img', '--img-size', nargs='+', type=int, default=[640], help='inference size h,w')
    parser.add_argument('--conf-thres', type=float, default=0.25, help='confidence threshold')
    parser.add_argument('--iou-thres', type=float, default=0.45, help='NMS IoU threshold')
    parser.add_argument('--max-det', type=int, default=1000, help='maximum detections per image')
    parser.add_argument('--device', default='', help='cuda device, i.e. 0 or 0,1,2,3 or cpu')
    parser.add_argument('--view-img', action='store_true', help='show results')
    parser.add_argument('--save-txt', action='store_true', help='save results to *.txt')
    parser.add_argument('--save-conf', action='store_true', help='save confidences in --save-txt labels')
    parser.add_argument('--save-crop', action='store_true', help='save cropped prediction boxes')
    parser.add_argument('--nosave', action='store_true', help='do not save images/videos')
    parser.add_argument('--classes', nargs='+', type=int, help='filter by class: --classes 0, or --classes 0 2 3')
    parser.add_argument('--agnostic-nms', action='store_true', help='class-agnostic NMS')
    parser.add_argument('--augment', action='store_true', help='augmented inference')
    parser.add_argument('--visualize', action='store_true', help='visualize features')
    parser.add_argument('--update', action='store_true', help='update all models')
    parser.add_argument('--project', default=ROOT / 'runs/detect', help='save results to project/name')
    parser.add_argument('--name', default='exp', help='save results to project/name')
    parser.add_argument('--exist-ok', action='store_true', help='existing project/name ok, do not increment')
    parser.add_argument('--line-thickness', default=3, type=int, help='bounding box thickness (pixels)')
    parser.add_argument('--hide-labels', default=False, action='store_true', help='hide labels')
    parser.add_argument('--hide-conf', default=False, action='store_true', help='hide confidences')
    parser.add_argument('--half', action='store_true', help='use FP16 half-precision inference')
    parser.add_argument('--dnn', action='store_true', help='use OpenCV DNN for ONNX inference')
    parser.add_argument('--vid-stride', type=int, default=1, help='video frame-rate stride')
    opt = parser.parse_args()
    opt.imgsz *= 2 if len(opt.imgsz) == 1 else 1  # expand
    print_args(vars(opt))
    return opt
```

我们也可以在其中手动配置检测参数。

## 模型

![image-20220915102058051](https://liaoyk-markdown.oss-cn-hangzhou.aliyuncs.com/markdownImg/image-20220915102058051.png) 

模型越大检测越准确，同时对性能的消耗也越高，一般我们用 s 或者 m 即可！

在 detect.py 中配置模型参数后，yolov5 会自动去下载模型到 models 文件夹！
