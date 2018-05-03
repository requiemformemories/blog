---
layout: post
current: post
class: post-template
subclass: 'post tag-speeches'
cover: assets/images/rails-post.jpg
title:  "[Rails 學習筆記]善用namespace建立後台"
date:   2016-04-27 16:07:19
categories: Rails
tags: rails 
author: fumitsuki
---
在寫blog的時候一直很苦惱要怎麼寫後台，
是要另外有一個叫backend的controller嗎？還是怎麼樣呢？

後來才知道原來有namespace這東西，覺得人間愉快(X)

所以廢話少說就開始吧！

### route的部分
首先到 **route.rb** 加入：

```ruby
namespace :admin do
    resources :articles
 end
```

這樣index的路徑就換變成 **admin/articles** 了， 
可以用`$ rake routes`來看一下路徑長什麼樣子。

然後這邊要特別注意（大師提點的，不是我自己說的w），
盡量不要用admin、backend這種大家都想得到的名稱。
否則後台很容易被恐怖的大黑黑所攻陷。(黑黑最喜歡戳戳了(́◉◞౪◟◉‵))

當然，article的部分就是controller的名字，要叫posts...什麼的都可以。

### Controller的部分
接下來是contorller的部分，這邊只要有基本的CRUD加上認證就OK了。
（其他功能就自己盡情加吧，那就跟本文無關了w）
可以善用command line來建立controller，像是：`$ rails g controller admin/articles`
這個時候可以發現多出了 **app/controllers/admin/articles_controller.rb** 這個檔案，
到這裡後台的路徑就差不多了。

### 使用者認證
這個部分看是多使用者還是單一使用者，
如果是單一使用者的話可以用簡單的 **http basic authenticate** 來處理。

在剛剛的 **admin/articles_controller.rb** 中加入：

``` ruby
http_basic_authenticate_with name: "your_name", password: "your_password"
```

這邊的`your_name`跟`your_password`就自由填入自訂的帳號密碼吧！
多使用者的話，可以用 **devise** 這個方便好用的套件。

首先，在 **Gemfile** 中加入：

``` ruby
gem 'devise'
```


然後在命令列輸入`$ bundle`就可以安裝套件了。
接著呢，再命令列輸入`$ rails g devise:install`來產生devise設定檔
再輸入`$ rails g devise user`產生user model，然後便忘了`rake db:migrate`來migrate它。Devise也提供view的樣板可以使用，在命令列輸入`rails generate devise:views`就可以自動產生了。


除此之外，
其實也有後台的套件可以用，像說[rails_admin]之類的，可以參考看看呢。


[rails_admin]: https://github.com/sferik/rails_admin