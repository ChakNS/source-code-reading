[await-to-js](https://github.com/scopsy/await-to-js/blob/master/src/await-to-js.ts)

### 功能
> Async await wrapper for easy error handling

### 源码
```javascript

/**
 * @param { Promise } promise
 * @param { Object= } errorExt - Additional Information you can pass to the err object
 * @return { Promise }
 */
export function to<T, U = Error> (
  promise: Promise<T>,
  errorExt?: object
): Promise<[U, undefined] | [null, T]> {
  return promise
    .then<[null, T]>((data: T) => [null, data])
    .catch<[U, undefined]>((err: U) => {
      if (errorExt) {
        const parsedError = Object.assign({}, err, errorExt);
        return [parsedError, undefined];
      }

      return [err, undefined];
    });
}

export default to;
```
### 总结

- 学习一下泛型的用法
- 好巧不巧，前两天自己写了一个跟`await-to-js`类似的方法，当时还不知道这个库(比较简单，没有库那么严谨)

```typescript
async function silentHandle(fn, ...args) {
  let result = [null, null]

  try {
    result[1] = await fn(...args)
  } catch(e) {
    result[0] = e
  }

  return result
}

export default silentHandle
```
