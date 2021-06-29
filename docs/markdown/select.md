#### 选取第 n 个标签元素

###

- `first-child` 表示选择列表中的第一个标签
- `last-child` 表示选择列表中的最后一个标签
- `nth-child(3)` 表示选择列表中的第 3 个标签
- `nth-child(2n)` 这个表示选择列表中的偶数标签
- `nth-child(2n-1)` 这个表示选择列表中的奇数标签
- `nth-child(n+3) ` 这个表示选择列表中的标签从第 3 个开始到最后
- `nth-child(-n+3)` 这个表示选择列表中的标签从 0 到 3，即小于 3 的标签
- `nth-last-child(3) ` 这个表示选择列表中的倒数第 3 个标签

```css
li:first-child {
}
```
