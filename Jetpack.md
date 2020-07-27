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

运行效果：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Jetpack/ViewModel_2.gif)

***

## 二、Lifecycles
&emsp;&emsp;为什么需要感知 **Activity** 生命周期的情况？  
&emsp;&emsp;如果某个界面发起一条网络请求，但是当请求得到响应的时候，界面或许已经关闭了，这个时候就不应该继续对响应的结果进行处理，因此需要能够时刻感知 **Activity** 的生命周期，以便在适当的时候进行相应的逻辑控制。  
&emsp;&emsp;**Lifecycles** 可以让一个类轻松感知到 **Activity** 的生命周期，同时又不需要在 **Activity** 中编写大量的逻辑处理。  
&emsp;&emsp;  
* 新建一个```MyObserver```  
```kotlin
class MyObserver : LifecycleObserver{
    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    fun activityStart(){
        Log.d("MyObserver:","activityStart")
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    fun activityStop(){
        Log.d("MyObserver:","activityStop")
    }
}
```
&emsp;&emsp;```LifecycleObserver```是一个空方法接口，在此进行接口实现声明。在方法上使用了```@OnLifecycleEvent```注解，并传入了一种生命周期事件。  
&emsp;&emsp;生命周期有7种，```ON_CREATE```、```ON_START```、```ON_RESUME```、```ON_PAUSE```、```ON_STOP```、```ON_DESTROY```分别匹配 **Activity** 中对应的生命周期，还有一种```ON_ANY```类型表示匹配 **Activity** 的任何生命周期。上面的```activityStart()```和```activityStop```分别在 **Activity** 的```onStart()```和```onStop()```触发的时候执行。    

* 修改```LifecyclesTest```  
```kotlin
class LifecyclesTest : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_lifecycles_test)
        lifecycle.addObserver(MyObserver())
    }

}
```

&emsp;&emsp;调用```LifecycleOwner```的```getLifecycle()```方法，得到一个```Lifecycle```对象，然后调用它的```addObserver()```方法来观察```LifecycleOwner```的生命周期，再把```MyObserver```的实例传进去。  
```kotlin
ifecycleOwner.lifecycle.addObserver(MyObserver()) 
```
&emsp;&emsp;由于 **Activity** 是继承自 **AppCompatActivity** ，或者 **Fragment** 是继承自 **androidx.fragment.app.Fragment** ，他们本身就是一个```LifecycleOwner```的实例，所以可以直接调用```getLifecycle()```。  
&emsp;&emsp;  
打开页面然后退出：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Jetpack/Lifecycles.PNG)

&emsp;&emsp;想主动获取当前的生命周期状态的话，可以在```MyObserver```的构造函数中将```Lifecycle```对象传进来，通过```lifecycle.currentState```来主动获取当前的生命周期状态。  
```kotlin
class MyObserver(val lifecycle:Lifecycle) : LifecycleObserver{
    ......
}
```

&emsp;&emsp;```Lifecycle```返回的生命周期状态有5种类型：```INITIALZED```、```DESTROYED```、```CREATED```、```STARTED```、```RESUMED```，与 **Activity** 的生命周期回调所对应的关系如下图。  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Jetpack/Lifecycles_2.PNG)

***

## 三、LiveData
&emsp;&emsp; **LiveData** 是一种响应式编程组件。它可以包含任何类型的数据，并在数据发生变化的时候通知给观察者，**LiveData** 适合与 **ViewModel** 结合在一起使用。  

### 1. 基本用法
&emsp;&emsp;上面的计数器功能在单线程模式下确实可以正常工作。但如果 **ViewModel** 的内部开启了线程去执行一些耗时操作，那么在点击按钮后就立即去获取最新的数据，得到的还会是之前的数据。  
&emsp;&emsp;把 **Activity** 的实例传给 **ViewModel** ，这样 **ViewModel** 能主动对 **Activity** 进行通知，但是 **ViewModel** 的生命周期长于 **Activity** ，这样会因为 **Activity** 无法释放而造成内存泄漏。这时候就用到 **LiveData**。  

