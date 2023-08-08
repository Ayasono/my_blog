---
layout: '../../layouts/MarkdownPost.astro'
title: 'handle write debounce and throttle function'
pubDate: 2023-08-06
description: 'handle write debounce and throttle function'
author: 'AyaSono'
cover:
  url: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSPvTmL1YdTOToOkJ2ZJfeZ2Ki1MsKkHN0i800o4JMr3wEK_EJmCfJQX4EOsIt-o2iR7OQ&usqp=CAU'
  square: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSPvTmL1YdTOToOkJ2ZJfeZ2Ki1MsKkHN0i800o4JMr3wEK_EJmCfJQX4EOsIt-o2iR7OQ&usqp=CAU'
  alt: 'cover'
tags: [ "handle write", "JavaScript", "debounce", "throttle"]
theme: 'light'
featured: false
---

**Warning**

> This is only a naive approach to the problem. It does not care about the special types such as RegExp, Date, Set, Map,
etc. It also does not handle circular references.

```js
function debounce (fn, delay) {
	let timer = null
	return function (...args) {
		if (timer) clearTimeout(timer)

		timer = setTimeout(() => {
			fn.apply(this, args)
			timer = null
		}, delay)
	}
}
```

```js
function throttle (fn, delay) {
	let timer = null
	return function (...args) {
		if (timer) return

		timer = setTimeout(() => {
			fn.apply(this, args)
			timer = null
		}, delay)
	}
}
```
