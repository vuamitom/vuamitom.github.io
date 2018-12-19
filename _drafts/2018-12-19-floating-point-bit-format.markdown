---
layout: post
title:  Floating point binary representation in javascript
date: '2018-12-19 16:16:00'
tags:
- technical
---


Inspired by the good blog post explaining floating point. I decided to do some demo.

(1 sign bit) (8 exponent bit) (23 significant bit)

With the dot point is always between the first and second significant bit.

Let start with a very simple example. 0.5 , which can be represented neatly in binary without rounding error.

Expectation is:




```javascript
var buffer = new ArrayBuffer(4);
new DataView(buffer).setFloat32(0.5);
var bytes = new UInt8Array(buffer);
```
Output is 
[127, 192, 0, 0]

Assuming big endian, get sign bit
```javascript
// 0 - positive, 1 - negative
var sign = bytes[0] >> 7;
```

Get exponent
```javascript
var exp = (bytes[0] << 1) | (bytes[1] >> 7);
// if exp is negative, take 2 compliment to get value
var expSign = '+';
if ((bytes[0] & 0x40) > 0) {
	exp =(0x0100 - exp);
	expSign = '-';
}
```

Get the significant

```javascript
var sig = ((bytes[1] & 0x7F) << 16) | (bytes[2] << 8) | bytes[3];

```

## Reference
[https://floating-point-gui.de/formats/fp/](https://floating-point-gui.de/formats/fp/)