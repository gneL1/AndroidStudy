# Broadcast
## 接收系统广播
&emsp;&emsp;```BroadcastReceiver```能够接收广播。  

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


