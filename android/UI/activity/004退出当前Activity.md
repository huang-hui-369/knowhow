- ## 自动销毁

  只要按下back键，系统会自动销毁

- ## 手动销毁

  调动Activity的finish()方法.

  ```kotlin
  val btn3: Button = findViewById<Button>(R.id.button3);
  // setOnClickListener
  btn3.setOnClickListener {
      // Code here executes on main thread after user presses button
      Log.d(TAG, "back button onClickListerner: ");
      Toast.makeText(this@MainActivity, "click back button", Toast.LENGTH_SHORT).show();
      finish(); // 销毁activity
  }
  ```

  

