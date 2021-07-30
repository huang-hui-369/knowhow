

## create Google タグ マネージャー

1. access Google タグ マネージャー
![](img\2021-06-08-13-50-04.png)

2. create Google タグ マネージャー account
![](img\2021-06-08-13-51-19.png)
3. create containner
![](img\2021-06-08-13-52-11.png)
4. 同意规约
![](img\2021-06-08-13-53-46.png)
5. install google tag manager to html 
![](img\2021-06-08-13-54-46.png)


```
<head>

  <!-- Google Tag Manager -->
  <script>(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
  new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
  j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
  'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
  })(window,document,'script','dataLayer','GTM-53X882M');</script>
   <!-- End Google Tag Manager -->

   。。。

</head>
```

```
<body>
  <!-- Google Tag Manager (noscript) -->
  <noscript><iframe src="https://www.googletagmanager.com/ns.html?id=GTM-53X882M"
  height="0" width="0" style="display:none;visibility:hidden"></iframe></noscript>
  <!-- End Google Tag Manager (noscript) -->

  。。。

</body>  
```


6. containner dashboard画面
![](img\2021-06-08-14-00-10.png)

**GTM-XXXX 是containner ID**

### create tag

![](img\2021-06-08-14-09-21.png)

![](img\2021-06-08-14-10-04.png)

![](img\2021-06-08-14-17-49.png)