* 修改```ViewModel```  
```kotlin
class VMPageViewModel(countReserved : Int) : ViewModel() {

    val counter = MutableLiveData<Int>()

    init {
        counter.value = countReserved
    }

    fun plusOne(){
        val count = counter.value ?: 0
        counter.value = count + 1
    }

    fun clear(){
        counter.value = 0
    }

}
```
&emsp;&emsp;将```counter```变量修改成一个```MutableLiveData```对象，并指定它的泛型为Int，表示它包含的是整型数据。  
&emsp;&emsp;```MutableLiveData```是一种可变的```LiveData```，有三种读写数据的方法：  
&emsp;&emsp;```getValue()```获取```LiveData```中包含的数据。  
&emsp;&emsp;```setValue()```用于给```LiveData```设置数据，但是只能在主线程中调用。  
&emsp;&emsp;```postValue()```用于在非主线程中给```LiveData```设置数据。  
&emsp;&emsp;在```init```结构体中给```counter```设置数据，这样之前保存的计数值就可以在初始化的时候得到恢复。新增```plusOne()```和```clear()```方法，分别用于给技术加以及将计数清零。```plusOne()```先获取```counter```中包含的数据，然后给它加1，再重新设置到```counter```中。调用```LiveData```的```getValue()```方法所获取的数据是可能为空的，因此使用了```?:```操作符，当获取到的数据为空时，用0来作为默认计数。  
&emsp;&emsp;  
* 修改```Activity```  
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
            viewModel.plusOne()
        }

        Btn_2.setOnClickListener {
            viewModel.clear()
        }

        viewModel.counter.observe(this, Observer {
            count -> infoText.text = count.toString()
        })

    }

    override fun onPause() {
        super.onPause()
        sp.edit {
            putInt("count_reserved",viewModel.counter.value ?: 0)
        }
    }

}
```
&emsp;&emsp;调用```viewModel.counter```的```observe()```方法来观察数据的变化。```counter```变量是一个```LiveData```对象，任何```LiveData```对象都可以调用它的```observe()```方法来观察数据的变化。  
&emsp;&emsp;```observe()```方法接收两个参数，第一个参数是一个```LifecycleOwner```对象，**Activity** 本身就是一个```LifecycleOwner```对象，因此直接传入```this```。第二个参数是一个```Observer```接口，当```counter```中包含的数据发生变化时，就会回调到这里，在此处将最新的计数更新到界面上。  
&emsp;&emsp;  
&emsp;&emsp;```LiveData```的```observe()```方法是一个 **Java** 方法，```observe()```的其中一个参数是```Observer```接口，这是一个单抽象方法接口，只有一个待实现的```onChanged()```方法。```observe()```的另一个参数```LifecycleOwner```也是一个单抽象方法接口。当一个 **Java** 方法同时接收两个单抽象方法接口参数时，要么同时使用函数式API的写法，要么都不使用函数式API的写法。  
&emsp;&emsp;```lifecycle-livedata-ktx```是一个专门为 **Kotlin** 语言设计的库，在2.2.0版本中加入了对```observe()```方法的语法扩展。  
&emsp;&emsp;  
添加依赖：  
```gradle
implementation  'androidx.lifecycle:lifecycle-livedata-ktx:2.2.0'
```
使用如下语法结构的```observe()```方法：  
```kotlin
viewModel.counter.observe(this){
        count -> infoText.text = count.toString()
}
```

* 优化```ViewModel```  
&emsp;&emsp;上面的写法还是有问题，在于将```counter```这个可变的```LiveData```暴露给了外部，这样即使是在```ViewModel```的外面也可以给```counter```设置数据，从而破坏了```ViewModel```数据的封装性。  
&emsp;&emsp;正确的做法是永远只暴露不可变的```LiveData```给外部，这样在非```ViewModel```中就只能观察```LiveData```的数据变化，而不能给```LiveData```设置数据。  
```kotlin
class VMPageViewModel(countReserved : Int) : ViewModel() {

    val counter:LiveData<Int> get() = _counter

    private val _counter = MutableLiveData<Int>()

    init {
        _counter.value = countReserved
    }

    fun plusOne(){
        val count = _counter.value ?: 0
        _counter.value = count + 1
    }

