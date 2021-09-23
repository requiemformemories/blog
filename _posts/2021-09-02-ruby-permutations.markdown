---
layout: post
current: post
class: post-template
subclass: 'post'
cover: 'assets/images/permutations.jpg'
title: 'ruby 的 permutation'
date: 2021-09-02 9:00:00
categories: ruby
tags: ruby
author: fumitsuki
---

不知道大家刷題的時候是不是有寫過 permutation，例如我有一個 array `['a', 'b', 'c']`，我需要回傳的結果是這幾個元素的所有排列，例如

```ruby
["a", "b", "c"]
["a", "c", "b"]
["b", "a", "c"]
["b", "c", "a"]
["c", "a", "b"]
["c", "b", "a"]
```

一般來說會試著窮舉所有可能性。
底下自己試著實作了遞迴版和迴圈版（解題方面弱弱的，有什麼建議都可以跟我說QQ

<script src="https://gist.github.com/requiemformemories/305600eaf585d3467536aa266497e462.js"></script>


結果發現 ruby 的 array 竟然有 [permutation](https://www.rubydoc.info/stdlib/core/Array:permutation) 有這個神奇的 method。

```ruby
['a', 'b', 'c'].permutation.to_a
# => [["a", "b", "c"], ["a", "c", "b"], ["b", "a", "c"], ["b", "c", "a"], ["c", "a", "b"], ["c", "b", "a"]]

['a', 'a', 'b'].permutation.to_a
# => [["a", "a", "b"], ["a", "b", "a"], ["a", "a", "b"], ["a", "b", "a"], ["b", "a", "a"], ["b", "a", "a"]]
```

搭配 uniq 或 to_set 就可以去除重複的結果，不需要自己很辛苦的寫扣窮舉，ruby 拿來刷題果然太奇怪了XD
