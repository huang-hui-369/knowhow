レスポンシブ

# 什么是响应式布局

响应式布局就是一个网站能够兼容多个终端，可以根据屏幕的大小自动调整页面的的展示方式以及布局，我们不用为每一个终端做一个特定的版本。响应式网站的几个标志：

1. 同时通过css来适配PC + 平板 + 手机等；
2. 标签导航在接近手持终端设备时改变为经典的抽屉式导航；
3. 网站的布局会根据视口来调整模块的大小和位置；

# 怎么实现响应式布局

1.设置meta标签

```xml
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
```

2.使用[@Media查询](https://link.segmentfault.com/?enc=%2B0VtiqoHyBMKBrCdCdyuUQ%3D%3D.CxD839ANSCEcEaw2PYBifMZvTm3nSRkDiik3CU9LWppGiouS%2BhJapXlo56GuofjwFu3xJcgwDYwpqsdOVFA3kQ%3D%3D)来设置样式，这是响应式布局的核心 600px`,`900px`,``,`

```css
@media screen and (max-width: 1920px) { ... }
```

3.设置布局分界点，即通过设置多种视图宽度样式来控制页面布局

- 600px以下为手机屏幕
- 600px -- 1024px 为平板屏幕
- 1024px屏幕以上为pc屏幕

```css
@media screen and (max-width: 1024px) { ... }

@media screen and (max-width: 600px) { ... }
```

其实就是通过不同的布局分界点（也就是屏幕大小）来适配不同的布局样式。

4.举例通过css来适配

例如 pc上一行显示4列，平板上一行显示2列，手机上一行显示1列。(实际上是通过百分比来实现)

```css
/* pc width > 1024px */
    body {
        background-color: yellow;
    }
    .col { /*1列显示4分之一*/
        width: 25%
    }
/* tablet 600px - 1024px */
@media screen and (max-width: 1024px) {
    body {
        background-color: #FF00FF;
    }
    .col { /*1列显示2分之一*/
        width: 50%
    }
}
/* mobile < 600px */
@media screen and (max-width: 600px) {
    body {
        background-color: green;
    }
    .col { /*1列显示100%*/
        width: 100%
    }
}
```



