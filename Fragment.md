# Fragment
## 使用方式
### 基本用法
1. 新建两个布局文件```left_fragment.xml```和```right_fragment.xml```  
> **left_fragment_xml**  
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:text="Button"
    />

</LinearLayout>
```

> **right_fragment_xml**  
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:background="#00ff00"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:textSize="24sp"
        android:text="这是右边的Fragment"
    />

</LinearLayout>
```

2. 新建与布局对应的类继承自```Fragment```  
> 这里重写了 **onCreateView()** 方法，通过 **LayoutInflater** 的 **inflate()** 方法将布局动态加载进来  
```kotlin
class LeftFragment : Fragment() {

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        return inflater.inflate(R.layout.left_fragment, container, false)
    }

}

class RightFragment : Fragment() {

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        return inflater.inflate(R.layout.right_fragment, container, false)
    }

}
```

3. 修改```activity_main.xml```  
> 通过 **android:name** 来显示声明要添加的 **Fragment** 类名  
```xml
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal"
    tools:context=".MainActivity">

    <fragment
        android:id="@+id/leftFrag"
        android:name="com.example.fragmenttest.LeftFragment"
        android:layout_width="0dp"
        android:layout_weight="1"
        android:layout_height="match_parent"/>

    <fragment
        android:id="@+id/rightLayout"
        android:name="com.example.fragmenttest.RightFragment"
        android:layout_weight="1"
        android:layout_width="0dp"
        android:layout_height="match_parent"/>

</LinearLayout>
```
<img src="https://github.com/gneL1/AndroidStudy/blob/master/photos/Fragment/fragment_01.jpg" width="680" height="400" align=center/>

***

### 动态加载Fragment
1. 新建```another_right_fragment.xml```  
```kotlin
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#ffff00"
    android:orientation="vertical">

    <TextView
        android:layout_gravity="center_horizontal"
        android:textSize="24sp"
        android:text="这是另一个右边的Fragment"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

</LinearLayout>
```

2. 新建```AnotherRighrFragment```  
```kotlin
class AnotherRightFragment : Fragment() {
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.another_right_fragment,container,false)
    }
}
```

3. 修改```activity_main.xml```  
```xml
<fragment
        android:id="@+id/leftFrag"
        android:name="com.example.fragmenttest.LeftFragment"
        android:layout_weight="1"
        android:layout_width="0dp"
        android:layout_height="match_parent" />

    <FrameLayout
        android:id="@+id/rightLayout"
        android:layout_weight="1"
        android:layout_width="0dp"
        android:layout_height="match_parent"/>
```

4. 修改```MainActivity```  
> 这里创建一个要添加的 **Fragment** 实例  
> 调用 **getSupportFragmentManager()** 方法获取 **FragmentManager**  
> 调用 **FragmentManager** 的 **beginTransaction()** 方法开启一个事务  
> 使用事务的 **replace()** 方法向容器添加或替换 **Fragment** ,传入容器 **id** 和要添加的 **Fragment** 实例  
> 通过 **commit()** 方法提交事务
> 点击按钮后，会切换右边的**Fragment  
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        Btn_1.setOnClickListener {
            replaceFragment(AnotherRightFragment())
        }
        replaceFragment(RightFragment())
    }

    private fun replaceFragment(fragment : Fragment){
        val fragmentManager = supportFragmentManager
        val transaction = fragmentManager.beginTransaction()
        transaction.replace(R.id.rightLayout,fragment)
        transaction.commit()
    }
}
```
<img src="https://github.com/gneL1/AndroidStudy/blob/master/photos/Fragment/fragment_02.jpg" width="680" height="400" align=center/>

***

### Fragment实现返回栈
&emsp;&emsp;```addToBackStack()```可以接收一个名字用于描述返回栈的状态，一般传入```null```  
> 修改**replaceFragment()** 方法  
> 按下Back键后，如果右边存在 **Fragment** ，则优先关闭 **Fragment**  
```korlin
private fun replaceFragment(fragment : Fragment){
    val fragmentManager = supportFragmentManager
    val transaction = fragmentManager.beginTransaction()
    transaction.replace(R.id.rightLayout,fragment)

    transaction.addToBackStack(null)
    transaction.commit()
}
```
***

### Fragment与Activity的交互
1. ```Activity```调用```Fragment```的方法  
&emsp;&emsp;```FragmentManager```提供了一个类似于```findViewById()```的方法，用于从布局文件中获取```Fragment```的实例  
```kotlin
val fragment = supportFragmentManager.findFragmentById(R.id.leftFrag) as LeftFragment
```
&emsp;&emsp;可简化成：  
```kotlin
val fragment = leftFrag as LeftFragment
```

2. ```Fragment```调用```Activity```的方法  
&emsp;&emsp;```Fragment```可以通过```getActivity```获取相关联的```Activity```实例  
&emsp;&emsp;```getActivity```可能返回```null```,所以需要做判空处理  
&emsp;&emsp;当```Fragment```需要```Context```对象时，也可以使用```getActivity```，因为```Activity```本身就是一个```Context```  
```kotlin
if(activity != null){
    val mainActivity = activity as MainActivity
    mainActivity.test()
}
```

***

## Fragment的生命周期
&emsp;&emsp;```Fragment```拥有和```Activity```相同的6个生命周期方法，没有```onRestart()```。除此之外还多了```onAttach()```，```onCreateView()```，```onActivityCreated()```，```onDestoryView()```，```onDetach()```这五个。  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Fragment/Fragment_life.jpg)

> 修改 **RighrFragment**  
```kotlin
class RightFragment : Fragment() {

    companion object{
        const val TAG = "RightFragment"
    }

    override fun onAttach(context: Context) {
        super.onAttach(context)
        Log.d(TAG,"onAttach")
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        Log.d(TAG,"onCreate")
    }

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        Log.d(TAG,"onCreateView")
        return inflater.inflate(R.layout.right_fragment,container,false)
    }

    override fun onActivityCreated(savedInstanceState: Bundle?) {
        super.onActivityCreated(savedInstanceState)
        Log.d(TAG,"onActivityCreated")
    }

    override fun onStart() {
        super.onStart()
        Log.d(TAG,"onStart")
    }

    override fun onResume() {
        super.onResume()
        Log.d(TAG,"onResume")
    }

    override fun onPause() {
        super.onPause()
        Log.d(TAG,"onPause")
    }

    override fun onStop() {
        super.onStop()
        Log.d(TAG,"onStop")
    }

    override fun onDestroyView() {
        super.onDestroyView()
        Log.d(TAG,"onDestroyView")
    }

    override fun onDestroy() {
        super.onDestroy()
        Log.d(TAG,"onDestroy")
    }

    override fun onDetach() {
        super.onDetach()
        Log.d(TAG,"onDetach")
    }
}
```

1. 启动程序  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Fragment/Fragment_life_01.PNG)
  
  
2. 点击按钮打开另一个```Fragment```  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Fragment/Fragment_life_02.PNG)
  
3. 点击Back键回到```RighrFragment```  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Fragment/Fragment_life_03.PNG)
  
4. 点击Back键退出程序  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Fragment/Fragment_life_04.PNG)



