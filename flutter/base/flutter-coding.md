# プロジェクト構造
```
msp
 |－　imgs  画像フォルダ
 |－　icons
 |－　jsons　静データー
 |－　lib　
　　   |－　common    一些工具类，如通用方法类、网络接口类、保存全局变量的静态类等
　　   |－　models    Json文件对应的Dart Model类会在此目录下
　　   |－　states    保存APP中需要跨组件共享的状态类
　　   |－　pages     存放所有路由页面类
　　   |－　widgets   APP内封装的一些Widget组件都在该目录下
```

# 画面

##　共同画面组件
### 构造
1. top appbar 可以没有
2. center     中间页面      必须有
3. bootom appbar(tab bar)  共同

### tab click 事件
1. 先清空Navgator，再push


## ホーム画面
1. トップappbar　なし
2. bottom　appbar
   
   2.1 tab　button

    - ホーム
    - お気に入り
    - 位置情報
    - マイページ
    - 設定
  


# 技術点

## api呼び
## JsonからDart Model転化
## status管理
## 情報保存
## 
　　
