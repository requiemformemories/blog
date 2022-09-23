---
layout: post
current: post
class: post-template
subclass: 'post'
cover: 'assets/images/black-background-key.jpg'
title: '好淺好淺的談談訊息驗證碼 MAC'
date: 2022-09-22 16:00:00
categories: ruby
tags: ruby
author: fumitsuki
---

## 前言閒聊
在串各種金流交易的時候，常常會聽到 MAC 跟 HMAC。以前第一次聽到的時候不知道 MAC 是什麼、HMAC 跟 MAC 差異在哪裡，直接問同事說「MAC 是什麼啊？」，結果被同事鄙視（誤）了一番。同事一邊說著「你不會查 wiki 嗎？」一邊翻了 wiki 頁面給我，結果他打開的是「Minimum Alveolar Concentration」的頁面（最小肺泡分壓之類的，我也不太清楚）。打了這麼長的篇幅，其實只是想笑他 owo （但其實受到同事的照顧很多啦）

但如果有記得他的英文全名 Message authentication code，其實就會比較直觀地知道，有個 code，是個用來做訊息驗證用的資訊，比較清楚他的角色是什麼。

## MAC
訊息驗證碼（Message authentication code，MAC）是一種常見驗證訊息來源、資料完整性（確保訊息沒有被竄改）的方式。

sender 在傳送訊息時會根據與 receiver 約定好的方法，用金鑰跟傳輸資料產出驗證碼，除了要送過去的資料以外帶上這個驗證碼。receiver 收到訊息以後，可以根據之前約定好的方法用同一把金鑰跟傳輸資料檢驗這個驗證碼是不是按照說好的方式產生的，來驗證送訊息過來的人就是 sender，不是別人。

![The example usage of MAC](assets/images/mac_usage.png)

「約定好的方法」可以有很多種，HMAC 就是一種產出驗證碼可以使用的方式。

## 範例

比方說 A 公司 server 和 B 公司 server 要互相傳送訊息，傳輸過程中經過大海般茫茫又不安全的網路，所以他們約定好一把 secret key，每次互送 request 的時用 key 和 payload 用 HMAC-SHA256 算出訊息驗證碼，在 HTTP header 的 `X_HMAC` 帶上（實務上似乎是放 query string 也有）。收到 request 方會需要用同一把 secret key 跟 payload 在產一次 mac，驗證和 header 上帶來的 mac 是否一樣，如果一樣才當作是可以信任的訊息。

## 變動的 IV 值

有些合作廠商可能是使用像是 AES-256-GCM、AES-256-CBC 之類的方式產生訊息驗證碼，產出驗證碼的過程中，除了 key 和  payload 還需要動態產生一組 IV 值，一起帶過去給對方做驗證。如果這個 IV 值每次都送同一組的話，會怎麽樣呢？

我對於密碼學不太熟悉，比較多的說明這裡附上 [stack overflow](https://crypto.stackexchange.com/questions/57645/is-using-the-same-iv-in-aes-similar-to-not-using-an-iv-in-the-first-place) ><

但簡言之，帶同一組 IV 是有風險的。如果攻擊者可以找到明文類似的訊息，或是攻擊者有辦法製造出他想要的明文內容的訊息，有可能可以破解出 key 或是可以竄改訊息，讓訊息驗證碼失去其效用。因此，上述的情境裡不應該使用同一把 IV。
