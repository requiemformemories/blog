---
layout: post
title:  "[Rails學習筆記] 用Settingslogic把重要資訊藏起來!"
date:   2016-05-12 12:33:54
categories: Rails
tags: hide route authenticate settingslogic
image: /images/pic03.jpg
---
首先來碎碎念一下XD
最近花了好多時間在寫自己的blog，
其實一直很想把它推到Github上(為了綠綠的attribute!!)，
但是又怕後台路徑跟帳號密碼會被看光光，其實煩惱了一陣子XD

經過大師提點(?)之後才知道原來這是可以解決的呢XD
要解決這個問題有兩種方法，
一種是把設定值放在ENV，
另一種則是用settinglogic來解決這個問題。
這邊會提到的是後者~

(正文要開始了w)

### 安裝gem

首先呢，先在 **Gemfile** 中加入：

```
gem  "settingslogic"
```

然後在terminal執行 `$ bundle` 就安裝完成了。

### 使用Settinglogic
這個gem的詳細說明可以看github上的[README]。
那這邊就簡單介紹初步的使用~

在 **app/model** 中加入 **settings.rb**，
裡面加入以下的內容：

```
class Settings < Settingslogic
  source "#{Rails.root}/config/application.yml"
  namespace Rails.env
end
```

那code中的 `application.yml` 就是等一下要放後台路徑跟帳密的地方~

接著務必記得把 **application.yml** 加到 **.gitignore** 中，
否則所有努力都是白費的噢~~~

```
$ echo "config/application.yml" >> .gitignore
```

### 後台路徑
上次在文章 [善用namespace建立後台] 中有提到後台路徑的作法，
這邊要更進階的把後台路徑藏起來~

打開 **config/route.rb** ，原本的後台路徑應該是：

```
 namespace :admin do
  # ...
end
```

這邊要把它改成：

```
 namespace :admin, path: Settings.admin_path_prefix do
  # ...
end
```

這裡的 'admin_path_prefix` 到時候會放在 **application.yml** 存後台路徑。
(可以自己取，要叫 `backend_path` 什麼的隨意~)
所以接著就在 **config/** 中加入 **application.yml**，裡面加上：

```
defaults: &defaults
development:
  <<: *defaults
  admin_path_prefix: your_admin_path
test:
  <<: *defaults
  admin_path_prefix: your_admin_path
production:
  <<: *defaults
  admin_path_prefix: your_admin_path
```

這裡的 ` your_admin_path` 就是後台路徑~ 
可以根據不同的環境自訂後台~
如果 `$ rails s` 的時候會出錯，建議檢查一下有沒有多打少打什麼，還有縮排正不正確。

### http basic authetication
常常用 http basic authetication 都是在 controller 中加入：

```
http_basic_authenticate_with name: "my_name", password: "my_secret"
```

如果這行跟著 controller 一起被推上github repo的話，肯定會被黑黑攻陷的！(誤)
所以也要想辦法放到 application.yml 中。
所以先把現在這行改成：

```
http_basic_authenticate_with  Settings.http_basic_auth.to_h
```

這裡的 `http_basic_auth` 就是會在 **application.yml** 中加入的帳密，
`.to_h` 會把 `http_basic_auth` 轉成雜湊。

接著呢，在 **application.yml**加入`http_basic_auth` ：

```
defaults: &defaults
development:
 <<: *defaults
 admin_path_prefix: your_admin_path
 http_basic_auth:
    :name: your_name 
    :password: Your_password
test:
 <<: *defaults
production:
 <<: *defaults
admin_path_prefix: your_admin_path
 http_basic_auth:
    :name: your_name 
    :password: Your_password
```

這樣就大功告成了！接著只要開啟server確定有好好運作就OK了~
照理來說後台路徑會變成 `your_admin_path/your_controller_name` 噢~

到這裡告一段落~ 趕快來試試看吧！

[README]: https://github.com/binarylogic/settingslogic
[善用namespace建立後台]: http://fumitsuki-blog.herokuapp.com/articles/backend-with-namespace