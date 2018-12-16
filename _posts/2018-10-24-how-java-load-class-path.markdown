---
layout: post
title:  How java load classes
date: '2018-10-24 16:00:00'
tags:
- technical
---

For a while, I have delegated the task of managing java run command to IDE. Eclipse, Netbean and IntelliJ and all seem to do a decent job of masking away complexity of supplying java with JVM parameters, classpath, debugging options... Compared to other languages, the java run command can get horrendously long, not very suitable for handcrafting. Recently, when working on some sort of command generating at work and playing around with java command line arguments, I encountered some minor gotchas. 

##### 1. -jar overrides -classpath
```
java -classpath lib1.jar:lib2.jar -jar sample.jar
```
When `-jar` argument is used, `-classpath` and `-cp` are ignored. `Class-Path` attribute in manifest file of `sample.jar` becomes the class path instead.

##### 2. -classpath 

This is the obivous option. Jar file or directory path supplied to this option can be absolute or relative to the working directory. 

##### 3. Class path of dependent jar are searched as well.

Let say we run this command
```
java -classpath lib1.jar lib1.MainClass
```

And `lib1.jar` has the following manifest file:
```
Manifest-Version: 1.0
Main-Class: lib1.MainClass
Class-Path: lib2.jar lib3.jar
```

Dependent jar `lib2.jar` and `lib3.jar` will be searched by class loader as well. Thus, there is no need to specify those in `-classpath` argument. 

