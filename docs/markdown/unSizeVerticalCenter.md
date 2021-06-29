#### 不固定高宽 div 垂直居中的方法

###

- 伪元素和 inline-block / vertical-align（兼容 IE8）

```css
.box-wrap:before {
  content: "";
  display: inline-block;
  height: 100%;
  vertical-align: middle;
  margin-right: -0.25em; //微调整空格
}
.box {
  display: inline-block;
  vertical-align: middle;
}
```

- flex(不兼容 ie8 以下)

```css
.box-wrap {
  height: 300px;
  justify-content: center;
  align-items: center;
  display: flex;
  background-color: #666;
}
```

- transform(不兼容 ie8 以下)

```css
.box-wrap {
  width: 100%;
  height: 300px;
  background: rgba(0, 0, 0, 0.7);
  position: relative;
}
.box {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translateX(-50%) translateY(-50%);
  -webkit-transform: translateX(-50%) translateY(-50%);
}
```

- 设置 margin:auto（非严格意义上的非固定宽高，而是 50%的父级的宽高）

```css
.box-wrap:before {
  content: "";
  display: inline-block;
  height: 100%;
  vertical-align: middle;
  margin-right: -0.25em; //微调整空格
}
.box {
  display: inline-block;
  vertical-align: middle;
}
```
