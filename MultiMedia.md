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
### 1. 调用摄像头拍照
**修改```CameraAlbumTest```的布局界面**  
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".CameraAlbumTest">

    <Button
        android:id="@+id/Btn_takePhoto"
        android:text="照相"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <ImageView
        android:id="@+id/Iv_image"
        android:layout_gravity="center_horizontal"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

</LinearLayout>
```

**修改```CameraAlbumTest```**  
1. 创建```File```对象，用于存储拍照后的图片,这里把图片命名为```output_image.jpg```。  
图片存放在手机SD卡的应用关联缓存目录下，调用```getExternalCacheDir()```得到该目录。  
应用关联缓存目录是SD卡中专门用于存放当前应用缓存数据的位置，具体路径是```/sdcard/Android/data/包名/cache```。  
```kotlin
outputImage = File(externalCacheDir,"output_image.jpg")
if(outputImage.exists()){
    outputImage.delete()
}
outputImage.createNewFile()
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/MultiMedia/camera_path.PNG)

2. 进行版本判断，系统版本低于7.0 就调用```Uri```的```fromFile()```方法将```File```对象转换成```Uri```对象，这个```Uri```对象标识着```output_image.jpg```的本地真实路径。  
不低于7.0，就调用```FileProvider```的```getUriForFile()```方法将```File```对象转换成一个封装过的```Uri```对象。  
Build.VERSION_CODES.N = 24 对应 Android版本7.0 (Nougat)，从Android 7.0 开始，直接使用本地真实路径的Uri被认为是不安全的，会抛出一个```FileUriExposedException```异常。  
```FileProvider```是一种特殊的```ContentProvider```,使用了和```ContentProvider```类似的机制来对数据进行保护，可以选择性的将封装过的```Uri```共享给外部，提高安全性。  
```kotlin
imageUri = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N){
    FileProvider.getUriForFile(this,"com.example.multimediatest.fileprovider",outputImage)
}
else{
    Uri.fromFile(outputImage)
}
```

3. 构建一个```Intent```对象，并将这个```Intent```的```action```指定为```android.media.action.IMAGE_CAPTURE```，再调用```Intent```的```putExtra()```方法指定图片的输出地址，即刚获得的```Uri```对象，最后调用```startActivityForResult()```启动```Activity```。  
这里用的是隐式```Intent```,系统会找出能够响应这个```Intent```的```Activity```去启动。  
```kotlin
val intent = Intent("android.media.action.IMAGE_CAPTURE")
intent.putExtra(MediaStore.EXTRA_OUTPUT,imageUri)
startActivityForResult(intent,takePhoto)
```

4. 拍完照会有结果返回到```onActivityResult()```。  
如果拍照成功，调用```BitmapFactory.decodeStream()```方法将```output_image.jpg```解析成```Bitmap```对象，设置到```ImageView```中显示出来。  
```kotlin
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)
    when(requestCode){
        takePhoto -> {
            //注意这里比较的时返回结果resultCode
            if (resultCode == Activity.RESULT_OK){
                //将拍摄的照片显示出来
                val bitmap = BitmapFactory.decodeStream(contentResolver.openInputStream(imageUri))
                Iv_image.setImageBitmap(rotateIfRequired(bitmap))
            }
        }
    }
}
```

