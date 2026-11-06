---
title: "JavaScript 异步：事件循环、Promise 与 async/await"
date: 2026-05-17T21:20:00+08:00
draft: false
description: "从事件循环讲清 setTimeout 和 Promise 谁先执行，再整理 Promise 组合方法与 async/await 的常见坑。"
categories: ["前端"]
tags: ["JavaScript", "异步", "Promise"]
cover:
  relative: true
  image: "cover.png"
  alt: "JavaScript 异步笔记封面"
  caption: "宏任务与微任务"
---

「为什么 console.log 的顺序和我想的不一样」——答案全在事件循环。

## 事件循环：一次只做一件事

{{< figure src="event-loop.png" caption="调用栈清空后先清微任务队列，再取下一个宏任务" alt="事件循环示意" >}}

```
同步代码 → 调用栈清空
   ↓
微任务队列（Promise.then、queueMicrotask、MutationObserver）← 全部执行完
   ↓
取一个宏任务（setTimeout、setInterval、IO、UI 事件）
   ↓
再清空微任务队列 → 循环
```

所以这段代码的输出顺序是：

```js
console.log('1');

setTimeout(() => console.log('2'), 0);   // 宏任务

Promise.resolve().then(() => console.log('3'));  // 微任务

console.log('4');

// 输出：1 → 4 → 3 → 2
```

同步的 `1`、`4` 先跑；栈清空后微任务 `3` 先于宏任务 `2`。**微任务永远优先于宏任务**，这句话能解释绝大多数顺序问题。

## Promise 的三种状态

```
pending（进行中） ──→ fulfilled（成功）
                └──→ rejected（失败）
```

状态一旦改变就固定，之后再 `resolve` 或 `reject` 都不会生效。这也是「Promise 只能决议一次」的来源。

```js
const p = new Promise((resolve, reject) => {
  const ok = true;
  ok ? resolve('成功') : reject(new Error('失败'));
});

p.then(v => console.log(v))        // 成功走这里
 .catch(e => console.error(e))     // 失败走这里
 .finally(() => console.log('结束'));  // 无论如何都走
```

`then` 返回一个新的 Promise，所以可以链式；链中任意一环抛错或被 `reject`，都会跳到后面的 `catch`。

## 组合方法怎么选

| 方法 | 行为 | 典型场景 |
| --- | --- | --- |
| `Promise.all([...])` | 全部成功才成功，任一失败立即失败 | 并发拉多个接口，缺一个就不可用 |
| `Promise.allSettled([...])` | 等全部结束，返回每项的状态与结果 | 批量任务，允许部分失败 |
| `Promise.race([...])` | 第一个结束的说了算（成功或失败） | 超时控制 |
| `Promise.any([...])` | 第一个**成功**的说了算，全失败才失败 | 多个镜像源取最快可用的 |

```js
// 超时控制：5 秒没响应就抛错
const withTimeout = (p, ms = 5000) =>
  Promise.race([
    p,
    new Promise((_, reject) => setTimeout(() => reject(new Error('超时')), ms)),
  ]);
```

## async/await：本质是 Promise 的语法糖

```js
async function loadUser(id) {
  try {
    const res = await fetch(`/api/users/${id}`);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (err) {
    console.error('加载失败', err);
    throw err;              // 继续往上抛，交给调用方处理
  }
}
```

要点：

- `async` 函数永远返回 Promise，`return 1` 等价于 `Promise.resolve(1)`。
- `await` 后面的表达式会被包成 Promise，非 Promise 值也能 `await`（立即返回）。
- `await` 只在 `async` 函数或模块顶层（top-level await）里可用。

### 循环里别用 forEach + await

```js
// 错误：forEach 不会等待，回调里的 await 只影响回调内部
items.forEach(async item => {
  await save(item);      // 实际是并发跑，且外层拿不到结果
});

// 正确：需要顺序执行
for (const item of items) {
  await save(item);
}

// 正确：允许并发，用 map + Promise.all
await Promise.all(items.map(item => save(item)));
```

需要「限并发」时，把任务切片分批跑：

```js
async function runWithLimit(tasks, limit = 5) {
  const results = [];
  for (let i = 0; i < tasks.length; i += limit) {
    results.push(...(await Promise.all(tasks.slice(i, i + limit).map(t => t())));
  }
  return results;
}
```

## 常见坑

| 坑 | 说明 | 处理 |
| --- | --- | --- |
| 忘了 `catch` | 出现 unhandled rejection，某些环境直接崩溃 | 顶层加 `.catch`，或配 `unhandledrejection` 监听 |
| `await` 串行 | 互不依赖的请求一个个等，白等 | 先 `Promise.all` 再赋值 |
| 在 `then` 里 `return` 了新 Promise 却忘了 `await` | 链断了，错误不冒泡 | `return await` 或直接 `return` 正确链 |
| `new Promise` 包裹已有 Promise | 多余且容易吞错误 | 直接返回原 Promise |
| `setTimeout(fn, 0)` 以为立即执行 | 实际是下一轮宏任务，且最小延迟约 4ms | 需要「本轮结束立刻执行」用 `queueMicrotask` |

## 小结

- 顺序记住一句：同步 → 微任务全清 → 一个宏任务 → 再清微任务。
- 组合按需求选：全都要用 `all`，允许失败用 `allSettled`，抢先用 `race`/`any`。
- 循环里用 `for...of` 或 `map + Promise.all`，别用 `forEach + await`。
