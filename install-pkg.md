> Install package programmatically. Detect package managers automatically (npm, yarn and pnpm).
> 以编程方式安装包。自动检测包管理器（`npm`、`yarn` 和 `pnpm`）。


[Vue团队核心成员开发的39行小工具 install-pkg 安装包，值得一学！](https://github.com/lxchuan12/install-pkg-analysis)
[install-pkg](https://github.com/antfu/install-pkg)

### 源码
```javascript
// install-pkg/src/install.ts

import execa from 'execa' // 执行脚本 对child_process的改进
import { detectPackageManager } from '.'

// 可选参数接口
export interface InstallPackageOptions {
  cwd?: string
  dev?: boolean
  silent?: boolean
  packageManager?: string
  preferOffline?: boolean
  additionalArgs?: string[]
}

export async function installPackage(names: string | string[], options: InstallPackageOptions = {}) {
  // 如果没有指定包管理器，则自动探测所使用的包管理器
  // 以上都没有，则使用npm
  const agent = options.packageManager || await detectPackageManager(options.cwd) || 'npm'
  if (!Array.isArray(names)) names = [names]
  const args = options.additionalArgs || []

  if (options.preferOffline)
    args.unshift('--prefer-offline') // 离线优先

  return execa(
    agent,
    [
      agent === 'yarn'
        ? 'add'
        : 'install',
      options.dev ? '-D' : '',
      ...args,
      ...names,
    ].filter(Boolean),
    {
      stdio: options.silent ? 'ignore' : 'inherit',
      cwd: options.cwd, // 当前工作目录
    },
  )
  
  // 如 pnpm install -D --prefer-offine release-it react antd
}
```
```javascript
// install-pkg/src/detect.ts

import path from 'path'
import findUp from 'find-up'

// 只允许分配pnpm yarn npm
export type PackageManager = 'pnpm' | 'yarn' | 'npm'
const LOCKS: Record<string, PackageManager> = {
  'pnpm-lock.yaml': 'pnpm',
  'yarn.lock': 'yarn',
  'package-lock.json': 'npm',
}

export async function detectPackageManager(cwd = process.cwd()) {
  // 查找目标文件并获取路径，返回第一个查找结果并返回promise，否则返回undefined
  const result = await findUp(Object.keys(LOCKS), { cwd })
  // 提取包管理器
  const agent = (result ? LOCKS[path.basename(result)] : null)
  return agent
}
```
### 部分工具或配置
> `execa.options.stdio`
> - pipe：相当于['pipe', 'pipe', 'pipe']，子进程的stdio和父进程的stdio通过管道进行连接
> - ignore：相当于['ignore','ignore', 'ignore']，子进程的stdio绑定到/dev/null,丢弃数据的输入输出。
> - inherit：继承父进程相关的stdio,等同于[process.stdin,process.stdout,process.sterr]或者[0,1,2],此时子进程的stdio都是绑定在同一个地方。

> `findUp`
> Find a file or directory by walking up parent directories
> 通过遍历父目录查找文件或目录
> ### findUp([...name], 选项？)
> 返回获取到的第一个路径，如果没有，则为undefined


### scripts
```json
"scripts": {
  "prepublishOnly": "nr build",
  "dev": "nr build --watch",
  "start": "esno src/index.ts",
  "build": "tsup src/index.ts --format cjs,esm --dts --no-splitting",
  "release": "bumpp --commit --push --tag && pnpm publish",
  "lint": "eslint \"{src,test}/**/*.ts\"",
  "lint:fix": "nr lint -- --fix"
},
```

- [ ni](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fantfu%2Fni)
> 自动根据锁文件 yarn.lock / pnpm-lock.yaml / package-lock.json 检测使用 yarn / pnpm / npm 的包管理器。

- [esno](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fantfu%2Fesno%23readme)
> TypeScript / ESNext node runtime powered by esbuild
> 使用 esbuild 即时传输TypeScript 和 esnext 功能

- [tsup](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fegoist%2Ftsup%23readme)
> 打包ts

- [bumpp](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fantfu%2Fbumpp)
> 交互式 CLI 可增加版本号等

### github action workflows
GitHub的持续集成
之前在项目中有使用Jenkins做自动化部署，脚本的整体的结构有些类似
GitHub Action特别的地方在于可以引用官方或别人的action
第一次看起来倒有点像docker的感觉
Mark一下，有时间动手做个实践
```yaml
// install-pkg/.github/workflows/release.yml

# workflow的名称
name: Release 

# trigger push tags为vxxx时触发
on:
  push:
    tags:
      - 'v*'

jobs:
  release:
  	# 运行所需的虚拟机环境
    runs-on: ubuntu-latest  
    steps:
    	# 第一步，使用checkout action拉取源码
      - uses: actions/checkout@v2 
        with:
          fetch-depth: 0
      # 第二步，使用setup-node 安装node环境
      - uses: actions/setup-node@v2 
        with:
          node-version: '14'
          registry-url: https://registry.npmjs.org/
      # 执行一系列命令
      # 全局安装pnpm @antfu/ni
      - run: npm i -g pnpm @antfu/ni   
      # npm ci 不更新锁文件 --frozen-lockfile
      - run: nci 
      # 若存在test命令则执行
      - run: nr test --if-present
      - run: npx conventional-github-releaser -p angular
        env:
          CONVENTIONAL_GITHUB_RELEASER_TOKEN: ${{secrets.GITHUB_TOKEN}}

```
### 总结

- 学习构建一个ts的npm包
- 学习GitHub action
- 学习findUp、execa、ni等的使用
- 特别的，一个简单的写法，着实让我眼前一亮，以前没想过这么用，果然看源码能发现很多惊喜
```javascript
[ 1, 2, undefined ].filter(Boolean)
// 常用的写法
[ 1, 2, undefined ].filter(item => item)
```
