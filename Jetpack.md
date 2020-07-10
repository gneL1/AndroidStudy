# Jetpack
## 一、ViewModel
&emsp;&emsp;**ViewModel** 抓门用来存放与界面相关的数据，界面上能看得到的数据，相关变量都应该存放在 **ViewModel** 中，而不是 **Activity** 中，这样可以在一定程度上减少 **Activity** 中的逻辑。  
&emsp;&emsp;当手机发生横竖屏旋转时， **Activity** 会被重建，同时存放在 **Activity** 中的数据也会丢失。而 **ViewModel** 的生命周期和 **Activity** 不同，它可以保证在手机屏幕发生旋转的时候不会被重新创建，只有当 **Activity** 退出的时候才会跟着 **Activity** 一起销毁。  
&emsp;&emsp;  
**ViewModel**的生命周期：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Jetpack/ViewModel_life.png)

### 1. 基本用法
1. 在```app/build.gradle```文件中添加依赖  
```gradle
implementation  'androidx.lifecycle:lifecycle-extensions:2.2.0'
```

2. 新建一个```ViewModel```类  
&emsp;&emsp;一般每一个 **Activity** 和 **Fragment** 都创建一个对应的 **ViewModel**。```counter```变量用于计数。  
```kotlin
class VMPageViewModel : ViewModel() {
    var counter = 0
}
```

3. 修改布局文件  
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".VMPage">

    <TextView
        android:id="@+id/infoText"
        android:layout_gravity="center_horizontal"
        android:textSize="32sp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

    <Button
        android:id="@+id/Btn_1"
        android:layout_gravity="center_horizontal"
        android:text="+1s"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

</LinearLayout>
```

4. 修改```Activity```  
```kotlin
class VMPage : AppCompatActivity() {

    lateinit var viewModel: VMPageViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_v_m_page)
        viewModel = ViewModelProvider(this).get(VMPageViewModel::class.java)
        Btn_1.setOnClickListener {
            viewModel.counter++
            refreshCounter()
        }
        refreshCounter()
    }

    private fun refreshCounter(){
        infoText.text = viewModel.counter.toString()
    }
}
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Jetpack/ViewModel_1.gif)

&emsp;&emsp;通过```ViewModelProvider```来获取```ViewModel```的实例。  
&emsp;&emsp;```ViewModelProvider(<Activity 或 Fragment 实例>).get(<ViewModel>::class.java)```  
&emsp;&emsp;  
&emsp;&emsp;这样写是因为```ViewModel```有其独立的生命周期，并且生命周期要长于```Activity```。如果在```onCreate()```方法中创建```ViewModel```的实例，那么每次```onCreate()```方法执行的时候，```ViewModel```都会创建一个新的实例。这样当手机屏幕发生旋转的时候，就无法保留其中的数据了。

***

### 2. 向```ViewModel```传递参数
实现在退出程序后又重新打开的情况下，数据仍然不会丢失的效果。  
&emsp;&emsp;  

1. 修改```VMPageViewModel```  
```kotlin
class VMPageViewModel(countReserved : Int) : ViewModel() {
    var counter = countReserved
}
```

2. 新建一个```VMPageViewModelFactory```类，并让它实现```ViewModelProvider.Factory```接口  
```kotlin
class VMPageViewModelFactory(private val countReserved : Int) : ViewModelProvider.Factory{
    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
        return VMPageViewModel(countReserved) as T
    }
}
```
&emsp;&emsp;```VMPageViewModelFactory```的构造函数中也接收了一个```countReserved```参数。```ViewModelProvider.Factory```接口要求必须实现```create()```方法，这里在```create()```方法中创建了```VMPageViewModel```的实例，并将参数```countReserved```传了进去。```create()```方法的执行时机和```Activity```的声明周期无关，不会产生重复创建的问题。  

3. 修改```VMPage```  
```kotlin
class VMPage : AppCompatActivity() {

    lateinit var viewModel: VMPageViewModel

    lateinit var sp : SharedPreferences

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_v_m_page)
        
        sp = getPreferences(Context.MODE_PRIVATE)
        val countReserved = sp.getInt("count_reserved",0)
        viewModel = ViewModelProvider(this,VMPageViewModelFactory(countReserved)).get(VMPageViewModel::class.java)
        
        Btn_1.setOnClickListener {
            viewModel.counter++
            refreshCounter()
        }
        
        Btn_2.setOnClickListener {
            viewModel.counter = 0
            refreshCounter()
        }
        
        refreshCounter()
    }

    private fun refreshCounter(){
        infoText.text = viewModel.counter.toString()
    }

    override fun onPause() {
        super.onPause()
        sp.edit {
            putInt("count_reserved",viewModel.counter)
        }
    }

}
```
&emsp;&emsp;在```onCreate()```方法中，首先获取```SharedPreferences```的实例，然后读取之前保存的计数值，如果没有读到就用0作为默认值。在```ViewModelProvider```的构造方法中传入了一个```VMPageViewModelFactory```参数，将读取到的计数值传给```VMPageViewModelFactory```的构造参数。  
&emsp;&emsp;在```onPause()```方法中对当前的计数进行保存，这样可以保证不管程序是退出还是进入后台，计数都不会丢失。  
&emsp;&emsp;  
如果```sp.edit```出现报错如下图：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Jetpack/ViewModel_Error.png)  
> ```Cannot inline bytecode built with JVM target 1.8 into bytecode that is being built with JVM target 1.6. Please specify proper '-jvm-target' option```  

&emsp;&emsp;这是因为项目用jvm1.6进行构建，而库用到1.8版本导致不兼容。  
&emsp;&emsp;解决方法如下：  
* 修改```app/build.gradle```  
```gradle
android {
    ...
    compileOptions {
        sourceCompatibility = 1.8
        targetCompatibility = 1.8
    }
 
    kotlinOptions {
        jvmTarget = "1.8"
    }
}
```
* 然后 File -> Settings -> Kotlin Compiler，确保```Target JVM version```这一栏为1.8  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Jetpack/ViewModel_fix_1.png)  
