---
layout: post
current: post
class: post-template
subclass: 'post tag-speeches'
cover: 'assets/images/sql.jpg'
title: '[SQL] 什麼要 primary key？primary key 一定要叫 id 嗎？'
date: 2020-04-27 06:13:00
categories: SQL
tags: sql
author: fumitsuki
---

## 問題一：可不可以不要 primary key？ 沒有 primary key 會怎樣？
primary key 主要的作用為：

### 1. 避免產生一堆重複又無法識別的 row

例如底下的表格，上一筆跟下一筆長一模一樣，根本無法識別。

|name|price|quantity|
|--|--|--|
|banana|100|1|
|banana|100|1|

### 2. 有辦法撈到特定的 row

如下，每個 row 都長一樣，根本無法撈到特定某個 row。

|name|price|quantity|
|--|--|--|
|banana|100|1|
|banana|100|1|

但是改成這樣：

|id|name|price|quantity|
|--|--|--|--|
|1|banana|100|1|
|2|banana|100|1|

就可以用 `WHERE id = 1` 來撈到第一筆資料

### 3. foreign key reference

比方說這張表是 `orders`

|customer|date|total|
|--|--|--|
|Carol|2020-01-01|2000|
|Carl|2020-01-01|500|
|Carol|2020-01-06|2500|

有另一張表是 `order_items`

|name|price|quantity|
|--|--|--|
|banana|100|1|
|banana|100|1|

如果 `orders` 上沒有 primary key，無法透過在 `order_items` 加上 foreign key 來 reference order。

如果 `orders` 改成這樣：

|id|customer|date|total|
|--|--|--|--|
|1|Carol|2020-01-01|2000|
|2|Carl|2020-01-01|500|
|3|Carol|2020-01-06|2500|

就可以簡單透過在 `order_items` 上加 `order_id`，加上 foreign key 來做 reference

|name|price|quantity|order_id|
|--|--|--|--|
|banana|100|1|1|
|banana|100|1|2|

沒有 primary key 的資料，就像是沒有書名的書，沒有歌名的音樂（欸）**很容易造成整理上的困難。**

## 問題二：如何選擇 primary key？

primary key 有個必帶條件：一定要是**唯一（unique）**的，不可以重複。

所以你通常會找個 unique 的欄位來當 primary key。然而很多大家以為 unique 的東西都沒有 unique，例如身分證字號。（身分證字號在古早年代是有可能不小心重複發的XD）

或者是你看完 spec 以後以為應該要 unique，結果哪一天老闆跑來跟你說：「這個欄位的內容可以重複嗎？」的資料。（這個時候都會 OS：要不是我還不能離職，要不然真想打扁你XD）

像是使用者的 email、電話等等。

如果 primary key 這麼難選擇，選錯就會造成自己日後還得要連夜加班，有的時候不如就用 pseudokey 就好了。那具體來說 pseudokey 是什麼呢？

### Pseudokey (Surrogate key)
基本上是一些刻意塞給每個 row 的資料，key 本身對資料來說沒有任何意義。

我們看回剛剛的 `order_items`：

|id|name|price|quantity|
|--|--|--|--|
|1|banana|100|1|
|2|banana|100|1|

上面的 `id` 對我來說沒有什麼意義可言， `id` 為 2 什麼的對我來說一點意義都沒有。但是有了這個 key 以後我就不用去思考，要用哪些欄位來組合成 primary key 了。

各家 DB 提供 AUTO_INCREMENT、GENERATOR 等功能來讓 pseudokey 自動增加，且多個 client 同時來 DB 塞資料也不會重複

的確很常見到教材裡面，每張表都有用來放一個 id 的 primary key 欄位，會 auto increment，甚至有些 web framework 就直接這樣設計。這個慣例很不錯，但這並不是絕對的。如果有本來就有可以拿來當 primary key 的欄位，直接拿來用也無妨。

|order_id|customer|date|total|
|--|--|--|--|
|1|Carol|2020-01-01|2000|
|2|Carl|2020-01-01|500|
|3|Carol|2020-01-06|2500|

很多 web framework 的設計讓或書籍裡，都會讓每張表的 primary key 叫 id，然而這不是絕對的！比方說把 `products` 這張 table 的 primary key 叫 product_id，`users` 的叫 user_id 也挺好的。而且這種命名法還有一種優點，可以透過 **USING** 來 JOIN 資料表。

當 bugs 的 primary key 叫 bug_id，product 上對應的 foreign key 也叫 bug_id。 除了一般的 `JOIN ... ON ...`：
```SQL
SELECT * FROM bugs
JOIN bug_products ON bugs.bug_id = bug_products.bug_id
```

還可以改寫成：

```SQL
SELECT * FROM bugs
JOIN bug_products USING(bug_id)
```

## 問題三：只有 primary key 無法避免資料重複

如果 code 沒寫好，還是有可能出現很多重複資料。比方說這個是管理訂單跟負責員工的關聯表 `assignments`：

|id|order_id|employee_id|
|--|--|--|
|1|2|330|
|2|2|330|
|3|3|331|

除了去檢查為什麼你的 code 沒有好好地 validate 資料、避免重複，也可以檢討一下 DB 這邊沒有把 **unique key** 加上去的部分。

在新增 table 的時候，可以針對 order_id 和 employee_id 加上 unique key，避免出現組合重複的 order_id、employee_id：

```SQL
CREATE TABLE assignments(
   ...,
   UNIQUE(order_id,employee_id)
);
```

除此之外，直接拿 order_id 和 employee_id 組合成 compound key 來當 primary key 也是很 OK 的作法。


## 問題四：`MAX(id)` vs `LAST_INSERT_ID() `

有些人會自己找 `MAX(id)` (資料庫裡最大的id) 去 generate 新 id。
這樣會有 Race Condition 的問題，當多個 client 同時操作資料庫，有可能不同 client 試圖塞一樣的 id 進來。為了避免 race condition 只好做 table lock，避免同時 insert，但這樣會導致效能瓶頸。

如果是用 AUTO_INCREAMENT ，MYSQL 可以用 LAST_INSERT_ID() 來取最後被 insert 進來的資料 id：

```
SELECT LAST_INSERT_ID() FROM products LIMIT 1;
```

## 結論

 - primary key 是一種 constraint，不是一種 data type
 - primary key 可以是 varchar, integer 等等等，沒有一定要是 integer
 - primary key 可以是 auto_increment 的 pseudokey，也可以自己定義（反正 unique 就好）


像 Rails 的理念是慣例優於設定（convention over configuration），這樣的理念使得大家可以降低開發成本，善用框架本身的設計帶來的便捷功能等等。一般來說就算不需要 Rails mibgration 幫你產生的 id，你也不會特別去拿掉他。

但是這可能會讓新手誤以為 primary key 就是一個叫 id、值為 integer 的欄位，其實不是這樣的。

最後，附上作者的提醒：

> Don't let inflexible conventions get in the way of good design

## 補充：ISO/IEC 11179
針對 metadata 制訂的標準，和其他標準一樣不好閱讀。Joe Celko 在他的書 SQL propgramming style 裡有實際使用這個規範。

---

本文是參考 Bill Karwin 的 SQL Antipatterns 第四章 ID required 彙整而成。原書說明了很多不太好的設計模式，什麼時候可以使用以及為何要避免使用等等。大家有空可以去翻閱看看哦。
