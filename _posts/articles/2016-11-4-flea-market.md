---
layout: post
title: React Project - Flea Market
excerpt: "Flea Market 拼裝車總覽"
modified: 2016-07-08
categories: articles
tags: [React]
image:
  feature: japan-3.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
comments: true
share: true
---

距離上篇Webpack技術文已過了4個月XD
趕快交代一下最近的React進度，要學新框架果然還是直接實作比較有效果，
這也是我做`Flea Market`這個網站的原因，我們公司的跳蚤市場一直以來都是同仁們帶東西來賣，
要賣的東西旁邊會附一張下標單，所以要下標就是直接現場手寫下標，我是公司的福委，
前一次的跳蚤市場是我主辦的，我那時一直覺得這件事可以電子化，做成一個網站，大家用手機電腦就可以下標了，
還可以解決4點截標，3:55後大家就會聚在那間擺放物品的會議室，搶著當3:59下標的最後得標者XD
這個構想我跟g0v的大大[劉邦皓]討論過，他想跳坑XD(誤)，再加上拉了以前project的技術leader [Orga Shih]，
也同意跳坑(他說寫Rails他就跳wwwww)，還有想學Rails的同事Alice，我們4人小團隊誕生啦!

> 想學React，又有想做的網站!還有神人隊友!!!那還等什麼!!!!衝一發RR!!!!!

### Functional Conception

我們的網站主要有幾個feature是團隊開會一起討論出來的:

<figure>
	<a href="http://i.imgur.com/MFXQpaJ.png"><img src="http://i.imgur.com/MFXQpaJ.png" alt="image"></a>
</figure>
<figure>
	<a href="http://i.imgur.com/gP3PYAU.png"><img src="http://i.imgur.com/gP3PYAU.png" alt="image"></a>
</figure>

因為網頁的短期目標為公司的跳蚤市場，所以我們有綁公司的mail(mail一定要是`happygorgi.com`才能登入)，
所以可能無法讓外面的人看到全部的成品，只有主頁跟下標頁能看到，其它只能靠截圖了QQ 之後到v2.0考慮把這一部分露出去，
加入群組的概念...有做出來再來po文XD 我們是10/20舉辦跳蚤市場，所以現在所有的商品都已經結標了，
圖片都是半透明。

<div markdown="0"><a href="http://flea.fubotech.com.tw/" class="btn">Demo</a></div>

User賣的商品頁:
<figure>
	<a href="http://i.imgur.com/CEAsZAv.png"><img src="http://i.imgur.com/CEAsZAv.png" alt="image"></a>
</figure>

User下標過的商品頁:
<figure>
	<a href="http://i.imgur.com/aC0vUEU.png"><img src="http://i.imgur.com/aC0vUEU.png" alt="image"></a>
</figure>

上傳商品頁:
<figure>
	<a href="http://i.imgur.com/OcQNV16.png"><img src="http://i.imgur.com/OcQNV16.png" alt="image"></a>
</figure>

RWD完整支援手機喔!!

<figure class="half">
	<img src="http://i.imgur.com/zd7RWJU.jpg" alt="image">
	<img src="http://i.imgur.com/TiqEv2k.jpg" alt="image">
</figure>
<figure class="half">
	<img src="http://i.imgur.com/heP16LP.jpg" alt="image">
	<img src="http://i.imgur.com/F73YOuz.jpg" alt="image">
</figure>

##### 短期重要的feature

為了解決4點結標大家都拚3:59分下標，我們訂了一個結標規則:

* 只要有人在結標前3分鐘下標就會觸發延長結標時間，之後下標的金額一定要比最高下標金額高出10塊錢才能下標成功。
* 延長時間會最後會收斂成１分鐘：3, 3, 3, 1, 1, 1, 1, 1...
* 4:00結標:<br />
    在3:57~4:00有任何人下標，結標時間延長至4:03。<br />
    在4:00~4:03有任何人下標，結標時間延長至4:06。<br />
    在4:03~4:06有任何人下標，結標時間延長至4:09。<br />
    在4:06~4:09有任何人下標，結標時間延長至4:10。<br />
    在4:09~4:10有任何人下標，結標時間延長至4:11。<br />
    在4:10~4:11有任何人下標，結標時間延長至4:12。<br />
    在4:11~4:12有任何人下標，結標時間延長至4:13。<br />
    在4:12~4:13有任何人下標，結標時間延長至4:14。<br />

