---
layout: '../../layouts/MarkdownPost.astro'
title: 'text highlight optimization'
pubDate: 2023-08-06
description: 'experience from internship'
author: 'AyaSono'
cover:
  url: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSPvTmL1YdTOToOkJ2ZJfeZ2Ki1MsKkHN0i800o4JMr3wEK_EJmCfJQX4EOsIt-o2iR7OQ&usqp=CAU'
  square: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSPvTmL1YdTOToOkJ2ZJfeZ2Ki1MsKkHN0i800o4JMr3wEK_EJmCfJQX4EOsIt-o2iR7OQ&usqp=CAU'
  alt: 'cover'
tags: ["handle write", "JavaScript", "optimization"]
theme: 'light'
featured: false
---

## Text highlight optimization

> The requirement is to highlight the text in the search results, and the highlight color is the same as the search keyword


### Solution with for loop
```js
function highlight (text, keywords = []) {
	for (let i = 0; i < keywords.length; i ++) {
		const keyword = keywords[i]
		const reg = new RegExp(keyword, 'g')
		text = text.replace(reg, `<span class="highlight">${keyword}</span>`)
	}
}
```

### Solution with prefix tree / trie

> Trie is more efficient than for loop if text is long and keywords are many

```js
class TrieNode {
	constructor () {
		this.children = new Map();
		this.isEndOfWord = false;
	}
}

class Trie {
	constructor () {
		this.root = new TrieNode();
	}

	insert (word) {
		let currentNode = this.root;
		for (const char of word) {
			if (!currentNode.children.has(char)) {
				currentNode.children.set(char, new TrieNode());
			}
			currentNode = currentNode.children.get(char);
		}
		currentNode.isEndOfWord = true;
	}
}

// Sample keywords list
const keywords = ['sample', 'keywords'];

// Build the prefix tree
const trie = new Trie();
keywords.forEach(keyword => trie.insert(keyword.toLowerCase()));

// Get the paragraph element
const paragraphElement = document.getElementById('paragraph');
const paragraphText = paragraphElement.textContent;

// Split paragraph into words
const words = paragraphText.split(/\s+/);

// Create a document fragment to build the highlighted content
const fragment = document.createDocumentFragment();

words.forEach((word, index) => {
	const span = document.createElement('span');
	span.textContent = word;

	if (trieContainsWord(trie.root, word.toLowerCase())) {
		span.classList.add('highlight');
	}

	fragment.appendChild(span);
	if (index < words.length - 1) {
		fragment.appendChild(document.createTextNode(' '));
	}
});

// Clear the paragraph and append the fragment
paragraphElement.innerHTML = '';
paragraphElement.appendChild(fragment);

function trieContainsWord (node, word) {
	for (const char of word) {
		if (!node.children.has(char)) {
			return false;
		}
		node = node.children.get(char);
	}
	return node.isEndOfWord;
}
```
