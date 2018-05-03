---
layout: post
current: post
class: post-template
subclass: 'post tag-speeches'
cover: 'assets/images/rails-post.jpg'
title:  "[Rails 學習筆記]用Vagrant打造舒適美好的rails環境吧！"
date:   2016-05-26 15:40:58
categories: Rails
tags: rails
author: fumitsuki
---

相信這個世界上有許多人和我一樣，窮窮的買不起mac，又沒有可以用linux的腦袋跟耐心，所以只好一直被windows慘虐QAQ"
到目前為止在下都是用c9.io來寫rails的，但畢竟不是在本機寫，又沒錢買付費版，所以一直都要等c9.io的workspace緩緩慢慢的開啟，每個指令也都緩緩慢慢的。只是下個 `rake db:migrate` 之類的CPU就會爆衝(下次開c9可以看右上角的值)，感覺這個環境根本不堪負荷啊...

所以呢，想說用Vagrant來建虛擬機，這似乎是比較簡單的方法呢。對於我這個小腦殘來說應該不會耍雷得太嚴重XD

其實網路上Vagrant的教學文還不少(而且還是中文的)，遇到問題還可以上stackoverflow查，個人覺得資源蠻多的。如果是docker的話我個人絕對沒有信心把它裝起來XD

不過說實在的，心底還是默默的期望可以買mac啊><" 

總而言之廢話不多說(已經講很多了)就來裝裝看吧！

BTW，在下的電腦目前是萬惡的 windows 10，所以安裝過程可能跟大家不太一樣，細節可以參考底下的教材。然後安裝過程中的問題跟雷點我也會一併分享的w

### 安裝步驟

#### 安裝Vagrant、Virtual Box、MobaxTerm

首先來安裝等等會需要用到的東西：

- [Vagrant]
	- 依照自己的作業系統環境找適合的版本安裝
- [VirtualBox]
	- VirtualBox platform packages
	- VirtualBox 5.0.20 Oracle VM VirtualBox Extension Pack
- [MobaXterm]
	- 可以拿來連telnet、ssh、http、ftp...等等的，很方便
	- 這次會拿來連ssh

安裝過程中只要一直按 `next` 基本上就會裝好了XD
個人安裝沒有遇到什麼問題，有遇到的話可以google一下解決辦法。

#### 生出虛擬機
接著到[Vagrant Cloud]物色一下要安裝的box，個人這次用的是 `ubuntu/trusty64`。
找到需要的版本以後會看到這樣一行command line，像這樣：

```{}
$ vagrant init ubuntu/trusty64; vagrant up --provider virtualbox
```

這個時候只要愉快地把這行指令貼到terminal上，虛擬機就生出來了呢！這種時候就會覺得Vagrant真是強大呢！

不過第一次開會跑很久很久很久，請耐心等待XD

#### ssh連線
接著差不多跑玩了的時候，稍微翻一下terminal上的訊息，應該會有類似這樣的訊息：

```{}
default: SSH address: 127.0.0.1:2222
default: SSH username: vagrant
default: SSH auth method: private key
```

到virtualbox那邊看，多了一個虛擬機。
![new virtualbox]

把MobaXterm打開以後，到session新增ssh session，在host、username、port填入：
![new session]

在左邊側欄對 `127.0.0.1(vibrant)` 點兩下，照理來說會出現類似 `vagrant@vagrant-ubuntu-trusty-64:~$` 的東西，或是要你輸入password。 
沒有的話，到 **Settings > Configurations > Terminal** 把 `Use Windows PATH environment`打勾。

預設的username和password都會是vagrant。


#### 問題: VT-x/AMD-V 硬體加速在您的系統不可用
我個人就是遇到這個狀況，虛擬機開不起來。
這個時候可以檢查一下虛擬化技術有沒有開起來，這個部分可以到 **工作管理員 > CPU** 去看。
![虛擬化技術]
英文版的應該會寫virtualization之類的。

如果上面不是寫著「已啟用」的話，那就到BIOS去把virtualization technology 開起來吧！怎麼把BIOS開起來可以google一下。

至於windows10的話，就一邊按著shift一邊重新啟動，出現藍藍畫面後選 **進階選項 > UEFI韌體設定 > 重新啟動** 。
接著電腦重新啟動後就可以來改設定了！按F10進去改設定，在advanced應該會有一個叫virtualization technology的選項，把它改成enabled，就OK了！
再按F10電腦就會愉快的開機了，這一次虛擬機應該開得起來了。

