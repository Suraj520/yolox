# More Than YOLO

TensorFlow & Keras Implementations & Python

YOLOv3, YOLOv3-tiny, YOLOv4

YOLOv4-tiny(unofficial)

**requirements:** TensorFlow 2.x (not test on 1.x), OpenCV, Numpy, PyYAML

---

## News !

- Small batch size is used, because available GPU (8G) has small memory. 
- Online High Level augmentation will slow down training speed.
- When I tried to train yolov3 or yolov4, 'NaN' problem made me crazy.

---

This repository have done:

- [x] Backbone for YOLO (YOLOv3, YOLOv3-tiny, YOLOv4, YOLOv4-tiny[unofficial])
- [x] YOLOv3 Head
- [x] Keras Callbacks for Online Evaluation
- [x] Load Official Weight File
- [x] K-Means for Anchors
- [x] Fight with 'NaN'
- [x] Train (Strategy and Model Config)
  - Define simple training in [train.py](./train.py)
  - Use YAML as config file in [cfgs](./cfgs)
  - [x] Cosine Annealing LR
  - [x] Warm-up
- [ ] Data Augmentation
  - [x] Standard Method: Random Flip, Random Crop, Zoom, Random Grayscale, Random Distort, Rotate
  - [x] Hight Level: Cut Mix, Mix Up, Mosaic （These Online Augmentations is Slow）
  - [ ] More, I can be so much more ... 
- [ ] For Loss
  - [x] Label Smoothing
  - [x] Focal Loss
  - [x] L2, D-IoU, G-IoU, C-IoU
  - [ ] ...

---

[toc]

## 0. Please Read Source Code for More Details

You can get official weight files from https://github.com/AlexeyAB/darknet/releases or https://pjreddie.com/darknet/yolo/.

## 1. Samples

### 1.1 Data File

We use a special data format like that,

```txt
path/to/image1 x1,y1,x2,y2,label x1,y1,x2,y2,label 
path/to/image2 x1,y1,x2,y2,label 
...
```

Convert your data format firstly. We present a script for Pascal VOC in https://github.com/yuto3o/yolox/blob/master/data/pascal_voc/voc_convert.py

More details and a simple dataset could be got from https://github.com/YunYang1994/yymnist.

### 1.2 Configure 

```yaml
# voc_yolov3_tiny.yaml
yolo:
  type: "yolov3_tiny" # must be 'yolov3', 'yolov3_tiny', 'yolov4', 'yolov4_tiny'.
  iou_threshold: 0.45
  score_threshold: 0.5
  max_boxes: 100
  strides: "32,16"
  anchors: "10,14 23,27 37,58 81,82 135,169 344,319"
  mask: "3,4,5 0,1,2"

train:
  label: "voc_yolov3_tiny" # any thing you like
  name_path: "./data/pascal_voc/voc.name"
  anno_path: "./data/pascal_voc/train.txt"
  # "416" for single mini batch size, "352,384,416,448,480" for Dynamic mini batch size.
  image_size: "416" 

  batch_size: 4
  # if you want to load .weights file, you should use something like coco.yaml.
  init_weight_path: "./ckpts/yolov3-tiny.h5"
  save_weight_path: "./ckpts"

  # Must be "L2", "DIoU", "GIoU", "CIoU" or something like "L2+FL" for focal loss
  loss_type: "L2" 
  
  # turn on hight level data augmentation
  mix_up: false
  cut_mix: false
  mosaic: false
  label_smoothing: false
  normal_method: true

  ignore_threshold: 0.5

test:
  anno_path: "./data/pascal_voc/test.txt"
  image_size: "416" # image size for test
  batch_size: 1
  init_weight_path: ""
```

### 1.3 K-Means

Edit the kmeans.py as you like

```python
# kmeans.py
# Key Parameters
K = 6 # num of clusters
image_size = 416
dataset_path = './data/pascal_voc/train.txt'
```

### 1.4 Inference

#### A Simple Script for Video, Device or Image

Only support mp4, avi, device id, rtsp, png, jpg (Based on OpenCV) 

![gif](./misc/street.gif)

```shell
python detector.py --config=./cfgs/coco_yolov4.yaml --media=./misc/street.mp4 --gpu=false
```

#### A simple demo for YOLOv4

![yolov4](./misc/dog_v4.jpg)

```python
from core.utils import decode_cfg, load_weights
from core.model.one_stage.yolov4 import YOLOv4
from core.image import draw_bboxes, preprocess_image, preprocess_image_inv, read_image, Shader

import numpy as np
import cv2
import time

# read config
cfg = decode_cfg('cfgs/coco_yolov4.yaml')

model, eval_model = YOLOv4(cfg)
eval_model.summary()

# assign colors for difference labels
shader = Shader(cfg['yolo']['num_classes'])

# load weights
load_weights(model, cfg['test']['init_weight_path'])

img_raw = read_image('./misc/dog.jpg')
img = preprocess_image(img_raw, (512, 512))
imgs = img[np.newaxis, ...]

tic = time.time()
boxes, scores, classes, valid_detections = eval_model.predict(imgs)
toc = time.time()
print((toc - tic)*1000, 'ms')

# for single image, batch size is 1
valid_boxes = boxes[0][:valid_detections[0]]
valid_score = scores[0][:valid_detections[0]]
valid_cls = classes[0][:valid_detections[0]]

valid_boxes *= 512
img, valid_boxes = preprocess_image_inv(img, img_raw.shape[1::-1], valid_boxes)
img = draw_bboxes(img, valid_boxes, valid_score, valid_cls, names, shader)

cv2.imshow('img', img[..., ::-1])
cv2.imwrite('./misc/dog_v4.jpg', img)
cv2.waitKey()
```

