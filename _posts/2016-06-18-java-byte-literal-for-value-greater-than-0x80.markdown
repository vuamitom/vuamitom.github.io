---
layout: post
title: Java byte literal for value greater than 0x80
date: '2016-06-18 15:09:01'
tags:
- technical
---

In porting a piece of code from C++ to Java, I encountered statement like this: 

```language-cpp
uint8_t a = 0x84; 
```

`uint8_t` in one single byte unsigned integer. In Java, the equivalent single byte primitive type would be `byte`. However, `byte` type is signed. Since Java employs the notion of negative hex literal, the valid value range for `byte` would be [-0x80, 0x7F]. The line of code below would refuse to compile. 

```language-java
byte a = 0x84;  // out of range 
```
To work around it, we can go with either of these option. 

1) Take two compliment of the literal to get its absolute value and apply negative sign. 
```language-java 
byte a = -((1 << 8) - 0x84)
```

2) Try to work with larger-storage type. 
```language-java
int a = 0x84 & 0xFF 
```

