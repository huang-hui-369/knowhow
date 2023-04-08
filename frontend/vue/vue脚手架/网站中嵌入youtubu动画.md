# 嵌入方法

1. 打开yutube的动画网页，点击共有
   ![](img\2021-08-17-12-48-33.png)

2. 在共有dialog中点击嵌入
   ![](img\2021-08-17-12-50-28.png)

3. copy ifram的代码 到site代码中
   ![](img\2021-08-17-12-51-44.png)

    ![](img\2021-08-17-12-53-32.png)

    embed/{视频ID} 后的字符数字为视频ID

# responsive

主旨思想为在div中不设置宽度，默认就是页面的一行的宽度，通过padding-bottom设置高度为宽度的56.25%，然后div中加入iframe 它定位于绝对于div的坐标为0，0，设置宽度为div的宽度，

1. 在iframe外加入`<div style="position: relative; padding-bottom: 56.25%;">`

- 56.25% 为16：9的比例


2. 在iframe中加入 `style="position: absolute; top: 0; left: 0; width: 100%;`

完整代码
```js
<div style="position: relative; padding-bottom: 56.25%;">
<!-- width="1104" height="621"  -->
    <iframe 
        style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"
        src="https://www.youtube.com/embed/aNYWBvko" 
        title="YouTube video player" 
        frameborder="0" 
        allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
        allowfullscreen>
    </iframe>
</div>
```

# 播放参数


## 播放动画ID
在iframe中谈过设置src的URL来播放动画
URL： `https://www.youtube.com/embed/[YouTube動画ID]`

## 播放动画List
`https://www.youtube.com/embed?listType=playlist&list=[YouTube 再生リスト ID]`

## 播放用户上传的视频
`https://www.youtube.com/embed?listType=user_uploads&list=[YouTube ユーザー名]`

## 播放检索结果的视频
`https://www.youtube.com/embed?listType=search&list=[検索キーワード])`


## 播放完后是否推荐的相关视频
默认推荐相关视频

通过url后加入参数rel=0来设置不推荐相关视频

`src="https://www.youtube.com/embed/aNYWJvko>rel=0"> `

2018年9月25日以后，即使设置也已经不起作用了

## 是否显示字幕
### 设置默认语言
cc_lang_pref=ja

### 设置是否表示字幕
- cc_load_policy=1 表示字幕
- cc_load_policy=0 不表示字幕
## 是否自动播放
默认不自动播放

- autoplay=1 来设置自动播放
- autoplay=0 设置不自动播放

## 是否显示视频控制按钮面板
默认为显示

- controls=0 来设置不显示
- controls=1 来设置显示

## 是否设置为静音
默认为不静音

- mute=1 来设置静音
- mute=0 来设置不静音

## 设置视屏播放的开始，结束时间
默认播放整段视频

- start=30&end=60 来设置播放一段时段的视频

## 是否循环播放
默认不循环播放

- loop=1 设置为循环播放
- loop=0 设置为不循环播放

## 设置全屏显示按钮
默认显示

- fs=0 全屏显示按钮不显示
- fs=1 全屏显示按钮显示

## 禁止键盘操作
默认不禁止

- disablekb=0 不禁止键盘操作
- disablekb=1 禁止键盘操作

# javascript 控制播放器
https://developers.google.com/youtube/iframe_api_reference?hl=ja

