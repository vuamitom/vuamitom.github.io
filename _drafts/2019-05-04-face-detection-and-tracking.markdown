Everyone who has used Facebook messenger video call may have tried their delightful feature of overlaying caller's face with cartoonish mask. Behind the scene, the app would need to analyze the camera's bitmap to locate positions of eyes, nose, lips and chin...

Most available blog posts focus on landmark detection using opencv and dlib. Those are easy to use but are hard to customize and often large in size. This is a collection of useful references in practical face landmark detection without using those library with more control. I'm in the process of 

Object detection:

https://github.com/tensorflow/models/tree/master/research/object_detection/

pretrained face model: https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md

Face landmark:

Pointing out by my colleague. 
https://www.groundai.com/project/pfld-a-practical-facial-landmark-detector/#bib.bib13


Object tracking:
https://github.com/davheld/GOTURN#train-the-tracker
https://www.learnopencv.com/goturn-deep-learning-based-object-tracking/
https://www.learnopencv.com/object-tracking-using-opencv-cpp-python/
https://arxiv.org/pdf/1611.10012.pdf - compare different models (by google)
Speed:

- quantize
- reduce size of input image

Dataset

https://www.kaggle.com/dataturks/face-detection-in-images/kernels

https://towardsdatascience.com/faced-cpu-real-time-face-detection-using-deep-learning-1488681c1602

http://shuoyang1213.me/WIDERFACE/

