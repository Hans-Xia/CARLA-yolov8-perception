# CARLA Traffic Perception with YOLOv8

[English](#english) | [中文](#中文)

## English
An end-to-end simulated autonomous-driving perception pipeline for **traffic sign and traffic light detection** using CARLA and YOLOv8.

The project covers the complete workflow from sensor-based data generation in a driving simulator to YOLO-format annotation, model training, evaluation, and frame-by-frame video inference.

## Highlights

- Generated road-scene data using the **CARLA 0.9.15** simulator.
- Configured RGB, semantic-segmentation, and instance-segmentation cameras.
- Automatically extracted traffic-sign and traffic-light instances from simulator segmentation outputs.
- Converted simulator annotations into YOLO bounding-box labels.
- Trained and compared YOLOv8n models from scratch and with transfer learning.
- Achieved **mAP@50 = 0.5036** and **mAP@50–95 = 0.3258** using pretrained YOLOv8n.
- Applied trained detectors to a **2,948-frame driving video**.

## Overview

Autonomous vehicles require robust perception systems capable of recognizing safety-critical road infrastructure.

Real-world data collection and annotation can be expensive. CARLA provides a controlled simulation environment where synchronized sensor information and ground-truth semantic information can be generated automatically.

This project connects simulation and deep learning through the following pipeline:

```text
CARLA Simulator
      │
      ▼
Simulated Vehicle
      │
      ├── RGB Camera
      ├── Semantic Segmentation Camera
      └── Instance Segmentation Camera
              │
              ▼
     Object Instance Extraction
              │
              ▼
     Bounding-Box Generation
              │
              ▼
        YOLO Dataset
              │
              ▼
        YOLOv8 Training
              │
              ▼
     Validation + Video Inference
```

## CARLA Environment

The simulation is implemented using **CARLA 0.9.15**.

A vehicle is spawned in the CARLA urban environment and tested under both manual-control and autopilot modes.

Multiple camera sensors are attached to the vehicle:

### RGB Camera

Provides the visual road-scene frames used as YOLO model inputs.

### Semantic Segmentation Camera

Provides pixel-level semantic categories for scene elements.

### Instance Segmentation Camera

Provides individual object identities, allowing separate traffic signs and traffic lights to be localized.

The combination of semantic and instance information enables automatic bounding-box generation without manually annotating every image.

## Automatic Annotation

CARLA segmentation information is used to locate:

- Traffic signs
- Traffic lights

For each detected instance, the visible pixel region is extracted and converted into a rectangular bounding box.

The bounding boxes are then transformed into standard YOLO format:

```text
class_id x_center y_center width height
```

with normalized image coordinates.

The final detection classes are:

| Class ID | Object |
| ---: | --- |
| 0 | Traffic sign |
| 1 | Traffic light |

## Dataset

The generated dataset contains:

| Split | Images |
| --- | ---: |
| Training | 210 |
| Validation | 60 |
| Test | 30 |
| **Total** | **300** |

### Object Instances

| Class | Instances |
| --- | ---: |
| Traffic signs | 619 |
| Traffic lights | 649 |

Traffic objects are often relatively small compared with the full road-scene image, making localization challenging despite the controlled simulation environment.

## YOLOv8 Training

Three YOLOv8n experiments are compared.

### Experiment 1

YOLOv8n trained from scratch for **10 epochs**.

This short run is mainly used to observe initial learning behavior and does not produce a strong detector.

### Experiment 2

YOLOv8n trained from scratch for **100 epochs**.

### Experiment 3

Pretrained YOLOv8n fine-tuned for **100 epochs**.

All main experiments use:

- Input resolution: 640 × 640
- Batch size: 8
- GPU acceleration
- Random seed: 42

## Results

| Model | Precision | Recall | mAP@50 | mAP@50–95 |
| --- | ---: | ---: | ---: | ---: |
| YOLOv8n scratch, 100 epochs | 0.9571 | 0.7087 | 0.8156 | 0.6713 |
| YOLOv8n pretrained, 100 epochs | **0.9402** | **0.9176** | **0.9585** | **0.8567** |


The pretrained YOLOv8n model achieved the best overall detection performance, reaching **mAP@50 = 0.9585** and **mAP@50–95 = 0.8567**.

## Video Inference

The trained 100-epoch models are evaluated on a driving video using frame-by-frame inference.

A total of **2,948 frames** are processed.

Detection outputs include:

- Object class
- Bounding box
- Confidence score
- Frame index

Annotated videos and structured detection logs can be generated for subsequent analysis.

## Technology Stack

- CARLA 0.9.15
- Python
- YOLOv8 / Ultralytics
- PyTorch
- OpenCV
- NumPy
- CUDA


## 中文

# 基于 CARLA 与 YOLOv8 的交通感知

基于 CARLA 自动驾驶仿真环境和 YOLOv8 实现交通标志与交通信号灯检测。

## 项目概述

本项目构建了一套从仿真数据生成到目标检测的完整流程：

1. 在 **CARLA** 中生成驾驶场景
2. 获取 RGB、语义分割与实例分割数据
3. 自动生成目标检测标注
4. 训练 YOLOv8
5. 对驾驶视频进行逐帧推理

## 数据集

仿真数据集规模如下：

| 数据划分 | 图像数量 |
| --- | ---: |
| 训练集 | 210 |
| 验证集 | 60 |
| 测试集 | 30 |

检测类别包括：

- 交通标志
- 交通信号灯

最终共生成 **1,268 个目标标注框**，其中：

- **619 个交通标志**
- **649 个交通信号灯**

## 检测流程

```text
CARLA 仿真器
     │
     ├── RGB 相机
     ├── 语义分割相机
     └── 实例分割相机
             │
             ▼
        自动生成标注
             │
             ▼
         YOLO 数据集
             │
             ▼
        YOLOv8 训练
             │
             ▼
          视频推理
```

## 实验结果

| 模型 | Precision | Recall | mAP@50 | mAP@50–95 |
| --- | ---: | ---: | ---: | ---: |
| YOLOv8n 从头训练，100 epochs | 0.9571 | 0.7087 | 0.8156 | 0.6713 |
| YOLOv8n 预训练权重，100 epochs | **0.9402** | **0.9176** | **0.9585** | **0.8567** |

使用预训练权重的 YOLOv8n 获得了最佳整体检测性能，达到 **mAP@50 = 0.9585**、**mAP@50–95 = 0.8567**。

训练完成后，模型还在一段包含 **2,948 帧**的驾驶视频上进行了逐帧推理。

## 技术栈

- CARLA 0.9.15
- Python
- YOLOv8 / Ultralytics
- PyTorch
- OpenCV
- NumPy
- CUDA
