# 追加onClickListenner

## 方法1 通过设计tool加

在onClick中指定函数名

![image-20220627175311056](D:\github\knowhow\android_studio\android_native\activity\002button.assets\image-20220627175311056.png)

在activity中定义函数

```kotlin
fun btnToastClick(view: View) {
        // Do something in response to button
        Log.d(TAG, "btnToastClick: ");
        Toast.makeText(this@MainActivity, "click toast button", Toast.LENGTH_SHORT).show();
    }
```

## 方法2通过button的setOnClickListener方法

1. 在onCreate方法中通过findViewById<Button>(R.id.button1);先得到button
2. 在调用button的方法setOnClickListener

```kotlin
class MainActivity : AppCompatActivity() {
    private val TAG = "MainActivity";
    override fun onCreate(savedInstanceState: Bundle?) {
        Log.d(TAG, "onCreate: ");
        super.onCreate(savedInstanceState)
        // set activity1
        setContentView(R.layout.activity1);
        // find toast button
        val btn1: Button = findViewById<Button>(R.id.button1);
        // setOnClickListener
        btn1.setOnClickListener {
                    // Code here executes on main thread after user presses button
                    Log.d(TAG, "toast button onClickListerner: ");
                    Toast.makeText(this@MainActivity, "click toast button", Toast.LENGTH_SHORT).show();
        }

    }

    fun btnToastClick(view: View) {
        // Do something in response to button
        Log.d(TAG, "btnToastClick: ");
        Toast.makeText(this@MainActivity, "click toast button", Toast.LENGTH_SHORT).show();
    }
}
```

