---
title: 我不是很懂 Promise
date: 2018-03-12 17:29:53
tags: javascript, Promise
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
這是一個最簡單的 Promise 用法。then 接受兩個函數作爲參數，分別爲成功和失敗時的處理。
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

## 關於 Promise 的幾道題
### 打印結果是?
```javascript
Promise.resolve('foo')
  .then(string => {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        string += 'bar'
        resolve(string)
      }, 100)
    })
  })
  .then(string => {
    setTimeout(() => {
      string += 'baz'
      console.log('timeout 1000: ', string)
    }, 1000)
    return string
  })
  .then(string => {
    console.log('last then: ', string)
  })
```
第二個 then 直接返回了 string, 接着第 3 個 then 狀態改變。又過了 1s, 第 2 個 then 里的 setTimeout 執行。所以結果是:
```
last then: foobar
timeout 1000: foobarbaz
```

### 在 setTimeout 里 `reject` 和 `throw` 有什么不同?
下面这道题来自 [MDN-catch](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch)：
```javascript
const p = new Promise(function (resolve, reject) {
  setTimeout(function () {
    // throw 'Oh-throw!'
    reject('Oh-reject')
  }, 1000)
})

p.then(
  value => console.log('resolve: ' + value),
  err => console.log('err: ' + err)
)
```

在 setTimeout 里调 `reject` 会使 Promise 状态变为 'rejected'，結果打印：`err: On-reject`
然而，在 setTimeout 里調 `throw` 我們得到了錯誤！ MDN 是這樣解釋的:
>// Errors thrown inside **asynchronous** functions will act like **uncaught errors**

如果不是異步的 `throw`，`reject` 和 `throw` 都能使 Promise 狀態變爲 `rejected`，效用相同。

# 實現一個 Promise
## 實現特性
美劇里常出現這句話: `You have my word!`, 與 `I promise you` 一個意思。據此，我們給自己實現的 Promise 命名爲 **Makeword**，好在字典里沒有這個詞，姑且作爲構造函數名。

在前面的分析中，我們瞭解了 Promise 的特性，我們暫時給 Makeword 實現下面幾個特性:
- then 鏈式調用
- 狀態改變即結局
- catch
- Makeword.resolve
- Makeword.reject

## 代碼實現
- 下面的代碼參考了 [史上最易读懂的 Promise/A+ 完全实现](https://zhuanlan.zhihu.com/p/21834559), [实现一个超简单的Promise](https://segmentfault.com/a/1190000009792439)。
- 代碼力求清晰簡潔，容易理解，也許有的地方考慮不周，如果我想到的話，會更新到 [github](https://github.com/xtimespace/jslib/blob/master/src/makeword.js) 上。
- 下面的代碼不再作多餘的說明，一方面過多的說明反而將問題本身變複雜，二來已有可以參考的文章了。

```javascript
function Makeword (fn) {
  this.status = 'pending'
  this.value
  this.onFulfilled = []
  this.onRejected = []
  const me = this

  function resolve (newValue) {
    if (me.status === 'pending') {
      me.status = 'resolved'
      me.value = newValue
      me.onFulfilled.forEach(fn => {
        fn(newValue)
      })
    }
  }

  function reject (reason) {
    if (me.status === 'pending') {
      me.status = 'rejected'
      me.value = reason
      me.onRejected.forEach(fn => fn(reason))
    }
  }

  try {
    fn(resolve, reject)
  } catch(e) {
    reject(e)
  }
}

Makeword.resolve = function (value) {
  return new Makeword((resolve, reject) => {
    resolve(value)
  })
}

Makeword.reject = function (reason) {
  return new Makeword((resolve, reject) => {
    reject(reason)
  })
}

Makeword.prototype.then = function (done, fail) {
  const me = this
  done = typeof done === 'function' ? done : () => {}

  let retP
  let tmp

  switch (me.status) {
    case 'pending':
      me.onFulfilled.push(done)
      me.onRejected.push(fail || null)
      retP = me
      break
    case 'resolved':
      tmp = done(me.value)
      if (tmp instanceof Makeword) {
        retP = tmp
      } else {
        retP = Makeword.resolve()
      }
      break
    default:
      if (typeof fail === 'function') {
        tmp = fail(me.value)
        if (tmp instanceof Makeword) {
          retP = tmp
        } else {
          retP = Makeword.resolve()
        }
      } else {
        retP = Makeword.reject(me.value) // 將 rejected 冒泡出去
      }
      break
  }

  return retP
}

Makeword.prototype.catch = function (cb) {
  return this.then(undefined, cb)
}

module.exports = Makeword
```

# 後記
格拉德威尔在其書《異類》中提出了 “一萬小時定律”，即要成爲某個領域的專家，需要10000小時。

王國維在《人间词话》中說：
>古今之成大事業、大學問者，必經過三種之境界:
- ‘昨夜西風凋碧樹 獨上高樓 望盡天涯路’。 此第一境也。
- '衣帶漸寬終不悔 爲伊消得人憔悴‘。此第二境也。
- ‘衆裏尋他千百度 驀然回首 那人卻在 燈火闌珊處’。此第三境也。

要在某一領域有所成就，必須方向明確，堅定不移，並爲之努力付出，縱然辛勞也無怨無悔。終有一天，撥雲見日，融會貫通，達到前所未有的成就。

10000 小時，不是機械的勞動，它包含了 **聽過 &rarr; 用過 &rarr; 知其所以然 &rarr; 改進** 的過程。如果你知道了某個技術，但總是停留在 會用 的階段，也用得滾瓜爛熟，歷經了10000小時，但仍然無法成爲專家。要融會貫通，舉一反三，非老老實實走完所有的階段是不行的。

10000 小時，考慮到人的精力有限，按每天 4 ~ 5 小時算，約等於 7 年。所以，如果想要在某一領域成爲專家，你至少需要 7 年的時間。如果你頻繁的從一個領域跳到另一領域，則在每個領域都是膚淺的。恰好有本書名字叫做《[七年就是一輩子](https://b.xinshengdaxue.com/)》，作者說: "七年就是一辈子。每一辈子都要至少习得一个重要的技能，进而获得不可逆的重生。"

回到博客本身，一開始我是沒有把握寫完的。學習《ES6 標準入門》關於 Promise 的章節後，是會用了，可以寫出幾個 promise 的適用場合，但一回到原理性的東西，就感覺好幾個地方邏輯銜接不起來。自己寫了好幾個 demo，有好幾個地方不明所以。Promise 看起來是簡單的，但真正弄明白卻需要付出更多的努力。

在查閱資料，並最終寫完 Makeword 後，才算真正明白了 Promise，也知道那些疑惑性的問題是如何發生的。如果不認真弄明白一件事，最終可能對它一無所知。學習似乎是量子的，是跳躍的，不是 1 就是 0，沒有 0.6。如果只想隨隨便便弄一弄，隨便學一學，不明真相，不窮其理，最終只能說 “我曾經聽過，不過以及即不清楚是幹什麼的了”。


# 參考
- [史上最易读懂的 Promise/A+ 完全实现](https://zhuanlan.zhihu.com/p/21834559)
- [Promises/A+](https://promisesaplus.com/)
