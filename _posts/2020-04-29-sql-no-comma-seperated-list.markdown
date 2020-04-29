---
layout: post
current: post
class: post-template
subclass: 'post tag-speeches'
cover: 'assets/images/sql.jpg'
title: '[SQL] 為什麼要多一張表處理多對多關係？把數值都逗號分隔存起來不行嗎？'
date: 2020-04-29 06:13:00
categories: SQL
tags: sql
author: fumitsuki
---

想當初小的在學校學 SQL 的時候，從來沒有仔細想過為什麼的多對多關聯都要再多用一個 table 來處理。

如果直接把資料想辦法塞在一個欄位裡，比方說在 table `users` 上新增 `orders` 欄位（type 為 `VARCHAR`），把 order_id 們像是 `1, 34, 135` 這樣地塞在 `orders`欄位中，不是也可以嗎？（或是反過來新增 `users` 欄位在資料表 `orders` 上）

|id|name| .... | orders |
|-|-|-|-|
|1|Grace|....| 2, 14, 65, 11|
|1|Carol|....| 3, 2, 1, 12, 100|

乍聽之下，這個想法真是聰明，這樣就可以省得新增一個表格呢！

但是今天就要為大家破除這個迷思，一但試了一次這個方法，**未來恐怕後患無窮**。

**未來恐怕後患無窮**。

**未來恐怕後患無窮**。

因為很重要所以說三次（咦）

大家務必三思以後再考慮這樣設計 schema 呢。


## Problem 1: 當我想要在 `users` 裡 `WHERE order_id = 11`，卻發現行不通。

如果我~~沒有採用這大膽的想法~~乖乖地把 user 跟 order 用另一張表 `user_orders` 來整理他們之間的多對多關係，就可以像這樣查找 id 為 11 的 order 對應到哪些 user：

```sql
SELECT users.* FROM users JOIN user_orders
                          ON (users.id = user_orders.user_id)
WHERE order_id = 11
```

但如果今天是上面那個存 comma-seperated list 的做法呢？

這個時候只好改寫醜一點的 SQL query 來查找你要的 users，比方說想辦法用 REGEXP 去 match 它：

```sql
SELECT * FROM users WHERE orders REGEXP '[[:<:]]11[[:>:]]'
```

一方面要提心吊膽自己的 REGEXP 沒寫法，會 match 到不該 match 的，另一方面還要擔心資料有不符合規範的東西，例如 `1, 2, 4, 11aaaa, 12` 等等。自己 match 資料時出錯的風險也大大提高。

除此之外，查找 order_id 的部分也無法加 index 來提升效能。

## Problem 2: 很難反過來找使用者的訂單資料

接著，如果我今天想要找`id` 為 1 的 user 的訂單資料，如果是乖乖新增第三張表的時候，可以簡單地 join table 解決。如果是現在這個 comma-seperated list 的作法，就只好再來寫醜醜的 SQL：

```sql
SELECT * FROM users JOIN orders
                    ON users.orders REGEXP '[[:<:]]' || orders.id || '[[:>:]]'
WHERE users.id = 1
```

是否開始感覺到為了一時輕率的選擇，造成的後果要用一世來彌補（欸）

然而問題還沒完，讓我們望向第三個問題。

## Problem 3: 不能用`SUM()`、`AVG()` 等等 function 來做 aggregate queries

如果今天你想要統計每個使用者有多少訂單，如果是好好地使用第三張表的做法，就可以簡單地使用 `SUM()` 來找出訂單數量：

```sql
SELECT SUM(user_orders.order_id) FROM users JOIN user_orders
                                            ON users.id = user_orders.user_id
GROUP BY users.id
```

但是用了 comma-seperated list 以後，這些資料不是分開一個 row 一個 row，而是全部擠在某個 user 的一個欄位裡。這個時候要計算訂單數量只好用一些奇妙的方法來解決。

比方說，如果保證每個使用者一定有對應的帳單的話，就可以透過「逗點的數量」來計算有幾筆訂單。

- `12`: 一筆訂單
- `12, 27`: 兩筆訂單
- `12, 27, 33`: 三筆訂單
- `12, 27, 33, 65`: 四筆訂單

也就是說，逗點數量 + 1 會是訂單數量。於是就可以透過 (總字數) - (去掉 `,` 的字數) 來計算有幾個逗號。

```sql
SELECT length(orders) - length(REPLACE(orders, ',','')) +1 FROM users
WHERE user_id = 5;
```

## Problem 4: 順序問題

想到這個 comma-seperated list 的做法的朋友們，可能會想說這個做法最簡單的地方在於，可以簡單地 append 新的 order_id 在欄位的最後面。比方說，id 為 5 的 user 原本的 orders 是 `11, 14, 18`，如果要建立他跟 id 為 20 帳單的關係，只要在 `11, 14, 18` 後面加上逗號與 20 就好，變成 `11, 14, 18, 20` 這樣。

