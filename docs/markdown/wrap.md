#### 中英文自动换行

###

- `word-break:break-all;` 只对英文起作用，以字母作为换行依据
- `word-wrap:break-word;` 只对英文起作用，以单词作为换行依据
- `white-space:pre-wrap;` 只对中文起作用，强制换行
- `white-space:nowrap;` 强制不换行，都起作用

```css
p {
  word-wrap: break-word;
  white-space: normal;
  word-break: break-all;
}
```

```css
//不换行
.wrap {
  white-space: nowrap;
}
//自动换行
.wrap {
  word-wrap: break-word;
  word-break: normal;
}
//强制换行
.wrap {
  word-break: break-all;
}
```
