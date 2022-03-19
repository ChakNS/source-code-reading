[pnpm/only-allow](https://github.com/pnpm/only-allow/blob/master/bin.js)

### 功能
统一规范包管理器
### 源码
```javascript
#!/usr/bin/env node
// 获取当前使用包管理器及其版本
const whichPMRuns = require('which-pm-runs')
// 终端盒子输出
const boxen = require('boxen')

// 从process上获取终端命令
const argv = process.argv.slice(2)
// 必须传入要锁定的包管理器
if (argv.length === 0) {
  console.log('Please specify the wanted package manager: only-allow <npm|pnpm|yarn>')
  process.exit(1)
}

// 仅支持npm、pnpm、yarn
const wantedPM = argv[0]
if (wantedPM !== 'npm' && wantedPM !== 'pnpm' && wantedPM !== 'yarn') {
  console.log(`"${wantedPM}" is not a valid package manager. Available package managers are: npm, pnpm, or yarn.`)
  process.exit(1)
}

// 校验使用的包管理器是否合法
const usedPM = whichPMRuns()
if (usedPM && usedPM.name !== wantedPM) {
  const boxenOpts = { borderColor: 'red', borderStyle: 'double', padding: 1 }
  switch (wantedPM) {
    case 'npm':
      console.log(boxen('Use "npm install" for installation in this project', boxenOpts))
      break
    case 'pnpm':
      console.log(boxen(`Use "pnpm install" for installation in this project.
If you don't have pnpm, install it via "npm i -g pnpm".
For more details, go to https://pnpm.js.org/`, boxenOpts))
      break
    case 'yarn':
      console.log(boxen(`Use "yarn" for installation in this project.
If you don't have Yarn, install it via "npm i -g yarn".
For more details, go to https://yarnpkg.com/`, boxenOpts))
      break
  }
  process.exit(1)
}
```
### 总结

- 学习到`whichPMRuns``boxen`的使用
- `process`获取终端命令
- `only-allow`内部逻辑
- `npm`生命周期的使用 `preinstall` `install` `postinstall`
- `"preinstall":"npx only-allow pnpm"`一行代码实现统一包管理器
