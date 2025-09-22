# Promises

Promise 是一個建構函式，主要是一個非同步(asynchronous)的操作，需要使用 `new` 語法實例化它，不管過程如何，都一定會回傳資訊。

## 三種狀態

Promise 主要分為三個狀態：

1. Pending (等待中)：初始狀態，當下還是屬於同步操作
2. Fulfilled (已實現)：操作成功，進入 Microtask Queue (微任務佇列)
3. Rejected (已拒絕)：操作失敗，進入 Microtask Queue (微任務佇列)


## 基本語法
最常見的創建方式是使用 new Promise() 建構函式，如下程式碼：

``` javascript
// 第一種
const promise = new Promise((resolve, reject)=> {
  const success = true;
  if(success)  {
    resolve('操作成功！');
  } else {
    reject('出了點問題。');
  }
});

// 第二種
function promise () {
  return new Promise(function(resolve, reject) {
    const success = true;
    if(success)  {
      resolve('操作成功！');
    } else {
      reject('出了點問題。');
    }
  });
}
```

## 重點說明：

- 建構函式接受一個有兩個參數的函數：resolve 和 reject
- resolve 用於將 Promise 轉為 fulfilled 狀態
- reject 用於將 Promise 轉為 rejected 狀態
- Promise 的狀態是不可逆的，一旦從 pending 變為 fulfilled 或 rejected，就無法再改變

一旦有結果後，Promise 就會被認定為「settled」（已確定）。

## Promise 使用
當宣告使用 `Promise` 後，可以使用 `then` 、 `catch` 跟 `finally` 來處理 `Promise` 回傳的結果(如下)。

```javascript
promise
.then(result => {
  console.log(result); // Promise fulfilled 時執行
})
.catch(error => {
  console.error(error); // Promise rejected 時執行
})
.finally(() => {
  console.log('Promise 已完成'); // 無論成功還是失敗都會執行
});
```

- then() - 處理成功的結果
- catch() - 處理錯誤和拒絕的結果
- finally() - 不管成功或失敗都會執行

## Promise 鏈式調用
Promise 是可以將多個不同的非同步(asynchronous)操作鏈接在一起

```javascript
const { setTimeout: delay } = require('node:timers/promises');

const promise = delay(1000).then(() => '第一個任務完成');

promise
  .then(result => {
    console.log(result); // '第一個任務完成'
    return delay(1000).then(() => '第二個任務完成');
  })
  .then(result => {
    console.log(result); // '第二個任務完成'
  })
  .catch(error => {
    console.error(error); // 如果任何 Promise 被拒絕，捕獲錯誤
  });
```

## Async/Await 語法糖

`Promise` 的出現解決了早期的 callback hell 問題。但是實際開發來講，還是有可能會讓 Callback Hell 問題發生，因此 ES7 出現了一個新的語法，也就是 Async/Await，因為這個語法糖的出現，才真正的解決這個問題。因為 async/await 出現可以大幅的減少巢狀結構
```javascript
async function performTasks() {
  try {
    const result1 = await promise1;
    console.log(result1); // 'First task completed'

    const result2 = await promise2;
    console.log(result2); // 'Second task completed'
  } catch (error) {
    console.error(error); // Catches any rejection or error
  }
}
performTasks();
```
### async：用於定義傳回的 Promise 的函數
```javascript
// 使用 async 的做法
async function fn() {
  return 'async'
}

// 轉換成 Promise 的程式碼
function fn() {
  return Promise.resolve('async');
}
```
由上面可以看到，使用 `async` 就等同於使用了 `Promise.resolve`

### await
```javascript
// 使用 await 的做法
async function fn() {
  return await 'await'
}

// 轉換成 Promise 的程式碼
function fn() {
  return Promise.resolve('await').then(() => undefined)
}
```
由上面可以看到，使用 `await` 就等同於使用了 `then`

### 優點

- 更容易閱讀跟維護
- 更直觀的可以解決錯誤處理
- 解決 Callback Hell 的問題

### 高階的 await
在使用 ECMAScript 模組時，因為模組本身會被視為支援非同步的作用域，因此可以在模組頂層直接使用 await ，不需要在外層包 async
``` javascript
import { setTimeout as delay } from 'node:timers/promises';

await delay(1000); // 不需要包在 async 函數中
```

## 基於 Promise 的 Node.js API
Node.js 在很多核心功能中，有提供 Promise 版本，使用 Node.js API 和 Promise 更加容易，並降低了「回調地獄」的風險。

如下: 
- fs.promises
- dns.promises
- stream/promises
- timers/promises
- child_process → util.promisify 或新版 node:child_process/promises

```javascript
const fs = require('node:fs').promises;
// Or, you can import the promisified version directly:
// const fs = require('node:fs/promises');

async function readFile() {
  try {
    const data = await fs.readFile('example.txt', 'utf8');
    console.log(data);
  } catch (err) {
    console.error('Error reading file:', err);
  }
}

readFile();
```

## 高級 Promise 方法

在處理多個非同步的任務時，Promise 本身也支援了好幾種方式。

