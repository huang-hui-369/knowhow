http头信息

头信息的作用很多，最主要的有下面几个：
1、跳转
当浏览器接受到头信息中的 Location: xxxx 后，就会自动跳转到 xxxx 指向的URL地址，这点有点类似用 js 写跳转。但是这个跳转只有浏览器知道，不管体内容里有没有东西，用户都看不到。
例：header("Location: http://www.xker.com/");