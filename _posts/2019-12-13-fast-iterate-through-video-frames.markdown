---
layout: post
title: Fast way to iterate through video frames with Python and OpenCV
date: '2019-12-13 14:36:18'
tags:
- technical
---

Recently, I was working on a program to sample N frames from a source video, and then assign a score to each frame (from 0 to 1) in terms of thumbnail-worthiness. Before long, it became apparent that decoding video frames was one performance bottleneck. 

Consider that a video is F frame long, we will need to select frame at the sample rate of `S = F/N`. Naiively, one could loop through the video and read every single frame. 

```python
# naive version
cap = cv2.VideoCapture(video_path)
success, img = cap.read()
fno = 0
while success:
	if fno % sample_rate == 0:
		do_something(img)
	# read next frame
	success, img = cap.read()
```

However, each call to `cap.read()` would result in unnecessary decoding of frames to be discarded. Performance was intolerable. A better alternative would be to skip through frames and only decode the one that we need. Opencv `grab` and `retrieve` function can be used for this purpose (which are called by `read` function internally). 

```python
success = cap.grab() # get the next frame
fno = 0
while success:
	if fno % sample_rate == 0:
		_, img = cap.retrieve()
		do_something(img)
	# read next frame
	success, img = cap.grab()	
```

This simple modification would result in a much needed performance boost. But can we do better? Looking through OpenCV documentation, it's possible to read from a specific frame by call to `cap.set(cv2.CAP_PROP_POS_FRAMES, fno)`. The above snippet can be re-written as below:

```python
for fno in range(0, total_frames, sample_rate):
	cap.set(cv2.CAP_PROP_POS_FRAMES, fno)
	_, image = cap.read()
	do_something(image)
```

So which option is faster. It depends on number of frames read. The sparser the read, the faster it is to read with random-seeking method. Since each `cap.set` takes a fixed amount of time to reach certain frame, the latter method's time cost increases linearly with number of frames to sample. On the other hand, the incremental read method results in a relatively fix amount of time regardless of number of frames to sample. 

![Compare performance of two methods](/content/images/opencv_read_frame.png)

