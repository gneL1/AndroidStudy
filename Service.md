# Service
## 一、Android多线程
### 1. 线程的基本用法
&emsp;&emsp;定义一个线程：新建一个类继承```Thread```，然后重写父类的```run()```方法，在里面编写耗时逻辑。  
```kotlin
class MyThread : Thread(){
    override fun run() {
        //编写具体的逻辑
    }
}
```

&emsp;&emsp;启动线程：创建```MyThread```的实例，调用它的```start()```方法，这样```run()```方法中的代码就会在子线程中运行了。  
```kotlin
MyThread().start()
```

&emsp;&emsp;使用实现```Runnable```接口的方式定义一个线程，可以减轻耦合性。  
```kotlin
class MyThread : Runnable{
    override fun run() {
        //编写具体的逻辑
    }
}
```
&emsp;&emsp;对应的启动线程方法也要进行相应改变。  
&emsp;&emsp;```Thread```的构造函数接收一个```Runnable```参数，创建的```MyThread```实例正是一个实现了```Runnable```接口的对象，将它传入```Thread```的构造函数里，再调用```Thread```的```start()```方法，```run()```方法中的代码就会在子线程中运行了。  
```kotlin
val myThread = MyThread()
Thread(myThread).start()
```

&emsp;&emsp;如果不想再定义一个类去实现```Runnable```接口，可以使用```Lambda```方式。  
```kotlin
Thread{
  //编写具体的逻辑
}.start()
```

&emsp;&emsp;Kotlin开启线程的方式：  
&emsp;&emsp;```thread```是Kotlin内置的顶层函数，只需要在```Lambda```表达式中编写具体的逻辑，```start()```方法在```thread```函数内部处理好了。  
```kotlin
thread { 
  //编写具体的逻辑            
}
```

***

### 2. 在子线程中更新UI
&emsp;&emsp;Android的UI是线程不安全的，不允许在子线程中进行UI操作。如果想要更新应用程序里的UI元素，必须在主线程中进行，否则会出现异常。  
&emsp;&emsp;  
**修改```activity_android_thtead_test.xml```布局文件：**  
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".AndroidThteadTest">

    <Button
        android:id="@+id/Btn_1"
        android:text="改变文本"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <TextView
        android:id="@+id/Tv_1"
        android:text="测试文本"
        android:layout_gravity="center_horizontal"
        android:textSize="20sp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

</LinearLayout>
```

**修改```AndroidThteadTest```文件：**  
```kotlin
class AndroidThteadTest : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_android_thtead_test)

        Btn_1.setOnClickListener {
            thread {
                Tv_1.text = "当前文本已改变"
            }
        }
    }
    
}
```
&emsp;&emsp;在子线程里尝试更新UI，报错：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Service/thread_error.PNG)

&emsp;&emsp;  
**在子线程中进行UI操作的方法：**  
&emsp;&emsp;定义一个整型变量```updateText```，用于表示更新```TextView```这个动作。新增一个```Handler```对象，并重写父类的```handleMessage()```方法，在这里对具体的```Message```进行处理。  
&emsp;&emsp;创建一个```Message```对象，并将它的```what```字段的值指定为```updateText```，然后调用```Handler```的```sendMessage()```方法将这条```Message```发送出去。
```kotlin
class AndroidThteadTest : AppCompatActivity() {

    val updateText = 1

    private val handler = object : Handler(){
        override fun handleMessage(msg: Message) {
            //在这里进行UI操作
            when(msg.what){
                updateText -> Tv_1.text = "当前文本更新"
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_android_thtead_test)
        Btn_1.setOnClickListener {
            thread {
                val msg = Message()
                msg.what = updateText
                handler.sendMessage(msg)
            }
        }
    }
}
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Service/thread_1.gif)

***

### 3. 异步消息处理机制
1. **Message**  
&emsp;&emsp;```Message```是在线程之间传递的消息，它可以在内部携带少量的信息，用于在不同线程之间传递数据。除了```what```字段，还可以用```arg1```和```arg2```字段来携带一些整形数据，使用```obj```字段携带一个```Object```对象。  

2. **Handler**  
&emsp;&emsp;用于发送和处理消息。发送消息一般使用```Handler```的```sendMessage()```方法、```post()```方法等。发出的消息最终会传递到```Handler```的```handleMessage()```方法中。  

