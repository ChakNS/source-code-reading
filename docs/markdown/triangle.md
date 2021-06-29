#### 绘制三角形

###

```css
div {
  width: 0;
  height: 0;
  border-width: 0 40px 40px;
  border-style: solid;
  border-color: transparent transparent red;
}
```

#### 绘制带边框的三角形

####

```css
#blue {
  position: relative;
  width: 0;
  height: 0;
  border-width: 0 40px 40px;
  border-style: solid;
  border-color: transparent transparent blue;
}
#blue:after {
  content: "";
  position: absolute;
  top: 1px;
  left: -38px;
  border-width: 0 38px 38px;
  border-style: solid;
  border-color: transparent transparent yellow;
}
```

- 如果想绘制右直角三角，则将左 border 设置为 0；如果想绘制左直角三角，将右 border 设置为 0 即可（其它情况同理）
