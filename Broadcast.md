# Broadcast
## 接收系统广播
&emsp;&emsp;```BroadcastReceiver```能够接收广播。  
&emsp;&emsp;不要在```BroadcastReceiver```的```onReceive()```方法中添加过多逻辑或进行耗时操作。  
&emsp;&emsp;```BroadcastReceiver```不允许开启线程，```onReceive()```方法运行长时间不结束，程序会出错。  

### 动态注册监听时间变化
&emsp;&emsp;动态注册即在代码中注册,动态注册的```BroadcastReceiver```，一定要取消注册。  
&emsp;&emsp;动态注册必须在程序启动后才能接收广播。因为注册的逻辑是写在```onCreate()```方法中的。    
&emsp;&emsp;系统每隔一分钟会发送一次```android.intent.action.TIME_TICK```的广播。  
```kotlin
class MainActivity : AppCompatActivity() {
    
    lateinit var timeChangeReceiver : TimeChangeReceiver

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        /**
         * 创建了一个IntentFilter()实例
         * 给它添加了一个值为"android.intent.action.TIME_TICK"的action
         * 当系统时间发生变化时，系统发出的正是一条值为"android.intent.action.TIME_TICK"的广播
         * 即BroadcastReceiver想要监听什么广播，就在这里添加相应的action
         */
        val intentFilter = IntentFilter()
        intentFilter.addAction("android.intent.action.TIME_TICK")
        /**
         * 创建了一个TimeChangeReceiver的实例
         * 调用registerReceiver()方法注册
         * TimeChangeReceiver会受到所有值为"android.intent.action.TIME_TICK"的广播
         */
        timeChangeReceiver = TimeChangeReceiver()
        registerReceiver(timeChangeReceiver,intentFilter)
    }

    override fun onDestroy() {
        super.onDestroy()
        /**
         * 动态注册的BroadcastReceiver一定要取消注册
         */
        unregisterReceiver(timeChangeReceiver)
    }

    /**
     * 定义一个内部类继承自BroadcastReceiver
     * 重写onReceive()方法
     * 这里是每当系统时间发生变化时，onReceiver()方法就会得到执行
     */
    inner class TimeChangeReceiver : BroadcastReceiver(){
        override fun onReceive(context: Context?, intent: Intent?) {
            Toast.makeText(context,"时间已经改变",Toast.LENGTH_SHORT).show()
        }
    }

}
```
<img src="https://github.com/gneL1/AndroidStudy/blob/master/photos/Broadcast/broadcast_01.png" width="400" height="680" align=center/>

&emsp;&emsp;其他系统广播列表可以在```AndroidSDK\platforms\android-30\data\broadcast_actions.txt```路径下查看。  
&emsp;&emsp;获取当前 **SDK** 路径的方法：  
&emsp;&emsp;File -> Other Settings -> Default Project Structure...  

***