3. **MessageQueue**  
&emsp;&emsp;消息队列，用于存放所有通过```Handler```发送的消息。这部分消息会一直存在于消息队列中，等待被处理。每个线程中只会有一个```MessageQueue```对象。  

4. **Looper**  
&emsp;&emsp;```Looper```是每个线程中的```MessageQueue```的管家。调用```Looper```的```loop()```方法后，就会进入一个无限循环中，然后每当发现```MessageQueue```中存在一条消息时，就会将它取出，并传递到```Handler```的```handleMessage()```方法中。每个线程只会有一个```Looper```对象。  

**流程：**  
&emsp;&emsp;首先需要在主线程中创建一个```Handler```对象，并重写```handleMessage()```方法。然后当子线程中需要进行UI操作时，就创建一个```Message```对象，并通过```Handler```将这条消息发送出去。之后这条消息会被添加到```MessageQueue```的队列中等待被处理，而```Looper```则会一直尝试从```MessageQueue```中取出待处理消息，最后分发回```Handler```的```handleMessage()```方法中。由于```Handler```是在主线程中创建的，所以```handleMessage()```方法也会在主线程中运行。  

***

### 4. AsyncTask
待补充  

***

## 二、Service
### 1. Service的基本用法
1. **定义一个```Service```**  
右键包名 -> New -> Service -> Service  
&emsp;&emsp;```Exported```表示是否将这个```Service```暴露给外部其他程序访问，```Enabled```表示是否启用这个```Service```。  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Service/service_create.PNG)

&emsp;&emsp;```Myservice```继承自系统的```Service```类，```onBind()```方法是```Service```中唯一的抽象方法，必须在子类里实现。
&emsp;&emsp;  
&emsp;&emsp;```onCreate()```方法会在```Service```创建的时候调用。  
&emsp;&emsp;```onStartCommand()```方法会在每次```Service```启动的时候调用。  
&emsp;&emsp;```onDestroy()```方法会在```Service```销毁的时候调用。  
&emsp;&emsp;如果希望```Service```一旦启动就立刻去执行某个动作，就可以将逻辑写在```onStartCommand()```方法里。当```Service```销毁时，在```onDestroy()```方法中回收不再使用的资源。  
```kotlin
class MyService : Service() {

    override fun onBind(intent: Intent): IBinder {
        TODO("Return the communication channel to the service.")
    }

    override fun onCreate() {
        super.onCreate()
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        return super.onStartCommand(intent, flags, startId)
    }

    override fun onDestroy() {
        super.onDestroy()
    }
}
```
&emsp;&emsp;  
注意```Service```需要在```AndroidManifest.xml```中注册才能生效。  
```xml
<application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".ServicePageTest"></activity>

        <service
            android:name=".MyService"
            android:enabled="true"
            android:exported="true" />
        ......    
```

***

&emsp;&emsp;  
2. **启动和停止```Service```**  
修改```activity_service_page_test```布局文件：  
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".ServicePageTest">

    <Button
        android:id="@+id/Btn_startService"
        android:text="开始服务"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <Button
        android:id="@+id/Btn_stopService"
        android:text="停止服务"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

</LinearLayout>
```

修改```ServicePageTest```文件：  
&emsp;&emsp;构建一个```Intent```对象，调用```startService()```方法来启动```MyService```，调用```stopService()```来停止```MyService```。  
&emsp;&emsp;```Service```也可以自我停止运行，在```Service```内部调用```stopSelf()```方法。  
```kotlin
class ServicePageTest : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_service_page_test)

        Btn_startService.setOnClickListener {
            val intent = Intent(this,MyService::class.java)
            startService(intent)
        }

        Btn_stopService.setOnClickListener {
            val intent = Intent(this,MyService::class.java)
            stopService(intent)
        }
    }
}
```

修改```MyService```文件： 
```kotlin
class MyService : Service() {

    private val tag = "MyService"

    override fun onBind(intent: Intent): IBinder {
        TODO("Return the communication channel to the service.")
    }

    override fun onCreate() {
        super.onCreate()
        Log.d(tag,"onCreate executed")
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        Log.d(tag,"onStartCommand executed")
        return super.onStartCommand(intent, flags, startId)
    }