    fun clear(){
        _counter.value = 0
    }

}
```
&emsp;&emsp;将原来的```counter```变量改为```_counter```变量，并给它加上```private```修饰符，这样```_counter```变量对于外部就是不可见的了。然后又新定义了一个```counter```变量，将它的类型声明为不可变的```LiveData```，并在它的```get()```属性方法中返回```_counter```变量。  
&emsp;&emsp;当外部调用```counter```变量时，实际上获得的就是```_counter```的实例，但是无法给```counter```设置数据，从而保证了```ViewModel```的数据封装性。  
&emsp;&emsp;```LiveData```里的```setValue```：  
```kotlin
@MainThread
protected void setValue(T value) {
    assertMainThread("setValue");
    mVersion++;
    mData = value;
    dispatchingValue(null);
}
```

***

### 2. map和switchMap
**```map()```**  
&emsp;&emsp;将实际包含数据的```LiveData```和仅用于观察的```LiveData```进行转换。  
&emsp;&emsp;比如有一个```User```类，```User```中包含用户的姓名和年龄：  
```kotlin
data class User(val firstName : String, val lastName : String, var age : Int)
```
&emsp;&emsp;在```ViewModel```中。  
```kotlin
class VMPageViewModel(countReserved : Int) : ViewModel() {

    private val userLiveData = MutableLiveData<User>()

    val userName : LiveData<String> = Transformations.map(userLiveData){
        user -> "${user.firstName} ${user.lastName}"
    }
    ......
}    
```
&emsp;&emsp;调用了```Transformations```的```map()```方法来对```LiveData```的数据类型进行转换。```map()```方法接收两个参数，第一个参数是原始的```LiveData```对象。第二个参数是一个转换函数，在转换函数中编写具体的逻辑，这里将```User```对象转换成一个只包含用户姓名的字符串。  
&emsp;&emsp;将```userLiveData```声明称```private```，来保证数据的封装性。外部使用的时候只要观察```userName```这个```LiveData```。当```userLiveData```的数据发生变化时，```map()```方法会监听到变化并执行转换函数中的逻辑，然后将转换之后的数据通知给```userName```的观察者。  
&emsp;&emsp;  
**```switchMap()```**  
&emsp;&emsp;如果```ViewModel```中的某个```LiveData```对象时调用另外的方法获取的，那么就可以借助```switchMap()```方法，将这个```LiveData```对象转换成另外一个可观察的```LiveData```对象。  

* 新建一个```Repository```单例类  
```kotlin
object Repository {
    fun getUser(userId : String) : LiveData<User>{
        val liveData = MutableLiveData<User>()
        liveData.value = User(userId,userId,0)
        return liveData
    }
}
```
&emsp;&emsp;```getUser()```方法返回的是一个包含```User```数据的```LiveData```对象，每次调用```getUser()```方法都会返回一个新的```LiveData```实例。  

* 修改```ViewModel```  
```kotlin
class VMPageViewModel(countReserved : Int) : ViewModel() {

    private val userIdLiveData = MutableLiveData<String>()

    val user : LiveData<User> = Transformations.switchMap(userIdLiveData){
        userId -> Repository.getUser(userId)
    }

    fun getUser(userId:String){
        userIdLiveData.value = userId
    }
    ......

}
```
&emsp;&emsp;定义了一个新的```userIdLiveData```对象，用来观察```userId```的数据变化，然后调用```Transformations```的```switchMap()```方法，用来对另一个可观察的```LiveData```对象进行转换。  
&emsp;&emsp;```switchMap()```接收两个参数，第一个参数传入新增的```userIdLiveData```，```switchMap()```方法会对它进行观察。第二个参数是一个转换函数，必须在这个转换函数中返回一个```LiveData```对象，```switchMap()```的工作原理就是将转换函数中返回的```LiveData```对象转换成另一个可观察的```LiveData```对象。在转换函数中调用```Repository```的```getUser()```方法来得到```LiveData```对象，并将它返回。  
&emsp;&emsp;```switchMap()```的整个工作流程是当外部调用```ViewModel```中的```getUser()```方法来获取用户数据时，并不会发起任何请求或者函数调用，只会将传入的```userId```值设置到```userIdLiveData```中。一旦```userIdLiveData```的数据发生变化，那么观察```userIdLiveData```的```switchMap()```方法就会执行，并且调用编写的转换函数。然后在转换函数中调用```Repository.getUser()```方法获取真正的用户数据。```switchMap()```方法会将```Repository.getUser()```返回的```LiveData```对象转换成一个可观察的```LiveData```对象，对于 **Activity** 而言，只需要去观察这个```LiveData```。  

* 修改```Activity```  
```kotlin
class VMPage : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        ......
        Btn_3.setOnClickListener {
            val userId = (0..10000).random().toString()
            viewModel.getUser(userId)
        }
        viewModel.user.observe(this, Observer {
            user -> infoText.text = user.firstName
        })
    }    
}
```
&emsp;&emsp;在按钮中使用随机函数生成了一个```userId```，然后调用```ViewModel```的```getUser()```方法来获取用户数据。当数据获取完成后，可观察```LiveData```对象的```observe()```方法就会得到通知，在此处将获取的用户名显示到界面上。  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Jetpack/switchMap.gif)  
&emsp;&emsp;  
&emsp;&emsp;另外注意，以下写法是行不通的：  
```kotlin
class VMPageViewModel(countReserved : Int) : ViewModel() {
    ...
    fun getUser(userId:String) : LiveData<User>{
        return Repository.getUser(userId)
    }

}

