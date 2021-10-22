# SSMA4OracleSQLServerCollections,SSMA4OracleSQLServerExtensions手動で導入

1. SSMA tooのdataBaseのAssembilesに上記のAssembilesがあります。
SSMA4OracleSQLServerCollectionsとSSMA4OracleSQLServerExtensionsにはいろいろOracle拡張関すんがあります。
![](img\2021-06-17-12-18-23.png)

2. sql serverの中に上記のAssembilesがありませんので、同期前に導入しないといけない。
![](img\2021-06-17-12-22-35.png)

3. 導入する下記エラーが出た来ます。
   
* 現象
 ```
 「ｘｘｘ」のアセンブリを作成できませんでした。 sp_configure の ‘clr strict security’ オプションが 1 に設定されているため、SAFE または EXTERNAL_ACCESS オプションを指定したアセンブリ ‘SqlClrPadding’ の CREATE または ALTER ASSEMBLY が失敗しました。UNSAFE ASSEMBLY アクセス許可を持つ対応するログインを使用した証明書または非対称キーでアセンブリに署名することをお勧めします。または、sp_add_trusted_assembly を使用してアセンブリを信頼することができます。
```
   
* 原因
  
```
このDLLはサインしてないので、信頼できない。 

```

* 解決策

DATABASEのセキュリティーラベルを低くにする。

```
ALTER DATABASE SAITAMA SET TRUSTWORTHY ON;
```
