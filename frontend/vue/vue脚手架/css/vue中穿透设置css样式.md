# Vue中的scoped和scoped穿透

## 什么是scoped
在Vue文件中的style标签上有一个特殊的属性，scoped。当一个style标签拥有scoped属性时候，它的css样式只能用于当前的Vue组件，可以使组件的样式不相互污染。如果一个项目的所有style标签都加上了scoped属性，相当于实现了样式的模块化。

## scoped的实现原理

Vue中的scoped属性的效果主要是通过PostCss实现的。

以下是转译前的代码:

```js
<style scoped lang="less">
    .example{
        color:red;
    }
</style>
<template>
    <div class="example">scoped测试案例</div>
</template>
```

转译后:

```js
.example[data-v-5558831a] {
  color: red;
}
<template>
    <div class="example" data-v-5558831a>scoped测试案例</div>
</template>

```

## scoped穿透
scoped有不污染其他组件的好处的一面，但是当在Vue项目中，当我们引入第三方组件库时(element)，需要在局部组件中修改第三方组件库的样式，而又不想去除scoped属性造成组件之间的样式覆盖。这时我们需要通过特殊的方式穿透scoped。

### style为css时的写法如下
需要>>>
```css
.a >>> .b {
    ***
}
```

### style使用css的预处理器为sass，scss的写法如下
需要加入::v-deep

```css
.a ::v-deep .b{
    ***
}

.login {
    ::v-deep .el-button {
      width: 100px;
      height: 30px;
      border: 0;
      padding: 0 0;
      color: $whiteColor;
      font-weight: 600;
      font-size: 14px;
      border-radius: 15px;
    }
    .new-login {
      background: $redBgColor;
    }
    .login,
    .logout {
      background: $blueBgColor;
    }
  
```

### style使用css的预处理器为less的写法如下
需要加入::v-deep

```css
 .a /deep/ .b{
    ***
}
```

# 在组件中修改第三方组件库样式的其它方法
- 在vue组件中不使用scoped属性
- 在vue组建中使用两个style标签，一个加上scoped属性，一个不加scoped属性，把需要覆盖的css样式写在不加scoped属性的style标签里
- 建立一个reset.css(基础全局样式)文件，里面写覆盖的css样式，在入口文件main.js 中引入