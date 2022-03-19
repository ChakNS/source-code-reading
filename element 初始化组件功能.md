[初始化组件](https://github.com/ElemeFE/element/blob/dev/build/bin/new.js)

<a name="VcXls"></a>
### 功能
> 脚本自动生成组件文件并导入

<a name="qZF5k"></a>
### 源码
```javascript
'use strict';
// 一些模块和路径
const path = require('path');
const fs = require('fs');
// 写文件
const fileSave = require('file-save');
const uppercamelcase = require('uppercamelcase');
const componentname = process.argv[2];
const chineseName = process.argv[3] || componentname;
const ComponentName = uppercamelcase(componentname);
const PackagePath = path.resolve(__dirname, '../../packages', componentname);
// 需要创建的文件列表
const Files = [
  {
    filename: 'index.js',
    content: `import ${ComponentName} from './src/main';

/* istanbul ignore next */
${ComponentName}.install = function(Vue) {
  Vue.component(${ComponentName}.name, ${ComponentName});
};

export default ${ComponentName};`
  },
  
  ...

];

// 添加到 components.json
const componentsFile = require('../../components.json');
if (componentsFile[componentname]) {
  console.error(`${componentname} 已存在.`);
  process.exit(1);
}
componentsFile[componentname] = `./packages/${componentname}/index.js`;
fileSave(path.join(__dirname, '../../components.json'))
  .write(JSON.stringify(componentsFile, null, '  '), 'utf8')
  .end('\n');



...



// 生成文件
Files.forEach(file => {
  fileSave(path.join(PackagePath, file.filename))
    .write(file.content, 'utf8')
    .end('\n');
});



...




console.log('DONE!');
```
<a name="W2Fki"></a>
### 总结

- 以上省略了一些步骤，基本类似，都是创建文件或者写入一些声明信息
- 核心是fileSave的使用，思路比较清晰简单
- 刚好最近在做个后台demo，所以调到这一期，顺手撸了一个，真香