## 2. Train

!!! Please Read the abve guide (1.1, 1.2).

```shell
python train.py --config=./cfgs/voc_yolov4.yaml
```

## 3. Experiment

**i7-9700F+16GB**

| Model       | 416x416 | 512x512 | 608x608 |
| ----------- | ------- | ------- | ------- |
| YOLOv3      |         |         |         |
| YOLOv3-tiny |         |         |         |
| YOLOv4      |         |         |         |
| YOLOv4-tiny |         |         |         |

**i7-9700F+16GB / RTX 2070S+8G**

| Model       | 416x416 | 512x512 | 608x608 |
| ----------- | ------- | ------- | ------- |
| YOLOv3      |         |         |         |
| YOLOv3-tiny |         |         |         |
| YOLOv4      | 61 ms   |         |         |
| YOLOv4-tiny | 29 ms   |         |         |

We freeze backbone for the first 30 epoches (lr=1e-4), and then finetune  all of the trainable variables for another 50 epoches (lr=1e-5), and final 10 epoches for evaluation(lr=1e-6).

| Name                    | Abbr |
| ----------------------- | ---- |
| Standard Method         | SM   |
| Dynamic mini batch size | DM   |
| Label Smoothing         | LS   |
| Focal Loss              | FL   |
| Mix Up                  | MU   |
| Cut Mix                 | CM   |
| Mosaic                  | M    |

Standard Method Package includes Flip left and right,  Crop and Zoom(jitter=0.3), Grayscale, Distort, Rotate(angle=7).

**YOLOv3-tiny**(Pretrained on COCO)

| SM   | DM   | LS   | FL   | MU   | CM   | M    | Loss | mAP  | mAP@50 | mAP@75 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ------ | ------ |
|      |      |      |      |      |      |      | L2   | 18.5 | 44.9   | 10.4   |
| ✔    |      |      |      |      |      |      | L2   | 22.0 | 49.1   | 15.2   |
| ✔    | ✔    |      |      |      |      |      | L2   | 22.8 | 49.8   | 16.3   |
| ✔    | ✔    | ✔    |      |      |      |      | L2   | 21.9 | 48.5   | 15.4   |
| ✔    | ✔    |      |      |      |      |      | CIoU | 25.3 | 50.5   | 21.8   |
| ✔    | ✔    |      | ✔    |      |      |      | CIoU | 25.6 | 49.4   | 23.6   |
| ✔    | ✔    |      | ✔    | ✔    |      |      | CIoU |      |        |        |
| ✔    | ✔    |      | ✔    |      | ✔    |      | CIoU |      |        |        |
| ✔    | ✔    |      | ✔    |      |      | ✔    | CIoU | 23.7 | 46.1   | 21.3   |

Maybe the model is underfitting, so **Label Smoothing** doesn't work ???

**YOLOv3**(TODO; Pretrained on COCO)

| SM   | DM   | LS   | FL   | MU   | CM   | M    | Loss | mAP  | mAP@50 | mAP@75 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ------ | ------ |
| ✔    | ✔    | ✔    | ✔    |      |      |      | CIoU |      |        |        |
| ✔    | ✔    | ✔    | ✔    | ✔    |      |      | CIoU |      |        |        |
| ✔    | ✔    | ✔    | ✔    |      | ✔    |      | CIoU |      |        |        |
| ✔    | ✔    | ✔    | ✔    |      |      | ✔    | CIoU |      |        |        |

**YOLOv4-tiny**(TODO; Pretrained on COCO, part of YOLOv3-tiny weights)

| SM   | DM   | LS   | FL   | MU   | CM   | M    | Loss | mAP  | mAP@50 | mAP@75 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ------ | ------ |
| ✔    | ✔    |      | ✔    |      |      |      | CIoU | 27.6 | 48.3   | 28.9   |
| ✔    | ✔    |      | ✔    | ✔    |      |      | CIoU |      |        |        |
| ✔    | ✔    |      | ✔    |      | ✔    |      | CIoU |      |        |        |
| ✔    | ✔    |      | ✔    |      |      | ✔    | CIoU |      |        |        |

**YOLOv4**(TODO; Pretrained on COCO)

| SM   | DM   | LS   | FL   | MU   | CM   | M    | Loss | mAP  | mAP@50 | mAP@75 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ------ | ------ |
| ✔    | ✔    | ✔    | ✔    |      |      |      | CIoU |      |        |        |
| ✔    | ✔    | ✔    | ✔    | ✔    |      |      | CIoU |      |        |        |
| ✔    | ✔    | ✔    | ✔    |      | ✔    |      | CIoU |      |        |        |
| ✔    | ✔    | ✔    | ✔    |      |      | ✔    | CIoU |      |        |        |

## 4. Reference

- https://github.com/YunYang1994/TensorFlow2.0-Examples/tree/master/4-Object_Detection/YOLOV3
- https://github.com/hunglc007/tensorflow-yolov4-tflite
- https://github.com/experiencor/keras-yolo3

## 5. History

- Slim version: https://github.com/yuto3o/yolox/tree/slim
- Tensorflow2.0-YOLOv3: https://github.com/yuto3o/yolox/tree/yolov3-tf2