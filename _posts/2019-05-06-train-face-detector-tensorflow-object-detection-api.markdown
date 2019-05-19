---
layout: post
title: Train a face dectector using TensorFlow object detection API.
date: '2019-05-13 21:41:40'
mathjax: false
tags:
- technical
---

About 3 years ago, putting together a face detection camera application for mobile devices was more involving a task. I remember a colleague sitting next to me back then tinkering with [OpenCV](https://opencv.org/) and [dlib](http://dlib.net/) to produce a demo with the right trade-off between size, speed and accuracy. As with every engineering problem, there is no one-size-fit-all solution. A on-device face detector may choose to reduce the size of input images to quicken detection, though lower resolution results in lower accuracy. Fast forward to the moment, it has never been as easier to customize your own face dection model thanks to folks at Google who open source their [Tensorflow object dection api](https://github.com/tensorflow/models/tree/master/research/object_detection). Besides, platforms like Colab provide hobbists with free access to ML training-capable machines. 

This post summarizes a few steps taken to retrain a face-dectector on Google Colab using Tensorflow object dection api. 

## Prepare training data

Though the pre-trained face detection model provided [facessd_mobilenet_v2_quantized_open_image_v4](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md) used OpenImage v4 for training, ground truth bounding boxes are overly large. Besides, the data do not appear to contain suffcient rotated head poses. Instead, I used [WIDERFace dataset](http://shuoyang1213.me/WIDERFACE/) for this training. After downloading the dataset, it can be preprocessed to omit invalid data such as images marked with invalid flag or ones with too small dimensions. Example script can be found [here ](https://github.com/vuamitom/models/blob/master/research/object_detection/dataset_tools/create_widerface_tf_record.py).

## Configure training pipeline

[facessd_mobilenet_v2_quantized_320x320_open_image_v4.config](https://github.com/vuamitom/models/blob/master/research/object_detection/samples/configs/facessd_mobilenet_v2_quantized_320x320_open_image_v4.config) can be a good base for customization. For my purpose, I reduced the size of input images and replace label file with face-only [label map](https://github.com/tensorflow/models/blob/master/research/object_detection/data/face_label_map.pbtxt). 

```
model {
	ssd {
		num_classes: 1
		image_resizer {
		  	fixed_shape_resizer {
			    height: 224
			    width: 224
		  	}
		}
		...
	}
	...
}
```

Some other options that can be tinkered with are number of ssd anchors generated for each grid. 

```
anchor_generator {
  ssd_anchor_generator {
    aspect_ratios: 1.0
    aspect_ratios: 2.0
    aspect_ratios: 0.5
  }
}
```

## Run model on Colab

The [pet detection tutorial](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/running_pets.md) trains on Google AI platform. Initially I did the same and had my cloud billing overshot my budget. Fortunately, Google Colab came to the rescue. Google Colab is a version of Jupyter notebook that lets you run your code on Google's highend machines for free. The downside is Colab requires an active browser session otherwise allocated machines are freed up. So it's better to keep your files on mounted Google Drive.    

```python
# mount drive
from google.colab import drive
drive.mount('/gdrive', force_remount=False)
```

In my case, I uploaded Tensorflow object detection code to `/gdrive/My Drive/models-master` and start training with the below code: 

```python
import sys
sys.path.append('/gdrive/My Drive/models-master/research')
sys.path.append('/gdrive/My Drive/models-master/research/slim')
from tensorflow.python.platform import flags
from object_detection import model_main

class FlagValues:
  pipeline_config_path='/gdrive/My Drive/models-master/research/object_detection/wider_face_models/ssd224/ssd_mobilenet_v2_quantized_224x224_widerface.config'
  model_dir='/gdrive/My Drive/models-master/research/object_detection/wider_face_models/ssd224'
  num_train_steps=50000
  sample_1_of_n_eval_examples=1
  sample_1_of_n_eval_on_train_examples=5
  alsologtostderr=True
  eval_training_data=False
  hparams_overrides=None
  checkpoint_dir=None
  run_once=False
  
model_main.FLAGS = FlagValues()

model_main.main(None)
```


