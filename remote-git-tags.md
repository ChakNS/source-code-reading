#### 源码地址

- [https://github.com/sindresorhus/remote-git-tags](https://github.com/sindresorhus/remote-git-tags)

#### 功能

- 从远程仓库获取所有标签

#### 源码阅读

- 代码结构

```javascript
// promisify 把callback形式转换成promise形式
import {promisify} from 'node:util';
// child_process 子进程模块
import childProcess from 'node:child_process';

// execFile 执行命令
const execFile = promisify(childProcess.execFile);

export default async function remoteGitTags(repoUrl) {
	// 执行 git ls-remote --tags repoUrl
	const {stdout} = await execFile('git', ['ls-remote', '--tags', repoUrl]);
	const tags = new Map();

	// 储存在map
	for (const line of stdout.trim().split('\n')) {
		const [hash, tagReference] = line.split('\t');
		// `refs/tags/v9.6.0^{}` → `v9.6.0`
		const tagName = tagReference.replace(/^refs\/tags\//, '').replace(/\^{}$/, '');

		tags.set(tagName, hash);
	}

	return tags;
}
```

#### promisify

```javascript
function promisify(original) {
	// 核心代码
  function fn(...args) {
  	// 增加回调 转换为promise
    return new Promise((resolve, reject) => {
      ArrayPrototypePush(args, (err, ...values) => {
        if (err) {
          return reject(err);
        }
        if (argumentNames !== undefined && values.length > 1) {
          const obj = {};
          for (let i = 0; i < argumentNames.length; i++)
            obj[argumentNames[i]] = values[i];
          resolve(obj);
        } else {
          resolve(values[0]);
        }
      });
      // 执行原函数
      ReflectApply(original, this, args);
    });
  }

  ObjectSetPrototypeOf(fn, ObjectGetPrototypeOf(original));

  ObjectDefineProperty(fn, kCustomPromisifiedSymbol, {
    value: fn, enumerable: false, writable: false, configurable: true
  });
  return ObjectDefineProperties(
    fn,
    ObjectGetOwnPropertyDescriptors(original)
  );
}
```

#### 总结&收获

- 认识`promisify`、`child_process`、`execFile`
- 认识命令`git ls-remote --tags repoUrl`