### 静态注册实现开机启动
&emsp;&emsp;静态注册可以在程序未启动的情况下接收广播。  
&emsp;&emsp;隐式广播是指没有具体指定发送给哪个应用程序的广播，大多数系统广播属于隐式广播。  
&emsp;&emsp;少数特殊的系统广播允许使用静态注册的方式来接收，参考
[https://developer.android.google.cn/guide/components/broadcast-exceptions.html](https://developer.android.google.cn/guide/components/broadcast-exceptions.html)  


&emsp;&emsp;右键 -> NEW -> Other -> Broadcast Receiver  
&emsp;&emsp;```Exported```属性表示是否允许这个```BroadcastReceiver```接收本程序以外的广播。  
&emsp;&emsp;```Enable```属性表示是否启用这个```BroadcastReceiver```。  
```kotlin
class BootCompleteReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        // This method is called when the BroadcastReceiver is receiving an Intent broadcast.
//        TODO("BootCompleteReceiver.onReceive() is not implemented")
        Toast.makeText(context,"Boot Complete",Toast.LENGTH_SHORT).show()
    }
}
```
&emsp;&emsp;通过快捷方式创建的```BroadcastReceiver```，在```AndroidManifest.xml```中自动完成了注册。  
&emsp;&emsp;所有静态的```BroadcastReceiver```都是在标签```<receiver>```中注册的。  
```xml
<application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <receiver
            android:name=".BootCompleteReceiver"
            android:enabled="true"
            android:exported="true">
        </receiver>
    </application>
```

&emsp;&emsp;一些敏感操作，需要权限声明，如果不进行权限声明，那么程序会直接崩溃。  
&emsp;&emsp;为了收到广播，再进行修改：  
> 在```<receiver>``` 标签中添加一个```<intent-filter>``` 标签，并在里面声明相应的 ```action```  

```xml
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        ......

        <receiver
            android:name=".BootCompleteReceiver"
            android:enabled="true"
            android:exported="true">

            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED"/>
            </intent-filter>

        </receiver>
    </application>
```

***

## 发送自定义广播
### 发送标准广播
&emsp;&emsp;标准广播是一种完全异步的广播，发出后，所有```BroadcastReceiver```几乎都会在同一时刻收到广播，没有先后顺序可言，效率高且无法被截断。  
1. 新建一个```MyBroadcastReceiver```  
```kotlin
class MyBroadcastReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        // This method is called when the BroadcastReceiver is receiving an Intent broadcast.
//        TODO("MyBroadcastReceiver.onReceive() is not implemented")
        Toast.makeText(context,"收到广播了",Toast.LENGTH_SHORT).show()
    }
}
```

2. 修改```AndroidManifest.xml```  
> 这里让**MyBroadcastReceiver** 接收一条值为 **com.example.broadcasttest.MY_BROADCAST** 的广播  
```xml
<receiver
    android:name=".MyBroadcastReceiver"
    android:enabled="true"
    android:exported="true">

    <intent-filter>
        <action android:name="com.example.broadcasttest.MY_BROADCAST"/>
    </intent-filter>

</receiver>
```

3. 修改```MainActivity```  
> 广播使用 **Intent** 发送，还可以在 **Intent** 中携带一些数据传递给相应的 **BroadcastReceiver**  
```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        Btn_1.setOnClickListener {
            /**
             * 构建一个Intent对象，传入要发送的广播的值
             * 调用Intent的setPackage方法，传入当前应用程序的包名
             * getPackageName()获取当前应用程序的包名
             * sendBroadcast()将广播发送出去
             * 静态注册的BroadcastReceiver无法接收隐式广播，默认情况下发出的自定义广播恰恰都是隐式广播
             * 这里调用setPackage()方法指定这条广播发送给哪个应用程序
             * 让它变成一条显示广播
             */
            val intent = Intent("com.example.broadcasttest.MY_BROADCAST")
            intent.setPackage(packageName)
            sendBroadcast(intent)
        }
    }

}
```
<img src="https://github.com/gneL1/AndroidStudy/blob/master/photos/Broadcast/broadcast_02.png" width="400" height="680" align=center/>

***

### 发送有序广播
&emsp;&emsp;有序广播是一种同步执行的广播，广播发出后，同一时刻只会有一个```BroadcastReceiver```收到广播。当此```BroadcastReceiver```中的逻辑
执行完毕后，广播才会继续传递。优先级高的```BroadcastReceiver```先收到广播，前面的```BroadcastReceiver```可以截断正在传递的广播。  
&emsp;&emsp;```sendOrderedBroadcast()```发送有序广播。  
&emsp;&emsp;```abortBroadcast()```方法表示将广播截断。  
1. 新建一个```AnotherBroadcastReceiver```  
```kotlin
class AnotherBroadcastReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        Toast.makeText(context,"接收到了有序广播",Toast.LENGTH_SHORT).show()
    }
}
```

2. 修改```AndroidManifest.xml```  
> 设置与上面的 **MyBroadcastReceiver** 接收同样的广播 **com.example.broadcasttest.MY_BROADCAST**  
```xml
<receiver
    android:name=".AnotherBroadcastReceiver"
    android:enabled="true"
    android:exported="true">

    <intent-filter>
        <action android:name="com.example.broadcasttest.MY_BROADCAST"/>
    </intent-filter>

</receiver>
```

3. 修改```MainActivity```  
&emsp;&emsp;```sendOrderedBroadcast(Intent intent,String receiverPermission)```即发送有序广播。  
&emsp;&emsp;第一个参数是```Intent```,第二个参数是一个与权限相关的字符串，如果为```null```，则不需要权限。  
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    Btn_1.setOnClickListener {
        val intent = Intent("com.example.broadcasttest.MY_BROADCAST")
        intent.setPackage(packageName)
        sendOrderedBroadcast(intent,null)
//            sendBroadcast(intent)
    }
}
```

4. 修改```AndroidManifest.xml```设置广播的先后顺序。  
&emsp;&emsp;通过```android:priority```属性给```BroadcastReceiver```设置优先级，优先级高的```BroadcastReceiver```先收到广播。  
```xml
<receiver
    android:name=".MyBroadcastReceiver"
    android:enabled="true"
    android:exported="true">
    <intent-filter android:priority="100">
        <action android:name="com.example.broadcasttest.MY_BROADCAST" />
    </intent-filter>
</receiver>
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Broadcast/broadcast_ordered_1.gif)

5. 修改```MyBroadcastReceiver```使用```abortBroadcast()```截断广播  
```kotlin
class MyBroadcastReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        Toast.makeText(context,"收到广播了",Toast.LENGTH_SHORT).show()
        abortBroadcast()
    }
}
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Broadcast/broadcast_ordered_2.gif)

***

## 实现一个强制下线的功能
1. 新建一个```ActivityController```管理所有```Activity```  
```kotlin
object ActivityController {
    private val activities = ArrayList<Activity>()

    fun addActivity(activity: Activity){
        activities.add(activity)
    }

    fun removeActivity(activity: Activity){
        activities.remove(activity)
    }

