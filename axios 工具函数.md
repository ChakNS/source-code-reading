### axios 工具函数

#### 源码地址

- [https://github.com/axios/axios/blob/master/lib/utils.js](https://github.com/axios/axios/blob/master/lib/utils.js)
- [axios/bind.js at master · axios/axios · GitHub](https://github.com/axios/axios/blob/master/lib/helpers/bind.js)

#### 功能

- `utils` 是一个非特定于 axios 的通用辅助函数库

#### 源码阅读

```javascript
'use strict';

// helpers中封装的bind函数
var bind = require('./helpers/bind');

// helpers-bind
// 封装后返回一个wrap函数，调用时更改this指向为thisArg
function bind(fn, thisArg) {
  return function wrap() {
    var args = new Array(arguments.length);
    for (var i = 0; i < args.length; i++) {
      args[i] = arguments[i];
    }
    return fn.apply(thisArg, args);
  };
};

// 指向Object的原型方法toString
var toString = Object.prototype.toString;

// 判断一个值是否为数组
function isArray(val) {
  return Array.isArray(val);
}

// 判断一个值是否为undefined
function isUndefined(val) {
  return typeof val === 'undefined';
}

// 判断一个值是否为Buffer 二进制数据类型
// 主要判断构造函数是否为Buffer类
function isBuffer(val) {
  return val !== null
    && !isUndefined(val)
    && val.constructor !== null
    && !isUndefined(val.constructor)
    && typeof val.constructor.isBuffer === 'function'
    && val.constructor.isBuffer(val);
}

// 判断一个值是否为ArrayBuffer二进制数据缓冲区
function isArrayBuffer(val) {
  return toString.call(val) === '[object ArrayBuffer]';
}

// 判断一个值是否为FormData
function isFormData(val) {
  return toString.call(val) === '[object FormData]';
}

// 判断一个值是否为ArrayBuffer视图，比如new Uint8Array()、new Float32Array()
function isArrayBufferView(val) {
  var result;
  if ((typeof ArrayBuffer !== 'undefined') && (ArrayBuffer.isView)) {
    // 若存在ArrayBuffer类，直接使用类上的方法isView
    result = ArrayBuffer.isView(val);
  } else { // 否则使用上面的isArrayBuffer方法
    result = (val) && (val.buffer) && (isArrayBuffer(val.buffer));
  }
  return result;
}

// 判断一个值是否为字符串类型
function isString(val) {
  return typeof val === 'string';
}

// 判断一个值是否为数字类型
function isNumber(val) {
  return typeof val === 'number';
}

// 判断一个值是否为对象
function isObject(val) {
  return val !== null && typeof val === 'object';
}

// 判断一个值是否为纯对象 如{} | new Object()
function isPlainObject(val) {
  if (toString.call(val) !== '[object Object]') {
    return false;
  }

  // 主要判断指向的原型是否为空 或 为Object的原型对象
  var prototype = Object.getPrototypeOf(val);
  return prototype === null || prototype === Object.prototype;
}

// 判断一个值是否为日期类型
function isDate(val) {
  return toString.call(val) === '[object Date]';
}

// 判断一个值是否为文件类型
function isFile(val) {
  return toString.call(val) === '[object File]';
}

// 判断一个值是否为Blob类型
function isBlob(val) {
  return toString.call(val) === '[object Blob]';
}

// 判断一个值是否为Function类型 即函数
function isFunction(val) {
  return toString.call(val) === '[object Function]';
}

// 判断一个值是否为流
// 通过判断是否存在pipe方法
function isStream(val) {
  return isObject(val) && isFunction(val.pipe);
}

// 判断一个值是否为URLSearchParams类型
// 处理 URL 的查询字符串
// URLSearchParams实例化的对象是可枚举的
function isURLSearchParams(val) {
  return toString.call(val) === '[object URLSearchParams]';
}

// 去除字符串首尾空格，优先使用trim方法，不支持则使用正则
function trim(str) {
  return str.trim ? str.trim() : str.replace(/^\s+|\s+$/g, '');
}

// 判断是否标准浏览器环境
// 官方不再推荐navigator.product
function isStandardBrowserEnv() {
  // 若支持navigator，优先使用navigator.product判断
  // 若为 ReactNative | NativeScript | NS，则不是
  if (typeof navigator !== 'undefined'
    && (navigator.product === 'ReactNative'
    || navigator.product === 'NativeScript'
    || navigator.product === 'NS')) {
    return false;
  }
  // 不支持navigator，则判断是否存在window和document
  return (
    typeof window !== 'undefined' &&
    typeof document !== 'undefined'
  );
}

// 实现forEach方法 应该是考虑兼容的问题
function forEach(obj, fn) {
  // 值为空，不处理
  if (obj === null || typeof obj === 'undefined') {
    return;
  }

  // 基础数据类型，转化为数组
  if (typeof obj !== 'object') {
    /*eslint no-param-reassign:0*/
    obj = [obj];
  }

  if (isArray(obj)) { // 如果是数组 for循环
    for (var i = 0, l = obj.length; i < l; i++) {
      fn.call(null, obj[i], i, obj);
    }
  } else { // 如果是对象，for/in迭代对象
    for (var key in obj) {
      if (Object.prototype.hasOwnProperty.call(obj, key)) {
        fn.call(null, obj[key], key, obj);
      }
    }
  }
}

// 实现Object.assign 考虑兼容
// 使用了上面的isPlainObject、isArray、forEach
function merge(/* obj1, obj2, obj3, ... */) {
  var result = {};

  // 合并主函数
  function assignValue(val, key) {
    if (isPlainObject(result[key]) && isPlainObject(val)) {
      // 如果结果对象中key对应的值存在且为对象，合并值也为对象，则递归合并这两个对象
      result[key] = merge(result[key], val);
    } else if (isPlainObject(val)) {
      // 如果result中对应key的值为空，且并入值为对象，合并（拷贝）
      result[key] = merge({}, val);
    } else if (isArray(val)) {
      // 如果result中对应key的值为空，且并入值为数组，合并（拷贝）
      result[key] = val.slice();
    } else {
      // 基础类型，直接赋值
      result[key] = val;
    }
  }
  // 迭代arguments，可能为对象、数组、基础类型、null、undefined
  // 按照forEach的规则，null、undefined不处理
  // 基础类型转换为数组
  for (var i = 0, l = arguments.length; i < l; i++) {
    forEach(arguments[i], assignValue);
  }
  return result;
}

// 对象扩展
// a是被扩展的对象，b是扩展属性的来源，thisArg是扩展函数的this指向
function extend(a, b, thisArg) {
  // 迭代对象b
  forEach(b, function assignValue(val, key) {
    // 值为函数类型，使用bind绑定函数并赋值给a
    if (thisArg && typeof val === 'function') {
      a[key] = bind(val, thisArg);
    } else {
    // 其他类型直接赋值给a
      a[key] = val;
    }
  });
  return a;
}

// 移除字节顺序标记 BOM - byte order mark
// 出现在文本文件头部，Unicode编码标准中用于标识文件是采用哪种格式的编码
function stripBOM(content) {
  if (content.charCodeAt(0) === 0xFEFF) {
    content = content.slice(1);
  }
  return content;
}

module.exports = {
  isArray: isArray,
  isArrayBuffer: isArrayBuffer,
  isBuffer: isBuffer,
  isFormData: isFormData,
  isArrayBufferView: isArrayBufferView,
  isString: isString,
  isNumber: isNumber,
  isObject: isObject,
  isPlainObject: isPlainObject,
  isUndefined: isUndefined,
  isDate: isDate,
  isFile: isFile,
  isBlob: isBlob,
  isFunction: isFunction,
  isStream: isStream,
  isURLSearchParams: isURLSearchParams,
  isStandardBrowserEnv: isStandardBrowserEnv,
  forEach: forEach,
  merge: merge,
  extend: extend,
  trim: trim,
  stripBOM: stripBOM
};
```

#### 总结

-  axios的utils工具函数中大量使用了Object.prototype.toString来判断数据类型 
-  同时为了兼容不同的浏览器，对一些常用的方法例如`forEach`|`assign`做了封装实现 
-  代码结构清晰，每个函数只实现了一个功能，这是在工具函数封装上非常重要的规范 
-  代码规范方面很值得学习 
