---
layout: post
current: post
class: post-template
subclass: 'post'
cover: 'assets/images/fan.jpg'
title: 'BigSur 上開發 Rails， Rails 把 CPU 吃好吃滿？！'
date: 2021-09-01 9:00:00
categories: rails
tags: rails
author: fumitsuki
---

不知道大家換到 BigSur 有沒有遇到比較舊的 Rails 專案會讓 mac 發燙起飛的狀況。
看一下 top 發生了什麼事，發現沒有在幹嘛的 Rails app 竟然把 CPU 吃好吃滿（咦

![top 截圖](assets/images/listen_high_cpu_usage_1.png)

原本我以為是 M1 有什麼特別的 issue，查了一下發現原來不少換到 BigSur 的人都有這個困擾

會發生這個狀況是因為開發時會監控 file change 的 gem `listen` 沒有套用到 Darwin 的 adapter，fall back to polling 形成高 CPU usage 的現象。
 
[https://github.com/guard/listen/issues/478](https://github.com/guard/listen/issues/478)

至於為什麼沒有套用到 Darwin-adapter 呢？因為之前的 regex 不包含 darwin20 XD

會需要把 listen 升級至 3.3.0 以上，才不會有筆電起飛的困擾


```ruby
# Gemfile
gem 'listen', '~> 3.3'
```


