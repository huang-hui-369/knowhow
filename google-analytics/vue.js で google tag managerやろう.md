# vue.js で google tag managerを利用する


### 1. vue-gtmを導入
vue2 で使う場合は必ずバージョンを指定。
でないと、vue3 用のが入ってしまいバグる

```
npm install vue-gtm@3.0.0-vue2
```

### 2. main.jsにvue-gtmを導入

```js
main.js

import router from './router';

import gtm from 'vue-gtm';

...

Vue.use(gtm, {
    id: 'GTM-XXX',//your GTM ID

    // 以下はOption
    queryParams: {
      // Add url query string when load gtm.js with GTM ID (optional)
      gtm_auth: "AB7cDEf3GHIjkl-MnOP8qr",
      gtm_preview: "env-4",
      gtm_cookies_win: "x",
    },
    defer: false, // Script can be set to `defer` to speed up page load at the cost of less accurate results (in case visitor leaves before script is loaded, which is unlikely but possible). Defaults to false, so the script is loaded `async` by default
    compatibility: false, // Will add `async` and `defer` to the script tag to not block requests for old browsers that do not support `async`
    nonce: "2726c7f26c", // Will add `nonce` to the script tag
    enabled: true, // defaults to true. Plugin can be disabled by setting this to false for Ex: enabled: !!GDPR_Cookie (optional)
    debug: true, // Whether or not display console logs debugs (optional)
    loadScript: true, // Whether or not to load the GTM Script (Helpful if you are including GTM manually, but need the dataLayer functionality in your components) (optional)
    vueRouter: router, // Pass the router instance to automatically sync with router (optional)
    ignoredViews: ["homepage"], // Don't trigger events for specified router names (case insensitive) (optional)
    trackOnNextTick: false, // Whether or not call trackView in Vue.nextTick
  })

  ...

});
```

3. user gtm instance in vue

```js
export default {
  name: "MyComponent",
  data() {
    return {
      someData: false,
    };
  },
  methods: {
    onClick() {
      this.$gtm.trackEvent({
        event: null, // Event type [default = 'interaction'] (Optional)
        category: "Calculator",
        action: "click",
        label: "Home page SIP calculator",
        value: 5000,
        noninteraction: false, // Optional
      });
    },
  },
  mounted() {
    this.$gtm.trackView("MyScreenName", "currentPath");
  },
};
```

### 3. send completely custom data to GTM with GTM data layer 

```
if (this.$gtm.enabled()) {
  window.dataLayer?.push({
    event: "myEvent",
    // further parameters
    target: category,
    action: action,
    "target-properties": label,
    value: value,
    "interaction-type": noninteraction,
    ...rest,
    
  });
}
```

### 4. use meta.gtm replace your view name

* If you defined a meta field for your route named gtm this will take the value of this field for the view name.
* Otherwise, if the plugin don't have a value for the meta.gtm it will fallback to the internal route name.

```js
const myRoute = {
  path: "myRoute",
  name: "MyRouteName",
  component: SomeComponent,
  meta: { gtm: "MyCustomValue" },
};
```

### 5. use gtm inside your setup method

```js
<template>
  <button @click="triggerEvent">Trigger event!</button>
</template>

<script>
import { useGtm } from "vue-gtm";

export default {
  name: "MyCustomComponent",

  setup() {
    const gtm = useGtm();

    function triggerEvent() {
      gtm.trackEvent({
        event: "event name",
        category: "category",
        action: "click",
        label: "My custom component trigger",
        value: 5000,
        noninteraction: false,
      });
    }

    return {
      triggerEvent,
    };
  },
};
</script>
```

## Vue.jsからGTMを使ってGAのカスタマイズイベントを送る

GTMのタグはindex.htmlに公式通りに入れているため、今回はスルーした。

### 1. dataLayerを使う

GTMから独自変数を送るにはdataLayerというグローバル変数を使うらしい。

```js
dataLayer.push({'event': 'event_name'});

dataLayer.push({
  'color': 'red',
  'conversionValue': 50,
  'event': 'customizeCar'
});
```

### 2. シンプルなVueプラグインを作る

```
gtm-my-error-event-handle.js

const gtmMyErrorEventHandle = {
  install: function (Vue, options) {
    const dataLayer = window.dataLayer || []
    Vue.prototype.$gtm = {
      sendErrorEvent: function(category, action, label){
        dataLayer.push({
          event: 'error',
          eventCategory: category || 'defaultCategory',
          eventAction: action || 'defaultAction',
          eventLabel: label || 'defaultLabel'
        })
      }
    }
  }
}
export default gtmMyErrorEventHandle
```

基本的にGTMのタグはHTMLの上の方で先に読み込んでいるはずで、dataLayerがセットされていないことはないはずだけど念の為、dataLayerがなくてもエラーが起きない（何も起きない）ようにしている。クリティカルな場面では使わないはず。

main.jsでインストールする

```js
import GtmDataLayer from './plugins/gtm-datalayer'
Vue.use(GtmDataLayer)
```

これでvueのscript内から

```js
this.$gtm.sendErrorEvent('お好きなカテゴリ', 'お好きなアクション', 'お好きなラベル')
```
みたいな感じで使える。

### 3. GTMで変数追加

勝手に送った変数はそのままでは使えない

ユーザー定義変数に上記、eventCategory、eventAction、eventLabelを追加する

![](img\2021-06-09-15-18-25.png)

### 4. GTMでカスタムイベント追加

カスタムイベントを受け取るトリガーを追加する。

![](img\2021-06-09-15-19-22.png)

とりあえず「すべてのカスタムイベント」にしたけど、あとで分けるなら、追加した変数を指定すればよさそう。

### 5. GTMでGAイベントのタグを追加

上で作った変数とトリガーを使って作る。

![](img\2021-06-09-15-23-16.png)

あとは公開するだけ。
