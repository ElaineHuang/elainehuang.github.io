---
layout: post
title: My First Collection - BigFace
excerpt: "My First Collection - 大臉仔"
modified: 2016-05-08
categories: articles
tags: [sample-post]
image:
  feature: so-simple-sample-image-1.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
comments: true
share: true
---

最近終於擠出時間來整理自己的作品惹，這個作品是我大三的時候寫的，css的全部是自己造輪子，沒有套用任何framework，RWD也是自己刻的(流汗)，一整個很肌樂的作品...
我那時做這個作品時就只是最單純的html檔玩玩而已，但現在回頭看code真的寫超醜搭!!!但我現在也沒時間再改惹(裝死)，還想留時間碰碰React跟webpack咩!~(合理化懶蟲的行為)大家就將就將就點吧哈哈XD

### Harp

首先來介紹Harp這個東東，首先為什麼會找到這個tool是因為，我的作品在local端看起來完全沒有問題，
但推到github的gh-pages就壞掉啦~!!首先是音樂出不來，再來是破圖的問題，但我其實不曉得到底出了什麼問題，
畢竟在local看是好好的呀~後來google了一下發現Harp這個簡單的tool，Harp其實並不有名，但我自己覺得很好用啦~
簡單的來說Harp就是:

> The static web server with built-in preprocessing

* [Harp home page]
* [Harp introduction]
* [stack overflow]  6 樓

> Preprocessors are built-in
Harp already includes the preprocessors you love: Markdown, Jade, EJS, LESS, Stylus, Sass and CoffeeScript, are all available without any configuration or setup necessary.

是不是超級方便!!他支援Jade, LESS, SASS正好是我需要DER
就決定是你惹!!最重要的是可以Compile後Deploy到github把demo site起起來

> Whether you’re making a GitHub project page, or a mobile application using Apache Cordova/PhoneGap, you can easily compile your code to HTML, CSS & JavaScript and host it anywhere.

### Install Harp

先介紹我用的開發環境，我使用的是[Cloud 9]上的VM喔，非常適合Developer使用。

1. 安裝Harp `$ sudo npm install harp -g`
2. `$ git clone your_project_repo` 
3. 進入你的project `$ cd your_project_name`
4. Initialise a new Harp app `$ harp init _harp` (記得命名前要加個底線，這樣Deploy到Github時Source file才不會被傳上去)
 
目錄結構像是這樣
your_project_name/_harp/

* _layout.jade
* 404.jade
* index.jade
* main.less

### Start web server

your_project_name/ 底下執行: `$ harp server _harp` 你就可以看見最基本的頁面了!

> 補充: 在c9.io的環境中想要看到自己的demo site該怎麼做呢?
你可以發現c9.io有幫你設定兩個變數，在terminal中可以看到，分別是`$PORT=8080`與`$IP=0.0.0.0`，在c9.io中你的port一定要開在8080($PORT)才會管用喔!
所以你的指令會變成`$ harp server _harp --port $PORT`，並開啟網址: https://`vm_name`-`your_name`.c9users.io/

### Compile and Deploy

1. Compile your Harp app: `$ harp compile _harp ./` 幫你生成靜態的HTML, css, js檔
2. 接下來就可以Deploy到GitHub上去囉~!! `$ git add -A` `$ git commit -m 'Harp'` `$ git push` 

> 補充: 記得再開一個branch命名為gh-pages, 這樣github才會產生demo site呦~網址可以去gh-pages/Settings查看喔!

### After all the talking, where is your demo site?

<div markdown="0"><a href="http://elainehuang.github.io/BigFace/index.html" class="btn">Demo</a></div>

[Harp home page]: http://harpjs.com/
[Harp introduction]: http://kennethormandy.com/journal/start-a-blog-with-harp
[stack overflow]: http://stackoverflow.com/questions/15718649/how-to-publish-a-website-made-by-node-js-to-github-pages
[Cloud 9]: https://c9.io/