> 完整代码：  
```kotlin
class CameraAlbumTest : AppCompatActivity() {

    private val takePhoto = 1
    lateinit var imageUri : Uri
    lateinit var outputImage : File

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_camera_album_test)

        Btn_takePhoto.setOnClickListener {
            outputImage = File(externalCacheDir,"output_image.jpg")
            if(outputImage.exists()){
                outputImage.delete()
            }
            outputImage.createNewFile()

            imageUri = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N){
                FileProvider.getUriForFile(this,"com.example.multimediatest.fileprovider",outputImage)
            }
            else{
                Uri.fromFile(outputImage)
            }
            
            val intent = Intent("android.media.action.IMAGE_CAPTURE")
            intent.putExtra(MediaStore.EXTRA_OUTPUT,imageUri)
            startActivityForResult(intent,takePhoto)
        }
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        when(requestCode){
            takePhoto -> {
                //注意这里比较的时返回结果resultCode
                if (resultCode == Activity.RESULT_OK){
                    //将拍摄的照片显示出来
                    val bitmap = BitmapFactory.decodeStream(contentResolver.openInputStream(imageUri))
                    Iv_image.setImageBitmap(rotateIfRequired(bitmap))
                }
            }
        }
    }

    /**
     * 调用照相机程序拍照，会发生照片旋转的问题
     * 手机认为摄像头打开进行拍摄时手机是横屏的，因此回到竖屏的情况下会发生90度旋转
     * 在这里对图片方向进行修正
     */
    private fun rotateIfRequired(bitmap: Bitmap) : Bitmap{
        val exif = ExifInterface(outputImage.path)
        val orientation = exif.getAttributeInt(ExifInterface.TAG_ORIENTATION,ExifInterface.ORIENTATION_NORMAL)
        return when(orientation){
            ExifInterface.ORIENTATION_ROTATE_90 -> rotateBitmap(bitmap,90)
            ExifInterface.ORIENTATION_ROTATE_180 -> rotateBitmap(bitmap,180)
            ExifInterface.ORIENTATION_ROTATE_270 -> rotateBitmap(bitmap,270)
            else -> bitmap
        }
    }

    private fun rotateBitmap(bitmap: Bitmap,degree : Int):Bitmap{
        val matrix = Matrix()
        matrix.postRotate(degree.toFloat())
        val rotateBitmap = Bitmap.createBitmap(bitmap,0,0,bitmap.width,bitmap.height,matrix,true)
        //不再需要的Bitmap对象回收
        bitmap.recycle()
        return rotateBitmap
    }
}
```

**修改```AndroidManifest.xml```**  
&emsp;&emsp;因为用到了```FileProider```，它继承自```ContentProvider```，所以要在```AndroidManifest.xml```中进行注册。  
&emsp;&emsp;```android:name```属性的值固定，```android:authority```的值必须与```FileProvider.getUriForFile()```方法中的第二个参数一致。  
&emsp;&emsp;```<provider>```标签内部使用```<meta-data>```指定Uri的共享路径，并引用了一个```@xml/file_paths```资源。  
```xml
<application
    android:allowBackup="true"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:supportsRtl="true"
    android:theme="@style/AppTheme">
    ......
    <provider
        android:authorities="com.example.multimediatest.fileprovider"
        android:name="androidx.core.content.FileProvider"
        android:exported="false"
        android:grantUriPermissions="true">
        <meta-data
            android:name="android.support.FILE_PROVIDER_PATHS"
            android:resource="@xml/file_paths"/>
    </provider>
</application>
```

**新增```file_path.xml```文件**  
&emsp;&emsp;右键res目录 -> New -> Directory,创建一个xml目录，右键xml目录 -> New -> File，创建一个```file_paths.xml```文件。  
&emsp;&emsp;```external-path```指定Uri共享，```name```属性随便填，```path```属性表示共享的具体路径，使用单斜线表示将整个SD卡进行共享，也可以仅共享存放```output_image.jpg```这张图片的路径。    
```xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path
        name="my_images"
        path="/"/>
</paths>
```
打开摄像头拍照：  
<img src="https://github.com/gneL1/AndroidStudy/blob/master/photos/MultiMedia/camera_1.png" width="400" height="680" align=center/>

拍照结果：  
<img src="https://github.com/gneL1/AndroidStudy/blob/master/photos/MultiMedia/camera_2.png" width="400" height="680" align=center/>

***

