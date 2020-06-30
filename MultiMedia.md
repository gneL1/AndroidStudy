# 多媒体
## 一、通知  
&emsp;&emsp;每条通知都属于一个对应的渠道，也就是通知渠道，通知渠道创建后不能再修改。  
&emsp;&emsp;通知渠道一旦创建后就不能再修改了。  

### 1. 统治的基本用法
1. 首先需要一个```NotificationManager```对通知进行管理，通过```Context```的```getSystemService()```方法获取。  
```getSystemService(@RecentlyNonNull String name)```接收一个字符串用于确定获取系统的哪个服务。  
```kotlin
val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
```

2. 使用```NotificationChannel```类构建一个通知渠道,并调用```NotificationManager.createNotificationChannel()```方法完成创建。  
由于```NotificationChannel```类和```createNotificationChannel()```方法都是Android 8.0系统新增的API,所以要进行版本判断。  
```Build.VERSION_CODES.O = 26``` ，即Android API = 26 对应Android 8.0(Oreo)  
```Build.VERSION.SDK_INT```是系统当前的版本号  
```NotificationChannel(String id, CharSequence name, int importance)```  
```id```：渠道id,保证全局唯一  
```name```：渠道名称，给用户看的  
```importance```：重要等级，通过```NotificationManager```来调用级别程度  
