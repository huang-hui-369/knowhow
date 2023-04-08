- # 通过显式intent来迁移

  只要给intent传递两个参数，1： 当前Activity的context  2： 要迁移的Activity的class名。

  ```kotlin
   val intent: Intent = Intent(this@MainActivity, Activity2::class.java);
   startActivity(intent);
  ```

  

- # 通过隐式intent来迁移

  并不明确的指出要迁移到那个Activity，而是创建一个定义了action和category的Intent，交由系统分析Intent应该启动那个合适的Activity（有可能有多个合适的，会弹出菜单让用户选择）

  假如要迁移到Activity2

  1. 在androidManefest.xml的.Activity2中定义IntentFilter，指定action和category

     ```xml
      <activity android:name=".Activity2">
          <intent-filter>
              <action android:name="com.example.activity1.ACTION_START_ACTIVITY" />
              <category android:name="android.intent.category.DEFAULT" />
          </intent-filter>
     </activity>
     ```

     

  2. 在MainActivity中创建Intent并指定action和category （android.intent.category.DEFAULT是默认的可以省略）

     ```kotlin
     val intent: Intent = Intent("com.example.activity1.ACTION_START_ACTIVITY");
     startActivity(intent);
     ```

     

  