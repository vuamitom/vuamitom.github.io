---
layout: post
title: Measuring Android WebView memory consumption
---

The Memory Monitor panel in Android Studio can be misleading when it comes to measuring total memory used by an application as it only reflects the Java heap's size. Java heap, which is the memory JVM reserves, is only part of the total memory used by a Java program. Besides the Java heap, an application also makes use of native heap, which is where objects created by C/C++ code resides. Android application makes use of this heap when it creates Bitmap instances (Honeycomb and before) or allocates object via JNI calls. 

When an instance of WebView got created and loadUrl is called on it, Memory Monitor shows an increase of roughly 3MB

![](/content/images/2016/09/Screen-Shot-2016-08-10-at-6-06-42-PM.png)

However, the amount is only reflective of allocation by JVM. To have a accurate measure of memory cost, we need to check dumpsys meminfo
```
dumpsys meminfo <package-name> -d
```

Before webview 
![](http://)

After webview 
![](http://)

In app summary 
Java heap = Davik heap + stack + ttf
