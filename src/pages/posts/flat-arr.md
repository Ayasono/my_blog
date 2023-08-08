---
layout: '../../layouts/MarkdownPost.astro'
title: 'handle write flat array function'
pubDate: 2023-08-06
description: 'handle write flat array function'
author: 'AyaSono'
cover:
  url: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSPvTmL1YdTOToOkJ2ZJfeZ2Ki1MsKkHN0i800o4JMr3wEK_EJmCfJQX4EOsIt-o2iR7OQ&usqp=CAU'
  square: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSPvTmL1YdTOToOkJ2ZJfeZ2Ki1MsKkHN0i800o4JMr3wEK_EJmCfJQX4EOsIt-o2iR7OQ&usqp=CAU'
  alt: 'cover'
tags: ["handle write", "JavaScript", "flat array"]
theme: 'light'
featured: false
---

## handle write flat array function

### solution with for loop
```js
function flat (arr, n) {
	let result = [];
	
	for (let i = 0; i < arr.length; i ++) {
		if (Array.isArray(arr[i]) && n > 0) {
			result = result.concat(flat(arr[i], n - 1));
		} else {
			result.push(arr[i]);
		}
	}
	return result;
}
```

### solution with recursion
```js
function flat (arr, n) {
	if (n === 0) return arr

	const result = []

	for (const ele of arr) {
		result.push(...(Array.isArray((ele)) ? flat(ele, n - 1) : [ele]))
	}

	return result
}
```