viewModel.getUser(userId).observe(this){
    user ->
    ...
}
```
&emsp;&emsp;这里每次调用```getUser()```方法返回的都是一个新的```LiveData```实例，而```ViewModel```会一直观察老的```LiveData```实例，从而无法观察到数据的变化。  
&emsp;&emsp;  

* 当```ViewModel```中某个获取数据的方法没有参数  
```kotlin
class VMPageViewModel(countReserved : Int) : ViewModel() {
    
    private val refreshLiveData = MutableLiveData<Any?>()
    
    val refreshResult = Transformations.switchMap(refreshLiveData){
        Repository.refresh()
    }
    
    fun refresh(){
        refreshLiveData.value = refreshLiveData.value
    }
    ......
}    
```
&emsp;&emsp;定义了一个不带参数的```refresh()```方法，对应地定义了一个```refreshLiveData```，不需要指定具体包含的数据类型，将```LiveData```的泛型指定成```Any?```。  
&emsp;&emsp;在```refresh()```方法中，将```refreshLiveData```原有的数据取出来(默认为空)，再重新设置到```refreshLiveData```当中，这样就能触发一次数据变化。```LiveData```内部不会判断即将设置的数据和原有数据是否相同，只要调用了```setValue()```或```postValue()```方法，就一定会触发数据变化事件。  
&emsp;&emsp;在 **Activity** 中观察```refreshResult```这个```LiveData```对象。只要调用了```refresh()```方法，观察者的回调函数就能够得到最新的数据。  
&emsp;&emsp;```LiveData```在内部使用了```Lifecycles```组件来自我感知生命周期的变化，从而可以在 **Activity** 销毁的时候及时释放引用，避免产生内存泄漏的问题。  
&emsp;&emsp;由于要减少性能消耗，当 **Activity** 处于不可见状态时(手机息屏、被其他 **Activity** 遮挡)，如果```LiveData```中的数据发生了变化，是不会通知给观察者的。只有当 **Activity** 重新恢复可见状态时，才会将数据通知给观察者。  
&emsp;&emsp;如果在 **Activity** 处于不可见状态时，```LiveData```发生了多次数据变化，当 **Activity** 恢复可见状态时，只有最新的那份数据才会通知给观察者，前面的数据在这种情况下相当于过期了。会被直接丢弃。  

***

## 四、Room
&emsp;&emsp;**ORM**(Object Relational Mapping)，对象关系映射。使用的编程语言时面向对象语言，使用的数据库时关系型数据库，将面向对象语言和面向关系的数据库之间建立一种映射关系，就是 **ORM** 。  
### 1. 使用Room进行增删改查
&emsp;&emsp;**Room** 主要有 **Entity**、**Dao**、**Database** 3部分组成。  
&emsp;&emsp;**Entity** ：用于定义封装实际数据的实体类，每个实体类都会在数据库中有一张对应的表，并且表中的列是根据实体类中的字段自动生成的。  
&emsp;&emsp;**Dao** ：**Dao** 是数据访问对象的意思，通常会在这里对数据库的各项操作进行封装，在实际编程时，逻辑层就不需要和底层数据库打交道了，直接和 **Dao** 层进行交互即可。  
&emsp;&emsp;**Database** ：用于定义数据库中的关键信息，包括数据库的版本号、包含哪些实体类以及提供 **Dao** 层的访问实例。  

* 添加依赖  
在```app/build.gradle```文件中  
```gradle
apply plugin: 'kotlin-kapt'

......

