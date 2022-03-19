#### 【本文参加了[每周一起学习200行源码共读活动](https://www.yuque.com/ruochuan12/topics/1)】

#### 前言
有幸参加若川大佬的源码共读活动，从简单的源码开始，跟上大佬的步伐！

#### 源码地址

- [https://github.com/npm/validate-npm-package-name/blob/main/index.js](https://github.com/npm/validate-npm-package-name/blob/main/index.js)

#### 功能

- 检测一个字符串是否是一个有效的包命名

#### 源码阅读

- 代码结构
```javascript
// 匹配作用域包名 如@user/package
var scopedPackagePattern = new RegExp('^(?:@([^/]+?)[/])?([^/]+?)$')
// node.js/io.js的核心模块名
var builtins = require('builtins')
// 命名黑名单
var blacklist = [
  'node_modules',
  'favicon.ico'
]

var validate = module.exports = function (name) {
	// 主函数 校验字符串是否符合规则
}

var done = function (warnings, errors) {
	// 结果处理函数
}
```

- `validate`函数
```javascript
// 默认警告和错误信息
var warnings = []
var errors = []

// errors校验

// 不能为null
if (name === null) {
  errors.push('name cannot be null')
  return done(warnings, errors)
}

// 不能为undefined
if (name === undefined) {
  errors.push('name cannot be undefined')
  return done(warnings, errors)
}

// 必须为string类型
if (typeof name !== 'string') {
  errors.push('name must be a string')
  return done(warnings, errors)
}

// 不能为空字符串
if (!name.length) {
  errors.push('name length must be greater than zero')
}

// 不能以 . 开头
if (name.match(/^\./)) {
  errors.push('name cannot start with a period')
}

// 不能以 _ 开头
if (name.match(/^_/)) {
  errors.push('name cannot start with an underscore')
}

// 首尾不能有空格
if (name.trim() !== name) {
  errors.push('name cannot contain leading or trailing spaces')
}

// 不能在黑名单内
blacklist.forEach(function (blacklistedName) {
  if (name.toLowerCase() === blacklistedName) {
    errors.push(blacklistedName + ' is a blacklisted name')
  }
})

// warnings校验
// 以前允许的规则，作为警告输出

// 与核心模块名称冲突
builtins.forEach(function (builtin) {
  if (name.toLowerCase() === builtin) {
    warnings.push(builtin + ' is a core module name')
  }
})


// 长度不能超出214
if (name.length > 214) {
  warnings.push('name can no longer contain more than 214 characters')
}

// 必须小写
if (name.toLowerCase() !== name) {
  warnings.push('name can no longer contain capital letters')
}

// 不能包含 ~)('!* 等字符
if (/[~'!()*]/.test(name.split('/').slice(-1)[0])) {
  warnings.push('name can no longer contain special characters ("~\'!()*")')
}

// 不能包含非url安全字符
if (encodeURIComponent(name) !== name) {
  // 有可能是个作用域包名
  var nameMatch = name.match(scopedPackagePattern)
  if (nameMatch) {
    var user = nameMatch[1]
    var pkg = nameMatch[2]
    // 拆开作用域包名，分别校验是否包含非url安全字符
    if (encodeURIComponent(user) === user && encodeURIComponent(pkg) === pkg) {
      return done(warnings, errors)
    }
  }

  errors.push('name can only contain URL-friendly characters')
}

// 返回结果
return done(warnings, errors)
```

- `done`函数
```javascript
var done = function (warnings, errors) {
  var result = {
    // 是否符合现行规则
    validForNewPackages: errors.length === 0 && warnings.length === 0,
    // 是否符合以前的规则
    validForOldPackages: errors.length === 0,
    // 警告信息
    warnings: warnings,
    // 错误信息
    errors: errors
  }
  if (!result.warnings.length) delete result.warnings
  if (!result.errors.length) delete result.errors
  return result
}
```
#### 
#### 总结

- 比较简单的项目，包含一些基本的校验规则
- 代码逻辑清晰，值得学习
