---
layout: post
title:  Floating point binary representation in javascript
date: '2018-12-19 16:16:00'
mathjax: true
tags:
- technical
---

Inspired by [this post](https://floating-point-gui.de/formats/fp/) which explains how computer stores floating point number, Here is a bit of javascript code that print out a float32 number in binary format. Firstly, the main idea is, floating point number is represented similar to scientific representation of numbers using `E` notation.

\\[ 0.07 = 7.0 * 10^{-2} =  7.0E-2 \\]

Although, instead of decimal, computer representation uses binary digit. For example with \\(0.5 = \frac{1}{2}\\):

\\[ 0.5 = 2^{-1} = 1.0 * 2^{-1} = 1.0 E_{2} -1 \\]

Lef of `E` is `significant` part. Right of `E` is `exponent` part. And the actual bit layout is 1 `sign` bit, followed by `exponent` followed by `significant`. The most significant bit (MSB) of `significant` part is always assumed to be 1, and is omitted from the representation.

(1 sign bit) - (8 exponent bit) - (23 significant bit)

For the 0.5 example, from the scientific representation above, we could derive the binary representation as below:

- Sign bit: 0 for positive number
- Exponent = -1. Binary representation = -1 + 127 = 126
- Significant = 1.0. Since only the MSB is 1, which is omitted, significant part should have 23 zero bits. 

This representation has the advantage of flexible position of dot, (hence the name floating point), with the `exponent` part control numbe of decimal places, thus trades off between precision (more decimal places) and top value range. Now let's put out some code to verify this representation. First, to get the underlying byte sequences of a float32 number.

```javascript
var buffer = new ArrayBuffer(4);
new DataView(buffer).setFloat32(0, 0.5);
var bytes = new Uint8Array(buffer);

// Expected output: [63, 0, 0, 0]
```

Assuming big endian, sign bit is obtained by:
```javascript
// 0 - positive, 1 - negative
var sign = (bytes[0] >> 7) > 0 ? '-': '';
```

Get exponent
```javascript
var exp = (bytes[0] << 1) | (bytes[1] >> 7);
// expected: 126 , in binary 01111110
// for float32, minus 127 from raw bits to get signed value
var expVal = exp - 127;
// expected: -1
var expSign = expVal > 0? '': '-';
```

Get the significant

```javascript
// the most significant bit is 1 and is ommited
// So we put it back here
var sig = (((bytes[1] & 0x7F) | 0x80) << 16) | (bytes[2] << 8) | bytes[3];

// expected: 8388608 , which is 10...0 (23 zeros)
```

Putting things together, we have the mathematical binary representation of float literal 0.5. `toBinaryStr` and `toBits` are helper functions listed at the bottom of this post.

```javascript
var mathRep = toBinaryStr(toBits(sig, 24)) + 'E' + expSign + toBits(expVal, 8).join('').replace(/^0+/,'');

//output: 1.0E-1
```

Try it on for another few numbers:

```javascript
toMathString(0.5);
//1.0E-1 = 2 ** -1
toMathString(0.25);
//1.0E-10 = 2 ** -2
toMathString(0.375);
//1.1E-10 = 2 ** -2 + 2 ** -3 

toMathString(0.15)
//1.0011001100110011001101E-11
toMathString(0.1)
//1.10011001100110011001101E-100
```

Numbers like 0.5, 0.25, 0.375 is neatly representable in a binary system. On the other hand, numbers like 0.1, 0.15 are rounded. To get rounding error:
```javascript
//TODO
```

TODO: This post is a work in progress. embed a widget for inspecting floating point

Though this post examines single precision floating point number (32-bit), number in javascript is floating point with double precision (64-bit), minor changes are needed to print out the actual binary representation used by javascript VM. 

## Helper functions

```javascript
// assuming BigEndian
/**
 * getting bit sequence of an input number
 */
function toBits(n, bitCount) {
	var x = n;
	var bs = [];
	for (var b = 0; b < bitCount; b++) {
		x = n >> b;		
		bs.push(x & 0x01);
	}
	return bs.reverse();
}

/**
 * nicely print a binary number. omit trailing 0 after the dot.
 */
function toBinaryStr(bits) {	
	var suffix = bits.slice(1).join('').replace(/0+$/, '');
	suffix = suffix.length == 0? '0': suffix;
	return bits[0] + '.' + suffix;
}
```

## Reference
[https://floating-point-gui.de/formats/fp/](https://floating-point-gui.de/formats/fp/)