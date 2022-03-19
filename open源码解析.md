[open](https://github.com/sindresorhus/open/blob/main/index.js)

> Open stuff like URLs, files, executables. Cross-platform.


在webpack中配置devServer.open可以在服务启动后自动打开浏览器<br />包括在vue-cli和create-react-app中也有相同的功能<br />三者都引用了[open](https://github.com/sindresorhus/open/blob/main/index.js)这个库

[open](https://github.com/sindresorhus/open/blob/main/index.js)根据当前系统，使用node的子进程child_process模块的spawn方法开启子进程调用相应的命令打开浏览器

```bash
#mac
open https://www.baidu.com

#windows
start https://www.baidu.com

#linux
xdg-open https://www.baidu.com
```
<a name="uqd8M"></a>
### 源码
```javascript
 // 代码有省略，只列出核心部分


const childProcess = require('child_process');

// 获取平台信息
const {platform, arch} = process;

// 主函数
const open = (target, options) => {
	if (typeof target !== 'string') {
		throw new TypeError('Expected a `target`');
	}

  // 执行baseOpen
	return baseOpen({
		...options,
		target
	});
};

const baseOpen = async options => {
	// 参数处理...

	let command;

  // mac
	if (platform === 'darwin') {
		command = 'open';

    // 参数处理...
	} else if (platform === 'win32' || (isWsl && !isDocker())) {
    // windows
    const encodedArguments = ['Start'];
    
		// 参数处理...
	} else {
    // linux
		if (app) {
			command = app;
		} else {
			const useSystemXdgOpen = process.versions.electron ||
				platform === 'android' || isBundled || !exeLocalXdgOpen;
			command = useSystemXdgOpen ? 'xdg-open' : localXdgOpenPath;
      
      // 参数处理...
		}
	}

	// 开启子进程 执行command
	const subprocess = childProcess.spawn(command, cliArguments, childProcessOptions);

  // 独立于父进程
	subprocess.unref();

	return subprocess;
};

module.exports = open;
```
<a name="OB4q2"></a>
### 总结

- 根据不同的系统，初始化相应的命令
- spawn开启子进程执行命令打开浏览器