    fun finishAll(){
        for (activity in activities){
            if (!activity.isFinishing){
                activity.finish()
            }
        }
        activities.clear()
    }
}
```

2. 新建```BaseActivity```类作为所有```Activity```的父类  
```kotlin
open class BaseActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ActivityController.addActivity(this)
    }

    override fun onDestroy() {
        super.onDestroy()
        ActivityController.removeActivity(this)
    }
    
}
```

3. 创建一个```LoginActivity```作为登陆界面，将主```Activity```设置为```LoginActivity```,登录成功后跳转至```MainActivity```    
**AndroidManifest.xml**  
```xml
<activity android:name=".LoginActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
```
&emsp;&emsp;**activity_login.xml**  
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".LoginActivity">

    <LinearLayout
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="60dp">

        <TextView
            android:layout_gravity="center_vertical"
            android:textSize="18sp"
            android:text="账号："
            android:layout_width="90dp"
            android:layout_height="wrap_content"/>

        <EditText
            android:id="@+id/Edit_Account"
            android:layout_weight="1"
            android:layout_gravity="center_vertical"
            android:layout_width="0dp"
            android:layout_height="wrap_content"/>

    </LinearLayout>

    <LinearLayout
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="60dp">

        <TextView
            android:textSize="18sp"
            android:text="密码："
            android:layout_gravity="center_vertical"
            android:layout_width="90dp"
            android:layout_height="wrap_content"/>

        <EditText
            android:id="@+id/Edit_Password"
            android:layout_weight="1"
            android:layout_gravity="center_vertical"
            android:layout_width="0dp"
            android:layout_height="wrap_content"/>

    </LinearLayout>

    <Button
        android:id="@+id/Btn_Login"
        android:text="登录"
        android:layout_gravity="center_horizontal"
        android:layout_width="200dp"
        android:layout_height="60dp"/>

</LinearLayout>
```
&emsp;&emsp;**LoginActivity**  
```kotlin
class LoginActivity : BaseActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_login)

        Btn_Login.setOnClickListener {
            val account = Edit_Account.text.toString()
            val password = Edit_Password.text.toString()
            if(account == "admin" && password == "123"){
                val intent = Intent(this,MainActivity::class.java)
                startActivity(intent)
                finish()
            }
            else{
                Toast.makeText(this,"账号或密码错误",Toast.LENGTH_SHORT).show()
            }
        }
    }
}
```

4. 修改```MainActivity```  
&emsp;&emsp;点击按钮发送一条广播，通知程序强制下线。  
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    Btn_1.setOnClickListener {
        val intent = Intent("com.example.broadcasttest.MY_BROADCAST")
        intent.setPackage(packageName)
        sendOrderedBroadcast(intent,null)
    }

    Btn_Off.setOnClickListener {
        val intent = Intent("com.example.broadcastbestpractice.FORCE_OFFLINE")
        sendBroadcast(intent)
    }
}
```

5. 修改```BaseActivity```  
&emsp;&emsp;创建一个```BroadcastReceiver```接收广播，```BroadcastReceiver```弹出一个对话框来阻塞用户的正常操作。  
&emsp;&emsp;如果是静态注册的```BroadcastReceiver```，是没法在```onReceiver()```方法里弹出对话框。  
&emsp;&emsp;重写```onResume()```和```onPause()```方法，分别在这两个方法中注册和取消```ForceOfflineReceiver```。  
&emsp;&emsp;因为只要处于栈顶的```Activity```才能接收到广播，非栈顶的```Activity```没必要接收广播，当一个```Activity```失去栈顶位置时，自动取消```BroadcastReceiver```的注册。  
```kotlin
open class BaseActivity : AppCompatActivity() {

    lateinit var receiver : ForceOffLineReceiver

    inner class ForceOffLineReceiver : BroadcastReceiver() {
        override fun onReceive(context: Context?, intent: Intent?) {
            AlertDialog.Builder(context).apply {
                setTitle("警告")
                setMessage("当前强制下线，请尝试重新登陆！")
                /**
                 * setCancelable将对话框设置为不可取消
                 * 否则按一下Back键就可以关闭对话框继续使用程序
                 */
                setCancelable(false)
                /**
                 * 设置确定按钮
                 */
                setPositiveButton("确定"){_,_ ->
                    //销毁所有Activity
                    ActivityController.finishAll()
                    val i = Intent(context,LoginActivity::class.java)
                    //重新启动LoginActivity
                    context?.startActivity(i)
                }
                show()
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ActivityController.addActivity(this)
    }

    override fun onResume() {
        super.onResume()
        val intentFilter = IntentFilter()
        intentFilter.addAction("com.example.broadcastbestpractice.FORCE_OFFLINE")
        receiver = ForceOffLineReceiver()
        registerReceiver(receiver,intentFilter)
    }

    override fun onPause() {
        super.onPause()
        unregisterReceiver(receiver)
    }
    
    override fun onDestroy() {
        super.onDestroy()
        ActivityController.removeActivity(this)
    }

}
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Broadcast/broadcast_off_line.gif)



