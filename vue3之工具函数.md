#### 源码地址

- [https://github.com/vuejs/core/blob/main/packages/shared/src/index.ts](https://github.com/vuejs/core/blob/main/packages/shared/src/index.ts)

#### 功能

- 工具函数

#### 参考

- [https://juejin.cn/post/6994976281053888519](https://juejin.cn/post/6994976281053888519)

#### 源码阅读

```javascript
// 空对象 只读属性
// __DEV__相当于process.env.NODE_ENV !== 'production'
// 开发环境下修改会报错
// freeze是冻结对象 冻结对象最外层无法修改
export const EMPTY_OBJ: { readonly [key: string]: any } = __DEV__
  ? Object.freeze({})
  : {}
```

```javascript
// 空数组 只读
// __DEV__相当于process.env.NODE_ENV !== 'production'
// 开发环境下修改会报错
export const EMPTY_ARR = __DEV__ ? Object.freeze([]) : []
```

```javascript
// 用于判断 便于压缩
export const NOOP = () => {} // 空函数
export const NO = () => false // 永远返回false
```

```javascript
// 判断是否on开头，并且下一个不是小写字母 如onClick
const onRE = /^on[^a-z]/
export const isOn = (key: string) => onRE.test(key)
```

```javascript
// 字符串是否是onUpdate:开头
export const isModelListener = (key: string) => key.startsWith('onUpdate:')
```

```javascript
// 合并
export const extend = Object.assign
```

```javascript
// 移除数组元素
// splice由于元素的移动，比较耗性能
// 在axios中有采用删除置null的方法删除元素，执行时为null的不执行，提升性能
export const remove = <T>(arr: T[], el: T) => {
  const i = arr.indexOf(el)
  if (i > -1) {
    arr.splice(i, 1)
  }
}
```

```javascript
// 是否为自身拥有的属性
const hasOwnProperty = Object.prototype.hasOwnProperty
export const hasOwn = (
  val: object,
  key: string | symbol
): key is keyof typeof val => hasOwnProperty.call(val, key)
```

```javascript
// 判断数据类型

export const isArray = Array.isArray
export const isMap = (val: unknown): val is Map<any, any> =>
  toTypeString(val) === '[object Map]'
export const isSet = (val: unknown): val is Set<any> =>
  toTypeString(val) === '[object Set]'
export const isDate = (val: unknown): val is Date => val instanceof Date
export const isFunction = (val: unknown): val is Function =>
  typeof val === 'function'
export const isString = (val: unknown): val is string => typeof val === 'string'
export const isSymbol = (val: unknown): val is symbol => typeof val === 'symbol'

// 判断是否为对象，排除null
export const isObject = (val: unknown): val is Record<any, any> =>
  val !== null && typeof val === 'object'
  
// 判断是否为Promise
// 是对象 && 存在then函数 && 存在catch函数
export const isPromise = <T = any>(val: unknown): val is Promise<T> => {
  return isObject(val) && isFunction(val.then) && isFunction(val.catch)
}
// Object原型上的toString方法
export const objectToString = Object.prototype.toString
export const toTypeString = (value: unknown): string =>
  objectToString.call(value)

// 截取类型字段
export const toRawType = (value: unknown): string => {
  return toTypeString(value).slice(8, -1)
}

// 是否为纯对象 （在axios源码上也有这个工具，通过对象的原型去判断）
export const isPlainObject = (val: unknown): val is object =>
  toTypeString(val) === '[object Object]'
```

```javascript
// 是否为数字字符串
// parseInt(key, 10)第二个参数是进制
export const isIntegerKey = (key: unknown) =>
  isString(key) &&
  key !== 'NaN' &&
  key[0] !== '-' &&
  '' + parseInt(key, 10) === key
```

```javascript
// 接收一个数组，分割并存储在一个对象中
// 返回一个方法，用于判断val是否在map中
// 可选参数是是否小写
export function makeMap(
  str: string,
  expectsLowerCase?: boolean
): (key: string) => boolean {
  const map: Record<string, boolean> = Object.create(null)
  const list: Array<string> = str.split(',')
  for (let i = 0; i < list.length; i++) {
    map[list[i]] = true
  }
  return expectsLowerCase ? val => !!map[val.toLowerCase()] : val => !!map[val]
}

export const isReservedProp = /*#__PURE__*/ makeMap(
  // the leading comma is intentional so empty string "" is also included
  ',key,ref,ref_for,ref_key,' +
    'onVnodeBeforeMount,onVnodeMounted,' +
    'onVnodeBeforeUpdate,onVnodeUpdated,' +
    'onVnodeBeforeUnmount,onVnodeUnmounted'
)
```

```javascript
// 接收一个函数，同时返回一个函数用于获取fn的结果
// 返回的函数中，每次调用会先检查cache是否有结果
// 有则返回，没有则调用fn获取并保存在cache中，同时返回结果
// 缓存机制，类似单例模式
const cacheStringFunction = <T extends (str: string) => string>(fn: T): T => {
  const cache: Record<string, string> = Object.create(null)
  return ((str: string) => {
    const hit = cache[str]
    return hit || (cache[str] = fn(str))
  }) as any
}
```

```javascript
// \w 是 0-9a-zA-Z_ 数字 大小写字母和下划线组成
// () 小括号是 分组捕获
const camelizeRE = /-(\w)/g

// replace第二个参数可以是一个函数
// 函数参数为match,p1,p2,p3...,offset,string,NamedCaptureGroup，参考MDN
// 如on-select
// 第一个参数为匹配到的字符 -s
// 第二个参数为第一分组匹配到的字符 s
// 替换函数将-s替换成-S
export const camelize = cacheStringFunction((str: string): string => {
  return str.replace(camelizeRE, (_, c) => (c ? c.toUpperCase() : ''))
})

// \B 是指 非 \b 单词边界。
const hyphenateRE = /\B([A-Z])/g

// 该函数匹配非首位大写字母，转成-小写
// $1为第一分组匹配到的内容
export const hyphenate = cacheStringFunction((str: string) =>
  str.replace(hyphenateRE, '-$1').toLowerCase()
)

// 首字母转大写
export const capitalize = cacheStringFunction(
  (str: string) => str.charAt(0).toUpperCase() + str.slice(1)
)

// 字符串首字母转大写并加上on前缀 click => onClick
export const toHandlerKey = cacheStringFunction((str: string) =>
  str ? `on${capitalize(str)}` : ``
)
```

```javascript
// 判断是不是有变化 两个值是否严格相等
export const hasChanged = (value: any, oldValue: any): boolean =>
  !Object.is(value, oldValue)
// 原实现方法
export const hasChanged = (value, oldValue) => value !== oldValue && (value === value || oldValue === oldValue);
  
// Object.is 用于判断两个值是否严格相等 与===类似
// 不同的是
// 1、+0不等于-0
// 2、NaN等于NaN
```

```javascript
// 同时间内执行多个函数
export const invokeArrayFns = (fns: Function[], arg?: any) => {
  for (let i = 0; i < fns.length; i++) {
    fns[i](arg)
  }
}
```

```javascript
// 定义对象属性 不可枚举
export const def = (obj: object, key: string | symbol, value: any) => {
  Object.defineProperty(obj, key, {
    configurable: true,
    enumerable: false,
    value
  })
}

// value——当试图获取属性时所返回的值。
// writable——该属性是否可写。
// enumerable——该属性在for in循环中是否会被枚举。
// configurable——该属性是否可被删除。
// set()——该属性的更新操作所调用的函数。
// get()——获取属性值时所调用的函数。
```

```javascript
// 转数字类型 
export const toNumber = (val: any): any => {
  const n = parseFloat(val)
  return isNaN(n) ? val : n
}

// 特别的 isNaN('a')为true
// 因此用isNaN来判断是否为NaN不严谨
// ES6新增Number.isNaN来弥补该API
```

```javascript
// 统一全局对象
let _globalThis: any
export const getGlobalThis = (): any => {
  return (
    _globalThis ||
    (_globalThis =
      typeof globalThis !== 'undefined'
        ? globalThis // 全局this指向
        : typeof self !== 'undefined'
        ? self // Web Worker
        : typeof window !== 'undefined'
        ? window // window
        : typeof global !== 'undefined'
        ? global // Node
        : {})
  )
}
```

#### 总结

- 除了一些常见的类型判断，也学习到例如缓存、字符串转换、空值、统一指针等实现思路
