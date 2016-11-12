---
layout: post
title: React Redux-Auth 學習筆記
excerpt: "Flea Market 拼裝車part1"
modified: 2016-07-08
categories: articles
tags: [Redux Auth]
image:
  feature: japan-3.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
comments: true
share: true
---

跳蚤市場的實作用了非常多不同的libary，即所謂的拼裝車，
其中我們為了實作Google OAuth2.0登入，而用了[redux-auth]，非常方便地幫我們做掉了許多事，
完整的google OAuth2的流程可以參考[簡單易懂的OAuth2]，寫得極為詳盡!超級推薦!
這次就來說說我們的實作流程並且簡單的記錄下來。

### Reference

* [flea-market-frontend](https://github.com/FuBoTeam/fubo-flea-market.git)
* [flea-market-backend](https://github.com/FuBoTeam/flea-backend.git)
* [簡單易懂的OAuth2]
* [redux-auth]

### Redux Auth

[redux-auth]把很多元件都包成一個一個component，直接照著官網的文件取用即可，非常方便使用。

Flea Market的example:
{% highlight js %}
  store.dispatch(configure({
    apiUrl: 'http://flea.fubotech.com.tw/',
  }, {
    currentLocation: window.location.href,
    clientOnly: true,
    cleanSession: false,
  })).then(() => {
    // ... render your app ... //
  });
{% endhighlight %}

##### 踩雷
* 之所以加`currentLocation`是為了要解決手機登入時，有些手機的OAuth2.0登入視窗無法正常關閉，會造成登入失敗的問題。
* `clientOnly`與`cleanSession`的設定是因為我們發現網站重新整理時，他還需要再次登入，但是看Cookie是有被存下來的。翻redux-auth的source code發現問題在於它會destroySession!!所以我們必須傳入這兩個設定值就能修復這個問題。

[redux-auth configure.js]:

{% highlight js %}
  if (!settings.clientOnly && !settings.initialCredentials || settings.cleanSession) {
    destroySession();
  }
{% endhighlight %}

除了這兩個大雷包之外，用起來是相當順利，一個component可以幫你做完一堆事，並且幫你把OAuth2.0吐回來的資訊存在store裡，並且把fetch的需要的token資訊都塞好。
{% highlight html %}
<OAuthSignInButton
    provider="google"
    next={
      () => {
        changeLocationOnSignIn(next || '/');
        return;
      }
    }
    style={btnStyle}
/>
{% endhighlight %}

Fetch的header `access-token, uid, client` 都塞好了，再來我們要再去跟我們自己的api要user data。
分為兩種情況:

* user已經登入過的狀態下去要user data。
* user未登入->登入也必須去要user data。

Header/index.js:
{% highlight js %}
  componentWillMount() {
    const { getUser, isSignedIn } = this.props;
    if (isSignedIn) {
      getUser();
    }
  }
  componentWillReceiveProps(nextProps) {
    if (this.props.isSignedIn === false && nextProps.isSignedIn === true) {
      nextProps.getUser();
    }
  }
{% endhighlight %}

而當user被登出時，這個store裡的state必須被清空，發現redux-auth在登出時，會聽action: `SIGN_OUT_COMPLETE` or `SIGN_OUT_ERROR`，所以就利用這兩個action把user state清空。

reducers/user.js:
{% highlight js %}
    const defaultState = {
      isFetching: false,
      id: null,
      fakeName: null,
      error: null,
    };
    
    const graph = (state = defaultState, action) => {
      switch (action.type) {
        case 'GRAPH_READY/USER':
          return {
            ...state,
            isFetching: !action.data,
          };
        case 'GRAPH_DONE/USER':
          return {
            ...state,
            ...action.data.user,
          };
        case 'GRAPH_ERROR/USER':
          return {
            ...state,
            error: action.error,
          };
        case 'SIGN_OUT_COMPLETE':
          return {
            ...defaultState,
          };
        case 'SIGN_OUT_ERROR':
          return {
            ...defaultState,
          };
        default:
          break;
      }
      return state;
    };
    
    export default graph;
{% endhighlight %}

### Google OAuth2.0

這是我對於整個Google OAuth2.0整個流程的理解，如果有錯誤的地方歡迎留言指證喔!謝謝!
建議先看[簡單易懂的OAuth2]，會比較有完整的概念!這邊簡單提一下而已

<script async class="speakerdeck-embed" data-slide="18" data-id="c8317f4038ce013138be5694540c4f3c" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

##### Click Login button
<figure>
	<a href="http://i.imgur.com/YFckPtt.png"><img src="http://i.imgur.com/YFckPtt.png" alt="image"></a>
</figure>
敲backend會導到這個網址，client ID與secret key不會大辣辣的show在網址上(不能被user知道，secret同password)，會進行一些加密與處理(google規定的)再送給google做認證，認證這個client是已註冊過的，
並檢查entry point url與endpoint url有沒有一致，檢查OK進到下一步

{% raw %}
    https://accounts.google.com/AccountChooser
    ? continue=https://accounts.google.com/o/oauth2/auth
      ? access_type=offline
      & scope=email,profile
      & response_type=code
      & redirect_uri=http://flea.fubotech.com.tw/omniauth/google_oauth2/callback
      & state=012556da0137aa8fdf3529aee8a9bac2656ffe89b9448890
      & client_id=1096620245007-0qjobnfluq3fl7kh26tsg0n5joeogqkd.apps.googleusercontent.com
      & from_login=1
      & as=-7532907c15951391
      & btmpl=authsub
      & scc=1
      & oauth=1
{% endraw %}

##### Login and authorized
給user認證是否把資訊給Client, 按下接受後，便會回傳state&code給backend

{% raw %}
    http://flea.fubotech.com.tw/omniauth/google_oauth2/callback
    ? state=012556da0137aa8fdf3529aee8a9bac2656ffe89b9448890
    & code=4/ftz5EqCyOWY9gsx-unQo-qxxafzEtQq94uI67X3DIYY
{% endraw %}
backend或許會自己再去敲google OAuth確認給的認證是否正確

#### What redux-auth have done

##### Before redirect

backend 產生 new token

##### After redirect
backend將new token帶回來給Frontend, Frontend把client, expiry, auth_token, uid存到Cookie,
並且再call validation_token api 在去向backend認證他自己發出去的token是否符合，認證過了backend會再把uid傳回來，frontend則會把backend回的uid塞進header和redux state裡

{% raw %}
    http://localhost:8080/login
    ? auth_token=eoYFpsLr6fwUuNvbTCjNzw
    & blank=true
    & client_id=6g6o83HyjReALJ7dhc_E3g
    & config=default
    & expiry=1479197813
    & uid=111593213638941768034
{% endraw %}

只要backend的token拿回來了並存在cookie裡，之後的傳遞資料就跟OAuth Server沒關係了，就是我們backend自己發出去的的token自己認證而已。
每當Login後Refresh，redux-auth會去拿Cookie裡的資料塞好`client` & `access-token`在header，
去call backend的validation token api，backend會回uid回來，並把response header都塞好(`client & access-token & uid`)，
這時的`redux-auth fetch`已經塞好認證所需的資料了，這時就可以用`fetch`來call需要登入的api惹。

[簡單易懂的OAuth2]:https://speakerdeck.com/chitsaou/jian-dan-yi-dong-de-oauth-2-dot-0
[redux-auth]: https://github.com/lynndylanhurley/redux-auth.git
[redux-auth configure.js]: https://github.com/lynndylanhurley/redux-auth/blob/master/src/actions/configure.js
<div id="disqus_thread"></div>
<script>
    /**
     *  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
     *  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables
     */
    /*
    var disqus_config = function () {
        this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
        this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
    };
    */
    (function() {  // REQUIRED CONFIGURATION VARIABLE: EDIT THE SHORTNAME BELOW
        var d = document, s = d.createElement('script');
        
        s.src = '//elainehuang.disqus.com/embed.js';  // IMPORTANT: Replace EXAMPLE with your forum shortname!
        
        s.setAttribute('data-timestamp', +new Date());
        (d.head || d.body).appendChild(s);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>

<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

  ga('create', 'UA-78158205-1', 'auto');
  ga('send', 'pageview');

</script>