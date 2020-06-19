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
