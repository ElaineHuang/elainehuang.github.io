---
layout: post
title: React Redux 串 graphQL 學習筆記
excerpt: "Flea Market 拼裝車part2"
modified: 2016-07-08
categories: articles
tags: [Redux graphQL]
image:
  feature: pic-1.jpg
comments: true
share: true
---

我們的[flea-market-frontend]，Backend用的是graphQL API，
前端要怎麼接我們其實討論了一陣子，
前期我們用了[Relay]，後來又後悔改用了[Redux GraphQL Middleware]，
[劉邦皓]在一陣子研究過後，雖然它在github上星星數還未滿20顆，但卻非常符合我們的需要!
所以就決定是它惹! <br />
(p.s 文章內容如有錯誤歡迎留言告知喔!或是有更好的寫法也歡迎交流!謝謝~)

### Reference

* [Flea Market Demo Site](http://flea.hopebaytech.com/): 因為綁了公司email所以登入會失敗喔!物品已截標
* [flea-market-frontend]
* [flea-market-backend](https://github.com/FuBoTeam/flea-backend.git)
* [Redux GraphQL Middleware]
* [Relay]
* [Flea Market 起源](http://blog.elaine.me/articles/flea-market/)

### Why not use [Relay]?

我們project使用的是React & Redux，Redux和Relay的理念是非常不同的，
在Redux裡會有Store/Reducer/Component的交互，但這在Relay裡是不存在的，
也就是說你利用Relay call GraphQL API，這些回來的資料，並不會存進Redux裡的Store。
光是這點如果你要用Redux就不適合用Relay。

### How to use [Redux GraphQL Middleware]?

以下是[flea-market-frontend]使用範例。因為又有react-router，所以看起來比較雜，
也可以直接查看官網 ([Redux GraphQL Middleware]) !

##### Installation

{% raw %}
    npm install redux-graphql-middleware --save
{% endraw %}

##### Usage

{% highlight js %}
    import {
      createStore,
      applyMiddleware,
    } from 'redux';
    import { routerMiddleware } from 'react-router-redux';
    import { browserHistory } from 'react-router';
    import graphqlMiddleware from 'redux-graphql-middleware';
    import thunk from 'redux-thunk';
    import { fetch } from 'redux-auth';
    import rootReducer from '../reducers';
    
    const graphOptions = {
      fetch,
      server: 'http://flea.fubotech.com.tw/graphql',
      action: 'GRAPH',
      ready: 'GRAPH_READY',
      done: 'GRAPH_DONE',
      error: 'GRAPH_ERROR',
      transform: (data) => { return data; },
      errorTransform: (error) => { return error; },
    };
    
    const enhancer = applyMiddleware(
        routerMiddleware(browserHistory),
        graphqlMiddleware(graphOptions),
        thunk
    );
    
    const reducers = require('../reducers');
    
    export default function configureStore(initialState) {
      const store = createStore(rootReducer, initialState, enhancer);
      if (module.hot) {
        module.hot.accept('../reducers', () => {
          return store.replaceReducer(reducers);
        });
      }
    
      return store;
    }
{% endhighlight %}

### GraphQL - Query

以首頁的query來作範例，可以在[GraphiQL](http://flea.hopebaytech.com/graphiql)試試首頁的query。

{% raw %}
    query {
      allGoods{
        totalCount,
        edges {
          node {
            id,
            title,
            image,
            biddingTime,
            allBiddings {
              totalCount,
            }
            highestBidding {
              amount
              user {
                id
                fakeName
              }
            }
          }
        }
      }
    }
{% endraw %}

##### Action - Query All Goods

{% highlight js %}
    export const allGoodsQuery = () => {
      return {
        type: GRAPH,
        graphql: {
          action: 'GRAPH/ALL',
          ready: 'GRAPH_READY/ALL',
          done: 'GRAPH_DONE/ALL',
          error: 'GRAPH_ERROR/ALL',
        },
        data: {
          query: `
            query {
              allGoods{
                totalCount,
                edges {
                  node {
                    id,
                    title,
                    image,
                    biddingTime,
                    allBiddings {
                      totalCount,
                    }
                    highestBidding {
                      amount
                      user {
                        id
                        fakeName
                      }
                    }
                  }
                }
              }
            }
          `,
        },
      };
    };
{% endhighlight %}

##### Container

{% highlight js %}
    import { connect } from 'react-redux';
    import Market from '../components/Market';
    import {
      allGoodsQuery,
    } from '../actions';
    
    const mapStateToProps = (state) => {
      const goods = state.allGoods.goods; // 從Store裡的allGoods拿到reducers整理好的API資料
      const error = state.error;
      const isAllLoading = !state.allGoods.isFetched && state.allGoods.isFetching;
      return {
        goods,
        error,
        isAllLoading,
      };
    };
    
    const mapDispatchToProps = (dispatch) => {
      return {
        getAllGoods: () => {
          dispatch(allGoodsQuery());
        },
      };
    };
    
    const MarketContainer = connect(mapStateToProps, mapDispatchToProps)(Market);
    
    export default MarketContainer;
{% endhighlight %}

##### Reducer

isFeched表示從defaultState(false)到有資料就會一直為true(原本無資料再Fetch的期間UI會顯示轉轉摩天輪)
而isFetching則是不管原本有沒有資料，只要call API資料還沒回來就會是true。(UI會顯示progress bar)

{% highlight js %}
    const defaultState = {
      isFetched: false,
      isFetching: false,
      goods: null,
      error: null,
    };
    
    const allGoods = (state = defaultState, action) => {
      switch (action.type) {
        case 'GRAPH_READY/ALL':
          return {
            ...state,
            isFetching: !action.data,
          };
        case 'GRAPH_DONE/ALL':
          return {
            ...state,
            goods: action.data && action.data.allGoods || null,
            error: null,
            isFetched: true,
          };
        case 'GRAPH_ERROR/ALL':
          return {
            ...state,
            goods: null,
            error: action.error,
            isFetched: true,
          };
        default:
          break;
      }
      return state;
    };
    
    export default allGoods;
{% endhighlight %}

##### Component

{% highlight js %}
    import React, { PropTypes } from 'react';
    import CSSModules from 'react-css-modules';
    import Block from './Block';
    import styles from './goods.css';
    import Loading from '../Loading';
    import {
      Link,
    } from 'react-router';
    
    class Market extends React.Component {
      constructor(props) {
        super(props);
        this.displayName = 'Market';
      }
      componentWillMount() {
        const {
          params,
          getAllGoods,
        } = this.props;
        return getAllGoods(); // 在DOM Mount前call query allGoods的API
      }
      render() {
        const {
          isAllLoading,
          goods,
        } = this.props;
        if (isAllLoading) {
          return <Loading />; // 資料未拿到顯示loading畫面
        }
        if (goods === null || goods.totalCount === 0) {
          return <p className="bg-info" styleName="info-block">Oops! No items for sale. <Link to="/upload">Start selling.</Link></p>;
        }
        return (
          <ul styleName="goods-container">
          {
            goods.edges.map((g) => {
              return <Block key={g.node.id} good={g.node} />;
            })
          }</ul>
        );
      }
    }
    
    Market.propTypes = {
      getAllGoods: PropTypes.func.isRequired,
      isAllLoading: PropTypes.bool.isRequired,
      goods: PropTypes.object,
      error: PropTypes.object,
    };
    
    export default CSSModules(Market, styles);

{% endhighlight %}

### GraphQL - Mutation

以下標來做為範例好惹，但這是UI需要登入的狀態，所以就沒辦法再GraphiQL上試。

##### Action - Mutation

這時就又體會到GraphQL的一大好處，就是一個API發出去後不但更改了資料，還可以更新現有資料，
我call了mutation下了標後，它便回我物品頁的所有資訊使頁面得以翻新。

{% highlight js %}
    export const addBidMutation = (biddingData) => {
      return {
        type: GRAPH,
        graphql: {
          action: 'GRAPH/GOOD',
          ready: 'GRAPH_READY/GOOD',
          done: 'GRAPH_MUTATION/GOOD',
          error: 'GRAPH_ERROR/GOOD',
        },
        vars: {
          biddingData,
        },
        data: { // mutation回來後的資料
          mutation: `
            mutation($biddingData: AddBiddingInput!) {
              addBidding(input: $biddingData) {
                bidding {
                  good {
                    createdAt,
                    description,
                    id,
                    image,
                    title,
                    updatedAt,
                    biddingTime,
                    extendedCount,
                    allBiddings(first: 20) {
                      biddings {
                        id,
                        amount,
                        createdAt,
                        trashWord,
                        user {
                          fakeName,
                        }
                      }
                    }
                  }
                }
              }
            }
          `,
        },
      };
    };
{% endhighlight %}

##### Container

{% highlight js %}
    import { connect } from 'react-redux';
    import BidForm from '../components/GoodDetail/BidForm';
    import { addBidMutation } from '../actions';
    
    const mapStateToProps = (state) => {
      return {
        user: state.user,
      };
    };
    
    const mapDispatchToProps = (dispatch) => {
      return {
        handleBid: (bidData) => {
          dispatch(addBidMutation(bidData)); // mutation API
        },
      };
    };
    
    const BidFormContainer = connect(
      mapStateToProps, mapDispatchToProps
    )(BidForm);
    
    export default BidFormContainer;
{% endhighlight %}

##### Reducer

GoodDetail的存放方式為只要有點過的物品就會被存起來，key為ID，
所以每次的Query都會去檢查Object key有沒有這個ID，沒有的話創一個新的存起來，
有的話直接Update render資料。

{% highlight js %}
    const dateFormat = (utcDate) => {
      const date = new Date(utcDate);
      return date.toLocaleString();
    };
    
    const good = (state = {}, action) => {
      const id = action.vars &&
                 action.vars.biddingData &&
                 action.vars.biddingData.id ||
                 action.vars &&
                 action.vars.id;
      switch (action.type) {
        case 'GRAPH_READY/GOOD': {
          const defaultGoodState = {
            data: null,
            error: null,
            isFetching: false,
            isFetched: false,
          };
          if (!state[id]) {
            return {
              ...state,
              [id]: {
                ...defaultGoodState,
                isFetching: !action.data,
              },
            };
          }
          return {
            ...state,
            [id]: {
              ...state[id],
              isFetching: !action.data,
            },
          };
        }
        case 'GRAPH_DONE/GOOD': {
          const biddings = action.data.good.allBiddings.biddings.map((bidding) => {
            return {
              ...bidding,
              createdAt: dateFormat(bidding.createdAt),
            };
          });
          return {
            ...state,
            [id]: {
              ...state[id],
              data: {
                ...action.data.good,
                biddingTime: dateFormat(action.data.good.biddingTime),
                utcTime: action.data.good.biddingTime,
                updatedAt: dateFormat(action.data.good.updatedAt),
                createdAt: dateFormat(action.data.good.createdAt),
                allBiddings: {
                  biddings,
                },
              },
              error: null,
              isFetched: true,
            },
          };
        }
        case 'GRAPH_ERROR/GOOD':
          return {
            ...state,
            [id]: {
              ...state[id],
              error: action.error,
              isFetched: true,
            },
          };
        case 'GRAPH_MUTATION/GOOD': { // Mutation!!
          const newGood = action.data.addBidding.bidding.good;
          const biddings = newGood.allBiddings.biddings.map((bidding) => {
            return {
              ...bidding,
              createdAt: dateFormat(bidding.createdAt),
            };
          });
          return { // 下標完後順便更新資料!
            ...state,
            [id]: {
              ...state[id],
              data: {
                ...newGood,
                biddingTime: dateFormat(newGood.biddingTime),
                utcTime: newGood.biddingTime,
                updatedAt: dateFormat(newGood.updatedAt),
                createdAt: dateFormat(newGood.createdAt),
                allBiddings: {
                  biddings,
                },
              },
              error: null,
              isFetched: true,
            },
          };
        }
        default:
          break;
      }
      return state;
    };
    
    export default good;
{% endhighlight %}

##### Component

寫起來很長，重點是onSubmit的下標Action。

{% highlight js %}
    import React, { PropTypes } from 'react';
    import { Button } from 'belle';
    class BidForm extends React.Component {
      constructor(props) {
        super(props);
        this.displayName = 'BidForm';
        this.handleAmount = this.handleAmount;
        this.handleWord = this.handleWord;
        const { goodId, highestBid } = this.props;
        this.state = {
          bidding: {
            id: goodId,
            amount: highestBid + 10,
            trashWord: '',
          },
        };
      }
      handleAmount(event) {
        event.preventDefault();
        this.setState({
          bidding: {
            ...this.state.bidding,
            amount: parseInt(event.target.value, 10),
          },
        });
      }
      handleWord(event) {
        event.preventDefault();
        this.setState({
          bidding: {
            ...this.state.bidding,
            trashWord: event.target.value,
          },
        });
      }
      render() {
        const { user, handleBid, highestBid, extendedCount, goodId } = this.props;
        const gap = (extendedCount > 0) ? 10 : 1;
        return (
          <tr>
            <td style={nameStyle}>{user.fakeName}</td>
            <td>
              <input
                style={amountStyle}
                className="form-control"
                type="number"
                ref="amount"
                min={highestBid + gap}
                value={this.state.bidding.amount}
                onChange={this.handleAmount.bind(this)}
                form="bidForm"
                required
              />
            </td>
            <td>
              <input
                className="form-control"
                type="text"
                ref="trashWord"
                value={this.state.bidding.trashWord}
                onChange={this.handleWord.bind(this)}
                maxLength="64"
                form="bidForm"
              />
            </td>
            <td>
              <Button
                primary
                style={btnStyles}
                type="submit"
                form="bidForm"
              >Submit</Button>
              <form
                id="bidForm"
                onSubmit={
                  (e) => {
                    e.preventDefault();
                    this.setState({
                      bidding: {
                        id: goodId,
                        amount: this.state.bidding.amount + 10,
                        trashWord: '',
                      },
                    });
                    handleBid(this.state.bidding); // 下標的action
                  }
                }
              />
            </td>
          </tr>
        );
      }
    }
    
    BidForm.propTypes = {
      user: PropTypes.object,
      handleBid: PropTypes.func.isRequired,
      goodId: PropTypes.string,
      highestBid: PropTypes.number,
      extendedCount: PropTypes.number,
    };
    
    export default BidForm;
{% endhighlight %}

[劉邦皓]: https://www.facebook.com/ben196888
[Redux GraphQL Middleware]: https://github.com/gtg092x/redux-graphql-middleware
[Relay]: https://facebook.github.io/relay/
[flea-market-frontend]: https://github.com/FuBoTeam/fubo-flea-market.git

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