    override fun onDestroy() {
        super.onDestroy()
        Log.d(tag,"onDestroy executed")
    }
}
```

启动```Service```：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Service/service_1.PNG)

停止```Service```：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Service/service_2.PNG)

多次启动```Service```：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Service/service_3.PNG)

***

&emsp;&emsp;  
3. **```Activity```和```Service```进行通信**  
&emsp;&emsp;通过```onBind()```方法来进行通信。  
&emsp;&emsp;  
修改```Myservice```：  
&emsp;&emsp;新建一个```DownloadBinder```类，让它继承自```Binder```，在它的内部提供下载和查看进度的方法。然后创建```DownloadBinder```的实例，在```onBind()```方法里返回这个实例。  
```kotlin
class MyService : Service() {

    private val mBinder = DownloadBinder()

    class DownloadBinder : Binder(){
        fun startDownload(){
            Log.d("MyService","开始执行下载")
        }

        fun getProgress() : Int{
            Log.d("MyService","获取进度")
            return  0
        }
    }

    override fun onBind(intent: Intent): IBinder {
        return mBinder
    }
    ......
}
```
&emsp;&emsp;  
修改```activity_service_page_test.xml```布局文件：  
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".ServicePageTest">
    
    ......
    <Button
        android:id="@+id/Btn_bindService"
        android:text="绑定服务"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <Button
        android:id="@+id/Btn_unbindService"
        android:text="解绑服务"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

</LinearLayout>
```
&emsp;&emsp;  
修改```ServicePageTest```：  
&emsp;&emsp;创建一个```ServiceConnection```的匿名类实现，重写```onServiceConnected()```和```onServiceDisconnected()```方法。```onServiceConnected()```方法会在```Activity```和```Service```成功绑定的时候调用。```onServiceDisconnected()```方法只有在```Service```的创建进程崩溃或者被杀掉的时候才会调用，不常用。  
&emsp;&emsp;在```onServiceConnected()```方法中，通过向下转型得到```DownloadBinder```的实例，使用实例来调用```Service```里```DownloadBinder```的方法。  
&emsp;&emsp;构建一个```Intent```对象，调用```bindService(Intent service, ServiceConnection conn,int flags)```方法将```Activity```和```Service```进行绑定。  
&emsp;&emsp;```bindService```的第一个参数是构建出来的```Intent```对象，第二个参数是创建出来的```ServiceConnection```实例，第三个参数是一个标志位，传入```BIND_AUTO_CREATE```表示```Activity```和```Service```进行绑定后自动创建```Service```，这会使```MyService```的```onCreate()```方法得到执行，但```onStartCommand()```方法不会执行。  
&emsp;&emsp;```unbindService()```方法解除```Activity```和```Service```之间的绑定。  
```kotlin
class ServicePageTest : AppCompatActivity() {

    lateinit var downloadBinder: MyService.DownloadBinder

    private val connection = object : ServiceConnection{
        override fun onServiceDisconnected(name: ComponentName?) {
            TODO("Not yet implemented")
        }

        override fun onServiceConnected(name: ComponentName?, service: IBinder?) {
            downloadBinder = service as MyService.DownloadBinder
            downloadBinder.startDownload()
            downloadBinder.getProgress()
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        ......

        Btn_bindService.setOnClickListener {
            val intent = Intent(this,MyService::class.java)
            bindService(intent,connection,Context.BIND_AUTO_CREATE)
        }

        Btn_unbindService.setOnClickListener {
            unbindService(connection)
        }
    }
}
```
点击绑定服务：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Service/service_4.PNG)

***

### 2. Servide的生命周期
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Service/service_life.png)

* 4个手动调用的方法：  

手动调用方法|作用
:-|:-|
startService()|启动服务
stopService()|关闭服务
bindService()|绑定服务
unbindService()|解绑服务

* 5个自动调用的方法：  

内部自动调用的方法|作用
:-|:-|
onCreate()|创建服务
onStartCommand()|开始服务
onDestroy()|销毁服务
onBind()|绑定服务
onUnbind()|解绑服务

* 关于操作```Service```  
&emsp;&emsp;```startService()```、```stopService()```只能启动/关闭```Service```，不能操作```Service```。  
&emsp;&emsp;```bindService()```、```unbindService()```除了绑定```Service```，还能操作```Service```。  

