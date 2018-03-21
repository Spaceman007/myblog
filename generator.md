---
title: 理解 Generator
date: 2018-03-21 17:29:53
tags: generator, Promise, curry
---

# 理解 Generator (生成器)
generator 和 iterator 的概念非常類似，同樣都是調 `next()` 獲取當前元素和當前狀態，同樣都是惰性求值，同樣可以用 `for...of` 遍歷。

按我的理解，generator 就是一種 iterator，只不過 generator 是由函數轉換成的 itarator，這種轉換是 es6 內部提供的，而將一個函數轉爲 generator，只需藉助 `function*` 和 `yield`。

舉例來說：

```js
const gen = function* () {
    yield 'hello'
    yield 'world'
}
const gener = gen() // 得到一個生成器，或叫遍歷器
gener.next() // { value: 'hello', done: false }
gener.next() // { value: 'world', done: false }
gener.next() // { value: undefined, done: true}
```

理解了 generator 就是 `function` 變出來的 itarator，就可以參考 [iterator](http://blog.csdn.net/marshall001/article/details/79628181) 了。不過 genertor 也有與 iterator 不一樣的地方:

## 在 next 中回傳參數

```js
const gen = function* () {
    console.log('1st: ' + (yield 'hello'))
    console.log('2nd: ' + (yield 'world'))
}
const gener = gen()

console.log(gener.next('first next'))
console.log(gener.next('second next'))
console.log(gener.next('third next'))
```

打印結果爲：

```json
{ value: 'hello', done: false }
1st: second next
{ value: 'world', done: false }
2nd: third next
{ value: undefined, done: true }
```

> next 方法可以帶一個參數，可以當作上一條 yield 語句的返回值。

## generator 可以用 return 提前結束遍歷

```js
const gen = function* () {
    yield 'hello'
    yield 'world'
}
const gener = gen()
gener.next() // { value: 'hello', done: false }
gener.return('ended') // { value: 'ended, done: true }
gener.next() // { value: undefined, done: true }
```

## 可在遍歷器外部調用 `throw` 拋出錯誤，而在遍歷器內部捕獲

```js
const gen = function* () {
    try {
        yield 'hello'
        yield 'world'
    } catch (e) {
        console.log(e)
    }
}

const gener = gen()
gener.next()
gener.throw('unexpected')
```

## generator 嵌套

當 generator 嵌套時，內部 generator 需要 `yield* generator`

```js
const foo = function* () {
    yield 'hello'
    yield 'world'
}

const bar = function* () {
    yield 'bar'
    yield* foo()
}

const gener = bar()

for (let itm of gener) {
    console.log(itm)  // 'bar', 'hello', 'world'
}
```

# generator - 變異步爲 ‘同步’

generator 與 curry 的結合可以使異步應用用同步的方式書寫，使得代碼更爲清晰簡潔。

## curry （柯里化）

`curry` 是以數學家 **[Haskell Curry](https://en.wikipedia.org/wiki/Haskell_Curry)** 的名字命名的。Haskell Curry 也是 **[Haskell](https://www.haskell.org/)** 語言的發明者。

`curry` 是一種 *函數參數* 拆分技術。通過拆分函數參數，我們可以得到新的函數。使用 curry 以後，函數變得更加靈活，更具有可玩性。

下面以 add 函數舉例說明 curry 的用處:

```javascript
function add () {
    const args = Array.prototype.slice.apply(arguments)
    return args.reduce((a, b) => a + b)
}
const add1 = add.curry(1)
const add123 = add.curry(1, 2, 3)
console.log(add1(1, 2, 3))   // 7
console.log(add123(4, 5, 6)) // 21
```

`add1` 其實是將 1 與傳遞給 add1 的參數組合在一起，再傳遞給 add 函數。

`add123 ` 其實是將 `1, 2, 3` 與傳遞給 add123 的參數組合在一起，再傳遞給 add 函數。

下面給出 curry 技術的 js 實現：	

```javascript
Function.method('curry', function () {
    const slice = Array.prototype.slice,
        args = slice.apply(arguments),
        that = this
    return function () {
        return that.apply(null, args.concat(slice.apply(arguments)))
    }
})
```

上面的代碼有點繞，主要是 `arguments` 是一個具有 `length` 屬性的 `Array like`，不得不先用 `slice`做點轉換。使用 es6 可以簡化：

```javascript
Function.method('curry', function (...args) {
    const that = this
    return (...args2) => that.apply(null, args.concat(args2))
})
```
## generator 中使用 curry

> 參考了阮一峯老師的《[Generator 函数的异步应用](http://es6.ruanyifeng.com/#docs/generator-async)》

### 使用 promise 逐一讀取文件

`fs.readFile` 是一個異步函數，讀取一個文件的用法:

```javascript
const fs = require('fs')
fs.readFile('./file.js', (err, data) => {
    if (err) throw err
    else console.log(data.toString())
})
```

當有多個文件，需要逐一讀取時，如果用 promise 可以這樣做:

```javascript
const fs = require('fs')

const read = function (fileName) {
    return new Promise((resolve, reject) => {
        fs.readFile(fileName, (err, data) => {
            if (err) reject(err)
            else resolve(data)
        })
    })
}
read('./file1.js')
  .then(data => console.log(data.toString())) // 處理 file1.js
  .then(() => read('./file2.js'))
  .then(data => console.log(data.toString())) // 處理 file2.js
  .catch(err => console.log(err))

console.log('started...')
```

執行結果爲:

```json
started...
// file1 內容
// file2 內容
```

雖然 `.then` 的寫法避開了 回調黑洞，使代碼的寫法近似同步，但過多的 `then` 還不夠簡潔優雅。

### 使用 generator 逐一讀取文件

在使用 generator 時，我們可以用 `next(value)` 往生成器中回傳參數。當初並不明白 `next(value)` 有何意義，如今可以解惑了。看下面這個函數:

```javascript
function* readGen () {
    const r1 = yield read('./file1.js') // 暫且不管 read 函數的定義
    console.log(r1.toString())
    const r2 = yield read('./file2.js')
    console.log(r2.toString())
}
const g = readGen()
g.next()
```

我希望 `readGen` 中的 `file1.js`和 `file2.js` 能依次讀取。

要實現這個目標，我得利用 `g.next()` 獲取 generator 執行狀態，並且處理完一個文件後，再次調用 `g.next()` 進行下一步處理。直到 `done === 'true'`。

在 `g.next()` 的狀態中，`value` 可以用來放 `readFile` 的回調函數。而這，正式 `curry` 大顯身手的地方。

下面來實現 `read` 函數:

```javascript
const read = function (fileName) {
    return fs.readFile.curry(fileName) // 返回只含一個 callback 參數的函數
}
```

這樣就將 `readFile` 變成了只有一個 callback 參數的函數。當執行 `g.next()` 時，這個函數賦予了 `value`，我們就能在 `g.next().value(cb)` 中處理了。

注意 `read` 函數，它的功能是返回了一個函數，這個函數記錄了 `fileName`。這也可以用閉包實現:

```javascript
const read = function (fileName) {
    return function (cb) {
        return fs.readFile(fileName, cb)
    }
}
```

**關鍵** 之處在於，我們需要在 `value` 中獲取到 `readFile` 的回調信息，並且能夠繼續 `g.next()` 使函數繼續執行。下面的 `run` 函數確保 `readGen` 逐一執行直到完成:

```javascript
function run (gen) {
    const g = gen()

    function next (err, data) {
        if (data) console.log(data.toString())
        const ret = g.next(data)
        if (ret.done) return
        else ret.value(next)
    }

    next()
}
```

`next` 函數就是往 curry 化之後那個函數裏傳的參數。通過在 `next` 中判斷 generator 是否執行完成（如果還沒執行完，則繼續調用 `g.next()` ），最終實現文件的逐一讀取。

總結一下，使用 generator 將異步變爲同步，就是

- 利用 `function*` 和 `yield` 以同步的方式書寫代碼
- 定義一個**自動執行器**，使 generator 執行完成。

# 從 generator 到 sync

由於 generator 實現了以同步方式書寫代碼，並在實際使用中反饋良好，但畢竟需要自己實現執行方式，於是大家就想爲什麼不提供一種新的語法來做這件事呢，它能包含隱式的自動執行方式，使代碼更爲簡潔優雅。

這個語法就是 `sync` 和 `await ` 。注意它還不是 es6 的標準，而是屬於 es7 ，但也可以用 es6 實現。在不支持 es7 的環境中，可以用 babel 轉碼。我在 `node v8.9.0` 中測試已支持 `aync`。

`sync` 函數的使用十分簡單，只相當於對上面的 generator 的方式作了兩個修改:

- `*` &rarr; `sync`
- `yield` &rarr; await

於是，採用 `sync` 和 `await` 改寫上面的代碼：

```javascript
const fs = require('fs')
const readAll = async function () {
    const r1 = await fs.readFile('./file1.js')
    console.log(r1.toString())
    const r2 = await fs.readFile('./file2.js')
    console.log(r2.toString())
}
readAll()
console.log('--- begin ---')
```

執行結果:

```json
--- begin ---
// content of file1.js
// content of file2.js
```

採用 `async`  時，需注意:

- await 後面可以是 **Promise** 或者 原始類型值，如果是 promise，因爲可能 reject，最好放在 try...catch 裏。
- async 返回一個 Promise，可用 `then` 添加下一步操作。

# 結語

這是我學習 generator 過程中的總結，主要參考了《javascript 語言精粹》和 《[ES6 標準入門](http://es6.ruanyifeng.com)》。这两本书都非常不錯，可以作为手边书，隨時翻閱。

generator 的用法很好理解，但深入以後才發現並不簡單，其中牽扯了許多知識點，比如 `Iterator`、`curry`、`Promise`、`async`，還包含了人們爲解決異步書寫問題所作出的很多努力。

以上內容僅反映我對一些概念的當下理解，並不完善，比如 “generator 與 協程” 就沒提及。同時也深感要向讀者解釋清楚一個概念需要多少努力，除非自己能深入理解並舉一反三，而且要邏輯清晰，並用儘可能簡單的語言闡釋出來，否則就可能誤人子弟，浪費別人時間了。推薦購買正版《ES6 標準入門》，遇到不懂的好隨時查閱。
