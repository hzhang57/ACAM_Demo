# Actor Conditioned Attention Maps Demo

This repo contains the demo code for our action recognition model explained in ARXIV LINK. 

This repo only contains the demo code, training and evaluation codes will be released in https://github.com/oulutan/ActorConditionedAttentionMaps .

The demo code achieves 16 fps through webcam using a GTX 1080Ti using multiprocessing and some other tricks. See the video at 
https://drive.google.com/open?id=1T5AJYp1cF0wLnxG8FmRjoUEGtiR7vvYh 

Additionally, we implemented the activation map displaying functionality to the demo code. This shows where the model gets activated for each action and gives pretty interesting results on understanding what model sees. 

The demo code includes a complete pipeline including object detection (Using TF Object API: https://github.com/tensorflow/models), tracking/bbox matching (Using DeepSort: https://github.com/nwojke/deep_sort) and our action detection module.

# Requirements
1. Tensorflow (Tested with 1.7 and 1.12)
2. Sonnet (DeepMind) for I3D backbone ``` pip install dm-sonnet ```
3. DeepSort (explained in next section)
4. TF object detection API (explained in next section)
5. OpenCV for webcam input and displaying purposes
6. ImageIO for video IO ```pip install imageio```. I rather use ImageIO for video IO than OpenCV as it is (a lot) easier to setup. This is being used when the input is a video file instead of webcam.

# Installation

1. Clone the repo recursively

```bash
git clone --recursive https://github.com/oulutan/ACAM_Demo
```

2. Compile Tensorflow Object Detection API within object_detection/models following https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/installation.md

It should be just the protoc compilation like the following: 
```bash
# From object_detection/models/research/
protoc object_detection/protos/*.proto --python_out=.
```
If you are getting errors you have to download the required protoc and run that
```bash
# From object_detection/models/research/
wget -O protobuf.zip https://github.com/google/protobuf/releases/download/v3.0.0/protoc-3.0.0-linux-x86_64.zip
unzip protobuf.zip
./bin/protoc object_detection/protos/*.proto --python_out=.
```


3. Make sure the object detection API and Deepsort are within PYTHONPATH. I have an easy script for this. 
```bash
# From object_detection/
source SET_ENVIRONMENT.sh
```

4. Download and extract Tensorflow Object Detection models into object_detection/weights/ from: 
https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md

16 Fps from webcam was achieved by 
http://download.tensorflow.org/models/object_detection/ssd_mobilenet_v2_coco_2018_03_29.tar.gz

5. Download DeepSort re-id model into object_detection/deep_sort/weights/ from their author's drive: 
https://drive.google.com/open?id=1m2ebLHB2JThZC8vWGDYEKGsevLssSkjo

6. Download the action detector model weights into action_detection/weights/ from following link:
https://drive.google.com/open?id=138gfVxWs_8LhHiVO03tKpmYBzIaTgD70

# How to run
There are 2 main scripts in this repo. 

```detect_actions.py``` is the simple and slow version where each module works sequentially and it is easier to understand. 

```multiprocess_detect_actions.py``` is the fast version where each module runs separately on their own process.

There are 2 arguments to each script. --v (--videopath) is the path to the video and if it is not provided, webcam will be used. -d (--display) is the display flag where the results will be visualized using OpenCV.

Object detection model can be replaced by any model in the API model zoo. Additionally, there is a object detection batch size parameter which should be changed depending on the GPU memory size and object model detection requirements. 

Object detection and Action detection can use different GPUs for faster performance. 

# How is real-time performance achieved?

1. Object Detector batched processing. Instead of running the object detector on each frame separately, performance was improved by batching multiple frames together. 