implementation  'androidx.room:room-runtime:2.2.5'
kapt  "androidx.room:room-compiler:2.2.5"
```
&emsp;&emsp;新增一个```kotlin-kapt```插件，同时在dependencies闭包中添加了两个 **Room** 的依赖库。**Room** 会根据项目中生成的注解来动态生成代码，因此这里一定要使用```kapt```引入 **Room**的编译时注解库，而启用编译时注解功能一定要先添加```kotlin-kapt```插件。```kapt```只能在 **Kotlin** 项目中使用，如果是 **Java** 项目的话，使用```annotationProcessor```。  

* **Entity**  
给每个实体类都添加一个```id```字段，并将这个字段设为主键  
```kotlin
@Entity
data class User(var firstName:String,var lastName:String,var age:Int) {
    @PrimaryKey(autoGenerate = true)
    var id:Long = 0
}
```
&emsp;&emsp;在```User```类名上使用了```@Entity```注解，将它声明成了一个实体类。在```User```类中添加了一个```id```字段，并使用```@PrimaryKey```注解将它设为了主键，再把```autoGenerate```参数指定为```true```，使得主键的值是自动生成的。  

* **Dao**  
新建一个```UserDao```接口，必须使用接口  
```kotlin
@Dao
interface UserDao {

    @Insert
    fun insertUser(user: User):Long

    @Update
    fun updateUser(newUser: User)

    @Query("select * from User")
    fun loadAllUsers() : List<User>

    @Query("select * from User where age > :age")
    fun loadUserOlderThan(age:Int):List<User>

    @Delete
    fun deleteUser(user: User)

    @Query("delete from User where lastName = :lastName")
    fun deleteUserByLastName(lastName:String):Int
}
```
&emsp;&emsp;```UserDao```接口的上面使用了一个```@Dao```注解，这样 **Room** 才能将它识别成 **Dao**。  
&emsp;&emsp;```insertUser()```方法上面使用了```@Insert```注解，表示会将参数传入的```User```对象插入到数据库中，插入完成后还会将自动生成的主键```id```值返回。  
&emsp;&emsp;```@Update```注解会将参数传入的```User```对象更新到数据库中。  
&emsp;&emsp;```@Delete```注解会将参数传入的```User```对象从数据库中删除。  
&emsp;&emsp;如果想要从数据库中查询数据，或者使用非实体类参数来增删数据，那么就必须编写```SQL```语句，因为如果只使用```@Query```注解，**Room** 无法知道用户想要查询哪些数据。通过```:```符号还可以将方法中传入的参数指定到```SQL```语句中。使用非实体类参数来增删改数据，此时不能使用```@Insert```、```@Delete```、```@Update```注解，而是都要使用```@Query```注解，参考```deleteUserByLastName()```方法。  

* **Database**  
&emsp;&emsp;新建一个```AppDatabase.kt```文件，定义3个部分：数据库的版本号、包含哪些实体、提供 **Dao** 层的访问实例。  
```kotlin
@Database(version = 1,entities = [User::class])
abstract class AppDatabase : RoomDatabase(){

    abstract fun userDao():UserDao

    companion object{
        private var instance:AppDatabase? = null

        @Synchronized
        fun getDatabase(context: Context):AppDatabase{
            instance?.let {
                return it
            }
            return Room.databaseBuilder(
                context.applicationContext,AppDatabase::class.java,"app_database"
            ).build().apply {
                instance = this
            }
        }
    }
}
```
&emsp;&emsp;```@Database```注解中声明了数据库的版本号以及包含哪些实体类，多个实体类之间用逗号隔开。  
&emsp;&emsp;```AppDatabase```类必须继承自```RoomDatabase```类，并且一定要使用```abstract```关键字将它声明成抽象类，然后提供相应的抽象方法，用于获取 **Dao** 的实例，只进行方法声明，具体的实现由 **Room** 在底层自动完成。  
&emsp;&emsp;在```companion object```结构体中编写一个单例模式，全局只存在一份```AppDatabase```实例。定义```instance```变量来缓存```AppDatabase```的实例，然后在```getDatabase()```方法中判断，如果```instance```变量不为空就直接返回，否则就调用```Room.databaseBuilder()```方法来构建一个```Appdatabase```的实例。  
&emsp;&emsp;```databaseBuilder()```方法接收3个参数，第一个参数一定要使用```applicationContext```，而不能使用普通的```context```，否则容易出现内存泄露的问题。第二个参数是```AppDatabase```的 **Class** 类型，第三个参数是数据库名。最后调用```build()```方法完成构建，并将创建出来的实例赋值给```instance```变量，然后返回当前实例。  

* 修改布局文件和 **Activity**  
&emsp;&emsp;由于数据库操作属于耗时操作， **Room** 默认是不允许在主线程中进行数据库操作的，将增删改查的功能都放到子线程中。  
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".RoomTest">

    <Button
        android:id="@+id/Btn_add"
        android:textAllCaps="false"
        android:text="添加"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <Button
        android:id="@+id/Btn_update"
        android:textAllCaps="false"
        android:text="修改"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <Button
        android:id="@+id/Btn_delete"
        android:textAllCaps="false"
        android:text="删除"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <Button
        android:id="@+id/Btn_query"
        android:textAllCaps="false"
        android:text="查询"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

</LinearLayout>
```