但是如果今天正好要加進來的 id 是 `5`，就會變成 `11, 14, 18, 5`，經過日日夜夜的累積，你的 orders 變得順序雜亂。哪天需要照順序的 id 的時候，只好一個 row 一個 row 的資料抓出來排序QQ

這個時候你或許會想說：「哦哦，那我再把它塞進去資料庫之前，先把新加的 `5` 放到正確的位置！這樣就沒問題了吧？」

的確是如此，這個時候必須要跑兩個 query，一個把舊資料撈出來，下一個把你把 `5` 加好的資料塞回去。只是當初好好地新增一張表，現在就可以用一個 SQL 就新增新的 order_id，查詢時也可以用 ORDER BY 無痛解決排序問題。

## Problem 5: 刪除特定 id

再來下一個難題是，該如何刪掉特定的 id 呢？

一樣要跑兩個 query，一個把資料撈出來，自己寫程式找出 id 在哪，去掉以後再把新字串塞回去。

開始感覺到做出了錯誤的抉擇，會讓日後的維護很阿雜XD

## Problem 6: 資料驗證

用了 comma-seperated list 的做法以後，因為 order_id 的資料不是一個個 row 獨立存起來的，已經無法透過建立 constraint 來管理資料的正確性與完整性。當 SQL 不再幫你檢驗 constraint，如果程式那邊沒有檢查好，到時候連自己都不會知道欄位被亂塞了什麼東西（eg. `3, 12, 1, banana`）

## Problem 7: seperator 被當成 string 的一部分

假設今天 order_id 有可能使用者自訂的字串，今天有個壞心眼的使用者，把 `a,b,c` 當成 order_id。於是這個 user 的 orders 就多了 `a,b,c`。當你要解析這個使用者有哪些訂單時，`a,b,c` 就很開心地被解析成 `a`,`b`,`c`，然後使用者永遠都無法正常地找回他的資料。

於是大家可能會開始想，我就用一個正常人不可能拿來用的字元當 seperator 吧！或者是在存入 order_id 時就限制不可以用 `,` 這個字元。的確是可以這樣修正，但如果是功能都開發下去、被使用了才出事，中途搶救也會很麻煩。

## Problem 8:長度限制
畢竟是用一個欄位來存 order_id 們，就要特別注意長度限制。比方說 `user_id` 是 `VARCHAR(255)`，如果裡面塞長度為 9 的字串，可以塞到 25 個才會爆炸。於是你只好想辦法說服你的客戶，一個 user 就是不能有太多 order。

這個時候，你心中的小惡魔可能會冒出來跟你說：

> 齁齁齁，有什麼關係，用 TEXT 我可以塞到 65535 bytes 呢 ヽ(́◕◞౪◟◕‵)ﾉ
> TEXT 不夠我可以用 LONGTEXT 呀 (☝ ՞ਊ ՞）☝

拜託....拜託不要亂來（欸

以上已經提出這種方法的八個疑慮了，就不要再用邪魔歪道與大量 workaround 來處理事情了吧。

## 如何發現這個 Antipattern

當專案小夥伴問你：

1. 「這個 list 最多會有幾個 entries 呢？」
   代表他已經在算 VARCHAR 長度要給多少了 ヽ(́◕◞౪◟◕‵)ﾉ

2. 「用 SQL 要怎麼 match word boundary 呢？」
   極有可能他正在思量怎麼從 list 裡挑出某個 entry (例子裡的 order_id) ><

3. 「這些 entries 裡會出現哪些符號呢？有什麼絕對不可能出現的特定符號嗎？」
   代表他已經在考慮 seperator 要用哪個符號了 (☝ ՞ਊ ՞）☝


## 使用這個 pattern 的合理情況

1. 比方說你有個欄位要顯示 comma-seperated list，你不會需要取這個 list 裡的單一 item，這個時候你可能會考慮反正規化一下，加了欄位放這個 comma-seperated list

2. 從別的 source 剛取回來的資料，正好有 comma-seperated list。在你 process 這些資料之前可能會考慮原封不動地存起來。

## 解方：intersection table

就跟以前學校老師講得一樣（欸
乖乖地新增一個 table 來存其他兩張 table 多對多的關係。

|user_id|order_id|
|-|-|
|1|2|
|1|3|


## 補充：REGEXP
參考 [Regular Expressions - MYSQL](https://dev.mysql.com/doc/refman/5.7/en/regexp.html#operator_regexp)

### `[=character_class=]`, `[:character_class:]`

在剛剛內文裡有用到的：

### `[[:<:]], [[:>:]]`
可以用來查找有沒有包含特定字串：

```sql
mysql> SELECT 'a word a' REGEXP '[[:<:]]word[[:>:]]';   -> 1
mysql> SELECT 'a xword a' REGEXP '[[:<:]]word[[:>:]]';  -> 0
```

---

本文是參考 Bill Karwin 的 SQL Antipatterns 第二章 Jaywalking 彙整而成。原書說明了很多不太好的設計模式，什麼時候可以使用以及為何要避免使用等等。大家有空可以去翻閱看看哦。