* 关于```Service```何时销毁  
&emsp;&emsp;```startService()```开启的```Service```，调用者退出后，```Service```仍然存在。  
&emsp;&emsp;```bindService()```开启的```Service```，```Service```随着调用者退出而销毁。  
&emsp;&emsp;  
&emsp;&emsp;特别值得注意的是，当绑定```Service```后，关闭```Activity```，由于不会自动解绑，程序会报错  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Service/thread_error_2.PNG)

**解决方法：**  
在```MyService```中：  
```kotlin
class MyService : Service() {
    ......
    companion object{
        var isBinding = false
    }
    
    override fun onBind(intent: Intent): IBinder {
        isBinding = true
        return mBinder
    }
    
    override fun onUnbind(intent: Intent?): Boolean {
        Log.d(tag,"onUnbind executed")
        isBinding = false
        return super.onUnbind(intent)
    }
}
```
在```ServicePageTest```中：  
```kotlin
class ServicePageTest : AppCompatActivity() {
    ......
    override fun onDestroy() {
        super.onDestroy()
        if (MyService.isBinding){
            unbindService(connection)
        }
    }
}
```
运行效果：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Service/unbind_1.PNG)

***

### 3. 前台Service
&emsp;&emsp;从Android 8.0 开始，只有当应用保持在前台可见状态下，```Service```才能保证稳定运行。一旦应用进入后台，```Service```随时都有可能被系统回收。  
&emsp;&emsp;使用前台```Service```，```Service```可以一直保持运行状态，它会有一个正在运行的图标在系统的状态栏显示，下拉状态可以看到更加详细的信息。  
&emsp;&emsp;  
**修改```MyService```：**  
&emsp;&emsp;构建```Notification```对象后，调用```startForeground()```方法。  
&emsp;&emsp;```startForeground(int id, Notification notification)```  
&emsp;&emsp;第一个参数是通知的id,类似于```notify()```的第一个参数。  
&emsp;&emsp;第二个参数是构建的```Notification```对象。  
```kotlin
class MyService : Service() {
    override fun onCreate() {
        super.onCreate()
        val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){
            val channel = NotificationChannel("my_service","前台Service通知",NotificationManager.IMPORTANCE_DEFAULT)
            manager.createNotificationChannel(channel)
        }
        val intent = Intent(this,ServicePageTest::class.java)
        val pi = PendingIntent.getActivity(this,0,intent,0)
        val notification = NotificationCompat.Builder(this,"my_service")
            .setContentTitle("这是文本")
            .setContentText("这是内容")
            .setSmallIcon(R.drawable.ic_launcher_foreground)
            .setLargeIcon(BitmapFactory.decodeResource(resources,R.drawable.ic_launcher_background))
            .setContentIntent(pi)
            .build()
        startForeground(1,notification)
        Log.d(tag,"onCreate executed")
    }
    ......
}
```
&emsp;&emsp;  
**修改```AndroidManifest.xml```：**  
&emsp;&emsp;从Android 9.0 系统开始，使用前台```Service```必须在```AndroidManifest.xml```中进行权限声明，不然程序会报错。  
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.servicetest">

    <uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
    ......
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Service/service_foreground.gif)

***

### 4. IntentService
&emsp;&emsp;```Service```中的代码都是默认运行在主线程中的，如果在```Service```里执行一些耗时的逻辑，容易出现```ANR```(Application Not Responding)。  
&emsp;&emsp;可以在```Service```的每个具体方法里开启一个子线程，在子线程里处理耗时逻辑：  
```kotlin
class MyService : Service(){
    override fun onStartCommand(intent : Intent,flags : Int,startId : Int) : Int{
        thread {
            //处理具体逻辑
        }
        return super.onStartCommand(intent,flags,startId)
    }    
}
```
&emsp;&emsp;  
&emsp;&emsp;但是```Service```一旦启动，必须调用```stopService```或```stopSelf()```方法，或者被系统回收，```Service```才会停止。  
```kotlin
class MyService : Service(){
    override fun onStartCommand(intent : Intent,flags : Int,startId : Int) : Int{
        thread {
            //处理具体逻辑
            stopSelf()
        }
        return super.onStartCommand(intent,flags,startId)
    }    
}
```