```kotlin
class RoomTest : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_room_test)

        //获取UserDao的实例，创建两个User对象
        val userDao = AppDatabase.getDatabase(this).userDao()
        val user1 = User("Tom","Brady",40)
        val user2 = User("Tom","Hanks",63)

        //将insertUser()方法返回的主键id赋值给原来的User对象
        //这么做是因为使用@Update和@Delete注解去更新和删除数据时都是基于id来操作的
        Btn_add.setOnClickListener {
            thread {
                user1.id = userDao.insertUser(user1)
                user2.id = userDao.insertUser(user2)
            }
        }

        Btn_update.setOnClickListener {
            thread {
                user1.age = 42
                userDao.updateUser(user1)
            }
        }

        Btn_delete.setOnClickListener {
            thread {
                userDao.deleteUserByLastName("Hanks")
            }
        }

        Btn_query.setOnClickListener {
            thread {
                for (user in userDao.loadAllUsers()){
                    Log.d("RoomTest",user.toString())
                }
            }
        }
    }
}
```
增加后查询：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Jetpack/Room_add.PNG)

修改后查询：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Jetpack/Room_update.PNG)

删除后查询：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Jetpack/Room_delete.PNG)

***

### 2. Room的数据库升级
如果打算在数据库中添加一张表：  
1. 创建一个```Book```实体类  
```kotlin
@Entity
data  class Book(var name:String,var pages:Int) {
    @PrimaryKey(autoGenerate = true)
    var id:Long = 0
}
```
&emsp;&emsp;```Book```类包含主键```id```、书名、页数这几个字段。  


2.创建```BookDao```接口  
```kotlin
@Dao
interface BookDao {

    @Insert
    fun insertBook(book:Book):Long

    @Query("select * from Book")
    fun loadAllBooks():List<Book>
}
```

3. 修改```AppDatabase```  
```kotlin
@Database(version = 2,entities = [User::class,Book::class])
abstract class AppDatabase : RoomDatabase(){

    abstract fun userDao():UserDao

    abstract fun bookDao():BookDao

    companion object{

        val MIGRATION_1_2 = object : Migration(1,2){
            override fun migrate(database: SupportSQLiteDatabase) {
                database.execSQL(
                    "create table Book (id integer primary key autoincrement not null,name text not null,pages integer not null)"
                )
            }
        }

        private var instance:AppDatabase? = null

        @Synchronized
        fun getDatabase(context: Context):AppDatabase{
            instance?.let {
                return it
            }
            return Room.databaseBuilder(
                context.applicationContext,AppDatabase::class.java,"app_database"
            ).addMigrations(MIGRATION_1_2).build().apply {
                instance = this
            }
        }
    }
}
```
&emsp;&emsp;在```@Database```注解中，将版本号升级成了2，并将```Book```类添加到了实体类声明中，然后提供了一个```bookDao()```方法用于获取```BookDao```实例。  
&emsp;&emsp;在```companion object```结构体中，实现了一个```Migration```的匿名类，并传入了1和2这两个参数，表示当数据库版本从1升级到2的时候就执行这个匿名类中的升级逻辑。由于要新增一张```Book```表，所以需要在```migrate()```方法中编写相应的建表语句。```Book```表的建表语句必须和```Book```实体类中声明的结构完全一致，否则 **Room** 就会抛出异常。  
&emsp;&emsp;在构建```AppDatabase```实例的时候，加入一个```addMigrations()```方法，并把```MIGRATION_1_2```传入。当进行任何数据库操作时，**Room** 就会自动根据当前数据库的版本号执行这些升级逻辑，从而让数据库始终保证是最新版本。  
&emsp;&emsp;  
&emsp;&emsp;  
&emsp;&emsp;如果是向现有表中添加新的列，只需要使用```alter```语句修改表结构就可以了。  
1. 修改```Book```实体类，添加一个作者字段  
```kotlin
@Entity
data  class Book(var name:String,var pages:Int,var author:String) {
    @PrimaryKey(autoGenerate = true)
    var id:Long = 0
}
```

