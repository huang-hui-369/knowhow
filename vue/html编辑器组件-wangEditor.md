# 使用基本流程

## 需要的组件

- wangEditor html编辑器
- i18next    多语言

## npm install wangeditor 

```
npm i wangeditor --save
```

## 或者 引入js
https://cdn.jsdelivr.net/npm/wangeditor@latest/dist/wangEditor.min.js

## 创建一个div占位符

```html
  <div id="msgEditor" class="msgEditor">
  </div>
```

## 引入使用wangEditor

```js
import E from "wangeditor"
// 创建wangeditor实例并且和div占位符关联起来
const editor = new E("#msgEditor")
editor.create()
```

- 初始化时设置显示html内容
    ```js
    editor.txt.html('<p>用 JS 设置的内容</p>') // 重新设置编辑器内容
    ```
- 获取 html
    ```js
   editor.txt.html() // 获取 html 
    ```
- 追加新内容
    ```js
    editor.txt.append('<p>用 JS 设置的内容</p>') // 追加新内容
    ```
- 获取 text
    ```js
    editor.txt.text() // 获取 text 文本
    ```
- 清空内容
    ```js
    editor.txt.clear()
    ```

## 一个完整的vue例子
```js
<template>
<div class="text-left cont">
    <div class="aq-redColor aq-margin-bottom-10"><span class="aq-margin-right-4 font-weight-600">*</span>必須項目</div>
    <el-form
    ref="form"
    :model="form"
    :rules="rules"
    label-position="top"
    autocomplete="off"
    @submit.native.prevent
    >
    <el-form-item label="通知テキスト" class="item" prop="htmlText">
        <!-- wangEditor的占位符 -->
        <div id="htmlEditor" class="htmlEditor">
        </div>
    </el-form-item>
    </el-form>
</div>
</template>

<script>
import wEditor from "wangeditor";
import i18next from "i18next";

export default {
  name: "demo",
  data() {
    return {

      form: {
        // 和wangEditor绑定的变量存放用户输入html字符串
        htmlText: "",
      },
      // 利用element ui的校验功能
      rules: {
        htmlText: [
          {
            required: true,
            trigger: "blur",
            validator: isValidateNull,
          },
        ],
        // 创建的wangeditor实例变量
        editor : null,
      },
    };
  },

  mounted() {
    // 创建wangeditor实例并且和div占位符关联起来
    const editor = new wEditor('#htmlEditor');
    // 设置editor高度
    editor.config.height = 300;
    // zIndex
    editor.config.zIndex = 1;
    // 设置editor menu
    editor.config.menus = [
      "link",
    ];
    // showFullScreen
    editor.config.showFullScreen = false;
    editor.config.pasteFilterStyle = true;
    editor.config.pasteIgnoreImg = true;
    
    // 设置tooltip
    editor.config.menuTooltipPosition = 'upright'

    // 设置为日语
    editor.config.lang = "japan";
    // 设置替换的menu，tooltip，错误消息的文字
    editor.config.languages["japan"] = {
      wangEditor: {
        重置: 'リセット',
        插入: '挿入',
        默认: 'デフォルト',
        创建: 'クリエイト',
        修改: 'エディット',
        如: 'リンク',
        请输入正文: '本文を入力してください',
        menus: {
            title: {
                链接: 'リンクの挿入' 
            },
            dropListMenu: {
                设置标题: 'タイトルの設定',
                背景颜色: '背景色',
                文字颜色: 'テキストの色',
                设置字号: 'サイズを設定',
                设置字体: 'フォントの設定',
                设置缩进: 'インデント',
                对齐方式: 'アルギン',
                设置行高: '行の高さを設定',
                序列: 'シーケンス',
                head: {
                    正文: '本文',
                },
                indent: {
                    增加缩进: 'インデントを増やす',
                    减少缩进: 'インデントを減らす',
                },
                justify: {
                    靠左: '左に寄る',
                    居中: '中央にある',
                    靠右: '右に寄る',
                },
                list: {
                    无序列表: '無秩序リスト',
                    有序列表: '順序リスト',
                },
            },
            panelMenus: {
                emoticon: {
                    默认: 'デフォルト',
                    新浪: '新浪',
                    emoji: 'emoji',
                    手势: '手振り',
                },
                image: {
                    图片链接: '写真のリンク',
                    上传图片: '画像をアップロード',
                    网络图片: 'ネットワーク画像',
                },
                link: {
                    链接: 'リンク',
                    链接文字: 'リンクテキスト',
                    删除链接: 'リンクを削除',
                    查看链接: 'リンクを表示',
                    取消链接: 'リンクの解除'
                },
                video: {
                    插入视频: 'ビデオを挿入',
                },
                table: {
                    行: '行',
                    列: '列',
                    的: 'の',
                    表格: 'テーブル',
                    添加行: '行を追加',
                    删除行: '行を削除',
                    添加列: '列を追加',
                    删除列: '列を削除',
                    设置表头: 'テーブルの設定',
                    取消表头: 'ヘッダをキャンセル',
                    插入表格: '表の挿入',
                    删除表格: '表を削除',
                },
                code: {
                    删除代码: 'コードを削除',
                    修改代码: 'コードの変更',
                    插入代码: 'コードを挿入',
                },
            },
        },
        validate: {
            张图片: '写真',
            大于: '大きい',
            图片链接: '写真のリンク',
            不是图片: '画像ではありません',
            返回结果: '結果を返します',
            上传图片超时: '画像のアップロードがタイムアウトしました',
            上传图片错误: '画像のアップロードエラー',
            上传图片失败: '画像のアップロードに失敗しました',
            插入图片错误: '画像の挿入エラー',
            一次最多上传: '一回で最大アップロード',
            下载链接失败: 'リンクのダウンロードに失敗しました',
            图片验证未通过: '画像の検証が失敗しました。',
            服务器返回状态: 'サーバの戻り状態',
            上传图片返回结果错误: '画像のアップロード結果エラー',
            请替换为支持的图片类型: 'サポートされている画像の種類に置き換えてください。',
            您插入的网络图片无法识别: 'あなたが挿入したネットワーク画像は識別できませんでした。',
            您刚才插入的图片链接未通过编辑器校验: '先ほど挿入した写真のリンクはエディタで検証されませんでした。',
        },
    };
    
    // 设置i18
    editor.i18next = i18next;
    editor.config.onchangeTimeout = 0;
    editor.config.focus = false;

    
    editor.create();

    this.editor = editor;
  },

  created() {
    this.init();
  },

  // 在画面销毁前销毁wangEditor
  beforeDestroy() {
    this.editor.destroy()
    this.editor = null
  },

  methods: {
    // 初期化
    init() {
      console.log(this.form.info);
        if (this.adminData.adminMsgData) {
          this.form = this.adminData.adminMsgData;
          // 一定要在$nextTick中设置初期显示html内容
          this.$nextTick(() => {
            this.editor.txt.html(this.form.info);
          });
        }
    },

    // form 提交
    submit() {
      // 获取 html文字列 
      this.form.info = this.editor.txt.html();
      this.$store.commit("adminData/SET_ADMIN_MSG_DATA", this.form);
      // from 校验
      this.$refs["form"].validate((valid) => {
        if (valid) {
          // 向服务器提交
        } else {
          alertMessage("入力漏れの項目があります", "ERROR");
        }
      });
    },
  },
};
</script>

<style lang="scss" scoped>
  /* 设置editor的css样式，如果是scoped 必须要用 ::v-deep来定位，否则不起作用 */
  ::v-deep .w-e-toolbar {
  border: 1px solid #333 !important;
  border-bottom: 1px solid #eee !important;
  }

  ::v-deep .w-e-text-container {
    min-height: 200px;
    border: 1px solid #333 !important;
    border-top: 0 !important;
  }

   /* 设置editor下p tag的css样式 */
  ::v-deep .w-e-text p {
    margin: 0 0;
  }

  /* 设置editor下a tag的css样式 */
  ::v-deep .w-e-text a {
    text-decoration: underline;
    color: blue;
  }
  
}
</style>
```