### Promise.all()
這是可以接受 array 樣式的的方式，並且會回傳一個新的 Promise 的資訊，在過程中如果有其中一筆被拒絕，所有的 Promise 的會被立即拒絕，但是即使發生拒絕，本身 Promise 也會繼續執行，在處理大量 Promise 時，尤其是在批次中，使用此函數可能會增加系統記憶體壓力。

```javascript
const { setTimeout: delay } = require('node:timers/promises');

const fetchData1 = delay(1000).then(() => 'Data from API 1');
const fetchData2 = delay(2000).then(() => 'Data from API 2');

Promise.all([fetchData1, fetchData2])
  .then(results => {
    console.log(results); // ['Data from API 1', 'Data from API 2']
  })
  .catch(error => {
    console.error('Error:', error);
  });

```

### Promise.allSettled()
這個方法則是將所有的 Promise 分別記錄每個狀態，並且依照順序，回傳物件陣列。跟 `Promise.all()` 不同的地方，即使某些 Promise 失敗時，也不會導致其他 Promise 資料流失，並且會等待所有 Promise 都回傳。
```javascript
const promise1 = Promise.resolve('Success');
const promise2 = Promise.reject('Failed');

Promise.allSettled([promise1, promise2]).then(results => {
  console.log(results);
  // [ { status: 'fulfilled', value: 'Success' }, { status: 'rejected', reason: 'Failed' } ]
});
```

### Promise.race()
這個方法跟 Promise.all() 有點相似，在傳入的 Promise 的資料中，只要有其中一筆著狀態改變，無論是失敗、成功，都會直接回傳資訊。剩餘的 Promise 結果，就會被直接忽略。

``` javascript
const { setTimeout: delay } = require('node:timers/promises');

const task1 = delay(2000).then(() => 'Task 1 done');
const task2 = delay(1000).then(() => 'Task 2 done');

Promise.race([task1, task2]).then(result => {
  console.log(result); // 'Task 2 done' (since task2 finishes first)
});

```
### Promise.any()
當其中一個 Promise 解析成功，此方法就會解析。如果所有 Promise 都被拒絕，則會返回AggregateError。
```javascript
const { setTimeout: delay } = require('node:timers/promises');

const api1 = delay(2000).then(() => 'API 1 success');
const api2 = delay(1000).then(() => 'API 2 success');
const api3 = delay(1500).then(() => 'API 3 success');

Promise.any([api1, api2, api3])
  .then(result => {
    console.log(result); // 'API 2 success' (since it resolves first)
  })
  .catch(error => {
    console.error('All promises rejected:', error);
  });

```
# Promise 方法比較表

| 方法 | 使用場景 | 優點 | 缺點 | 注意事項 |
|------|----------|------|------|----------|
| **Promise.all()** | • 載入頁面必需資源<br>• 批次資料處理<br>• 需要所有結果才能繼續<br>• 資料庫事務操作 | • 快速失敗機制<br>• 效能最佳化（並行執行）<br>• 結果順序與輸入順序一致<br>• 語法簡潔 | • 任一失敗整個失敗<br>• 無法獲得部分成功結果<br>• 不適合容錯需求 | • 其他 Promise 仍會繼續執行<br>• 適合「全有或全無」場景<br>• 失敗時只返回第一個錯誤 |
| **Promise.allSettled()** | • 多平台同步<br>• 載入非必要資源<br>• 健康檢查多個服務<br>• 批次處理可容錯操作 | • 永不 reject<br>• 獲得所有操作結果<br>• 容錯性強<br>• 可分別處理成功/失敗 | • 無快速失敗機制<br>• 需要手動檢查每個結果<br>• 較新的 API（IE 不支援）<br>• 程式碼較複雜 | • 返回 `{status, value/reason}` 物件<br>• 適合部分成功有價值的場景<br>• 需檢查每個 result.status |
| **Promise.race()** | • 超時控制<br>• 多個資源競爭（CDN）<br>• 快取 vs 資料庫競爭<br>• 用戶取消操作 | • 最快回應<br>• 適合超時控制<br>• 可實現取消機制<br>• 效能優化 | • 只返回第一個完成的結果<br>• 其他 Promise 浪費資源<br>• 難以處理錯誤情況<br>• 可能導致資源競爭 | • 其他 Promise 仍在背景執行<br>• 第一個 settle（成功/失敗）就返回<br>• 常與 timeout Promise 結合 |
| **Promise.any()** | • 多個備用 API<br>• CDN 容錯<br>• 快取降級策略<br>• 多個數據源嘗試 | • 只要有一個成功就成功<br>• 容錯性強<br>• 適合備用方案<br>• 自動選擇最快成功的 | • 所有失敗才失敗<br>• 較新 API（支援度較低）<br>• 錯誤處理複雜<br>• 資源可能浪費 | • 返回第一個成功的結果<br>• 全部失敗時返回 AggregateError<br>• ES2021 新增，需注意瀏覽器支援 |



## 參考資料
- [Ray - JavaScript 中的 Promise 是什麼？以及為什麼你要懂 Promise](https://israynotarray.com/javascript/20211128/2950137358/)
- [Node.js 官方文件 - Discover Promises in Node.js](https://nodejs.org/en/learn/asynchronous-work/event-loop-timers-and-nexttick)
- [- Modern Web 前端藏寶圖系列 第 18 篇 Promise 方法](https://ithelp.ithome.com.tw/articles/10276827)