2. 修改```AppDatabase```  
```kotlin
@Database(version = 3,entities = [User::class,Book::class])
abstract class AppDatabase : RoomDatabase(){
    ......
    companion object{
        ......
        val MIGRATION_2_3 = object : Migration(2,3){
            override fun migrate(database: SupportSQLiteDatabase) {
                database.execSQL("alter table Book add column author text not null default 'unknown'")
            }

        }

        private var instance:AppDatabase? = null

        @Synchronized
        fun getDatabase(context: Context):AppDatabase{
            instance?.let {
                return it
            }
            return Room.databaseBuilder(
                context.applicationContext,AppDatabase::class.java,"app_database"
            ).addMigrations(MIGRATION_1_2, MIGRATION_2_3).build().apply {
                instance = this
            }
        }
    }
}
```
&emsp;&emsp;先将版本号升级成3，然后编写一个```MIGRATION_2_3```的升级逻辑并添加到```addMigrations()```方法中。如果```SQL```语句写错了，程序运行之后在首次操作数据库的时候就会直接触发崩溃，并给出具体错误原因。  

***

## 五、WorkManager
&emsp;&emsp;**WorkManager** 是一个处理定时任务的工具，可以保证即使在应用退出甚至手机重启的情况下，之前注册的任务仍然将会得到执行。**WorkManager** 适合用于执行一些定期和服务器进行交互的任务，比如周期性地同步数据等待。  
&emsp;&emsp;使用 **WorkManager** 注册的周期性任务不能保证一定会准时执行，可能会将触发事件临近的几个任务放在一起执行，能减少CPU被唤醒的次数，节约电池。  

### 1. 基本用法
在```app/build.gradle```文件中添加依赖  
```gradle
implementation  'androidx.work:work-runtime:2.3.4'
```

**WorkManager```的基本用法分为3步：  
* (1) 定义一个后台任务，并实现具体的任务逻辑；  
* (2) 配置该后台任务的运行条件和约束信息，并构建后台任务请求；  
* (3) 将该后台任务请求传入 **WorkManager** 的```enqueue()```方法中，系统会在合适的时间运行。  

**1. 定义一个后台任务，创建一个```SimpleWorker```类**  
```kotlin
class SimpleWorker(context:Context,params:WorkerParameters) : Worker(context,params){
    override fun doWork(): Result {
        Log.d("SimpleWorker","do work in SimpleWorker")
        return Result.success()
    }
}
```
&emsp;&emsp;每一个后台任务都必须继承自```Worker```类，并调用它唯一的构造函数，然后重写父类中的```doWork()```方法，在这个方法中编写具体的后台任务。  
&emsp;&emsp;```doWork()```方法不会运行在主线程当中，在此可执行耗时逻辑。```doWork()```方法要求返回一个```Result```对象，用于表示人物的运行结果。成功返回```Result.success()```，失败返回```Result.failure()```。还有一个```Result.retry()```方法，也代表失败，但是可以结合```WorkRequest.Build```的```setBackoffCriteria()```方法来重新执行任务。  

**2. 配置后台任务的运行条件和约束信息**  
```kotlin
val request = OneTimeWorkRequest.Builder(SimpleWorker::class.java).build()
```
&emsp;&emsp;把之前创建的后台任务所对应的 **Class** 对象传入```OneTimeWorkRequest.Builder```的构造函数中，调用```build()```方法完成构建。  
&emsp;&emsp;```OneTimeWorkRequest.Builder```是```WorkRequest.Builder```的子类，用于构建单次运行的后台任务请求。```WorkRequest.Builder```还有另一个子类```PeriodicWorkRequest.Builder```，可用于构建周期性运行的后台任务请求。但是为了降低设备性能消耗，```PeriodicWorkRequest.Builder```构造函数中传入的运行周期间隔不能短于15分钟。  
```kotlin
val request1 = PeriodicWorkRequest.
    Builder(SimpleWorker::class.java,15,TimeUnit.MINUTES).build()
