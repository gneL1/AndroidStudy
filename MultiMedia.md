# 多媒体
## 一、通知  
&emsp;&emsp;每条通知都属于一个对应的渠道，也就是通知渠道，通知渠道创建后不能再修改。  
&emsp;&emsp;通知渠道一旦创建后就不能再修改了。  

### 1. 通知的基本用法
1. 首先需要一个```NotificationManager```对通知进行管理，通过```Context```的```getSystemService()```方法获取。  
```getSystemService(@RecentlyNonNull String name)```接收一个字符串用于确定获取系统的哪个服务。  
```kotlin
val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
```

2. 使用```NotificationChannel```类构建一个通知渠道,并调用```NotificationManager.createNotificationChannel()```方法完成创建。  
&emsp;&emsp;  
由于```NotificationChannel```类和```createNotificationChannel()```方法都是Android 8.0系统新增的API,所以要进行版本判断。  
```Build.VERSION_CODES.O = 26``` ，即Android API = 26 对应Android 8.0(Oreo)  
```Build.VERSION.SDK_INT```是系统当前的版本号  
&emsp;&emsp;  
```NotificationChannel(String id, CharSequence name, int importance)```  
```id```：渠道id,保证全局唯一  
```name```：渠道名称，给用户看的  
```importance```：重要等级，通过```NotificationManager```来调用级别程度  
```kotlin
if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){
    val channel = NotificationChannel("normal","Normal",NotificationManager.IMPORTANCE_DEFAULT)
    manager.createNotificationChannel(channel)
}
```

3. 使用```NotificationCompat```类的```Builder```构造器构建```Notification```对象。  
&emsp;&emsp;  
```Builder(@NonNull Context context, @NonNull String channelId)```  
第二个参数```channelId```,需要和在创建通知渠道时指定的渠道ID相匹配。  
```setSmallIcon()```方法用于设置通知的小图标，只能用纯alpha图层的图片进行设置，小图标会显示在系统状态栏上。  
最后调用```build()```方法完成构建。  
&emsp;&emsp;  
```NotificationManager```的```notify()```方法让通知显示出来。  
```notify(int id, Notification notification)```  
第一个参数id要保证为每个通知指定的id都是不同的  
第二个参数是```Notification```对象  
```kotlin
val notification = NotificationCompat.Builder(this,"normal")
    .setContentTitle("这是标题")
    .setContentText("这是文本")
    .setSmallIcon(R.drawable.ic_launcher_foreground)
    .setLargeIcon(BitmapFactory.decodeResource(resources,R.drawable.ic_launcher_background))
    .build()
manager.notify(1,notification)
```

***

### 2. 创建一个通知
1. 修改```activity_notifucation_test.xml```布局文件  
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".NotifucationTest">

    <Button
        android:id="@+id/Btn_sendNotice"
        android:text="发送通知"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

</LinearLayout>
```
2. 修改```NotifucationTest```  
&emsp;&emsp;创建通知的代码只在第一次执行的时候才会创建，当下次再执行创建代码时，系统会检测到通知渠道已存在，不会重复创建。  
```kotlin
class NotifucationTest : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_notifucation_test)
 
        val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        
        if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){
            val channel = NotificationChannel("normal","Normal",NotificationManager.IMPORTANCE_DEFAULT)
            manager.createNotificationChannel(channel)
        }

        Btn_sendNotice.setOnClickListener {
            val notification = NotificationCompat.Builder(this,"normal")
                .setContentTitle("这是标题")
                .setContentText("这是文本")
                .setSmallIcon(R.drawable.ic_launcher_foreground)
                .setLargeIcon(BitmapFactory.decodeResource(resources,R.drawable.ic_launcher_background))
                .build()
            manager.notify(1,notification)
        }
    }
}
```


