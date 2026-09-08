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
| Traffic signs | 454 |
| Traffic lights | 278 |

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
| YOLOv8n, scratch, 100 epochs | 0.9245 | — | 0.3207 | — |
| YOLOv8n, pretrained, 100 epochs | **0.7151** | **0.4293** | **0.5036** | **0.3258** |

The pretrained model achieves the strongest overall validation performance.

Although the scratch model reaches high precision, its lower mAP indicates weaker overall detection coverage.

Transfer learning provides a substantially better balance between localization and classification quality for the relatively small simulated dataset.

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
- Google Colab
- CUDA


## Key Takeaways

This project demonstrates a complete simulation-to-perception workflow:

1. Generate autonomous-driving scenes in CARLA.
2. Collect synchronized visual and segmentation sensor data.
3. Automatically generate object-detection labels.
4. Train a YOLOv8 detector.
5. Evaluate detection performance.
6. Deploy the model on sequential driving video.

The experiment also demonstrates the benefit of transfer learning when the available task-specific dataset is relatively small.

## Future Work

Possible extensions include:

- Increasing the number and diversity of simulated scenes
- Collecting data under more weather and lighting conditions
- Adding vehicles and pedestrians as additional classes
- Class-specific confidence-threshold optimization
- Temporal tracking across video frames
- Domain adaptation from simulation to real-world driving scenes

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

最终共生成 **454 个交通标志实例** 和 **278 个交通信号灯实例**。

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
| 预训练 YOLOv8n | 0.7151 | 0.4293 | **0.5036** | **0.3258** |

在该仿真数据集上，使用预训练权重的 YOLOv8n 获得了最好的整体检测表现。

## Demo

交通标志、交通信号灯检测结果以及视频推理示例可放置于 `assets` 目录中。

## 技术栈

- CARLA 0.9.15
- YOLOv8
- PyTorch
- OpenCV
- Python
- CUDA

