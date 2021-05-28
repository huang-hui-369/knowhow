
### 1) 获得字符串的长度
语法：

${#var}

### 从指定位置开始截取

```
${string: start :length}
```

* start :   是起始位置（从左边开始，从 0 开始计数）
* length :  是要截取的长度（省略的话表示直到字符串的末尾）

```
url="c.biancheng.net"
echo ${url: 2: 9}
biancheng
```

### 使用 # 号截取右边字符

```
${string#*chars}
```


```
url="http://c.biancheng.net/index.html"
echo ${url#*:}
//c.biancheng.net/index.html
```

```
url="http://c.biancheng.net/index.html"
echo ${url#http:}
//c.biancheng.net/index.html
```

## 使用 % 截取左边字符
使用%号可以截取指定字符（或者子字符串）左边的所有字符，具体格式如下：

```
${string%chars*}

```
请注意*的位置，因为要截取 chars 左边的字符，而忽略 chars 右边的字符，所以*应该位于 chars 的右侧。其他方面%和#的用法相同，这里不再赘述，仅举例说明：


```
#!/bin/bash
url="http://c.biancheng.net/index.html"
echo ${url%/*}  #结果为 http://c.biancheng.net
echo ${url%%/*}  #结果为 http:
```



| 格式                       | 说明                                                                         |
| -------------------------- | ---------------------------------------------------------------------------- |
| ${string: start :length}   | 从 string 字符串的左边第 start 个字符开始，向右截取 length 个字符。          |
| ${string: start}           | 从 string 字符串的左边第 start 个字符开始截取，直到最后。                    |
| ${string: 0-start :length} | 从 string 字符串的右边第 start 个字符开始，向右截取 length 个字符。          |
| ${string: 0-start}         | 从 string 字符串的右边第 start 个字符开始截取，直到最后。                    |
| ${string#*chars}           | 从 string 字符串第一次出现 *chars 的位置开始，截取 *chars 右边的所有字符。   |
| ${string##*chars}          | 从 string 字符串最后一次出现 *chars 的位置开始，截取 *chars 右边的所有字符。 |
| ${string%*chars}           | 从 string 字符串第一次出现 *chars 的位置开始，截取 *chars 左边的所有字符。   |
| ${string%%*chars}          | 从 string 字符串最后一次出现 *chars 的位置开始，截取 *chars 左边的所有字符。 |