#### 回復原來的狀態
如果過程中搞砸了(?)或是覺得虛擬機髒髒的越看越不順眼(?)，都可以還原成一開始的狀態呢。

這裡就試著回復成乾淨的狀態吧。

windows的話可以照著這個做：

```{}
$ vagrant halt
$ vagrant destroy --force
$ del /F /Q  Vagrantfile
$ rmdir /S /Q  .vagrant
 
$ vagrant box remove ubuntu/trusty64
$ rmdir /S /Q %USERPROFILE%\.vagrant.d\boxes\ubuntu-VAGRANTSLASH-trusty64
```
mac的話可以參考以下的寫法：

```{}
$ vagrant halt ; vagrant destroy --force
$ rm -rf .vagrant Vagrantfile
 
$ vagrant box remove ubuntu/trusty64
$ rm -rf $HOME/.vagrant.d/boxes/ubuntu-VAGRANTSLASH-trusty64
```

#### 建立環境
到這裡，我們已經學會怎麼把虛擬機開好了噢！而且還知道怎麼把它砍掉呢！可是網羅(?)還沒有結束，懂得把虛擬機開起來還要懂得安裝環境才行XDD 
開起來的乾乾淨淨的虛擬機是沒有git、ruby...什麼都沒有的XD 所以還是要把環境架起來才能開心寫rails(淚顏)

所以就來重新建一次虛擬機吧，不過這一次用人家架好的環境XDD
原本看gorails上有一連串複雜有可怕的步驟，覺得頭暈目眩。後來發現github上有人家弄好的環境，最後一比commit是1小時前(還有在更新中)，而且star數也有一千多，就毫不猶豫拿來用了www

這一次的話就：

```{}
$ git clone https://github.com/rails/rails-dev-box.git
$ cd rails-dev-box
$ vagrant up
```

就有人家弄好好的環境可以用了，覺得開心愉快。
等到跑了好久好久以後，就有一台要git有git、要有ruby有ruby的虛擬機了~

#### 把rails架起來
不過rails-dev-box裡面有的只有寫rails會用到的東西，但是沒有rails！
對，沒~有~rails~~

所以還是要自己裝一下XDD

```{}
sudo gem install rails --no-ri --no-rdoc
```

到這裡恭喜大家，環境架得差不多了！
祝大家玩耍愉快！


### 相關教材
- [Vagrant Tutorial（1）雲端研發人員，你也需要虛擬機！]
- [Vagrant Tutorial（2）跟著流浪漢把玩虛擬機]
- [Vagrant Tutorial（3）細說虛擬機生滅狀態]
覺得以上的教材講得其實很詳細，除了安裝步驟還有很多的介紹跟閒聊w

- [在 Windows 用 Vagrant 快速建立你的 Linux 環境]
- [Using Vagrant for Rails Development]
gorails 上的教學，不過這篇就比較複雜一些了。
- [Rails dev box]


[Vagrant]: https://www.vagrantup.com/downloads.html
[VirtualBox]: https://www.virtualbox.org/wiki/Downloads
[MobaXterm]: http://mobaxterm.mobatek.net/download.html
[Vagrant Cloud]: https://atlas.hashicorp.com/boxes/search?utm_source=vagrantcloud.com&vagrantcloud=1
[new virtualbox]: http://i.imgur.com/0VPr8U6.png
[new session]: http://i.imgur.com/ZF8sTSt.png
[虛擬化技術]: http://i.imgur.com/75QAWrA.png
[Vagrant Tutorial（1）雲端研發人員，你也需要虛擬機！]: http://www.codedata.com.tw/social-coding/vagrant-tutorial-1-developer-and-vm
[Vagrant Tutorial（2）跟著流浪漢把玩虛擬機]: http://www.codedata.com.tw/social-coding/vagrant-tutorial-2-playing-vm-with-vagrant
[Vagrant Tutorial（3）細說虛擬機生滅狀態]: http://www.codedata.com.tw/social-coding/vagrant-tutorial-3-vm-lifecycle
[在 Windows 用 Vagrant 快速建立你的 Linux 環境]: https://gist.github.com/chgu82837/ab1255846b5335407105
[Using Vagrant for Rails Development]: https://gorails.com/guides/using-vagrant-for-rails-development
[Rails dev box]: https://github.com/rails/rails-dev-box