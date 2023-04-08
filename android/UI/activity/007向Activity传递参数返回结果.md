# 向Activity传递参数

通过Intent.putExtra方法传递参数给Activity

1. mainActivity put参数

   ```kotlin
    val intent: Intent = Intent(this@MainActivity, Activity2::class.java);
   intent.putExtra("param1","paramter from main activity");
   startActivity(intent);
   ```

   

2. activity2中取出参数

```kotlin
class Activity2 : AppCompatActivity() {

    lateinit var textview2: TextView;

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity2)
        // get parameter from prev activity
        val param1 = intent.getStringExtra("param1");
        // set param to textview2
        textview2 = findViewById<TextView>(R.id.textView2);
        textview2.setText(param1);
    }
}
```

# 返回结果给上一个Activity

1. 在mainActivity中调用startActivityForResult

   ```kotlin
    val intent: Intent = Intent(this@MainActivity, Activity2::class.java);
    intent.putExtra("param1","paramter from main activity");
    startActivityForResult(intent, 1);
   ```

   

2. 在Activity2中新建一个没有参数的空的Intent，调用putExtra设置返回值

3. 调用Activity2的setResult(Result_OK, intent)

   ```kotlin
   var intent = Intent();
   intent.putExtra("result", "result from activity2");
   setResult(Activity.RESULT_OK, intent);
   finish();
   ```

   

4. 在mainActivity中重载onActivityResult方法

   参数1 requestCode（有可能迁移到多个Activity中的一个，所以要设置requestCode来区分从哪个Activity返回）

   参数2 resultCode

   参数3 putExtra设置的Result data

   ```kotlin
   override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
   
           when(requestCode) {
               1 -> {
                   if(resultCode == Activity.RESULT_OK) {
                       val rst = data?.getStringExtra("result")
                       Toast.makeText(this@MainActivity, rst, Toast.LENGTH_SHORT).show()
                   }
                   Log.d(TAG, "onActivityResult: resultCode NG");
   
               }
               else -> Log.d(TAG, "onActivityResult: other requestCode");
           }
           super.onActivityResult(requestCode, resultCode, data);
       }
   ```

   



# 按下back键返回结果

在Activity2中重载onBackPressed方法