```

**3. 将构建出的后台任务请求传入```WorkManager```的```enqueue()```方法中**  
```kotlin
WorkManager.getInstance(this).enqueue(request)
```

使用：  
* 修改布局页面  
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".WorkManagerTest">

    <Button
        android:id="@+id/Btn_doWork"
        android:textAllCaps="false"
        android:text="Do Work"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

</LinearLayout>
```

* 修改```Activity```  
```kotlin
class WorkManagerTest : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_work_manager_test)

        Btn_doWork.setOnClickListener {
            val request = OneTimeWorkRequest.Builder(SimpleWorker::class.java).build()
            WorkManager.getInstance(this).enqueue(request)
        }
    }
}
```
&emsp;&emsp;后台任务的具体运行时间是由我们所指定的约束以及系统自身的一些优化所决定的，这里没有指定任何约束，因此后台任务基本会在点击按钮之后立刻运行。  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Jetpack/WorkManager_1.PNG)

***

### 2. 处理复杂的任务
* 让后台任务在指定的延迟时间后运行，使用```setInitialDelay()```方法：    
```kotlin
val request = OneTimeWorkRequest.Builder(SimpleWorker::class.java)
                .setInitialDelay(5,TimeUnit.SECONDS)
                .build()
```

* 给后台任务请求添加标签：  
```kotlin
val request = OneTimeWorkRequest.Builder(SimpleWorker::class.java)
                .addTag("simple")
                .build()
```

添加了标签后，可以通过标签来取消后台任务请求  
```kotlin
WorkManager.getInstance(this).cancelAllWorkByTag("simple")
```

即使没有标签，也可以通过```id```来取消后台任务请求  
```kotlin
WorkManager.getInstance(this).cancelWorkById(request.id)
```

使用```id```只能取消单个后台任务请求，使用标签可以将同一标签下的所有后台任务请求全部取消。  

* 一次性取消所有后台任务请求：  
```kotlin
WorkManager.getInstance(this).cancelAllWork()
```

* 如果后台任务的```doWork()```方法中返回了```Result.retry()```，可以结合```setBackoffCriteria()```方法来重新执行任务：  
```kotlin
val requestRetry = OneTimeWorkRequest.Builder(SimpleWorker::class.java)
                .setBackoffCriteria(BackoffPolicy.LINEAR,10,TimeUnit.SECONDS)
                .build()
```
&emsp;&emsp;```setBackoffCriteria()```方法接收3个参数，第二个和第三个参数用于指定在多久之后重新执行任务，时间最短不能少于10秒钟。第一个参数则用于指定如果任务再次执行失败，下次重试的时间应该以什么样的形式延迟。随着失败次数的增多，下次重试的时间也进行适当的延迟，可选值```Linear```代表下次重试的时间以线性方式延迟，```EXPONENTIAL```代表下次重拾事件以指数的方式延迟。  

* 对后台的运行结果进行监听：  
```kotlin
WorkManager.getInstance(this).getWorkInfoByIdLiveData(request.id)
            .observe(this){
                workInfo ->
                if(workInfo.state == WorkInfo.State.SUCCEEDED) {
                    Log.d("WorkManagerTest","do work success")
                }else if(workInfo.state == WorkInfo.State.FAILED){
                    Log.d("WorkManagerTest","do work failed")
                }
            }
```
&emsp;&emsp;调用了```getWorkInfoByIdLiveData()```方法，传入后台任务请求的```id```，会返回一个```LiveData```对象。然后调用```LiveData```对象的```observe()```方法来观察数据变化，以此监听后台任务的运行结果。  
&emsp;&emsp;也可以调用```getWorkInfoByTagLiveData()```方法，监听同一标签下所有后台任务请求的运行结果。  

* 链式任务：  
&emsp;&emsp;假设定义3个独立的后台任务：同步数据、压缩数据、上传数据。想要实现先同步、再压缩、最后上传的功能，就可以借助链式任务来实现。  
```kotlin
val sync = ...
val compress = ...
val upload = ...
WorkManager.getInstance(this)
            .beginWith(sync)
            .then(compress)
            .then(upload)         
            .enqueue()
```
&emsp;&emsp;```beginWith()```方法用于开启一个链式任务，用```then()```方法来连接。```WorkManager```要求必须在前一个后台任务运行成功之后，下一个后台任务才会运行。如果某个后台任务运行失败，或者被取消了，那么接下来的后台任务都得不到运行。  
