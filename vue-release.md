[vue-release](https://github.com/lxchuan12/vue-next-analysis/blob/master/vue-next/scripts/release.js)

### 功能
> vue3.2 版本发布

### 源码
```javascript
// 命令参数解析
const args = require('minimist')(process.argv.slice(2))
const fs = require('fs')
const path = require('path')
// 终端多色彩输出
const chalk = require('chalk')
// 版本语义化
const semver = require('semver')
// 获取当前版本
const currentVersion = require('../package.json').version
// 终端交互
const { prompt } = require('enquirer')
// 执行命令
const execa = require('execa')



/* 一些参数的处理 */

// beta
const preId =
  args.preid ||
  (semver.prerelease(currentVersion) && semver.prerelease(currentVersion)[0])
const isDryRun = args.dry
// 跳过测试
const skipTests = args.skipTests
// 跳过打包
const skipBuild = args.skipBuild
// 读取packages文件夹下的文件，过滤掉非ts文件和非.开头的文件夹
const packages = fs
  .readdirSync(path.resolve(__dirname, '../packages'))
  .filter(p => !p.endsWith('.ts') && !p.startsWith('.'))

// 跳过的包
const skippedPackages = []
// 生成版本号
const versionIncrements = [
  'patch',
  'minor',
  'major',
  ...(preId ? ['prepatch', 'preminor', 'premajor', 'prerelease'] : [])
]

const inc = i => semver.inc(currentVersion, i, preId)


/* 封装bin下的执行脚本函数 */

// 获取脚本路径
const bin = name => path.resolve(__dirname, '../node_modules/.bin/' + name)
// 执行脚本
const run = (bin, args, opts = {}) =>
  execa(bin, args, { stdio: 'inherit', ...opts })
const dryRun = (bin, args, opts = {}) =>
  console.log(chalk.blue(`[dryrun] ${bin} ${args.join(' ')}`), opts)
const runIfNotDry = isDryRun ? dryRun : run
// 获取包路径
const getPkgRoot = pkg => path.resolve(__dirname, '../packages/' + pkg)
// 控制台输出步骤
const step = msg => console.log(chalk.cyan(msg))


/* 主函数 */


async function main() {
  // 获取用户键入的版本号
  let targetVersion = args._[0]

  // 如果没有 询问用户选择版本号
  if (!targetVersion) {
    const { release } = await prompt({
      type: 'select',
      name: 'release',
      message: 'Select release type',
      choices: versionIncrements.map(i => `${i} (${inc(i)})`).concat(['custom'])
    })

    // 用户选择自定义，则让用户输入版本号
    if (release === 'custom') {
      targetVersion = (
        await prompt({
          type: 'input',
          name: 'version',
          message: 'Input custom version',
          initial: currentVersion
        })
      ).version
    } else {
      targetVersion = release.match(/\((.*)\)/)[1]
    }
  }
	// 校验版本号
  if (!semver.valid(targetVersion)) {
    throw new Error(`invalid target version: ${targetVersion}`)
  }

  // 询问用户确认版本号
  const { yes } = await prompt({
    type: 'confirm',
    name: 'yes',
    message: `Releasing v${targetVersion}. Confirm?`
  })

  if (!yes) {
    return
  }

  // run tests before release
  step('\nRunning tests...')
  // 是否跳过
  if (!skipTests && !isDryRun) {
    // 执行测试脚本
    await run(bin('jest'), ['--clearCache'])
    await run('yarn', ['test', '--bail'])
  } else {
    console.log(`(skipped)`)
  }

  // update all package versions and inter-dependencies
  step('\nUpdating cross dependencies...')
  // 更新版本号及依赖版本
  updateVersions(targetVersion)

  // build all packages with types
  step('\nBuilding all packages...')
  // 是否跳过
  if (!skipBuild && !isDryRun) {
    // 执行打包脚本
    await run('yarn', ['build', '--release'])
    // test generated dts files
    step('\nVerifying type declarations...')
    await run('yarn', ['test-dts-only'])
  } else {
    console.log(`(skipped)`)
  }

  // 生成日志
  await run(`yarn`, ['changelog'])

  // 提交代码
  const { stdout } = await run('git', ['diff'], { stdio: 'pipe' })
  if (stdout) {
    step('\nCommitting changes...')
    await runIfNotDry('git', ['add', '-A'])
    await runIfNotDry('git', ['commit', '-m', `release: v${targetVersion}`])
  } else {
    console.log('No changes to commit.')
  }
 

  // 发布包
  step('\nPublishing packages...')
  for (const pkg of packages) {
    await publishPackage(pkg, targetVersion, runIfNotDry)
  }

  // 推送到github
  step('\nPushing to GitHub...')
  await runIfNotDry('git', ['tag', `v${targetVersion}`])
  await runIfNotDry('git', ['push', 'origin', `refs/tags/v${targetVersion}`])
  await runIfNotDry('git', ['push'])

  if (isDryRun) {
    console.log(`\nDry run finished - run git diff to see package changes.`)
  }

  if (skippedPackages.length) {
    console.log(
      chalk.yellow(
        `The following packages are skipped and NOT published:\n- ${skippedPackages.join(
          '\n- '
        )}`
      )
    )
  }
  console.log()
}

// 更新版本号
function updateVersions(version) {
  // 1. 更新根版本
  updatePackage(path.resolve(__dirname, '..'), version)
  // 2. 更新依赖版本
  packages.forEach(p => updatePackage(getPkgRoot(p), version))
}

// 更新版本号
function updatePackage(pkgRoot, version) {
  const pkgPath = path.resolve(pkgRoot, 'package.json')
  const pkg = JSON.parse(fs.readFileSync(pkgPath, 'utf-8'))
  pkg.version = version
  updateDeps(pkg, 'dependencies', version)
  updateDeps(pkg, 'peerDependencies', version)
  fs.writeFileSync(pkgPath, JSON.stringify(pkg, null, 2) + '\n')
}

// 更新依赖版本
function updateDeps(pkg, depType, version) {
  const deps = pkg[depType]
  if (!deps) return
  Object.keys(deps).forEach(dep => {
    if (
      dep === 'vue' ||
      (dep.startsWith('@vue') && packages.includes(dep.replace(/^@vue\//, '')))
    ) {
      console.log(
        chalk.yellow(`${pkg.name} -> ${depType} -> ${dep}@${version}`)
      )
      deps[dep] = version
    }
  })
}

// 发布包
async function publishPackage(pkgName, version, runIfNotDry) {
  if (skippedPackages.includes(pkgName)) {
    return
  }
  const pkgRoot = getPkgRoot(pkgName)
  const pkgPath = path.resolve(pkgRoot, 'package.json')
  const pkg = JSON.parse(fs.readFileSync(pkgPath, 'utf-8'))
  if (pkg.private) {
    return
  }

  // For now, all 3.x packages except "vue" can be published as
  // `latest`, whereas "vue" will be published under the "next" tag.
  let releaseTag = null
  if (args.tag) {
    releaseTag = args.tag
  } else if (version.includes('alpha')) {
    releaseTag = 'alpha'
  } else if (version.includes('beta')) {
    releaseTag = 'beta'
  } else if (version.includes('rc')) {
    releaseTag = 'rc'
  } else if (pkgName === 'vue') {
    // TODO remove when 3.x becomes default
    releaseTag = 'next'
  }

  // TODO use inferred release channel after official 3.0 release
  // const releaseTag = semver.prerelease(version)[0] || null

  step(`Publishing ${pkgName}...`)
  try {
    await runIfNotDry(
      'yarn',
      [
        'publish',
        '--new-version',
        version,
        ...(releaseTag ? ['--tag', releaseTag] : []),
        '--access',
        'public'
      ],
      {
        cwd: pkgRoot,
        stdio: 'pipe'
      }
    )
    console.log(chalk.green(`Successfully published ${pkgName}@${version}`))
  } catch (e) {
    if (e.stderr.match(/previously published/)) {
      console.log(chalk.red(`Skipping already published: ${pkgName}`))
    } else {
      throw e
    }
  }
}

main().catch(err => {
  console.error(err)
})
```
### 总结

- 学习vue的发布流程
- 学习node操作文件和执行脚本的用法
- 读第一遍还有点懵，第二遍就着川哥的文章读就清晰了
- 最近工作很忙，但再忙也得抽时间陪我川哥读源码😈
