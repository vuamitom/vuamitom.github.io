---
layout: post
title: Implicit property getter can be harmful
date: '2019-02-19 19:30:00'
tags:
- technical
---

`Property` as a programming language's feature has been around for a while. I first got to use it while developing a multi-tenant cloud-based [point of sale application](https://www.kiotviet.vn/) on .NET platform. The idea is to avoid the verbosity of calling getter/setter methods by invoking them behind the scene whenever a field is accessed/assigned. 

Modern javascript implementation has introduced [this feature](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get). The code looks like this:

```javascript
var obj = {
  log: ['a', 'b', 'c'],
  get latest() {
    if (this.log.length == 0) {
      return undefined;
    }
    return this.log[this.log.length - 1];
  }
}

console.log(obj.latest);
// expected output: "c"
```

In the above example, property `latest` is evaluated whenever it is accessed. However, the benefit this language feature intends to offer is exactly why it can be problematic. To `obj` owner, `latest` looks just like another field. This unawareness of underlying complexity and cost can lead to performance oversight. The above example is trivial. But in real life, a software project often depends on tens of third party libraries and code from many authors, a property getter can turn out to be a beast. Nobody knows what is going on behind accessing a field until performance deteriorates. 

Consider this code fragment which retreive all direct and non-direct childrens of a binary tree-node in a breadth-first manner: 

```javascript
class Node {
	constructor() {
		this.left = ...; // left child
		this.right = ...; // right child
	}

	get offspring() {
		let l = [this],
			s = 0;
		while(s < l.length) {
			let n = l[s];
			if (n.left) l.push(n.left);
			if (n.right) l.push(n.right);
			s ++;
		}
		return l.slice(1);
	}
}
```

Trustful programmers who use this class may think of `offspring` as a simple field and are inclined to write code that accesses `offspring` multiple times, causing unnecessary loop and array allocation. 

```javascript
let node = ... // somehow create a tree node. 
if (node.offspring.length > 0) {
	for (let c of node.offspring) {
		// do somth
	}
}
```
In my opinion, an explicit method call `getOffspring()` is more desirable despite being more wordy. A method call highlights to the programmer that work would be done at the time of calling, thus prompt actions to inspect how much is the cost and to avoid unncessary method calls. 

Implicit property getters/setters can be helpful if their use is limited to aliasing object fields or lazy evaluation. However, in the wild, there is no guarantee that this language feature is ultilized properly. Unfortunately, this feature has made it into javascript implementation of major browsers. If you can, favor writing a program that is obvious and does not have hidden cost. 