#### 文字超出部分显示省略号

###

- 单行文本的溢出显示省略号

```css
p {
  width: 200px; // 宽度是必须的
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
```

- 多行文本溢出显示省略号

```css
p {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 3;
  overflow: hidden;
}
```
