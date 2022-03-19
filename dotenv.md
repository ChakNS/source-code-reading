[dotenv](https://github.com/motdotla/dotenv/blob/master/lib/main.js)

### 功能
> Dotenv is a zero-dependency module that loads environment variables from a .env file into process.env.

### 源码
```javascript
const fs = require('fs')
const path = require('path')
const os = require('os')

const LINE = /(?:^|^)\s*(?:export\s+)?([\w.-]+)(?:\s*=\s*?|:\s+?)(\s*'(?:\\'|[^'])*'|\s*"(?:\\"|[^"])*"|\s*`(?:\\`|[^`])*`|[^#\r\n]+)?\s*(?:#.*)?(?:$|$)/mg

// 将源代码字符串解析为对象
function parse (src) {
  const obj = {}

  // 将 buffer 转为 string
  let lines = src.toString()

  // 转换换行符\r \n为同一种格式
  lines = lines.replace(/\r\n?/mg, '\n')

  let match
  
  // 找出键值对并解析为对象
  while ((match = LINE.exec(lines)) != null) {
    const key = match[1]

    // 值为 undefined or null，默认为空字符串
    let value = (match[2] || '')

    value = value.trim()
    
    const maybeQuote = value[0]

    // "" 和 `` 替换成单引号
    value = value.replace(/^(['"`])([\s\S]*)\1$/mg, '$2')

    // 如果有双引号 则扩展至下一行
    if (maybeQuote === '"') {
      value = value.replace(/\\n/g, '\n')
      value = value.replace(/\\r/g, '\r')
    }

    obj[key] = value
  }

  return obj
}

// 提示日志
function _log (message) {
  console.log(`[dotenv][DEBUG] ${message}`)
}

// 解析传入的路径，如果是~ 转换为当前操作系统的根目录
function _resolveHome (envPath) {
  return envPath[0] === '~' ? path.join(os.homedir(), envPath.slice(1)) : envPath
}

// 解析源代码键值对，并加到procee.env上
function config (options) {
  // 默认目标文件为当前目录的.env文件
  let dotenvPath = path.resolve(process.cwd(), '.env')
  // 默认编码为utf-8
  let encoding = 'utf8'
  // 默认不开启debug
  const debug = Boolean(options && options.debug)
  // 默认不覆盖process.env原有的属性
  const override = Boolean(options && options.override)

  if (options) {
    // 如有自定义的配置路径，则采用
    if (options.path != null) {
      dotenvPath = _resolveHome(options.path)
    }
    // 如有自定义的编码规则，则采用
    if (options.encoding != null) {
      encoding = options.encoding
    }
  }

  try {
    // 解析
    const parsed = DotenvModule.parse(fs.readFileSync(dotenvPath, { encoding }))

    // 添加到process.env
    Object.keys(parsed).forEach(function (key) {
      // 原来不存在的属性，直接增加
      if (!Object.prototype.hasOwnProperty.call(process.env, key)) {
        process.env[key] = parsed[key]
      } else {
        if (override === true) {
          // 覆盖原有属性
          process.env[key] = parsed[key]
        }

        // 提示日志
        if (debug) {
          if (override === true) {
            _log(`"${key}" is already defined in \`process.env\` and WAS overwritten`)
          } else {
            _log(`"${key}" is already defined in \`process.env\` and was NOT overwritten`)
          }
        }
      }
    })

    // 返回解析的结果
    return { parsed }
  } catch (e) {
    if (debug) {
      _log(`Failed to load ${dotenvPath} ${e.message}`)
    }

    return { error: e }
  }
}

const DotenvModule = {
  config,
  parse
}

module.exports.config = DotenvModule.config
module.exports.parse = DotenvModule.parse
module.exports = DotenvModule
```
### 总结

- 核心的实现思路就是读取配置文件获取键值对并赋值到`process.env`上，这个不难想到
- 但一个库除了实现核心功能，也要足够灵活考虑到不同的场景，并尽量做到提示友好
- 例如自定义配置路径，那在实际使用中，我们就可以在webpack通过读取不同的环境文件，达到区分环境配置的效果
- 另外`env文件`的编写自由度高，合理的校验规则以及友好的提示就显得尤为重要