### Frontend - React

為什麼用React呢? 其實這樣的網站是相當輕量級的，功能也不多，那為什麼要套React呢?
因為潮R(誤XD)，其實我們真正的目的是要學東西的，我們並不是為了要做出這個網站才有這個project，
所以整個概念是顛倒的，一般正常公司的產品應該要是針對這個產品去挑選它合適的框架，
但我們不是公司，我們只是一個小團隊寫了一個open source的project，以學習為動機，
選定了想學習的框架，然後做出這個網站當做成果展示，順便解決公司這個問題。

我們的Frontend很潮的用了`React+Redux`，Backend也很潮的用了`graphql`... <br /><br />
所以我們會需要...React接graphql api比較方便的framework...像是`Relay`??
咦!? `Relay`跟`Redux`的概念完全不一樣，不太適合，試試`redux-graphql-middleware`?
我們要用Google OAuth2.0登入，所以用個`redux-auth`好了。
需要router!!來個`react-router`!!
要支援手機上傳，要支援drop圖片這個功能，那用了`react-dropzone-component`好了...哇哩要怎麼上傳? 接imgurl的api好了...<br /><br />
很好，拼裝車來惹，難免會發生銜接困難的問題，像是`redux`和`Relay`就不太適合擺一起，
所以我們研究許久之後決定換掉!還好有大神[劉邦皓]解決了這個問題，
找到另一個`redux-graphql-middleware`，他甚至已經練到準備要貢獻回去的程度了，簡直是猛!
公司跳蚤市場的舉辦日期比我們預期早了很多，所以我們做到後面還蠻趕的，用了自己不熟悉的技術，
再加上framework互相配來配去的技術文章其實還不多，所以各種踩雷，還好有神一般的隊友!
差點要為deadline捏把冷汗!

##### Frontend用到的tool

* [react-redux](https://github.com/reactjs/react-redux)
* [react-router-redux](https://github.com/reactjs/react-router-redux)
* [redux-auth](https://github.com/lynndylanhurley/redux-auth)
* [redux-graphql-middleware](https://github.com/gtg092x/redux-graphql-middleware)
* [react-dropzone-component](https://github.com/felixrieseberg/React-Dropzone-Component)
* [react-css-modules](https://github.com/gajus/react-css-modules)
* [postcss-cssnext](https://github.com/MoOx/postcss-cssnext)
* [eslint](https://github.com/eslint/eslint)

之後有時間會慢慢補上學習筆記:

* [React Redux-Auth 學習筆記](http://elainehuang.github.io/so-simple-theme/articles/FleaMarket-online/)

賣一下g0v神人隊友[劉邦皓Github](https://github.com/orgs/FuBoTeam/people/ben196888)

> frontend repo: [https://github.com/FuBoTeam/fubo-flea-market.git](https://github.com/FuBoTeam/fubo-flea-market.git)

### Backend - graphql

我主要介紹Frontend，Rails只知道他們用了graphql-ruby哈哈哈，只好期盼Orga大神寫技術文啦~

* [graphql-ruby](https://github.com/rmosolgo/graphql-ruby)

附上神人[Orga Shih Github](https://github.com/sinorga)，貌似是想練習用graphql才跳坑XD
不是我在說R!!他超罩搭!!神人隊友之我都要哭了。

> backend repo: [https://github.com/FuBoTeam/flea-backend.git](https://github.com/FuBoTeam/flea-backend.git)

[劉邦皓]: https://www.facebook.com/ben196888
[Orga Shih]: https://www.facebook.com/sinorga