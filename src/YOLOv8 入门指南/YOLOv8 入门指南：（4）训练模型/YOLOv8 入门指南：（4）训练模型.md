# YOLOv8 入门指南：（4）训练模型

[TOC]

## 安装 YOLOv8

在命令行中运行命令:
```shell
pip install ultralytics
```

## 数据集配置文件

在开始模型训练之前，新建一个 `yaml` 文件对数据集进行配置。
```yaml
# 这是一个工作文件夹示例
# ├── data
# └── assets
#     └── qr
#         └── images
#             ├── train
#             └── val
#         └── labels
#             ├── train
#             └── val
# └── ...

path: ../assets/qr      # 数据集所在的文件夹
train:                  # 训练集
  - images/train
val:                    # 验证集
  - images/val
names:                  # 数据集的种类
  0: blue_qr
  1: green_qr
  2: red_qr
```

## 预训练权重下载

预训练模型是一种在大型数据集上进行预先训练的神经网络模型，一般来说训练这些模型需要大量的资源，费时费力。通常我们需要在预训练模型的基础之上进行微调，去定制化地训练某些任务，使得预训练模型“更懂”这个任务。

预训练模型可以直接使用 Python 代码进行下载：
```python
model = YOLO('yolov8n.pt')  # 加载 YOLOv8n 预训练模型
```
但是这些模型托管在 GitHub 上，网络不好的情况下可能无法下载，我们可以提前下载好需要用到的 `.pt` 文件类型的模型。下载地址：https://github.com/ultralytics/assets/releases

预训练模型有很多种，这些不同参数量级的模型主要区别在于网络的深度和宽度，n、s、m、l、x 表示不同尺寸规模的模型，其中 n 为最小，x 为最大规模。比如：

YOLOv8n：规模最小的模型，具有较少的参数量。它适用于资源受限的设备或需要快速推理的场景。

YOLOv8x：参数最多的模型，也是最大的模型。它提供了最高的检测精度，但需要更多的计算资源。适用于对精度要求较高的场景，如精细粒度目标检测。

| Model | size(pixels) | mAPval 50-95 | Speed CPU ONNX(ms) | Speed A100 TensorRT(ms) | params(M) | FLOPs(B) |
| :-: | :-: | :-: | :-: | :-: | :-: | :-: |
| [YOLOv8n](https://github.com/ultralytics/assets/releases/download/v8.1.0/yolov8n.pt) | 640 | 37.3 | 80.4 | 0.99 | 3.2 | 8.7 |
| [YOLOv8s](https://github.com/ultralytics/assets/releases/download/v8.1.0/yolov8s.pt) | 640 | 44.9 | 128.4 | 1.20 | 11.2 | 28.6 |
| [YOLOv8m](https://github.com/ultralytics/assets/releases/download/v8.1.0/yolov8m.pt) | 640 | 50.2 | 234.7 | 1.83 | 25.9 | 78.9 |
| [YOLOv8l](https://github.com/ultralytics/assets/releases/download/v8.1.0/yolov8l.pt) | 640 | 52.9 | 375.2 | 2.39 | 43.7 | 165.2 |
| [YOLOv8x](https://github.com/ultralytics/assets/releases/download/v8.1.0/yolov8x.pt) | 640 | 53.9 | 479.1 | 3.53 | 68.2 | 257.8 |

## 开始训练

```python
from ultralytics import YOLO
# 加载预训练模型
model = YOLO("weights/yolov8n.pt")
# 图像大小为640的数据集上进行100次训练
result = model.train(data="data/qr.yaml", epochs=100, imgsz=640)
# 验证模型
metrics = model.val()
metrics.box.map
metrics.box.map50
metrics.box.map75
metrics.box.maps
# 目标检测
predict = model("assets/qr/images/val/1.jpg")
# 保存训练的模型
success = model.export(format="tflite")
```
