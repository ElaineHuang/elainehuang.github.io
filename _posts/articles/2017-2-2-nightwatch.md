---
layout: post
title: Automation Test - Nightwatch
excerpt: "Nightwatch 入門筆記"
modified: 2017-02-02
categories: articles
tags: [Nightwatch, Automation Test]
image:
  feature: japan-4.jpg
comments: true
share: true
---

最近的任務是要Survey一下好用的Automation Test Framework, 以Node來開發為準，
看了幾個框架後決定來寫寫看Nightwatch和webdriver.io，以判斷哪種比較好用。
我非常謝謝[線上讀書會](https://github.com/onlinereadbook/bookreactjs/blob/master/%E8%AE%80%E6%9B%B8%E6%9C%83%E9%81%8E%E5%BE%80%E5%BD%B1%E7%89%87%E5%8F%8A%E8%A8%8E%E8%AB%96%E5%8D%80/README.md)
，我對於Automation Test並不是非常有經驗，之前寫過的是以Python為主的Robot Framework(而且只寫了幾個禮拜)，
這次以Node為主算是新的嘗試，身旁也沒什麼人可以問的，還好有影片教學可以讓我快速入門(鞠躬)。

### Reference

* [第一個 Nightwatch 自動化測試](http://alincode.github.io/blog/2016/04/21/start-with-nightwatch/)
* [End to End testing of React apps with Nightwatch](https://www.syncano.io/blog/testing-syncano/)
* [Custom commands and before(), after() hooks](https://www.syncano.io/blog/end-to-end-testing-of-react-apps-with-nightwatch-part-2/)
* [線上讀書會 - andy 主講 nightwatch & react 實例演練](https://www.youtube.com/watch?v=xZ9k_9kvI_Q)
* [線上讀書會 - 海門 主講 nightwatch 進階](https://www.youtube.com/watch?v=kuHpi_GsfDc)
* [認識 Selenium](https://learngeb-ebook.readbook.tw/intro/selenium.html)

### What is selenium? What is Nightwatch?

Selenium 是為Browser Automation需求所設計的一套工具，讓程式可以直接驅動瀏覽器進行各種網站操作。
Selenium 可以利用不同瀏覽器的Driver來控制瀏覽器，並且能夠模擬使用者在網站上的行為，
例如點擊按鈕或連結、操作網頁表單資料、取得網頁內容並進行檢驗，可以滿足相當多的測試需求。
這裡可以直接看海門大大的影片了，講得蠻清楚的! 可以跳到2m26s。

<iframe width="480" height="360" src="https://www.youtube.com/embed/kuHpi_GsfDc?t=2m26s" frameborder="0" allowfullscreen></iframe>

### Getting Started

My Repo: [Nightwatch Example](https://github.com/ElaineHuang/nightwatch_example.git)

應該可以很容易地發現我是fork Andy的[Nightwatch Example](https://github.com/bbandydd/nightwatch_example.git)入門的。
變更的部分是多加了幾個範例，並把遇到的雷改掉，等等會一一說明。

##### Set up

{% raw %}
    npm install
    npm install -g nightwatch
{% endraw %}

##### Execute

Run main.js
{% raw %}
    npm run start
{% endraw %}

Run github.js
{% raw %}
    npm run github
{% endraw %}

Run all tests
{% raw %}
    npm run test
{% endraw %}

Run all tests on Firefox
{% raw %}
    npm run firefox
{% endraw %}

### What I changed?

##### Add jshint

不管是什麼專案，jshint總要加的，先來加加~
{% raw %}
    npm install jshint --save-dev
{% endraw %}

新增 `.jshintrc` 檔
{% raw %}
    {
      "bitwise":true,
      "curly":true,
      "noempty":true,
      "nonew":true,
      "undef":true,
      "node":true,
      "supernew":true,
      "validthis":true,
      "sub":true,
      "loopfunc":true,
      "quotmark":"single",
      "indent":2,
      "newcap":true,
      "trailing":true,
      "esnext": true
    }
{% endraw %}

Excute
{% raw %}
    npm run jshint
{% endraw %}

##### Add nightwatch-html-reporter

reporter.js 已經寫好了一個可以輸出html reporter的程式，
要記得安裝它喔!

{% raw %}
    npm install nightwatch-html-reporter --save
{% endraw %}

裝好後要記得在nightwatch的指令後面加上 `--reporter reporter.js`
{% raw %}
    nightwatch --test test/main/main.js --reporter reporter.js
{% endraw %}

##### Install selenium-server-standalone-jar chrome-driver-standalone geckodriver

這是我遇到的雷之一，用Mac執行一切都很順利，當換成ubuntu的時候，
因為Browser版本問題，driver版本問題，或是Selenium的版本問題，
執行起來會有錯誤，後來看了讀書會的影片後，發現這些檔案可以用npm裝起來，
並把路徑指過去。

{% raw %}
    npm install selenium-server-standalone-jar chrome-driver-standalone geckodriver --save
{% endraw %}

裝好後把`nightwatch.conf.js`改成以下:

{% highlight js %}
    var os = require('os');
    var selenium = require('selenium-server-standalone-jar');
    var chromeDriver = require('chrome-driver-standalone');
    var geckoDriver = require('geckodriver');
    
    var config = {
        ...
        "selenium": { // downloaded by selenium-download module (see readme)
            "start_process": true, // tells nightwatch to start/stop the selenium process
            "server_path": selenium.path,
            "host": "127.0.0.1",
            "port": 4444, // standard selenium port
            "cli_args": { // chromedriver is downloaded by selenium-download (see readme)
                "webdriver.chrome.driver": chromeDriver.path,
                "webdriver.gecko.driver": geckoDriver.path
            }
        },
        ....
{% endhighlight %}

##### Using Page Objects

我們在寫測試程式時會盡量想把spec (main/main.js, github.js) 寫的很容易懂，
盡量不會出現一長串的selector像是
`#rso > div:nth-child(3) > div > div:nth-child(5) > div > h3 > a`，
重點是看完你還不知道這到底是頁面的哪一個element。
再者是把頁面的操作行為精簡化，取一個合適的method name再放進spec裡，
讓人一看到spec就馬上可以了解一個testcase做了什麼樣的事。
所以page object做了兩件事:

* 把頁面的 selector 都集中在一起，用好懂的名字表示它。
* 把頁面會重複用到的method抽出來，取個好名字在spec裡使用。

先聲明範例是參考海門大大的[example](https://github.com/chnbohwr/nightwatch_example.git)
當然又有做了些小改變。

nightwatch_example/test/pages/github.js
{% highlight js %}
    module.exports = {
      url: 'https://github.com',
      elements:{
        index_login_btn: 'body > header > div > div > div > a.btn.site-header-actions-btn.mr-1',
        login_field_username: '#login_field',
        login_field_password: '#password',
        login_field_submit: 'input.btn',
        index_avatar: 'img.avatar',
        index_create_button: '.header-nav-link.tooltipped.tooltipped-s.js-menu-target',
        index_dropdown: '.dropdown-menu.dropdown-menu-sw',
        index_dropdown_newrep: '.dropdown-item:first-child',
        newrep_field_name: '#repository_name',
        newrep_field_submit: '.btn.btn-primary.first-in-line',
        rep_container: '.pagehead-actions',
        // ... defined element here.
      },
      commands: [{
        createRepo(repositoryName){
            this.click('@index_create_button')
                .waitForElementVisible('@index_dropdown')
                .click('@index_dropdown_newrep')
                .waitForElementVisible('@newrep_field_name')
                .setValue('@newrep_field_name', repositoryName)
                .click('@newrep_field_submit')
                .waitForElementVisible('@rep_container');
    
            return this.api;
        },
        deleteRepo(account, repositoryName) {
            let resName = account + '/' + repositoryName;
            this.click('svg.octicon.octicon-mark-github')
                .waitForElementVisible('div.boxed-group-action')
                .click('span[title="' + resName + '"]')
                .waitForElementVisible('@rep_container')
                .click('@rep_setting')
                .waitForElementVisible('@setting_field_rename')
                .click('@setting_delete')
                .waitForElementVisible('@setting_modal')
                .setValue('@setting_modal_resname', repositoryName)
                .click('@setting_modal_del')
                .waitForElementVisible('@index_flashnotice');
            return this.api;
        }
        // ... defined method here
      }]
    };
{% endhighlight %}

在elements裡定義的可以拿到command的method中使用，並在前面加上`@`。

可以在spec裡這樣用 - nightwatch_example/test/main/github.js
{% highlight js %}
    module.exports = {
      ...
      'create repo': (browser) => {
        browser
          .page.github().createRepo(repositoryName)
          .page.github().deleteRepo(username, repositoryName);
      }
    };
{% endhighlight %}

##### Writing Commands

可參考[官方文件](http://nightwatchjs.org/guide#writing-commands)。
Commands一樣可以讓你定義Method，並且你可以直接用browser.mymethod去呼叫它，
我覺得如果是每個testcase都要用到的某個行為，像是登入，它可以放在commands裡，
但建議如果是某一頁才會有的動作，例如刪除帳號，則放在page裡的command應該更為適合。

nightwatch_example/test/command/goPage.js
{% highlight js %}
    exports.command = function() {
        let browser = this.page.todo();
        browser
            .navigate()
            .waitForElementPresent('@app', 'page ok')
            .assert.containsText('@title', 'todo MVC', 'title ok')
            .assert.elementPresent('@filters', 'filter ok')
            .assert.elementPresent('@new_todo', 'input field ok')
            .assert.elementPresent('@btn_add', 'add button ok')
            .assert.elementPresent('@btn_ajax', 'ajax button ok');
        return this;
    };
{% endhighlight %}

Use it.
{% highlight js %}
    browser.goPage();
{% endhighlight %}

##### Add before(), after() hooks

有點像Mocha，Nighwatch自己有支援這樣的語法，
一開始就執行登入，或是一開始要導到某個url。

> The done function must be called as the last step when the async operation completes. Not calling it will result in a timeout error.

done()這個用法其實雷我很多次了，這是我最不熟悉的用法，後來發現在關閉瀏覽器後也要在執行一次done，
不然一樣會造成Timeout Error。

最常見的用法如下:

{% highlight js %}
    before(browser) {
        browser
            .resizeWindow(1920, 1080)
            .login(username, password);
    },
    afterEach(browser, done) {
        done();
    },
    after(browser, done) {
        browser.end(() => {
            done();
        });
    }
{% endhighlight %}

如果有更好的寫法，歡迎交流討論喔!謝謝!再次感謝[線上讀書會](https://www.facebook.com/groups/906048196159262/)。

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