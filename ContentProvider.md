# ContentProvider
&emsp;&emsp;```ContentProvider```主要用于在不同的应用程序之间实现数据共享的功能，允许一个程序访问另一个程序中的数据，同时保证被访问数据的安全性。  
&emsp;&emsp;```ContentProvider```可以选择只对哪一部分数据进行共享。

## 运行时权限
&emsp;&emsp;用户不需要在安装软件的时候一次性授权所有申请的权限，而是可以在软件的使用过程中再对某一项权限申请进行授权。  
&emsp;&emsp;比如一款相机应用在运行时申请了地理位置定位权限，用户拒绝了这个权限，也可以使用这个应用的其他功能。  
&emsp;&emsp;  
### 在程序运行时申请权限
1. 创建一个```PermissionTest```  
> **activity_permission_test.xml** 文件  
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".PermissionTest">

    <Button
        android:id="@+id/Btn_makeCall"
        android:text="打电话"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
    
</LinearLayout>
```
> **PermissionTest** 文件  
```kotlin
class PermissionTest : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_permission_test)

        Btn_makeCall.setOnClickListener {
            /**
             * 构建一个隐式的Intent,Intent的action指定为Intent.ACTION_CALL，这是一个内置的打电话的动作
             * Intent.ACTION_CALL表示直接拨打电话，必须声明权限
             * 而Intent.ACTION_DIAL表示打开拨号界面，是不需要声明权限的
             * 在data部分指定了协议是tel,号码是10086
             * 为了防止程序崩溃，需要将操作放在异常捕获代码块中
             */
            try {
                val intent = Intent(Intent.ACTION_CALL)
                intent.data = Uri.parse("tel:10086")
                startActivity(intent)
            }catch (e : SecurityException){
                e.printStackTrace()
            }
        }

    }
}
```

