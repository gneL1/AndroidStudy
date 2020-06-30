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
&emsp;&emsp;创建通知渠道的代码只在第一次执行的时候才会创建，当下次再执行创建代码时，系统会检测到通知渠道已存在，不会重复创建。  
```kotlin
class NotifucationTest : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_notifucation_test)
 
        val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        
        if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){
            //创建了一个id为normal的通知渠道
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
<img src="https://github.com/gneL1/AndroidStudy/blob/master/photos/MultiMedia/notification_1.png" width="400" height="680" align=center/>
&emsp;&emsp;  
在设置里也可以看到通知渠道：  
<img src="https://github.com/gneL1/AndroidStudy/blob/master/photos/MultiMedia/notification_2.png" width="400" height="680" align=center/>

***

### 3. 给通知增加点击效果
&emsp;&emsp;```PendingIntent```在某个合适的实际执行某个动作，可以理解为延迟执行的```Intent```，可以使用```getActivity()```、```getBroadcast()```、```getService()```来获取```PendingIntent```实例。  
&emsp;&emsp;比如```getActivity(Context context, int requestCode, Intent intent, int flags)```  
&emsp;&emsp;```requestCode```一般传入0  
&emsp;&emsp;```flags```用于确定```PendingIntent```的行为，有```FLAG_ONE_SHOT```、```FLAG_NO_CAMERA```、```FLAG_CANCEL_CURRENT```、```FLAG_UPDATE_CURRENT```4种值可选，一般传入0。  
&emsp;&emsp;通过```Builder```的```setContentIntent(PendingIntent intent)```方法，将```PendingIntent```传递进去来构建一个延迟执行的意图。  
1. 新建一个```NotificationActivity```，修改布局文件  
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".NotificationActivity">

    <TextView
        android:text="通知跳转到的界面"
        android:textSize="24sp"
        android:layout_centerInParent="true"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

</RelativeLayout>
```

2. 修改```NotifucationTest```  
```kotlin
......
Btn_sendNotice.setOnClickListener {
    val intent = Intent(this,NotificationActivity::class.java)
    val pi = PendingIntent.getActivity(this,0,intent,0)
    val notification = NotificationCompat.Builder(this,"normal")
        .setContentTitle("这是标题")
        .setContentText("这是文本")
        .setSmallIcon(R.drawable.ic_launcher_foreground)
        .setLargeIcon(BitmapFactory.decodeResource(resources,R.drawable.ic_launcher_background))
        .setContentIntent(pi)
        .build()
    manager.notify(1,notification)
}
```
<img src="https://github.com/gneL1/AndroidStudy/blob/master/photos/MultiMedia/notification_3.gif" width="400" height="680" align=center/>

***

### 4. 取消通知
1. 点击通知时，通知自动取消  
```kotlin
val notification = NotificationCompat.Builder(this,"normal")
                ......
                .setAutoCancel(true)
                .build()
```

2. 点击按钮手动取消  
&emsp;&emsp;这里在```cancel()```传入了1，1表示通知的id,及对应```manager.notify()```里的id参数。  
```kotlin
Btn_cancelNotice.setOnClickListener {
    manager.cancel(1)
}
Btn_sendNotice.setOnClickListener {
    ......
    manager.notify(1,notification)
}
```
<img src="https://github.com/gneL1/AndroidStudy/blob/master/photos/MultiMedia/notification_4.gif" width="400" height="680" align=center/>

***

### 5. setStyle()的使用
&emsp;&emsp;```setStyle()```接受一个```NotificationCompat.Style```参数,这个参数能构建具体的富文本信息，如长文字、图片。  
&emsp;&emsp;如果把通知内容换成很长的文字，通知内容无法完整显示，多余的部分会用省略号代替。  
```kotlin
val notification = NotificationCompat.Builder(this,"normal")
    .setContentText("中文即 粘合剂，意思为粘合了两个不同的进程\n" +
        "\n" +
        "网上有很多对Binder的定义，但都说不清楚：Binder是跨进程通信方式、它实现了IBinder接口，是连接 ServiceManager的桥梁blabla，估计大家都看晕了，没法很好的理解\n" +
        "\n" +
        "我认为：对于Binder的定义，在不同场景下其定义不同\n" +
        "\n" +
        "作者：Carson_Ho\n" +
        "链接：https://www.jianshu.com/p/4ee3fd07da14\n" +
        "来源：简书\n" +
        "著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。")
    ...
    .build()
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/MultiMedia/notification_5.png)

1. 设置长文字  
&emsp;&emsp;使用```setStyle()```代替```setContentText()```方法，```NotificationCompat.BigTextStyle()```对象用于封装长文字信息，调用它的```bigText()```方法将文字内容传入。  
```kotlin
val notification = NotificationCompat.Builder(this,"normal")
    .setStyle(NotificationCompat.BigTextStyle().bigText("中文即 粘合剂，意思为粘合了两个不同的进程\n" +
        "\n" +
        "网上有很多对Binder的定义，但都说不清楚：Binder是跨进程通信方式、它实现了IBinder接口，是连接 ServiceManager的桥梁blabla，估计大家都看晕了，没法很好的理解\n" +
        "\n" +
        "我认为：对于Binder的定义，在不同场景下其定义不同\n" +
        "\n" +
        "作者：Carson_Ho\n" +
        "链接：https://www.jianshu.com/p/4ee3fd07da14\n" +
        "来源：简书\n" +
        "著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。"))
    ...
    .build()
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/MultiMedia/notification_6.png)

2. 设置图片。  
&emsp;&emsp;```Notification.BigPictureStyle()```对象用于设置大图片，通过```BitmapFactory```的```decodeResource```方法将图片解析成```Bitmap```对象，再传入```bigPicture()```方法中。  
```kotlin
val notification = NotificationCompat.Builder(this,"normal")
    .setStyle(NotificationCompat.BigPictureStyle().bigPicture(
        BitmapFactory.decodeResource(resources,R.drawable.acane)
    ))
    ...
    .build()
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/MultiMedia/notification_7.png)

***


### 6. 通知的重要等级
&emsp;&emsp;高重要等级的通知渠道可以弹出横幅，发出声音，低重要等级的通知渠道某些情况下可能会被隐藏，被改变显示顺序。  
&emsp;&emsp;开发者只能在创建通知渠道的时候指定初始的重要等级。  
&emsp;&emsp;修改```NotifucationTest```使通知弹出横幅：    
```kotlin
if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){
    val channel2 = NotificationChannel("important","Important",NotificationManager.IMPORTANCE_HIGH)
    manager.createNotificationChannel(channel2)
}
......
val notification2 = NotificationCompat.Builder(this,"important")
    .setContentTitle("这是标题2")
    .setContentText("这是文本2")
    .setSmallIcon(R.drawable.ic_launcher_foreground)
    .setLargeIcon(BitmapFactory.decodeResource(resources,R.drawable.ic_launcher_background))
    .build()
manager.notify(2,notification2)
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/MultiMedia/notification_8.png)

***

## 二、摄像头和相册