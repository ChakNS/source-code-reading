> Easily load and persist config without having to think about where and how
> 轻松加载和保存配置，不需要关心怎样存储或存储在哪


[configstore github](https://github.com/yeoman/configstore)

### 源码
```javascript
/*
* path                - 内置模块，操作文件和目录路径
* os                  - 内置模块，操作系统相关操作
* graceful-fs         - 操作文件，兼容不同环境，升级版的fs
* xdg-basedir         - Linux获取用户配置文件路径
* write-file-atomic   - 基于fs.writeFile的扩展模块，用于文件写入，设置文件权限
* dot-prop            - 多层级嵌套操作对象
* unique-string       - 随机生成32位字符串
*/

import path from 'path';
import os from 'os';
import fs from 'graceful-fs';
import {xdgConfig} from 'xdg-basedir';
import writeFileAtomic from 'write-file-atomic';
import dotProp from 'dot-prop';
import uniqueString from 'unique-string';

/*
* configDirectory     - 获取临时文件路径
* permissionError     - 无权限提示语
* mkdirOptions        - 创建文件默认参数，mode - 文件权限 0o0700 - 可读可写可执行， recursive - 是否递归创建
* writeFileOptions    - 写文件默认参数
*/

const configDirectory = xdgConfig || path.join(os.tmpdir(), uniqueString());
const permissionError = 'You don\'t have access to this file.';
const mkdirOptions = {mode: 0o0700, recursive: true};
const writeFileOptions = {mode: 0o0600};




export default class Configstore {
	constructor(id, defaults, options = {}) {
    // 如果配置了globalConfigPath 则将id和config.json拼接成存储文件路径名称
    // 否则默认为configstore/${id}.json
		const pathPrefix = options.globalConfigPath ?
			path.join(id, 'config.json') :
			path.join('configstore', `${id}.json`);
		
    // 如果配置了configPath，则将其作为完整路径
    // 否则将临时文件路径和存储文件路径名称拼接成完整路径
		this._path = options.configPath || path.join(configDirectory, pathPrefix);
		
    
    // 默认/初始化存储值
    // all为当前实例的存储数据 - 对象
		if (defaults) {
			this.all = {
				...defaults,
				...this.all
			};
		}
	}

  // 当获取all属性是，尝试以utf8格式读取存储数据
	get all() {
		try {
			return JSON.parse(fs.readFileSync(this._path, 'utf8'));
		} catch (error) {
			// 文件不存在，返回空对象
			if (error.code === 'ENOENT') {
				return {};
			}

			// 没有读取权限，抛出提示
			if (error.code === 'EACCES') {
				error.message = `${error.message}\n${permissionError}\n`;
			}

			// 无效的 JSON 清空文件并返回空对象
			if (error.name === 'SyntaxError') {
				writeFileAtomic.sync(this._path, '', writeFileOptions);
				return {};
			}

			throw error;
		}
	}

  // 当给all赋值时 尝试存储数据 覆盖写入
	set all(value) {
		try {
			// 确保文件存在，且有修改的权限
			fs.mkdirSync(path.dirname(this._path), mkdirOptions);
			
      // 写入数据
			writeFileAtomic.sync(this._path, JSON.stringify(value, undefined, '\t'), writeFileOptions);
		} catch (error) {
			// 抛出无权限提示
			if (error.code === 'EACCES') {
				error.message = `${error.message}\n${permissionError}\n`;
			}

			throw error;
		}
	}

  // 读取size属性时，获取存储对象keys长度
	get size() {
		return Object.keys(this.all || {}).length;
	}

  // 获取目标数据
	get(key) {
		return dotProp.get(this.all, key);
	}

  // 存储数据
	set(key, value) {
		const config = this.all;

    // 如果只有一个参数，视为对象，遍历对象的key进行存储
		if (arguments.length === 1) {
			for (const k of Object.keys(key)) {
				dotProp.set(config, k, key[k]);
			}
		} else {
			dotProp.set(config, key, value);
		}

		this.all = config;
	}

  // 检测是否存在目标数据
	has(key) {
		return dotProp.has(this.all, key);
	}

  // 删除目标数据
	delete(key) {
		const config = this.all;
		dotProp.delete(config, key);
		this.all = config;
	}

  // 清空存储数据
	clear() {
		this.all = {};
	}

  // 读取path属性时，返回当前存储数据的完整路径
	get path() {
		return this._path;
	}
}

```
### 总结

- 学习configstore的使用
- 学习开发JS库的一些规范
- 该库的核心在于多环境兼容

