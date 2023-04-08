# add option menu

1. 在res目录下创建menu目录

2. 在menu目录下new Menu Resource File

   ![image-20220627181750261](D:\github\knowhow\android_studio\android_native\activity\003optionMenu.assets\image-20220627181750261.png)

3. 在activity中重写onCreateOptionsMenu方法

   调用activity的menuInflater.inflate(R.menu.acv1optmenu, menu);方法加载已经创建的OptionsMenu资源

   ```kotlin
   class MainActivity : AppCompatActivity() {
       private val TAG = "MainActivity";
       
       ...
   
       override fun onCreateOptionsMenu(menu: Menu?): Boolean {
           Log.d(TAG, "onCreateOptionsMenu: ");
           menuInflater.inflate(R.menu.acv1optmenu, menu);
           return super.onCreateOptionsMenu(menu);
       }
       ...
   }
   ```

4. 在activity中重写onOptionsItemSelected方法

   ```kotlin
       override fun onOptionsItemSelected(item: MenuItem): Boolean {
           when (item.itemId) {
               R.id.addItem -> {
                   Log.d(TAG, "add item option menu click: ");
                   Toast.makeText(this@MainActivity, "add item option menu click:", Toast.LENGTH_SHORT).show()
               }
               R.id.removeItem -> {
                   Log.d(TAG, "remove item option menu click: ");
                   Toast.makeText(this@MainActivity, "remove item option menu click:", Toast.LENGTH_SHORT).show()
               }
               else -> { // 那个menu也没选，实际上不可能
                   Log.d(TAG, "none item option menu click: ");
               }
           }
           return super.onOptionsItemSelected(item)
       }
   ```

5. 运行结果

   ![image-20220627182455508](D:\github\knowhow\android_studio\android_native\activity\003optionMenu.assets\image-20220627182455508.png)

   ![image-20220627182357139](D:\github\knowhow\android_studio\android_native\activity\003optionMenu.assets\image-20220627182357139.png)

   ![image-20220627182519547](D:\github\knowhow\android_studio\android_native\activity\003optionMenu.assets\image-20220627182519547.png)