---
title: 我不是很懂 Promise
date: 2018-03-12 17:29:53
tags:
---

# 我不懂 Promise

是的，我不是很懂 Promise。儘管看了阮一峯老師的 [Promise 對象](http://es6.ruanyifeng.com/#docs/promise)，也能寫出 `axios.get(url).then(res => fn).catch(err => handle(err))` 這樣的代碼，但思考以下如下幾個問題，我發現我一無所知:

- `then` 返回的是什麼？仍是 Promise 嗎？是原來那個 Promise 嗎？
- 下面兩種寫法是一樣的嗎？有什麼不同？
  ```javascript
  axios.get(url)
    .then(...)
    .catch(err => {...})
  ```

  ```javascript
  axios.get(url)
    .then(
      () => {...},
      err => {...}
    )
  ```

- 有一個資源存在於多個服務器，怎樣以最快的速度獲取到資源？
- 有一個操作需要等多個異步函數執行完成，該怎麼做？
- 你能自己實現 Promise 嗎？

學習和掌握一種知識需要經過幾個階段：
1. 知道它是幹什麼的 (聽過)
2. 知道它怎麼用 (用過)
3. 知其原理 (實現過)

根據 Atwood定律，**任何可以使用 Javascript 來編寫的應用，最終都會由 Javascript 編寫。**作爲一名前端工程師，我們不能因爲會寫幾個 webpapp 就沾沾自喜，在如今這個時代，前端可以寫混合app，做微信小程序，寫桌面app，開發網頁遊戲，webgl實現三維炫酷效果, node.js 已經在服務端發揮着重要作用，而這對前端工程師的要求勢必會越來越高。我們得對 angular, vue, react信手拈來，得有審美，會寫算法，甚至自己實現新的技術。

未來的路任重而道遠。要建高樓，必先夯實基礎，基礎不牢，風雨飄搖。Javascript 基礎你真的掌握了嗎？

# Promise 是做什麼的？
javascript 是異步的，爲了等一個異步函數執行完成，我們會寫出這樣的代碼:
```javascript
fn (function () {
  // do something
  ...
  fn1 (function () {
    // do something
    ...
    fn2 (function () {
      // do something
      ...
      fn3 (function () {
        // do something
        ...
        fn4(function () {
          ...
        })
      })
    })
  })
})
```
當要等的東西多了，就會墮入**深淵**，通常叫做**回調地獄**或者**回調黑洞**。

javascript 的異步特性解決了很多問題，提高了資源利用率。我們當然不能因爲 **回調地域** 的問題就指責 javascript 異步特性不好。工程師們想出了新的寫法來解決這個問題，就是 Promise。有了 Promise，我們就能這樣寫：
```javascript
p1.then(() => p2)
  .then(() => p3)
  .then(() => p4)
  .catch(err => handle)
```

這種鏈式寫法在 javascript 里很常見，基本原理就是在函數里返回 this
```javascript
const obj = {
  say: function () {
    console.log('nice to meet you')
    return this
  },
  cook: function () {
    console.log('make a meel')
    return this
  },
  sleep: function () {
    console.log('zzz...')
    return this
  }
}

obj.say().cook().sleep()
```
可以猜想，Promise 一定也在 then 里返回了 this。

# Promise 的用法
## Promise 的含義
promise 即 **承諾** 之義。這裏 Promise 用來處理異步消息，當異步完成之後進行下一步處理，或處理成功，或處理失敗。Promise 裏面最重要的概念是 **狀態**，狀態只能發生一次改變，改變即結局，不可反覆，要信守承諾。

**狀態**:
有3 種狀態：`pending`, `resolved`, `rejected`，狀態之間只有 2 種流動方式:
- pending &rarr; resolved
- pending &rarr; rejected

如何理解 **改變即結局** 呢？ 我們來看下面這個例子:
```javascript
var p = new Promise(function (resolve, reject) {
  setTimeout(function () {
    resolve('resolved')
  }, 1000)

  setTimeout(function () {
    reject('something bad happenend')
  }, 2000)
})

p.then(res => console.log(res)).catch(err => console.log(err))

setTimeout(function () {
  p.then(res => console.log(res)).catch(err => console.log(err))
}, 3000)
```
在這個 promise 中，我們讓它 1000ms 後狀態變成 'resolved'，2000ms 後狀態變成 'rejected'。但我們最終只能得到兩個 'resolved'， 沒有任何 `bad thing` 發生，改變只能發生一次，就像潑出去的水收不回來。

## 定義一個 promise

```javascript
const p = new Promise(function (resolve, reject) {
  // ... some code
  if (/* 異步操作成功 */) {
    resolve(value)
  } else {
    reject(error)
  }
})
p.then(
  function (value) {
    // success
  },
  function (err) {
    // fail
  }
)
```
這就是一個最簡單的 Promise 用法。then 接受兩個函數作爲參數，分別爲成功和失敗時的處理。
then 返回的還是 promise，所以可以繼續 `.then` 鏈式調用。你可能會問 `.catch` 返回的是什麼？實際上 `.catch` 是 `.then(null, fn)` 的簡寫，既然仍是 `then`，當然也就返回 promise 了。

## Promise 上的方法

### then
- then 方法用來對 promise 結果進行處理，即處理 resolved 和 rejected，你需要傳遞兩個函數參數，裏面添加自己的代碼邏輯。
- then 函數返回的是一個新的 promise
	- `.then(value => { return 'jupiter' })` // 相當於返回 `Promise.resolve('jupiter')`
	- `.then(value => {})` // 相當於返回 `Promise.resolve(undefined)`
	- `.then(value => { return new Promise((resolve, reject) => { ... } )})` // 顯式返回一個 Promise

### catch
引用 [MDN catch](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch):
>calling obj.catch(onRejected) internally calls obj.then(undefined, onRejected)

在阮一峯老師所著的《ECMAScript 6入門》的 [Promise.prototype.catch](http://es6.ruanyifeng.com/#docs/promise#Promise-prototype-catch) 一章有如下的描述:
>Promise 对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获为止。也就是说，错误总是会被下一个catch语句捕获。
>
```javascript
getJSON('/post/1.json')
  .then(function(post) {
    return getJSON(post.commentURL);
  })
  .then(function(comments) {
    // some code
  })
  .catch(function(error) {
   // 处理前面三个Promise产生的错误
  });
```

那麼 `.then().catch()` 和 `.then(onFulfilled, onRejected)` 有什麼不同呢？
由於 Promise 具有 **冒泡** 屬性，當一個 Promise 沒有捕獲 rejected 時，相當於在 then 方法 onRejected 中隱式調用了 `return Promise.reject(err)`

```javascript
Promise.reject('hello')
  .then(() => console.log('I will not be called'))
  .catch(err => console.log(err))
```
相當於
```javascript
Promise.reject('hello')
  .then(() => console.log('I will not be called'), err => Promise.reject(err))
  .catch(err => console.log(err)) // 或 .then(undefined, err => console.log(err))
```

所以，不同之處在於:
- `.then().catch()` 中的 catch 處理的是沒有被 then 捕獲而冒泡後新的 `Promise.reject(err)`
- `.then(onFulfilled, onRejected)` 中的 onRejected 則直接處理的是本 Promise 的 rejected

這樣，阮老師關於 “Promise 是冒泡的” 也就好理解了，並且
>一般來說，不要在 then 方法中定義 Rejected 狀態的回調函數（即 then 的第二個參數），而應該總是使用 catch 方法。

一旦某個 then 返回了 rejected，就會進入最近的那個 catch, 而不管後面的 then。

### all
`Promise.all([p1, p2, p3, p4])` 回答了 *有一個操作需要等多個異步函數執行完成，該怎麼做* 這個問題。
比如，我們需要加載一組圖片，等所有圖片加載完成，將它們放到 body 裏面，可以這樣做:
```javascript
function loadImg (url) {
  return new Promise(function (resolve, reject) {
    const img = new Image()

    img.onload = () => resolve(img)
    img.onerror = () => reject('image loaded failed')

    img.src = url
  })
}

const imgs = [
  'http://desk-fd.zol-img.com.cn/t_s960x600c5/g5/M00/0B/0D/ChMkJ1e9jHqIWT4CAA2dKPU9Js8AAUsZgMf8mkADZ1A116.jpg',
  'http://desk-fd.zol-img.com.cn/t_s960x600c5/g5/M00/0B/0D/ChMkJle9jHSIUbUpAAzm7ILCVGQAAUsZQOmwMcADOcE736.jpg',
  'http://desk-fd.zol-img.com.cn/t_s960x600c5/g5/M00/0B/0D/ChMkJ1e9jHiIcv0hAAdXPPIMTekAAUsZgGrhloAB1dU798.jpg',
  'http://desk-fd.zol-img.com.cn/t_s960x600c5/g5/M00/0B/0D/ChMkJle9jH2ILYY3AAbui0bQOHkAAUsZwE752MABu6j537.jpg'
]

Promise.all(imgs.map(itm => loadImg(itm)))
  .then(arr => {
    arr.forEach(itm => {
      document.body.appendChild(itm)
    })
  })
  .catch(function (err) {
    console.log(err)
  })
```

### race
`Promise.race` 則回答了這個問題: *有一個資源存在於多個服務器，怎樣以最快的速度獲取到資源*
引用阮老師的示例：獲取資源，如果超時將 Promise 狀態置爲 `rejected`
```javascript
Promise
  .race([
    fetch('/resource-that-may-take-a-while'),
    new Promise((resolve, reject) => {
      setTimeout(() => reject(new Error('request timeout')), 5000)
    })
  ])
  .then(response => console.log(response))
  .catch(err => console.log(err))
```

### resolve
`Promise.resolve('foo')` 相當於 `new Promise(resolve => resolve('foo'))`

### reject
`Promise.reject(err)` 相當於 `new Promise((null, reject) => reject(err))`
