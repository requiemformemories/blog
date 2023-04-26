---
layout: post
current: post
class: post-template
subclass: 'post'
cover: 'assets/images/text.jpg'
title: '關於在 Ruby 中 unicode string 切成指定 bytes'
date: 2023-04-25 13:00:00
categories: ruby
tags: ruby rails
author: fumitsuki
---

在 Ruby 裡面如果要把字串切成最接近且小於 N bytes，大家可能會看到網路上提到 [byteslice](https://www.rubydoc.info/stdlib/core/String:byteslice) 這個 method。但是 byteslice 是沒在管 encoding 的，例如我想切一個 utf8 string，有機會切出 invalid characters。

我用 byteslice 來把 string 切成 9 bytes，先來切字串 `跳摟下殺價`：

```ruby
str = '跳摟下殺價'
str.byteslice(0, 9)
# => "跳摟下"
```

結果看起來非常正常。接著改切 `限時9折起`：
```ruby
str = '限時9折起'
str.byteslice(0, 9)
# => "限時9\xE6\x8A"
```

`折`直接被切爛了。為什麼會這樣呢？

utf8 每個字元的 byte 長度從 1 byte 到 4 bytes 不等，絕大多數的中文字是 3 bytes。在全部的字元都是漢字的時候切 3 倍數的 bytes 數可能不會注意到問題。但 `限時9折起` 中出現了 `9` 這個只有 1 byte 長度的字串，才導致`折`被切一半。

有一種做法是，把 byteslice 切完以後尾巴切爛的字元清除，透過 [valid_encoding?](https://www.rubydoc.info/stdlib/core/String:valid_encoding%3F) 可以去檢查每個字元是不是 valid 的字元。

```ruby
str.byteslice(0, 9).chars.select(&:valid_encoding?).join
```

或者是計算過到最接近 n byte 的字元時的 bytesize 後再來做 bytesclice 

```ruby
largest_valid_bytesize = (0..str.size-1).to_a.inject(0) do |accu, idx| 
  break accu if accu + str[idx].bytesize > 9

  accu + str[idx].bytesize
end

str.byteslice(0, largest_valid_bytesize)
```

如果是 Rails 的話，Rails 當中直接有 [ActiveSupport::Multibyte::Chars](https://api.rubyonrails.org/classes/ActiveSupport/Multibyte/Chars.html) 可以讓操作者不用理解 encoding 也能安全地對他們做操作。

```ruby
str.mb_chars.limit(9).to_s
```

Rails 真的是什麼都有呢。