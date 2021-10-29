# aes加密过程

要理解AES的加密流程, 会涉及到AES的五个关键词: 分组密码体制, Padding, 初始向量IV, 密钥, 加密模式.

- 分组密码体制: 所谓分组密码体制就是指将明文切成一段一段的来加密, 然后再把一段一段的密文拼起来形成最终密文的加密方
式. AES采用分组密码体制, 即AES加密会首先把明文切成一段一段的, 而且每段数据的长度要求必须是128位16个字节, 如果最后
一段不够16个字节了, 就需要用Padding来把这段数据填满16个字节, 然后分别对每段数据进行加密, 最后再把每段加密数据拼起
来形成最终的密文.

- Padding: Padding就是用来把不满16个字节的分组数据填满16个字节用的, 它有三种模式PKCS5、PKCS7和NOPADDING. PKCS5是
指分组数据缺少几个字节, 就在数据的末尾填充几个字节的几, 比如缺少5个字节, 就在末尾填充5个字节的5. PKCS7是指分组数据
缺少几个字节, 就在数据的末尾填充几个字节的0, 比如缺少7个字节, 就在末尾填充7个字节的0. NoPadding是指不需要填充, 也就
是说数据的发送方肯定会保证最后一段数据也正好是16个字节. 那如果在PKCS5模式下, 最后一段数据的内容刚好就是16个16怎么办？
那解密端就不知道这一段数据到底是有效数据还是填充数据了, 因此对于这种情况, PKCS5模式会自动帮我们在最后一段数据后再添
加16个字节的数据, 而且填充数据也是16个16, 这样解密段就能知道谁是有效数据谁是填充数据了. PKCS7最后一段数据的内容是
16个0, 也是同样的道理. 解密端需要使用和加密端同样的Padding模式, 才能准确的识别有效数据和填充数据. 我们开发通常采用
PKCS7 Padding模式.

- 初始向量IV: 初始向量IV的作用是使加密更加安全可靠, 我们使用AES加密时需要主动提供初始向量, 而且只需要提供一个初始向
量就够了, 后面每段数据的加密向量都是前面一段的密文. 初始向量IV的长度规定为128位16个字节, 初始向量的来源为随机生成.

- 密钥: AES要求密钥的长度可以是128位16个字节、192位或者256位, 位数越高, 加密强度自然越大, 但是加密的效率自然会低一
些, 因此要做好衡量. 我们开发通常采用128位16个字节的密钥, 我们使用AES加密时需要主动提供密钥, 而且只需要提供一个密钥
就够了, 每段数据加密使用的都是这一个密钥, 密钥来源为随机生成.

- 加密模式: AES一共有四种加密模式, 分别是ECB(电子密码本模式)、CBC(密码分组链接模式)、CFB、OFB, 我们一般使用的是CBC模
式. 四种模式中除了ECB相对不安全之外, 其它三种模式的区别并没有那么大. ECB模式是最基本的加密模式, 即仅仅使用明文和密钥
来加密数据, 相同的明文块会被加密成相同的密文块, 这样明文和密文的结构将是完全一样的, 就会更容易被破解, 相对来说不是那么
安全, 因此很少使用. CBC模式则比ECB模式多了一个初始向量IV, 加密的时候, 第一个明文块会首先和初始向量IV做异或操作, 然后
再经过密钥加密, 然后第一个密文块又会作为第二个明文块的加密向量来异或, 依次类推下去, 这样相同的明文块加密出的密文块就是
不同的, 明文的结构和密文的结构也将是不同的, 因此更加安全, 我们常用的就是CBC加密模式.

# java aes加密

## 问题

使用 AES-128 加密没有任何问题, 但当你把密钥增加到32个字节（AES-256 ）的时候, 可能会遇到如下异常:
```
java.security.InvalidKeyException: Illegal key size
```

## 解决方法
- 1.8.0_151 以前
  To solve that you have to go to [this website](https://www.oracle.com/java/technologies/javase-jce8-downloads.html), download the Unlimited Strength Jurisdiction Policy Files, unzip it, go to the <java-home>/lib/security directory,and replace the two files local_policy.jar and US_export_policy.jar with the two files from the download. 
- 1.8.0_151，152 
  Starting with Java 1.8.0_151 and 1.8.0_152 there is a new somewhat easier way to enable the unlimited strength jurisdiction policy for the JVM. Without enabling this you cannot use AES-256. Since this version, it is no longer necessary to download the policy files from the Oracle website and install it. You can now set the unlimited policy directly in your application with this one-liner:
  ```
  Security.setProperty("crypto.policy", "unlimited");
  ```
- 1.8.0_162 以后
  In Java 1.8.0_162, the unlimited policy is enabled by default. You no longer need to install the policy file in the JRE or set the security property crypto.policy.
  openjdk bugs: Enable unlimited cryptographic policy by default in Oracle JDK builds

# java aes加密字符串


# java aes加密文件流