---
layout: post
title: Webpack React 入門筆記
excerpt: "學習筆記 - Webpack React 基本工作流程"
modified: 2016-07-08
categories: articles
tags: [sample-post]
image:
  feature: japan.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
comments: true
share: true
---

最近終於有空來摸摸Webpack跟React了，雖然才剛起步，
但我想把Blog當作學習筆記把遇到的問題和解決方法都紀錄起來，
整理文章也有個優點就是會強迫自己去了解細節究竟是什麼原因，
之後如果有寫錯或理解錯誤的地方，
還請各路大神賜教，小妹還算是非常菜渣的開發者XDD 謝謝大家~!

### Reference

* [從無到有一]
* [從無到有二]
* [Gitbook - React入門]
* [Gitbook - Webpack]
* [webpack-howto]
* [WEBPACK入門教學筆記]
* [webpack-tutorial]
* [Babel入門教程]

### What is Webpack?

<figure>
	<a href="https://c8.staticflickr.com/9/8721/28111721471_5f676861c6_h.jpg"><img src="https://c8.staticflickr.com/9/8721/28111721471_5f676861c6_h.jpg" alt="image"></a>
</figure>

Webpack是所謂的模組打包工具，它可以幫你把各種文件(JS、JSX、coffee、less/sass...、圖片）打包成一系列的靜態資源來使用。

當我們在做前端網站時，大家所看到的Html如果像我以前什麼都不懂時做的project - [travel-site] (沒有RWD喔!先說無罪XDD手機看破圖BJ4)，真的會很肌樂，
首先是一直有重複的code, 再來是css沒有架構性超醜，開發時間慢透惹，檔案很大沒有經過壓縮...等等，
而這個project還只是只有靜態的網站!!如果又加了一些需要跟Backend串的功能，則會變得又臭又長又複雜又難維護，
所以你會需要用到一些需要編譯的語法來加速開發時間，例如:

* HTML - 或許會想用Jade
* CSS - 或許會想用Sass/Scss、less
* JavaScript - ES6 用 Babel

用了這些語法之後Browser當然是沒有支援，所以你會需要編譯器將這些語法轉譯成原生語法
才能上production，以下是流程:

* 各種編譯(ex. jade -> html, es6 -> es5 ...)
* 各種打包(ex. 把JS檔打包成bundle.js) 
* uglify(把code壓縮，變得不可識別)

而Webpack正是幫你做這些事的一個模組打包工具，而如果結合React的話，因為virtual dom的特性還能夠做到Live Reload!
真的超酷的!通常用React的人一定會用Webpack，而我最近一直被洗腦React有多好多好，
而且Angular的two way binding使用後發現一些蠻惱人的問題...

# How to Start?

### Install

你需要各種安裝:

Install [nodejs] [nvm]
建議用[nvm] (NPM 套件管理工具) 來安裝Node，如果各個project需要的Node版本都不一樣，
那用NVM就可以非常輕鬆自在地去切換版本，很推薦喔~!

{% raw %}
    $ curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.2/install.sh | bash
{% endraw %}
or wget:
{% raw %}
    wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.31.2/install.sh | bash
{% endraw %}
使用NVM:
{% raw %}
    $ nvm install 6.2.2
    $ nvm use 6.2.2
{% endraw %}

Install [npm] (套件管理工具)
{% raw %}
    $ sudo apt-get install npm
{% endraw %}

安裝webpack
{% raw %}
    $ npm install webpack -g
{% endraw %}

有時候project會因為版本問題有錯誤，我的範例用的版本是:

* Node: `v6.2.2`
* npm: `3.10.3`
* webpack: `1.13.1`

安裝好後我們建一個資料夾叫webpack-react-practive好了
{% raw %}
    $ mkdir webpack-react-practive
    $ cd webpack-react-practive
{% endraw %}

### Package.json

我們需要建立package.json檔，來管理我們的套件
{% raw %}
    $ npm init
{% endraw %}
就按照指令填上你要的就可以了，不知道要填什麼就按Enter, 它會自動升生成一個檔案
<figure>
	<a href="https://c3.staticflickr.com/8/7287/27573932154_5b70cdc1a6_b.jpg"><img src="https://c3.staticflickr.com/8/7287/27573932154_5b70cdc1a6_b.jpg" alt="image"></a>
</figure>

{% raw %}
    {
      "name": "react-webpack-practice",
      "version": "1.0.0",
      "description": "",
      "main": "index.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "author": "",
      "license": "ISC"
    }
{% endraw %}

然後我們可以使用 npm 來安裝套件 YAYA，
你現在的目錄應該長這樣:

* react-webpack-practice/
    * package.json

那現在開始安裝相關套件: 
* webpack, webpack-dev-server
webpack-dev-server 是個小型的 node.js express server，主要用來跑專案內的檔案，同時提供 LiveReload 的功能。

{% raw %}
    npm install webpack@1.13.1 webpack-dev-server@1.14.1 --save-dev
{% endraw %}

你應該會看到你的package.json多了一些行數，並會產生node_modules的folder

這邊解釋一下用法:

* `--save-dev`：用來安裝開發時用的工具(ex babel, webpack, webpack-dev-server...)，會更新到package.json裡的devDependencies(開發依賴)
* `--save`：用於上線時必要的套件(react, bootstrap...)，會更新到package.json裡的Dependencies(上線依賴)
* 如果`npm install <package>` 後面沒有加: 就會自己產生package在node_modules裡，但不會更新到你的package.json

所以通常把專案clone到開發環境，你會使用 npm install 來安裝所有的依賴套件。
但如果clone到正式環境，你會使用 npm install --production 來安裝 Dependencies(上線依賴)的套件。

因為我們會使用Javascript ES6 所以需要再Install babel-core babel-loader babel-preset-react babel-preset-es2015

* babel-core: 如果某些代碼需要調用Babel的API進行轉碼，就要使用babel-core
* babel-loader: 是用來解譯 jsx 與 ES6 語法的 loader
* babel-preset-react: react轉碼規則
* babel-preset-2015: ES2015轉碼規則

想更深入了解Babel -> [Babel入門教程]

{% raw %}
    npm install --save-dev babel-core babel-loader babel-preset-react babel-preset-es2015
{% endraw %}

Install React and React-dom(上線需要)

{% raw %}
    npm install --save react react-dom
{% endraw %}

### 建立webpack.config.js檔

其實這裡我也不太熟，我會就我知道的寫下來，順便發問一下好了XD 如果各路大神知道，還麻煩留言讓我長一下知識，我會很感謝你的!
 
* devtool: cheap-module-eval-source-map，求救XDD
* entry: js的進入點，每個進入點需設定來源，這邊設定了app/app.js當進入點，webpack/hot/dev-server應該就是跟webpack-dev-server會live reload有相關，
再者webpack-dev-server/client?http://0.0.0.0:8080，如果拿掉的話console會一直出現warning:[WDS] Disconnected!，但LiveRelaod不受影響。
* output: 設定路徑在build資料夾內，會打包產生一個bundle.js檔，跟Browserify做的事是一樣的
* resolve 指定可以被 import 的文件檔名。比如 Example.jsx 這樣的文件就可以直接用 import Example from 'Example' 引用。
* loaders 指定 babel-loader 編譯後檔名為 .js 或者 .jsx 的文件，這樣就可以在這類型的文件中自由使用 JSX 和 ES6 了。

{% highlight js %}
    const path = require('path');
    const webpack = require('webpack');
    
    module.exports = {
      devtool: 'cheap-module-eval-source-map',
      entry: ['webpack-dev-server/client?http://0.0.0.0:8080', 'webpack/hot/dev-server', path.resolve(__dirname, 'app/app.js')],
      output: { 
      	path: path.resolve(__dirname, 'build'),
       	filename: 'bundle.js'
      },
      resolve: {
        extensions: ['', '.js', '.jsx']
      },
      module: {
        loaders: [
          {
            test: /\.jsx?$/,
            loaders: ['babel'],
            include: path.join(__dirname, 'app'),
            exclude: /node_modules/
          }
        ]
      },
      plugins: [
        //new webpack.HotModuleReplacementPlugin(),
        new webpack.NoErrorsPlugin()
      ]
    };
{% endhighlight %}

### 建立.babelrc

Babel的配置文件是.babelrc，存放在項目的根目錄下。
該文件用來設置轉碼規則和插件。我們在preset裡加入規則ES2015轉碼規則和react轉碼規則。

{% highlight js %}
    {
    	"presets": [
    		"react",
    		"es2015"
    	],
    	"plugins": []
    }
{% endhighlight %}

你的專案目錄現在會長這樣

* react-webpack-practice/
    * package.json
    * webpack.config.js
    * .babelrc
    * app/
        * app.js
        * content.js
        * header.js
    * build/
        * index.html

### 新增build/index.html檔
{% raw %}
    <!DOCTYPE html>
     <html lang="en">
       <head>
         <meta charset="UTF-8">
         <title>React</title>
       </head>
     <body>
       <div id='app'></div>
       <script src="/webpack-dev-server.js"></script>
       <script src="bundle.js"></script>
     </body>
    </html>
{% endraw %}

### 新增app/*js檔

app.js
{% highlight js %}
    'use strict';
    import React from 'react';
    import ReactDOM from 'react-dom';
    import Header from './header';
    import Content from './content';
 
    class App extends React.Component {
      render() {
        return (
          <div className='app'>
            <Header />
            <Content />
          </div>
        );
      }
    }
    
    if(module.hot) {
      module.hot.accept();
    }
    
    ReactDOM.render(
      <App />, document.getElementById('app')
    )
{% endhighlight %}

header.js
{% highlight js %}
    'use strict';

    import React from 'react';
    
    export default class Header extends React.Component {
    	render() {
    		return (
    			<div className='header'>
    				Header
    			</div>
    		);
    	}
    }
{% endhighlight %}

content.js
{% highlight js %}
    'use strict';
    
    import React from 'react';
    
    export default class Content extends React.Component {
    	render() {
    		return (
    			<div className='content'>
    				Content
    			</div>
    		);
    	}
    }
{% endhighlight %}

### Setting package.json

安裝完後的package.json會長這樣
{% raw %}
    {
      "name": "react-webpack-practice",
      "version": "1.0.0",
      "description": "",
      "main": "index.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "author": "",
      "license": "ISC",
      "devDependencies": {
        "babel-core": "^6.10.4",
        "babel-loader": "^6.2.4",
        "babel-preset-es2015": "^6.9.0",
        "babel-preset-react": "^6.11.1",
        "webpack": "^1.13.1",
        "webpack-dev-server": "^1.14.1"
      },
      "dependencies": {
        "react": "^15.2.1",
        "react-dom": "^15.2.1"
      }
    }
{% endraw %}

把scripts修改一下
{% raw %}
    "scripts": {
        "test": "test",
        "build": "webpack",
        "start": "webpack-dev-server --devtool eval --progress --colors --hot --content-base build"
    }
{% endraw %}

start 內的指令在這邊做個解釋:

* webpack-dev-server 會把server起起來，在localhost:8080就能看見你的專案
* --devtool eval 會顯示出發生錯誤的行數與檔名
* --progress 會顯示出打包的過程
* --colors 會幫顯示的訊息加入顏色
* --content-based build 指向project最終輸出的資料夾

package.json
{% raw %}
    {
      "name": "react-webpack-practice",
      "version": "1.0.0",
      "description": "",
      "main": "index.js",
      "scripts": {
        "test": "test",
        "build": "webpack",
        "start": "webpack-dev-server --devtool eval --progress --colors --hot --content-base build"
      },
      "author": "",
      "license": "ISC",
      "devDependencies": {
        "babel-core": "^6.10.4",
        "babel-loader": "^6.2.4",
        "babel-preset-es2015": "^6.9.0",
        "babel-preset-react": "^6.11.1",
        "webpack": "^1.13.1",
        "webpack-dev-server": "^1.14.1"
      },
      "dependencies": {
        "react": "^15.2.1",
        "react-dom": "^15.2.1"
      }
    }
{% endraw %}

### 可以開始試Run溜~~~

輸入指令:
{% raw %}
    $ npm run start
{% endraw %}

就可以看到你的成果囉~~~
再來你可以把header.js的Header改成Header1就可以看見網頁在不重整的情況下Live Reload了!
這裡是Github上的範例，已經把sass也加進去裡面了! [react-webpack-practice]
{% raw %}
    $ git clone https://github.com/ElaineHuang/react-webpack-practice.git
    $ cd react-webpack-practice
    $ npm install
    $ npm run start
{% endraw %}

### Some Problem

其實我在建置環境的時候遇到幾個問題，說不定大家也會遇到，在這邊分享一下:

我在npm install react 的時候裝不起來!就在想說為蝦米裝不起來的時候!!!!
發現我的package.json name寫react..... package.json的name不要取跟你要裝的套件一樣的名字喔!
哈哈哈哈...(我在卡關的時候倒是笑不出來XDDD)

再來是terminal一直出現這樣的錯誤:

<figure>
	<a href="https://c3.staticflickr.com/8/7605/28155224746_f60fefeb8d_b.jpg"><img src="https://c3.staticflickr.com/8/7605/28155224746_f60fefeb8d_b.jpg" alt="image"></a>
</figure>

後來發現沒加.babelrc，所以react和es2015的轉碼規則就沒有被用到
{% raw %}
    {
    	"presets": [
    		"react",
    		"es2015"
    	],
    	"plugins": []
    }
{% endraw %}

或是你在webpack.config.js裡把module loader改成這樣也可以~~
{% raw %}
    module: {
        loaders: [
          {
            test: /\.jsx?$/,
            loaders: ['babel?presets[]=react,presets[]=es2015'],
            include: path.join(__dirname, 'app'),
            exclude: /node_modules/
          }
        ]
    }
{% endraw %}

謝謝大家~~第一次寫比較長的技術文，希望我以後可以堅持下去，你的留言就是我的動力來源喔!!
也希望如果大家看到有寫錯或是不清楚的地方留言糾正我!我會非常感激你的!! 我也還是初學者而已> <


[從無到有一]: https://medium.com/html-test/%E5%BE%9E%E7%84%A1%E5%88%B0%E6%9C%89%E5%BB%BA%E7%AB%8B-webpack-%E8%A8%AD%E5%AE%9A%E6%AA%94-%E4%B8%80-42fbc76a2d37#.nz3liwypv
[從無到有二]: https://medium.com/html-test/%E5%BE%9E%E7%84%A1%E5%88%B0%E6%9C%89%E5%BB%BA%E7%AB%8B-webpack-%E8%A8%AD%E5%AE%9A%E6%AA%94-%E4%BA%8C-%E8%A8%AD%E5%AE%9A%E6%A8%A3%E5%BC%8F-61c210d63411#.eir55us6h
[Gitbook - React入門]: https://hulufei.gitbooks.io/react-tutorial/content/index.html
[Gitbook - Webpack]: https://fakefish.github.io/react-webpack-cookbook/Introduction-to-Webpack.html
[webpack-howto]: https://github.com/petehunt/webpack-howto/blob/master/README-zh.md
[WEBPACK入門教學筆記]: http://blog.kkbruce.net/2015/10/webpack.html#.V39x9rh96he
[webpack-tutorial]: https://webpack.github.io/docs/tutorials/getting-started/
[travel-site]: https://github.com/ElaineHuang/travel-site
[npm]: https://github.com/nodejs-tw/nodejs-wiki-book/blob/master/zh-tw/node_npm.rst
[nodejs]: https://nodejs.org/en/
[nvm]: https://github.com/creationix/nvm
[Babel入門教程]: http://www.ruanyifeng.com/blog/2016/01/babel.html
[Browserify]: http://browserify.org/
[react-webpack-practice]: https://github.com/ElaineHuang/react-webpack-practice

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