---
layout: post
title: Use MobileNetV2 as feature extractor in Tensorflow
date: '2019-06-19 21:41:40'
mathjax: false
tags:
- technical
---

Applying machine learning in image processing tasks sometimes feel like toying with Lego blocks. One base block to extract feature vectors from images, another block to classify... Popular choices of feature extractors are MobileNet, ResNet, Inception. And as with any other engineering problem, choosing a feature extractor is about considering trade-offs between speed, accuracy, and size. For my current task of dealing with ML on mobile devices, [MobileNetV2](https://arxiv.org/abs/1801.04381) seem to be a good fit as it is fast, quantization friendly and does not sacrifice too much of accuracy. Tensorflow provides a [reference implementation](https://github.com/tensorflow/models/tree/master/research/slim/nets/mobilenet) of MobileNetV2 that makes using it much easier. 

First thing first, clone the [repo](https://github.com/tensorflow/models) and add its to Python path 

```python
import sys
sys.path.append('/path/to/tensorflow/models/research/slim')
```

`mobilenet_v2.mobilenet_base` returns output tensors that are convolved with input image. For MobileNetV2, the last layer is `layer_20`. Output from mobilenet can be used for classification or as input to ssdlite for object detection. 

```python

def mobilenet(inputs, is_training=True):
	with tf.variable_scope('myscope', reuse=reuse_weights) as scope:
        with slim.arg_scope(
            mobilenet_v2.training_scope(is_training=is_training, bn_decay=0.9997)), \
            slim.arg_scope(
              [mobilenet.depth_multiplier], min_depth=16):
            with (context_manager.IdentityContextManager()):
                _, image_features = mobilenet_v2.mobilenet_base(
                  od_ops.pad_to_multiple(inputs, 32),                  
                  depth_multiplier=1.0,
                  is_training=is_training,
                  use_explicit_padding=True,
                  scope=scope)
                
                # last layer of moblilenetV2 is layer_20
                # this is for demonstration purpuse.
                # Layers such as fully_connected can be put here. 
                return image_features['layer_20']
``` 

It is a pretty straight forward process. There is only one point to note, the `is_training` flag. When it is set to True, BatchNorm layers accumulate statistics such as moving variance and moving mean. When it is False, it would just use those values. When I first trained my MobileNetV2 based network, though loss value decreased fine, prediction worked as expected. Just that the model would produce junk output when `is_training` is set to False for evaluation. And quantizing the model seemed to make it gibberish. Turned out, I need to make the update of moving variances and means a dependency of training ops, which is cautioned [here](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/contrib/layers/python/layers/layers.py#L473). 

```python
	update_ops = tf.compat.v1.get_collection(tf.GraphKeys.UPDATE_OPS)
    with tf.control_dependencies(update_ops):
        print('add dependency on "moving avg" for batch_norm')        
        train_op = optimizer.minimize(l1_loss, global_step) 
```

Without it, Tensorflow does not write those statistics as part of the model, which breaks it after quantization. 

 