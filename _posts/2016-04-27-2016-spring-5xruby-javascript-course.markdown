---
layout: post
current: post
class: post-template
subclass: 'post tag-speeches'
cover: 'assets/images/5xruby-post.jpg'
title:  "[5xRuby實習日記] Javascript 課程心得"
date:   2016-04-27 16:02:15
categories: Javascript
tags: 5xruby
author: fumitsuki
---
覺得在五倍紅寶石實習的好處就是有各種免費的課程資源，跟喝不完的免費飲料(X)
不過由於個人還在就學的緣故，
因為衝到期中考跟各種學校事務，一直沒能把課程上好上滿QAQ"

這次的JS課程很遺憾地只上了最後一堂課QAQ" (而且只有後半段QAQ) 
(沒有HTML只上到第一堂課這麼淒慘就是了)
我覺得這次的確有對jQuery有比較多的了解，以往都是一直複製別人的code亂改，
終於可以了解每一行code都在做什麼，然後完整地寫出一段code來。

我覺得Kuro講師提供很多可以參考的例子提供觀摩跟練習，這一點真的有讓我覺得比較易學、易懂。
而且可以實際上了解要怎麼使用，有什麼時機可以使用。

接著提供一下最後一堂課的筆記：
### 善用find找子元素
```
$(this).find('.title').toggleClass('active');
```
### 用trigger觸發事件
例如：

```
	$(.b).on('click',function(){
	$(.a).trigger('click');
});
```

這樣觸發的是a的click事件，因此如果a的click事件中有this，會的是a自己而不是b

### 利用data存取、指定
- 存取 ex.

```
	$('.home').data('show')
```

如此一來可以存取有home這個class的物件的data-show的值
- 指定 ex.

```
	$('.home').data('show','home')
```

把有home這個class的物件的data-show的值改成home


剩下晚一點再來補，不過我有寫在[hackMD]上提供參考

[hackMD]: https://hackmd.io/s/rk_X2cdl
