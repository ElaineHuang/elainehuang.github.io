---
layout: post
title: Automation Test - Webdriver.io
excerpt: "Webdriver.io 入門筆記"
modified: 2017-02-03
categories: articles
tags: [Webdriver.io, Automation Test]
image:
  feature: japan-5.jpg
comments: true
share: true
---

在試過 [Nightwatch]，決定再來試試 [WebdriverIO] 來比較一下兩者差異，
目前看起來 [WebdriverIO] 非常吸引我的一點是他能夠使用現在火紅的framework像是[Mocha]，
還有[Chimp]，[官網](http://webdriver.io/)上寫著Extendable & Compatible 並且 support 
TDD and BDD test frameworks 這一點真的是它的優勢，API看起來又比 
[WebDriverJs](https://github.com/SeleniumHQ/selenium/wiki/WebDriverJs)簡單很多，
 [Slant](https://www.slant.co/topics/2814/~node-js-selenium-webdriver-client-libraries-bindings)
上排名第一耶，一定要試一下!特別感謝 [Alincode] ，她寫的自動化測試系列文章真是我入門時的好幫手!

### Reference

* [Setting up Selenium tests with Webdriver.io](https://medium.com/@boriscoder/setting-up-selenium-tests-with-webdriver-io-cc7fc3c86629#.xzd52mm3x)
* [alincode webdriver.io系列文章](http://alincode.github.io/blog/all-categories/) - 前端測試
* [JSDC 2016 alin webdriver.io slideshare](http://slides.com/alincode/jsdc2016_webdriverio#/)

### WebdriverIO Get Started

My Repo: [WebdriverIO Example](https://github.com/ElaineHuang/webdriverio_example.git)

應該能很輕易地發現我是forked from [alincode/jsdc-tw-2016-webdriverio](https://github.com/alincode/jsdc-tw-2016-webdriverio.git)，
變更的部分是多了幾個範例，並再加了一些有的沒的。

### Set up

##### Install [WebdriverIO]

你可以直接安裝全域，也可以直接clone我的[repo](https://github.com/ElaineHuang/webdriverio_example.git)執行 `npm install`

{% raw %}
    npm install -g webdriverio
    or
    npm install webdriverio --save-dev
{% endraw %}

它有Tool可以自動幫你generate config檔，這次的範例選用的是[Mocha](https://mochajs.org/)
<figure>
	<a href="https://camo.githubusercontent.com/c3a1ec4c4926b1bc534c6a1c104ec4052d4c5b25/687474703a2f2f7765626472697665722e696f2f696d616765732f636f6e6669672d7574696c6974792e676966"><img src="https://camo.githubusercontent.com/c3a1ec4c4926b1bc534c6a1c104ec4052d4c5b25/687474703a2f2f7765626472697665722e696f2f696d616765732f636f6e6669672d7574696c6974792e676966" alt="image"></a>
</figure>

##### Install [Babel]

{% raw %}
    npm install babel-plugin-transform-runtime babel-preset-es2015 babel-preset-stage-2 --save-dev
{% endraw %}

Add .babelrc
{% raw %}
    {
      "plugins": ["transform-runtime"],
      "presets": ["es2015", "stage-2"]
    }
{% endraw %}

##### Install [Mocha]

如果你的config檔，`framework: 'mocha',`，要記得安裝它。

{% raw %}
    npm install wdio-mocha-framework --save-dev
{% endraw %}

##### Install wdio-selenium-standalone-service

{% raw %}
    npm install wdio-selenium-standalone-service --save-dev
{% endraw %}

wdio.conf.js
{% raw %}
    // Test runner services
    // Services take over a specific job you don't want to take care of. They enhance
    // your test setup with almost no effort. Unlike plugins, they don't add new
    // commands. Instead, they hook themselves up into the test process.
    services: ['selenium-standalone'],
{% endraw %}

它的用途是當你執行`wdio wdio.conf.js`時它會自動幫你把selenium這個control server叫起來，
你就不需要自己手動去開啟它。你把services那行註解掉不用的話，
你就會自己需要起一個selenium的server起來，你可以安裝:
{% raw %}
    npm install selenium-standalone -g
    selenium-standalone install
{% endraw %}
並且把它起起來
{% raw %}
    selenium-standalone start
{% endraw %}
再執行`wdio wdio.conf.js`，他們的效果是一樣的。

### Add jshint

上篇有介紹過，就不再贅敘，有一點不一樣的是設定。
Mocha要設定為true, 並且加上globals變數設定。

.jshintrc
{% raw %}
    {
      ...
      "mocha": true,
      "globals": {
        "browser": true
      }
    }
{% endraw %}

### Add [Chai](http://chaijs.com/)

> Chai is a BDD / TDD assertion library for node and the browser that can be delightfully paired with any javascript testing framework.

在這裡設定chai的語法，寫spec的時候就能完美結合!

wdio.conf.js
{% highlight js %}
  // Gets executed before test execution begins. At this point you can access all global
  // variables, such as `browser`. It is the perfect place to define custom commands.
  before: function(capabilities, specs) {
    const chai = require('chai');
    global.expect = chai.expect;
    chai.Should();
  },
{% endhighlight %}

`.should.be.equal` 就是chai的語法喔!非常直觀!
{% raw %}
    describe('Page title', () => {
        it('should be set by the Meteor method @watch', () => {
          browser.url('http://google.com');
          browser.getTitle().should.be.equal('Google');
        });
    });
{% endraw %}

### Install [Chimp]

如下圖!!你可以完全理解他有多好用!!
<figure>
	<a href="https://raw.githubusercontent.com/xolvio/chimp/master/images/realtime.gif"><img src="https://raw.githubusercontent.com/xolvio/chimp/master/images/realtime.gif" alt="image"></a>
</figure>

我覺得蠻常用到的所以裝了全域。
{% raw %}
    npm install chimp --save-dev
    npm install -g chimp
{% endraw %}

Execute
{% raw %}
    chimp --mocha --watch --path=test
{% endraw %}

他會去尋找你的testcase敘述中有出現`@watch`的，便在他的監控範圍內。
{% highlight js %}
  describe('Page title', () => {
    it('should be set by the Meteor method @watch', () => {
      browser.url('http://google.com');
      browser.getTitle().should.be.equal('Google');
      // browser.element('input#lst-ib.gsfi').setValue('Taroko Software');
      // browser.submitForm('#tsf');
    });
  });
{% endhighlight %}

### TestRunner

你可以在config設定suite，以便可以設定一套一套執行。

wdio.conf.js

{% raw %}
    ....
    suites: {
        demo: ['./test/specs/demo/demo.spec.js', './test/specs/demo/demo2.spec.js'],
        github: ['./test/specs/github/github.spec.js']
    },
    ....
{% endraw %}

可以用以下的指令來呼叫它

{% raw %}
    wdio wdio.conf.js --suite demo
    wdio wdio.conf.js --suite github
{% endraw %}

### [Allure] Report

精美的測試報告就靠它了!它的Report真得比Nightwatch要好很多!

{% raw %}
    npm install wdio-allure-reporter allure-commandline --save-dev
{% endraw %}

wdio.config

{% raw %}
    reporters: ['spec', 'allure'],
    reporterOptions: {
        allure: {
            outputDir: 'allure-results'
        }
    },
{% endraw %}

執行一次測試後，會產生allure-results這個資料夾，再執行`allure report generate allure-results`，
會再產生html檔到allure-report這個資料夾，這時可以再執行`allure report open`它就會自動把結果打開了。

{% raw %}
    npm run test
    ./node_modules/.bin/allure report generate allure-results
    ./node_modules/.bin/allure report open
{% endraw %}

### Page Object Pattern

這個概念再[上篇文章](http://blog.elaine.me/articles/nightwatch/)是一樣的，只是換種寫法。
用法的話我覺得[官網的範例](http://webdriver.io/guide/testrunner/pageobjects.html)就寫得很清楚了，所以就不加以贅敘。
這裡我踩的雷比[Nightwatch]少很多，很多看著官網照著做就完全沒有任何問題，
讓我對它蠻有信心的。

如果有更好的寫法，歡迎交流討論喔!謝謝!再次感謝[Alincode]!!快變小粉絲了XD

[Nightwatch]: http://nightwatchjs.org/
[WebdriverIO]: http://webdriver.io/
[Mocha]: https://mochajs.org/
[Chimp]: https://github.com/xolvio/chimp
[Alincode]: http://alincode.github.io/blog/
[Babel]: https://babeljs.io/
[Allure]: http://allure.qatools.ru/

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

  ga('create', 'UA-88441714-1', 'auto');
  ga('send', 'pageview');

</script>