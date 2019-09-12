---
layout: post
title: Implement shapenet face landmark detection in Tensorflow
date: '2019-09-12 21:41:40'
tags:
- technical
---

In my previous post on [building face landmark detection model](/2019/04/03/face-landmark-detect), the [Shapenet](https://github.com/justusschock/shapenet) paper was implemented in Pytorch. With Pytorch, however, to run the model on mobile requires converting it to Caffe. Though there is tool to take care of that, some operations are not supported and in the case of Shapenet, it was not something I know how to fix yet. Turn out it was simpler to just re-implement Shapenet in Tensorflow and then convert it to Tensorflow Lite. 

## Implement Shapenet in Tensorflow

```python
def predict_landmarks(inputs, pca_components, is_training=True, feature_extractor=extractors.original_paper_feature_extractor):

    # shape means are stored at index 0
    shape_mean = tf.constant(pca_components[0], name='shape_means', dtype=tf.float32)
    components = tf.constant(pca_components[1:], name='components', dtype=tf.float32)

    in_channels = 1 if len(inputs.shape) == 3 else 3 

    # get number of PCA components
    n_components = components.shape.as_list()[0]
    n_transforms = 0
    for k, v in TRANSFORMS_OPS.items():
        n_transforms += v    

    num_out_params = n_components + n_transforms  

    if in_channels == 1:
        inputs = tf.expand_dims(inputs, -1)
        
    # extract features from input
    features = feature_extractor(inputs, num_out_params, is_training)
    features = tf.reshape(features, [-1, num_out_params, 1, 1])

    # run shape layer 
    shapes = shape_layer(shape_mean, components, features[:, 0:n_components])

    # run transform layers. Transform parameters are stored
    # in the last few indices of features vector.
    transformed_shapes = transform_layer(shapes, features[:, n_components:])
    return transformed_shapes
```

The implementatin in Tensorflow is pretty straightforward. 

1. Firstly, pre-calculated PCA components are loaded onto the Tensorflow graph as `tf.constant`. Secondly. 
2. A feature extractor network is run on input images to retreive feature vector.
3. Run the shape layer on the extracted features to get landmarks. 
4. Transform the output of shape layer with scale, translate and rotate parameters. 

Detailed implementation of each layer can be found [here](https://github.com/vuamitom/shapenet-tensorflow/tree/master/model/shapenet)

## Speeding up feature extraction with depthwise convolution.

When deploying neural network on mobile, speed is the key. Using tensorflow lite [bencmark tool](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/lite/tools/benchmark), we can discern that convolution ops are among the most costly operations. By replacing convolution layer with [depthwise separable convolution](https://towardsdatascience.com/a-basic-introduction-to-separable-convolutions-b99ec3102728), a speed up can be quickly achieved.

```python
def depthwise_conv_feature_extractor(inputs, num_out_params, is_training=True):
    return original_paper_feature_extractor(inputs, num_out_params, is_training=is_training, use_depthwise=True)
    
def original_paper_feature_extractor(inputs, num_out_params, is_training=True, use_depthwise=False):
    """  Original feature extractor accepts a use_depthwise flag
    """
    norm_class = tf.contrib.layers.instance_norm

    # use depthwise conv2d as a drop-in replacement for conv2d
    conv2d_class = slim.separable_conv2d if use_depthwise else slim.conv2d
```

## Converting to Tflite model.

In theory, converting a Tensorflow model to Tflite is just a matter of running converter tool. However, similar to Pytorch converter, Tflite model requires the Tflite runtime to support operations in the model graph. For Shapenet, `tf.broadcast_to` and `tf.cos` were two operations that are not supported at the time. Fortunately it is pretty simple to implement your own from from supported operations:

```python
# for tf.broadcast_to
def broadcast_to_batch(tensor, batch_size, name=None):
    if FOR_TFLITE:
        multiples = tf.concat([[batch_size], tf.ones(tf.size(tf.shape(tensor)), dtype=tf.int32)], 0)
        return tf.tile(tf.expand_dims(tensor, 0), multiples, name=name)
    else:
        return tf.broadcast_to(tensor, [batch_size, *tensor.shape], name=name)
```


```python
# for tf.cos
def do_cos(tensor, sin_tensor):
    if FOR_TFLITE:
        # calculate cos from sin 
        # since cos is not supported as of 1.13.1
        # cos = (1 - sin**2)** 0.5
        return tf.pow(1 - tf.pow(sin_tensor, 2), 0.5)
    else:
        return tf.cos(tensor)
```

