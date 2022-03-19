
#### 直接上源码（2.6.14）
[nextTick](https://github.com/vuejs/vue/blob/dev/src/core/util/next-tick.js)
```javascript
// 有删减

// 环境是否支持插入微任务
export let isUsingMicroTask = false

const callbacks = []
let pending = false

// 一次性执行注册的回调
function flushCallbacks () {
  // 重置标记
  pending = false
  const copies = callbacks.slice(0)
  callbacks.length = 0
  for (let i = 0; i < copies.length; i++) {
    copies[i]()
  }
}

let timerFunc

// 异步更新核心部分，根据环境降级处理
// 先尝试使用promise
// 再尝试使用MutationObserver
// 再尝试使用setImmediate
// 最后使用setTimeout

if (typeof Promise !== 'undefined' && isNative(Promise)) {
  const p = Promise.resolve()
  timerFunc = () => {
    // p为fulfilled 将 flushCallbacks 推入微任务队列
    p.then(flushCallbacks)
    if (isIOS) setTimeout(noop)
  }
  isUsingMicroTask = true
} else if (!isIE && typeof MutationObserver !== 'undefined' && (
  isNative(MutationObserver) ||
  MutationObserver.toString() === '[object MutationObserverConstructor]'
)) {
  let counter = 1
  const observer = new MutationObserver(flushCallbacks)
  // 创建textNode 并用 MutationObserver 实例监听
  const textNode = document.createTextNode(String(counter))
  observer.observe(textNode, {
    characterData: true
  })
  timerFunc = () => {
    // 修改textNode 触发监听回调，推入微任务队列
    counter = (counter + 1) % 2
    textNode.data = String(counter)
  }
  isUsingMicroTask = true
} else if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  timerFunc = () => {
    // 推入宏任务队列
    setImmediate(flushCallbacks)
  }
} else {
  timerFunc = () => {
    // 推入宏任务队列
    setTimeout(flushCallbacks, 0)
  }
}

// 主体函数
export function nextTick (cb?: Function, ctx?: Object) {
  let _resolve
  // 将当前任务插入callbacks 等待被执行
  // cb不存在则插入一个空的promise resolve
  callbacks.push(() => {
    if (cb) {
      try {
        cb.call(ctx)
      } catch (e) {
        handleError(e, ctx, 'nextTick')
      }
    } else if (_resolve) {
      _resolve(ctx)
    }
  })
  
  // 标记 只有在当前时间循环首次调用时才执行timerFunc
  if (!pending) {
    pending = true
    // 插入异步队列
    timerFunc()
  }
  // 没传入回调 同时环境支持promise
  // 返回promise
  // 缓存resolve插入callbacks中
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(resolve => {
      _resolve = resolve
    })
  }
}
```
#### 大致流程

- 初始化时就根据当前环境降级处理，定义`timerFunc`，用于将注册的回调队列 `callbacks`插入异步队列，优先插入微任务队列
- 调用 `nextTick`时，将回调推入 `callbacks`等待被执行，首次调用会执行一次 `timerFunc`
