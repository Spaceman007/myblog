---
title: 2018年1月前端薪資統計
date: 2018-01-25 10:19:10
tags:
---

本文利用node.js + superagent + d3.js 展示 2018年1月拉勾網前端薪資。

先出結果圖:

**深圳**
![](/images/shenzhen201801.svg)

**成都**
![](/images/chengdu201801.svg)

城市 | 總數 | 平均薪資      | 去掉薪資最高的10組平均薪資 | 去掉頭尾10組薪資後的平均薪資 | 1~3年平均工資 | 5~10年平均工資
-----|------|---------------|----------------------------|------------------------------|---------------|
成都 | 390  | 8392 ~ 14220  | 7418 ~ 12558               | 8037 ~ 13602                 | 6317 ~ 10875  | 12655 ~ 19896
深圳 | 1050 | 12020 ~ 20829 | 11674 ~ 20238              | 11965 ~ 20714                | 9118 ~ 15927  | 17848 ~ 29915

# 1、數據獲取

## 1. 1獲取一頁數據
在瀏覽器中打開控制臺，導航到 [拉勾](https://www.lagou.com), 在深圳站輸入前端，可以發現拉勾的數據獲取方式。
需要**注意**的是，拉勾對爬蟲作了一點限制，如果在superagent中不設置header的話，拔取數據會失敗。 我們可以將瀏覽器的 **Request Header** 部分拷貝出來， 在superagent中構造出：

```javascript
const request = require('superagent')

const metainfo = {
  url: 'https://www.lagou.com/jobs/positionAjax.json',
  requestHeader: {
    'Accept': 'application/json',
    'Referer': 'https://www.lagou.com/jobs/list_%E5%89%8D%E7%AB%AF?labelWords=&fromSearch=true&suginput=',
    'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.108 Safari/537.36',
    'Cache-Control': 'no-cache',
    'Origin': 'https://www.lagou.com',
    'Host': 'www.lagou.com',
    'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8',
    'Pragma': 'no-cache',
    'timeout': 5000
  },
  queryObj: {
    city: '深圳',
    needAddtionalResult: false,
    isSchoolJob: 0
  },
  postObj: {
    first: true, // 是否是第一頁
    pn: 1,
    kd: '前端'
  }
}

request
  .post(metainfo.url)
  .set(metainfo.requestHeader)
  .query(metainfo.queryObj)
  .send(metainfo.postObj)
  .then(res => {
  	const content = res.body.content
  	if (content) {
  		resolve(content)
  	} else {
  		reject('no content')
  	}
  })
  .catch(err => {
  	reject(err)
  })
```

## 1.2 獲取全部數據
要獲取所有數據，我們需要分析一下數據結構。
拉勾返回的數據是這樣的:

```
{
  content: {
    pageNo: 1,
    pageSize: 15,
    positionResult: {
      result: [{}],
      resultSize: 15,
      totalCount: 1109
    }
  }
}
```

比較 totalCount 和 pageNo * pageSize,　我們可以知道數據是否獲取完。
獲取的數據，我們可以用writeStream保存到本地。
另外，async 和 await 可以幫助我們寫出更爲簡潔的代碼。

完整的代碼如下:

```javascript
const request = require('superagent')
const fs = require('fs')

const writeStream = fs.createWriteStream('tea.json')

const metainfo = {
  url: 'https://www.lagou.com/jobs/positionAjax.json',
  requestHeader: {
    'Accept': 'application/json',
    'Referer': 'https://www.lagou.com/jobs/list_%E5%89%8D%E7%AB%AF?labelWords=&fromSearch=true&suginput=',
    'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.108 Safari/537.36',
    'Cache-Control': 'no-cache',
    'Origin': 'https://www.lagou.com',
    'Host': 'www.lagou.com',
    'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8',
    'Pragma': 'no-cache',
    'timeout': 5000
  },
  queryObj: {
    city: '深圳',
    needAddtionalResult: false,
    isSchoolJob: 0
  },
  postObj: {
    first: true,
    pn: 1,
    kd: '前端'
  }
}

function requestInfo (queryObj, postObj, ms) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      request
        .post(metainfo.url)
        .set(metainfo.requestHeader)
        .query(queryObj)
        .send(postObj)
        .then(res => {
          const content = res.body.content
          if (content) {
            resolve(content)
          } else {
            reject('no content')
          }
        })
        .catch(err => {
          reject(err)
        })
    }, ms)
  })
}

async function getTotalInfo () {
  let { first, pn, kd } = metainfo.postObj
  let pageSize = 1
  let totalCount = 1000

  writeStream.write('[\n')

  while (pn * pageSize <= totalCount) {
    console.log(`第${pn}頁`)
    await requestInfo(metainfo.queryObj, { first, pn, kd }, 2000).then(content => {
      writeStream.write(JSON.stringify(content.positionResult.result).slice(1, -1))
      writeStream.write(',\n')
      pn++
      pageSize = content.pageSize
      first = false  // 非第一頁，需要設置first爲false
      totalCount = content.positionResult.totalCount
    }).catch(err => {
      console.error(err)
    })
  }
  writeStream.write(']')
  writeStream.end()
}

getTotalInfo()
```

這樣，我們就得到了所有的數據。

# 2. 數據展示
拉勾的薪資格式爲: ` "salary": "20k-25k"`。我們將按照這個關鍵字對數據分組，得到下面的格式:
```json
[
  {
    "min": 7000,
    "max": 11000,
    "number": 11
  }
]
```

其中，number表示有幾個公司是這一水平。

然後，我們用d3.js繪製出圖形，顏色從橙到藍，表示公司的數量。矩形的位置和寬度表示薪資範圍。完整代碼爲:

```javascript
import * as d3 from 'd3'
import jobs from '../assets/shenzhen.json'

const getNumber = function (str) {
  return Number(/\d*/.exec(str)[0])
}

const getFormattedArray = function (arr) {
  const obj = {}
  const ret = []

  for (let i = 0; i < arr.length; i++) {
    const salary = arr[i].salary.toUpperCase()
    if (obj[salary]) {
      obj[salary]++
    } else {
      obj[salary] = 1
    }
  }

  for (let k in obj) {
    const tmp = k.split('-')
    ret.push({
      min: getNumber(tmp[0]) * 1000,
      max: getNumber(tmp[1]) * 1000,
      number: Number(obj[k])
    })
  }

  ret.sort((a, b) => {
    let rt = b.min - a.min
    if (rt === 0) {
      rt = b.max - a.max
    }
    return rt
  })

  return ret
}

const getAvg = function (arr) {
  let sumMin = 0
  let sumMax = 0
  let num = 0

  for (let i = 0; i < arr.length; i++) {
    sumMin += arr[i].min * arr[i].number
    sumMax += arr[i].max * arr[i].number
    num += arr[i].number
  }

  return {
    min: parseInt(sumMin / num),
    max: parseInt(sumMax / num)
  }
}

const getColor = (function () {
  const compute1 = d3.interpolate('orange', 'red')
  const compute2 = d3.interpolate('red', 'purple')
  const compute3 = d3.interpolate('purple', 'blue')

  const n1 = 20, n2 = 30, n3 = 100

  return function (d) {
    if (d.number < n1) {
      return compute1(d.number / n1)
    } else if (d.number < n2) {
      return compute2((d.number - n1) / (n2 - n1))
    } else if (d.number < n3) {
      return compute3((d.number - n2) / (n3 - n2))
    } else {
      return 'blue'
    }
  }
})()

// 用d3.js繪製圖形
const draw = function (arr) {
  const width = 1600,
    height = 500,
    padding = {
      left: 30,
      right: 30,
      top: 30,
      bottom: 30
    }

  const svg = d3.select('.content')
    .append('svg')
    .attr('width', width)
    .attr('height', height)

  // 比例尺
  const xScale = d3.scaleLinear()
    .domain([0, d3.max(arr, d => d.max)])
    .range([0, width - padding.right - padding.left])

  const yScale = d3.scaleLinear()
    .domain([0, d3.max(arr, d => d.number)])
    .range([height - padding.top - padding.bottom, 0])

  // 添加矩形
  svg.selectAll('rect')
    .data(arr)
    .enter()
    .append('rect')
    .attr('transform', `translate(${padding.left}, ${padding.top})`)
    .attr('x', d => xScale(d.min))
    .attr('y', d => yScale(d.number))
    .attr('fill', d => getColor(d))
    .attr('height', d => height - padding.top - padding.bottom - yScale(d.number))
    .attr('width', d => xScale(d.max - d.min))

  // 坐标轴
  const xAxis = d3.axisBottom()
    .scale(xScale)
  svg.append('g')
    .attr('transform', `translate(${padding.left}, ${height - padding.bottom})`)
    .call(xAxis)

  const yAxis = d3.axisLeft()
    .scale(yScale)
  svg.append('g')
    .attr('transform', `translate(${padding.left}, ${padding.top})`)
    .call(yAxis)
}

function showTableData (jobs) {
  // 總數
  document.querySelector('#td1').innerText = jobs.length

  // 總平均工資
  const list = getFormattedArray(jobs)
  const avg = getAvg(list)
  document.querySelector('#td2').innerText = `${avg.min} ~ ${avg.max}`

  // 去掉薪資最高的10組
  const listlow = list.slice(10)
  const avglow = getAvg(listlow)
  document.querySelector('#td3').innerText = `${avglow.min} ~ ${avglow.max}`

  // 去掉薪資最高最低的10組
  const list2 = list.slice(10, -10)
  const avg2 = getAvg(list2)
  document.querySelector('#td4').innerText = `${avg2.min} ~ ${avg2.max}`

  // 1 ~ 3年平均薪資
  const jobs3 = jobs.filter(d => d.workYear === '1-3年')
  const list3 = getFormattedArray(jobs3)
  const avg3 = getAvg(list3)
  document.querySelector('#td5').innerText = `${avg3.min} ~ ${avg3.max}`

  // 5 ~ 10年
  const jobs4 = jobs.filter(d => d.workYear === '5-10年')
  const list4 = getFormattedArray(jobs4)
  const avg4 = getAvg(list4)
  document.querySelector('#td6').innerText = `${avg4.min} ~ ${avg4.max}`
}

showTableData(jobs)
const list = getFormattedArray(jobs)
draw(list)
```

# 3. 後記
瞭解自身價值及其所處的行業環境是很重要的。互聯網技術給了我們另一雙看世界的眼睛，在數據面前，語言是蒼白的，謊言是站不住腳的。
數據分析可以更好的爲我們服務，破除無明，打破信息不對稱。

從上面的數據可以看出，2018年1月份，深圳的公司在拉勾上發佈的前端招聘信息遠多餘成都。從薪資來看，深圳的公司會傾向於在20000附近開價，而成都在15000左右。
