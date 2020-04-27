---
layout: post
current: post
class: post-template
subclass: 'post tag-speeches'
cover: 'assets/images/tailwind-post.jpg'
title:  "[工作日常] 如何用建立巢狀的 tailwindcss config？"
date:   2019-08-22 06:13:00
categories: CSS
tags: css javascript
author: fumitsuki
---

## 內容更新

Tailwind 在 v1.2.0 的時候推出了 [Allow plugins to extend the user's config](https://github.com/tailwindcss/tailwindcss/pull/1162)的功能，我在公司的技術文章中「[還在跟複雜的 CSS 的設定奮鬥嗎？用 Tailwind 來幫你實現真正的高效整潔！](http://5xruby.tw/posts/tailwind-css-plugin/)」有提到這個功能，與 Tailwind 的基本介紹。

## 前言

最近因為工作緣故使用到 [Tailwind CSS](https://tailwindcss.com/) 這款 css 框架。

有別於 bootstrap 把 各種元件都定義好（按鈕、導覽列、card...），Tailwind 提供比較基礎的 utility classes，讓大家可以根據自己的需求來刻元件、切版型。個人蠻喜歡這樣的工具的，要不然在大家都用 bootstrap 的年代，到處網站都一個樣，看得實在是蠻不舒服的w

Tailwind 本身就提供一些 default settings，像是 `margin: 0.25rem`（`.m-1`），或是 `color: #fff`（`.text-white`） 等等。如果需要客製化，也可以在 `tailwind.config.js ` 中新增顏色等等，而 Tailwind 會自動把 default config 和你的 customized config 合併在一起，搭配 purgeCSS 幫你把沒有用到的 css 設定拔掉，煞是方便呢。

## 可不可以建立 nested config？

然而工具雖方便好用，總是會遇到 basic usage 無法滿足的需求的狀況。比方說一個網站分成前台、後台，兩邊想用不一樣的 customized config。與此同時，有一些共用的設定（比方說兩邊都要有 `primary` 這組 color）兩邊都要有，這個時候該怎麼辦呢？

### 方案一：佛系解決問題，等待官方處理的那一天

如標題所示，緣份到來，問題自然解決。說不定哪一天官方的 new release 就幫你把問題處理好了。

~~如果對專案放置 play 問題就會解決，我也是很想這樣。~~

### 方案二：一邊一組 customized config

是個非常直觀簡單的解決方案，直接前台一個 `tailwind.config.js `，後台一個 `tailwind.config.js `，共同的部分就兩邊的 config 都放就好了。但是總覺得不夠 dry (Don't repeat yourself)，維護的時候要小心保持兩邊的共同設定都一樣，實在不是個漂亮的作法。

### 方案三：把 tailwind 裡面的 ResolveConfigObjects() 拿來 merge 你的 customized config

如果打開 `node_modules/tailwindcss` 觀察觀察，會發現檔案 `resolveConfig.js` 中，Tailwind 透過一個叫 resolveConfigObjects() 的 function 幫你把你的 customized config 跟 Tailwind 的 default config 合併起來。

```javascript
// node_modules/tailwindcss/resolveConfig.js

const resolveConfigObjects = require('./lib/util/resolveConfig').default
const defaultConfig = require('./stubs/defaultConfig.stub.js')

module.exports = function resolveConfig(config) {
  return resolveConfigObjects([config, defaultConfig])
}
```

那如果我們把這個 resolveConfigObjects() 抽出來用，是不是也可以達到 nested config 的效果呢？

```javascript
const resolveConfigObjects = require('tailwindcss/lib/util/resolveConfig').default
const defaultConfig = require('#{共同設定的那個 config 檔案}'.js)
const extendedConfig = { //這裡放你前台 or 後台的設定 }

module.exports = resolveConfigObjects([extendedConfig, defaultTheme])
```


經過實際測試後，發現會有難以解決的問題。

在 Tailwind 的 config 中，有一些部分的設定會像這樣：

```javascript
textColor: theme => theme('colors')
```

經過 resolveConfigObjects() 這個 function 以後，這樣的 function 會被處理成 Object：

```javascript
textColor: {
      transparent: 'transparent',
      black: '#000',
      white: '#fff',
      gray: [Object],
      red: [Object],
      orange: [Object],
      yellow: [Object],
      green: [Object],
      teal: [Object],
      blue: [Object],
      indigo: [Object],
      purple: [Object],
      pink: [Object]
}
```

如果這個時候在你的前台 config 裡加上一個新的顏色：

```javascript
const extendedConfig = {
  theme: {
    colors: {
      ...defaultTheme.theme.colors,
      sakura: '#f2a5b7',
    },
  },
}
```

textColor 是不會更新的！如果在網頁當中有 `.text-sakura`，會因為沒吃到設定而壞掉。

看來還是沒辦法漂亮地解決問題呢。

### 方案四：先把你的 customized config 都 deep merge 過

一樣分成共同的 config，還有前台、後台的 config。但是自行用 deepmerge 套件或 lodash 的 defaultsDeep() 把 customized config 合併起來。

```javascript
const deepMerge = require('deepmerge')
const baseTheme = require('#{共同設定的那個 config 檔案}')
const extendedTheme = {
  //這裡放你前台 or 後台的設定
}

const customizedTheme = deepMerge(baseTheme, extendedTheme)

module.exports = customizedTheme
```

這四種方案（其實只有三種？）中，比較推薦這種呢。

### 顏色的 shallow merge 問題

在 Tailwind 中，customize 你的設定基本上分兩種方式：

1. 蓋掉原本的設定（直接放到 theme 底下）

以下方的 code 為例，md 和 lg 的設定被更動成 666px 和 888px，同時 default 的其他設定也不復存在。比方說原本的 `sm: '640px'` 設定就沒有了。

```javascript
module.exports = {
  theme: {
    screens: {
      md: '666px',
      lg: '888px',
  	},
  },
}
```

2. 延展原本的設定（放到 theme.extend 底下）

和剛剛類似的範例，只是改成放在 extend底下。此時 md 和 lg 的設定被更動成 666px 和 888px之餘，其他 default 的設定依舊保持。比方說原本的 `sm: '640px'` 還會存在。

```javascript
module.exports = {
  theme: {
    extend: {
      screens: {
        md: '666px',
        lg: '888px',
  	  },
    },
  },
}
```

但很奇妙地，Tailwind 的顏色設定無法好好的 extend。以底下的範例來說，我們的想像是 gray 原本的設定(100 ~ 900)都在，但奇蹟似地都會壞光光（欸

```javascript
module.exports = {
  theme: {
    extend: {
      colors: {
        gray: {
          primary: '#9f9e9f',
        },
      },
    },
  },
}
```

這個時候必須再加上一行 `...#{你的 defaultTheme 的名字}.theme.colors.gray`，把 default 的資料代回來。

```javascript
module.exports = {
  theme: {
    extend: {
      colors: {
        gray: {
          ...defaultTheme.theme.colors.gray,
          primary: '#9f9e9f',
        },
      },
    },
  },
}
```

根據官方在 [github issue](https://github.com/tailwindcss/discuss/issues/242) 上的說法，似乎是因為設定還只是 shallow merge 而已。

這個問題就佛系等待官方解決吧。

#### variants 重複問題
如果同時在上層跟下層加上一樣的 variants 會發生什麼事情呢？
```javascript
// 上層：
module.exports = {
  variants: {
    borderColor: ['hover']
  },
}
// 下層：
module.exports = {
  variants: {
    borderColor: ['responsive', 'hover']
  },
}
```

因為兩個 config 是我們自己用 deepmerge 弄的，所以合併完會變成：

```javascript
borderColor: [ 'responsive', 'hover', 'hover' ]
```

接著你就會發現所有 hover 的 borderColor 的 css 都變成兩倍（哭）

目前想到的方式就是 variants 還是固定寫某一個檔案就好這樣。但願之後 Tailwind 官方會有更好的實務做法。

（筆者使用 tailwindcss v1.0.1）