### 2. 从相册中选择图片
```kotlin
//相册
private val fromAlbum = 2
override fun onCreate(savedInstanceState: Bundle?) {
   ......
   /**
     * 构建一个Intent对象，将action指定为Intent.ACTION_OPEN_DOCUMENT，表示打开系统的文件选择器
     * 给Intent对象设置条件过滤，只允许可打开的图片文件显示出来
     */
    Btn_album.setOnClickListener {
        //打开文件选择器
        val intent = Intent(Intent.ACTION_OPEN_DOCUMENT)
        intent.addCategory(Intent.CATEGORY_OPENABLE)
        //指定只显示的图片
        intent.type = "image/*"
        startActivityForResult(intent,fromAlbum)
    }
}

override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    ......
    when(requestCode){
        /**
         * 调用返回的Intent对象的getData()方法获取选中图片的Uri
         * 调用getBitmapFromUri()方法将Uri转换成Bitmap对象，显示到界面上
         */
        fromAlbum -> {
            if (resultCode == Activity.RESULT_OK && data != null){
                data.data?.let {
                    //将选择的图片显示出来
                    val bitmap = getBitmapFromUri(it)
                    Iv_image.setImageBitmap(bitmap)
                }
            }
        }
    }
}
private fun getBitmapFromUri(uri : Uri) = contentResolver.openFileDescriptor(uri,"r")?.use {
    BitmapFactory.decodeFileDescriptor(it.fileDescriptor)
}
```
选择图片：  
<img src="https://github.com/gneL1/AndroidStudy/blob/master/photos/MultiMedia/album_1.png" width="400" height="680" align=center/>

展示效果：  
<img src="https://github.com/gneL1/AndroidStudy/blob/master/photos/MultiMedia/album_2.png" width="400" height="680" align=center/>

***

## 三、播放多媒体文件
### 1. 播放音频
```MediaPlayer```类控制音频的播放。  

方法名|功能描述
:-|:-|
setDataSource()|设置要播放的音频文件的位置
prepare()|在开始播放之前调用，以完成准备工作
start()|开始或继续播放音频
pause()|暂停播放音频
reset()|将MediaPlayer对象重置到刚刚创建的状态
seekTo()|从指定的位置开始播放音频
stop()|停止播放音频。调用后的MediaPlauer无法再播放音频
release()|释放与MediaPlayer对象相关的资源
isPlaying()|判断当前MediaPlayer是否正在播放音频
getDuration()|获取载入的音频文件的时常

&emsp;&emsp;创建一个```MediaPlayer```对象，调用```setDataSource()```方法设置音频文件的路径，再调用```prepare()```方法进入准备状态。接下来调用```start()```方法开始播放音频，```pause```暂停播放，调用```reset()```方法停止播放。  

修改布局文件：  
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".PlayAudioTest">

    <Button
        android:id="@+id/Btn_play"
        android:text="播放"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <Button
        android:id="@+id/Btn_pause"
        android:text="暂停"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <Button
        android:id="@+id/Btn_stop"
        android:text="停止"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

</LinearLayout>
```
&emsp;&emsp;  
&emsp;&emsp;```assets```目录可以存放任意文件和子目录，这些文件和子目录可以在打包的时候一并被打包到安装文件中，在程序中可以借助```AssetManager```类提供的接口对```assets```目录下的文件进行读取。  
&emsp;&emsp;右键app/src/main -> New -> Directory，输入assets创建完成。  

修改```PlayAudioTest```文件：  
&emsp;&emsp;在```initMediaPlayer()```方法中，通过```getAssets()```方法获取```AssetManager```实例，```AssetManager```可用于读取```assets```目录下的任何资源。调用```openFd()```方法将音频文件句柄打开，再调用```setDataSource()```和```prepare()```为```MediaPlayer```做好播放前的准备。  
```kotlin
class PlayAudioTest : AppCompatActivity() {

    private val mediaPlayer = MediaPlayer()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_play_audio_test)
        
        initMediaPlayer()
        Btn_play.setOnClickListener {
            if (!mediaPlayer.isPlaying){
                mediaPlayer.start()
            }
        }

        Btn_pause.setOnClickListener {
            if(mediaPlayer.isPlaying){
                mediaPlayer.pause()
            }
        }

        Btn_stop.setOnClickListener {
            if(mediaPlayer.isPlaying){
                mediaPlayer.reset()
                initMediaPlayer()
            }
        }
    }

    private fun initMediaPlayer(){
        val assetManager = assets
        val fd = assetManager.openFd("ずっと真夜中でいいのに。-眩しいDNAだけ.mp3")
        mediaPlayer.setDataSource(fd.fileDescriptor,fd.startOffset,fd.length)
        mediaPlayer.prepare()
    }

    override fun onDestroy() {
        super.onDestroy()
        mediaPlayer.stop()
        mediaPlayer.release()
    }

}
```

***

### 2. 播放视频
