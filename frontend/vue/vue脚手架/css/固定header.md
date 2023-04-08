# 固定header的方法
主要是设置`position: fixed;`


```css
.header {
  width: 100%;
  /* 将header固定起来，即使滚动滚动条头部也会固定显示*/
  position: fixed;
  top: 0;
  left: 0;
  /* 将header始终显示在最上面，*/
  z-index: 100;
}

.body {
    /* 因为header position为fixed，所以下面的body也将会从0，0坐标开始呈现
    这样body就会被挡住，一部分，所以需要设置margin-top，或者padding-top,
    高度为header的高度
     */
    margin-top: 106px;
}

```