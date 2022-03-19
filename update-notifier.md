[update-notifier](https://github.com/yeoman/update-notifier/blob/main/index.js)

> Update notifications for your CLI app

<a name="qZF5k"></a>
### 源码

- 获取package.json的name和version
- 设置检查间隔时间、静默等参数
- configStore持久化配置及包信息
   - configStore会将缓存以json形式存储在临时文件区或者指定位置
- 实例化并调check方法
- 若有更新缓存，则更改当前版本信息
- 检查是否超过更新间隔，是则开启子进程跑check.js
- 调用fetchInfo获取当前npm registry上的包信息，拿到最新版本
- 设置更新信息，检查时间等
- notify输出更新信息
```javascript
constructor(options = {}) {
  	// 整合并校验参数
		this.options = options;
		options.pkg = options.pkg || {};
		options.distTag = options.distTag || 'latest';
		options.pkg = {
			name: options.pkg.name || options.packageName,
			version: options.pkg.version || options.packageVersion
		};
		if (!options.pkg.name || !options.pkg.version) {
			throw new Error('pkg.name and pkg.version required');
		}

		this.packageName = options.pkg.name;
		this.packageVersion = options.pkg.version;
  	// 检查间隔时间
		this.updateCheckInterval = typeof options.updateCheckInterval === 'number' ? options.updateCheckInterval : ONE_DAY;
    // 是否静默
		this.disabled = 'NO_UPDATE_NOTIFIER' in process.env ||
			process.env.NODE_ENV === 'test' ||
			process.argv.includes('d') ||
			isCi();
		this.shouldNotifyInNpmScript = options.shouldNotifyInNpmScript;

  	// 保存配置
		if (!this.disabled) {
			try {
				const ConfigStore = configstore();
				this.config = new ConfigStore(`update-notifier-${this.packageName}`, {
					optOut: false,
					lastUpdateCheck: Date.now()
				});
			} catch {
				// 抛错
			}
		}
	}
```
```javascript
check() {
  	// 异常 退出
		if (
			!this.config ||
			this.config.get('optOut') ||
			this.disabled
		) {
			return;
		}

  	
		this.update = this.config.get('update');
		
  	// 更新缓存的当前版本
		if (this.update) {
			// Use the real latest version instead of the cached one
			this.update.current = this.packageVersion;

			// Clear cached information
			this.config.delete('update');
		}

		// 是否到检查间隔
		if (Date.now() - this.config.get('lastUpdateCheck') < this.updateCheckInterval) {
			return;
		}

		// 开启子进程用于更新包信息
		spawn(process.execPath, [path.join(__dirname, 'check.js'), JSON.stringify(this.options)], {
			detached: true, // 子进程独立于父进程
			stdio: 'ignore' // 忽略控制台输入输出
		}).unref(); // 使子进程完全独立于父进程，不受父进程退出影响
	}
```
```javascript
// check.js

'use strict';
let updateNotifier = require('.');

const options = JSON.parse(process.argv[2]);

updateNotifier = new updateNotifier.UpdateNotifier(options);

(async () => {
	// 超时退出
	setTimeout(process.exit, 1000 * 30);

  // 获取包信息
	const update = await updateNotifier.fetchInfo();

	// 更新
	updateNotifier.config.set('lastUpdateCheck', Date.now());

	if (update.type && update.type !== 'latest') {
		updateNotifier.config.set('update', update);
	}

	// 主动中止子进程
	process.exit();
})().catch(error => {
	console.error(error);
	process.exit(1);
});
```
```javascript
// 获取包信息
async fetchInfo() {
		const {distTag} = this.options;
    // 获取包最新版本
		const latest = await latestVersion()(this.packageName, {version: distTag});

		return {
			latest,
			current: this.packageVersion,
			type: semverDiff()(this.packageVersion, latest) || distTag,
			name: this.packageName
		};
	}
```
```javascript
notify(options) {
  	// 是否静默
		const suppressForNpm = !this.shouldNotifyInNpmScript && isNpm().isNpmOrYarn;
		if (!process.stdout.isTTY || suppressForNpm || !this.update || !semver().gt(this.update.latest, this.update.current)) {
			return this;
		}

		options = {
			isGlobal: isInstalledGlobally(),
			isYarnGlobal: isYarnGlobal()(),
			...options
		};

		let installCommand;
  	// yarn global
		if (options.isYarnGlobal) {
			installCommand = `yarn global add ${this.packageName}`;
		} else if (options.isGlobal) {
    // npm global
			installCommand = `npm i -g ${this.packageName}`;
		} else if (hasYarn()()) {
    // yarn scoped
			installCommand = `yarn add ${this.packageName}`;
		} else {
    // npm scoped
			installCommand = `npm i ${this.packageName}`;
		}
		
  	// 提示及样式
		const defaultTemplate = 'Update available ' +
			chalk().dim('{currentVersion}') +
			chalk().reset(' → ') +
			chalk().green('{latestVersion}') +
			' \nRun ' + chalk().cyan('{updateCommand}') + ' to update';

		const template = options.message || defaultTemplate;

		options.boxenOptions = options.boxenOptions || {
			padding: 1,
			margin: 1,
			align: 'center',
			borderColor: 'yellow',
			borderStyle: 'round'
		};

		const message = boxen()(
			pupa()(template, {
				packageName: this.packageName,
				currentVersion: this.update.current,
				latestVersion: this.update.latest,
				updateCommand: installCommand
			}),
			options.boxenOptions
		);

		if (options.defer === false) {
			console.error(message);
		} else {
			process.on('exit', () => {
				console.error(message);
			});

			process.on('SIGINT', () => {
				console.error('');
				process.exit();
			});
		}

		return this;
	}
```
```javascript
// 暴露初始化方法和整个类
module.exports = options => {
	const updateNotifier = new UpdateNotifier(options);
	updateNotifier.check();
	return updateNotifier;
};

module.exports.UpdateNotifier = UpdateNotifier;
```
<a name="W2Fki"></a>
### 总结

- 了解update-notifier的处理流程
- 学会spawn开启独立子进程处理数据
- 巩固configstore的应用
- 克服看源码的恐惧心理，其实分析下来也没有特别复杂，只是大佬思维更严谨、考虑也更周全，值得学习
