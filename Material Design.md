# Material Design
## 一、Toolbar
&emsp;&emsp;任何一个新建的项目，默认都会显示 **ActionBar** ，即顶部的标题栏，**ActionBar** 是根据项目中指定的主题来显示的。  
&emsp;&emsp;  
打开```AndroidManifest.xml```文件  
&emsp;&emsp;这里使用了```android:theme```属性指定了一个 **AppTheme** 的主题。  
```xml
<application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
      ......
```

点击 **AppTheme** 打开了```res/values/style.xml```文件  
&emsp;&emsp;这里定义了一个叫 **AppTheme** 的主题，然后指定它的 **parent** 主题是 **Theme.AppCompat.Light.DarkActionBar** ，这个**DarkActionBar** 是一个深色的 **ActionBar** 主题。  
```xml
<resources>
    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>

</resources>
```

&emsp;&emsp;要使用 **Toolbar** 来代替 **ActionBar** ，就需要指定一个不带 **ActionBar** 的主题，通常有```Theme.AppCompat.NoActionBar```和```Theme.AppCompat.Light.NoActionBar```这两种主题可选。  
&emsp;&emsp;```Theme.AppCompat.NoActionBar```表示深色主题，会将界面的主体颜色设成深色，陪衬颜色设为浅色。  
&emsp;&emsp;```Theme.AppCompat.Light.NoActionBar```表示浅色主题，会将主体颜色设为浅色，陪衬颜色设为深色。  
```xml
<resources>
    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>
</resources>
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Material%20Design/Toolbar/toolbar_1.PNG)

&emsp;&emsp;注意下 **AppTheme** 中的属性重写，这里重写了```colorPrimary```、```colorPrimaryDark```、```colorAccent```这3个属性的颜色，除此之外，还可以通过```textColorPrimary```、```windowBackground```和```navigationBarColor```等属性控制更多位置的颜色，可参考下图。  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Material%20Design/Toolbar/toolbar_2.PNG)

***

使用 **Toolbar** 来代替 **ActionBar**  
```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".ToolBarTest">

    <androidx.appcompat.widget.Toolbar
        android:id="@+id/Tb_1"
        android:background="@color/colorPrimary"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
        app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"/>

</FrameLayout>
```
&emsp;&emsp;```xmlns:app```指定了一个新的命名空间，```xmlns```即```XML Namespaces```的缩写。由于每个布局文件都会使用```xmlns:android```来指定一个命名空间，才能一直使用```android:id```、```android:layout_width```等写法。这里指定```xmlns:app```，也就是说现在可以使用```app:attribute```这样的写法。指定```xmlns:app```的原因是由于许多 **Material** 属性是在新系统中增加的，老系统并不存在，那么为了能够兼容老系统，就不能使用```android:attribure```这样的写法了，而是应该使用```app:attribute```。  
&emsp;&emsp;定义了一个 **Toolbar** 控件，这个控件由appcompat库提供，高度设为```actionBar```的高度。由于上面在```style.xml```中将程序的主题指定成了浅色主题，因此 **Toolbar** 现在也是浅色主题，**Toolbar** 上面的各种元素就会自动使用深色系，从而和主体颜色区分开。之前使用 **ActionBar** 时文字是白色的，现在变成黑色的会很难看。为了让 **Toolbar** 单独使用深色主题，这里使用```android:theme```属性，将 **Toolbar** 的主题指定为```ThemeOverlay.AppCompat.Dark.ActionBar```。这样指定之后，如果 **Toolbar** 中有菜单按钮，那么弹出的菜单项也会变成深色主题，又会变得难看，因此使用```app:popupTheme```属性，单独将弹出的菜单项指定成浅色主题。  
&emsp;&emsp;  
修改```Activity```文件  
```kotlin
class ToolBarTest : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_tool_bar_test)
        setSupportActionBar(Tb_1)
    }
}
```
&emsp;&emsp;调用```setSupportActionBar()```方法并将 **Toolbar** 的实例传入，这样就做到既使用了 **Toolbar** ，又让它的外观与功能和 **ActionBar** 一致。  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Material%20Design/Toolbar/toolbar_theme_1.PNG)

如果去掉```android:theme```属性：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Material%20Design/Toolbar/toolbar_theme_2.PNG)

* 修改标题栏的文字内容  
在```AndroidManifest.xml```文件中  
```xml
<application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity
            android:name=".ToolBarTest"
            android:label="ToolbarPage" />
```
&emsp;&emsp;```android:label```属性用于指定在 **Toolbar** 中显示的文字内容，如果没指定，会默认使用```application```中指定的```label```内容，也就是应用名称。  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Material%20Design/Toolbar/toolbar_text.PNG)

* 添加一些```action```按钮  
&emsp;&emsp;右键res目录 -> New -> Directory，创建一个menu文件夹，右键menu文件夹 -> New -> Menu resource file，创建一个```toolbar.xml```文件。  
```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    >
    <item
        android:id="@+id/backup"
        android:icon="@drawable/ic_backup"
        android:title="Backup"
        app:showAsAction="always"/>

    <item
        android:id="@+id/delete"
        android:icon="@drawable/ic_delete"
        android:title="Delete"
        app:showAsAction="ifRoom"/>

    <item
        android:id="@+id/settings"
        android:icon="@drawable/ic_settings"
        android:title="Settings"
        app:showAsAction="never"/>

</menu>
```
&emsp;&emsp;通过```<item>```标签定义action按钮，```android:id```指定按钮的id,```android:icon```指定按钮图标，```android:title```指定按钮文字。  
&emsp;&emsp;```app:showAsAction```指定按钮的显示位置。**Toolbar** 的action按钮只会显示图标，菜单中的action按钮只会显示文字。  
&emsp;&emsp;```always```表示永远显示在 **Toolbar** 中，如果屏幕空间不够则不显示。  
&emsp;&emsp;```ifRoom```表示屏幕空间足够的情况下显示在 **Toolbar** 中，不够的话显示在菜单当中。  
&emsp;&emsp;```never```永远显示在菜单当中。  
&emsp;&emsp;  
修改```Activity```文件  
```kotlin
class ToolBarTest : AppCompatActivity() {
    ......

    override fun onCreateOptionsMenu(menu: Menu?): Boolean {
        menuInflater.inflate(R.menu.toolbar,menu)
        return true
    }

    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        when(item.itemId){
            R.id.backup -> Toast.makeText(this,"点击了备份", Toast.LENGTH_SHORT).show()
            R.id.delete -> Toast.makeText(this,"点击了删除", Toast.LENGTH_SHORT).show()
            R.id.settings -> Toast.makeText(this,"点击了设置", Toast.LENGTH_SHORT).show()
        }
        return true
    }
}
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Material%20Design/Toolbar/toolbar_menu.gif)

***

## 二、滑动菜单
### 1. DrawerLayout
&emsp;&emsp;```DrawerLayout```布局只允许放入两个直接子控件，第一个子控件是主屏幕显示的内容，第二个子控件是滑动菜单中显示的内容。  
```xml
<androidx.drawerlayout.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/drawerLayout"
    tools:context=".DrawerLayoutTest">

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.appcompat.widget.Toolbar
            android:id="@+id/toolBar"
            android:background="@color/colorPrimary"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"/>

    </FrameLayout>

    <TextView
        android:text="这是菜单"
        android:textSize="30sp"
        android:background="#FFF"
        android:layout_gravity = "start"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</androidx.drawerlayout.widget.DrawerLayout>
```
&emsp;&emsp;特别注意第二个子控件的```layout_gravity```属性，这个属性必须指定，它告诉```DrawerLayout```滑动菜单是在屏幕的左边还是右边。left左right右，如果指定为start，会根据系统语言进行判断。  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Material%20Design/SlideMenu/drawerLayout_1.PNG)

* 点击按钮关闭菜单  
&emsp;&emsp;调用```getSupportActionBar()```方法得到```ActionBar```的实例，```ActionBar```的具体实现由```Toolbar```完成。在```ActionBar```不为空的情况下调用```setDisplayHomeAsUpEnabled()```方法让导航按钮显示出来，调用```setHomeAsUpIndicator()```方法来设置一个导航按钮图标。```Toolbar```最左侧的按钮就叫Home按钮，默认图标是一个返回的箭头，返回上一个Activity。  
&emsp;&emsp;在```onOptionsItemSelected()```方法中对Home按钮的点击事件进行处理，Home按钮的id永远都是```android.R.id.home```。然后调用```DrawerLayout```的```openDrawer()```方法将滑动菜单展示出来，```openDrawer()```方法要求传入一个```Gravity```参数。  
```kotlin
class DrawerLayoutTest : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_drawer_layout_test)

        //绑定Toolbar
        setSupportActionBar(toolBar)

        supportActionBar?.let {
            it.setDisplayHomeAsUpEnabled(true)
            it.setHomeAsUpIndicator(R.drawable.ic_menu)
        }
    }

    override fun onCreateOptionsMenu(menu: Menu?): Boolean {
        menuInflater.inflate(R.menu.toolbar,menu)
        return true
    }

    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        when(item.itemId){
            ......
            android.R.id.home -> drawerLayout.openDrawer(GravityCompat.START)
        }
        return true
    }
}
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Material%20Design/SlideMenu/drawerLayout_2.gif)

***

### 2. NavigationView
&emsp;&emsp;```NavigationView```能将滑动菜单页面的实现简单化，它是 **Material** 库中提供的一个控件，所以需要添加依赖。  
```gradle
implementation  'com.google.android.material:material:1.1.0'
implementation  'de.hdodenhof:circleimageview:3.0.1'
```
&emsp;&emsp;第一行是 **Material** 库。  
&emsp;&emsp;第二行是一个开源项目```CircleImageView```，实现图片圆形化。  

构建过程：  
#### (1) 设置menu
```menu```用来在```NavigationView```中显示具体的菜单项。  
右键menu文件夹 -> New -> Menu resource file，创建一个```nav_menu.xml```。  
```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android">
<group android:checkableBehavior="single">
    <item
        android:id="@+id/navCall"
        android:icon="@drawable/nav_call"
        android:title="Call"/>

    <item
        android:id="@+id/navFriends"
        android:icon="@drawable/nav_friends"
        android:title="Friends"/>

    <item
        android:id="@+id/navLocation"
        android:icon="@drawable/nav_location"
        android:title="Location"/>

    <item
        android:id="@+id/navMail"
        android:icon="@drawable/nav_mail"
        android:title="Mail"/>

    <item
        android:id="@+id/navTask"
        android:icon="@drawable/nav_task"
        android:title="Tasks"/>

</group>
</menu>
```
&emsp;&emsp;```<group>```表示一个组，```checkableBehavior```指定为```single```表示组中的所有菜单项只能单选。  

#### (2) 设置headerLayout
```headerLayout```用来在```NavigationView```中显示头部布局。  
右键layout文件夹 -> New -> Layout resource file，创建一个```nav_header.xml```文件。  
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:padding="10dp"
    android:background="@color/colorPrimary"
    android:layout_height="180dp">

    <de.hdodenhof.circleimageview.CircleImageView
        android:id="@+id/iconImage"
        android:src="@drawable/nav_icon"
        android:layout_centerInParent="true"
        android:layout_width="70dp"
        android:layout_height="70dp"/>

    <TextView
        android:id="@+id/mailText"
        android:layout_alignParentBottom="true"
        android:text="aabb@gmail.com"
        android:textAllCaps="false"
        android:textColor="#fff"
        android:textSize="14sp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

    <TextView
        android:id="@+id/userText"
        android:layout_above="@+id/mailText"
        android:text="aabb"
        android:textColor="#fff"
        android:textSize="14sp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

</RelativeLayout>
```

#### (3) 修改布局文件
通过```app:menu```和```app:headerLayout```属性将上面的menu和headerLayout设置进去。  
```xml
<androidx.drawerlayout.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/drawerLayout"
    tools:context=".DrawerLayoutTest">

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.appcompat.widget.Toolbar
            android:id="@+id/toolBar"
            android:background="@color/colorPrimary"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"/>

    </FrameLayout>

    <com.google.android.material.navigation.NavigationView
        android:id="@+id/navView"
        app:menu="@menu/nav_menu"
        app:headerLayout="@layout/nav_header"
        android:layout_gravity = "start"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</androidx.drawerlayout.widget.DrawerLayout>
```

#### (4) 修改Activity，处理点击事件
```kotlin
class DrawerLayoutTest : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_drawer_layout_test)

        //绑定Toolbar
        setSupportActionBar(toolBar)

        supportActionBar?.let {
            it.setDisplayHomeAsUpEnabled(true)
            it.setHomeAsUpIndicator(R.drawable.ic_menu)
        }

        navView.setCheckedItem(R.id.navCall)
        navView.setNavigationItemSelectedListener {
            drawerLayout.closeDrawers()
            true
        }
    }
    ......
}
```
&emsp;&emsp;调用```NavigationView```的```setCheckedItem()```方法将Call菜单项设置为默认选中。调用```setNavigationItemSelectedListener()```设置一个菜单项选中事件的监听器，当用户点击了任意菜单项时，就会回调到传入的Lambda表达式中，可以在这里进行具体的逻辑处理。  
&emsp;&emsp;调用```DrawerLayout```的```closeDrawers()```方法将滑动菜单关闭，并返回true表示此事件已经被处理。  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Material%20Design/SlideMenu/drawerLayout_3.gif)

***

## 三、悬浮按钮和可交互提示
### 1. FloatingActionButton
&emsp;&emsp;默认以```colorAccent```作为按钮的颜色。```app:elevation```属性给```FloatingActionButton```指定一个高度值。高度值越大，投影范围也越大，投影效果越淡。高度值越小，投影范围越小，投影效果越浓。    
修改布局文件  
```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".FloatingAndSnackbar">

    <com.google.android.material.floatingactionbutton.FloatingActionButton
        android:id="@+id/fab"
        android:layout_margin="16dp"
        android:layout_gravity="bottom|end"
        android:src="@drawable/ic_done"
        android:elevation="8dp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

</FrameLayout>
```

修改```Activity```文件  
```kotlin
class FloatingAndSnackbar : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_floating_and_snackbar)

        fab.setOnClickListener {
            Toast.makeText(this,"点击了悬浮按钮",Toast.LENGTH_SHORT).show()
        }
    }
}
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Material%20Design/FloatingActionButton.gif)

***

### 2. Snackbar
&emsp;&emsp;```Snackbar```允许在提示中加入一个可交互的按钮，当用户点击按钮的时候，可以执行一些额外的逻辑操作。  
```kotlin
class FloatingAndSnackbar : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_floating_and_snackbar)

        fab.setOnClickListener {
            Snackbar.make(it,"删除数据",Snackbar.LENGTH_SHORT)
                .setAction("Undo"){
                    Toast.makeText(this,"数据恢复",Toast.LENGTH_SHORT).show()
                }
                .show()
        }
    }
}
```
&emsp;&emsp;调用```Snackbar```的```make()```方法来创建一个```Snackbar```对象。第一个参数传入一个```View```，```Snackbar```会使用这个```View```自动查找最外层的布局。第二个参数是```Snackbar```中显示的内容。第三个参数是展示时常。  
&emsp;&emsp;调用```setAction()```方法来设置一个动作，从而让```Snackbar```与用户交互。这里在动作按钮的点击事件里弹出一个Toast提示，最后调用```show()```方法让```Snackbar```显示出来。  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Material%20Design/snack.gif)

***

### 3. CoordinatorLayout
&emsp;&emsp;```CoordinatorLayout```是一个加强版的```FrameLayout```，可以监听其所有子控件的各种事件，并作出合理响应。  
&emsp;&emsp;上面的案例中，弹出的```Snackbar```将悬浮按钮遮挡了，使用```CoordinatorLayout```监听```Snackbar```的弹出事件，它会自动将内部的```FloatActionButton```向上偏移，从而确保不会被```Snackbar```遮挡。  
&emsp;&emsp;  
修改布局文件  
```xml
<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".FloatingAndSnackbar">

    <com.google.android.material.floatingactionbutton.FloatingActionButton
        android:id="@+id/fab"
        android:layout_margin="16dp"
        android:layout_gravity="bottom|end"
        android:src="@drawable/ic_done"
        android:elevation="8dp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Material%20Design/CoordinatorLayout.gif)

&emsp;&emsp;```Snackbar```看似不是```CoordinatorLayout```的子控件，但在```Snackbar```的```make()```方法中传入的第一个参数，是用来指定```Snackbar```基于哪个View触发。这里传入的是```FloatingActionButton```本身，而```FloatingActionButton```是```CoordinatorLayout```中的子控件，所以这个事件能被监听到。  

***

## 四、卡片式布局
### 1. MaterialCardView
&emsp;&emsp;```MaterialCardView```是一个```FrameLayout```，```app:cardBackgroundColor```属性设置卡片圆角的弧度，```android:elevation```属性设置卡片的阴影。  
```xml
<androidx.cardview.widget.CardView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:cardBackgroundColor="4dp"
        android:elevation="5dp"/>
```

实现一个水果列表效果：  
#### (1) 添加依赖
&emsp;&emsp;**Glide** 是一个开源图片加载库。  
```gradle
implementation  'androidx.recyclerview:recyclerview:1.1.0'
implementation  'com.github.bumptech.glide:glide:4.9.0'
```

#### (2) 修改布局文件
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MaterialCardViewTest">

    <androidx.coordinatorlayout.widget.CoordinatorLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.appcompat.widget.Toolbar
            android:id="@+id/Tb_1"
            android:background="@color/colorPrimary"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"/>

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/recyclerView"
            android:layout_width="match_parent"
            android:layout_height="match_parent"/>

        <com.google.android.material.floatingactionbutton.FloatingActionButton
            android:id="@+id/fab"
            android:layout_gravity="bottom|end"
            android:layout_margin="16dp"
            android:src="@drawable/ic_done"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

    </androidx.coordinatorlayout.widget.CoordinatorLayout>

</LinearLayout>
```

#### (3) 定义一个Fruit类
```kotlin
//name是水果名字，imageId是水果对应的图片id资源
class Fruit(val name : String,val imageId: Int)
```

#### (4) layout目录下新建一个fruit_item.xml
&emsp;&emsp;ImageVie中使用了一个```scaleType```属性，指定图片的缩放模式。```centerCrop```可以让图片保持原有比例填充满ImageView。  
```xml
<com.google.android.material.card.MaterialCardView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_margin="5dp"
    app:cardCornerRadius="4dp"
    android:layout_height="wrap_content">
        
    <LinearLayout
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <ImageView
            android:id="@+id/fruitImage"
            android:scaleType="centerCrop"
            android:layout_width="match_parent"
            android:layout_height="100dp"/>

        <TextView
            android:id="@+id/fruitName"
            android:layout_gravity="center_horizontal"
            android:layout_margin="5dp"
            android:textSize="16sp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

    </LinearLayout>

</com.google.android.material.card.MaterialCardView>
```

#### (5) 新建一个适配器FruitAdapter类
&emsp;&emsp;```Glide.with()```方法传入一个```Context```、```Activity```或```Fragment```参数，然后调用```load()```方法加载图片，可以是URL地址、本地路径或一个资源id，最后调用```into()```方法将图片设置到具体某个ImageView中。  
```kotlin
class FruitAdapter(private val context: Context, private val fruitList: List<Fruit>) :
    RecyclerView.Adapter<FruitAdapter.ViewHolder>() {

    inner class ViewHolder(view:View):RecyclerView.ViewHolder(view){
        val fruitImage : ImageView = view.findViewById(R.id.fruitImage)
        val fruitName : TextView = view.findViewById(R.id.fruitName)
    }
    
    override fun getItemCount(): Int {
        return  fruitList.size
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        val fruit = fruitList[position]
        holder.fruitName.text = fruit.name
        Glide.with(context).load(fruit.imageId).into(holder.fruitImage)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(context).inflate(R.layout.fruit_item,parent,false)
        return ViewHolder(view)
    }

}
```

#### (6) 修改Activity文件
```kotlin
class MaterialCardViewTest : AppCompatActivity() {

    private val fruits = mutableListOf(
        Fruit("Apple",R.drawable.apple),
        Fruit("Banana",R.drawable.banana),
        Fruit("Orange",R.drawable.orange),
        Fruit("Watermelon",R.drawable.watermelon),
        Fruit("Pear",R.drawable.pear),
        Fruit("Grape",R.drawable.grape),
        Fruit("Pineapple",R.drawable.pineapple),
        Fruit("Strawberry",R.drawable.strawberry),
        Fruit("Cherry",R.drawable.cherry),
        Fruit("Mango",R.drawable.mango)
    )

    private val fruitList = ArrayList<Fruit>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_material_card_view_test)

        setSupportActionBar(toolBar)

        initFruits()
        val layoutManager = GridLayoutManager(this,2)
        recyclerView.layoutManager = layoutManager
        val adapter = FruitAdapter(this,fruitList)
        recyclerView.adapter = adapter

    }

    private fun initFruits(){
        fruitList.clear()
        repeat(50){
            val index = (0 until fruits.size).random()
            fruitList.add(fruits[index])
        }
    }

}
```
运行后发现出错：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Material%20Design/Card/MaterialCardView_error_1.PNG)

解决方法：  
&emsp;&emsp;修改```fruit_item.xml```文件，增加```android:theme```属性，设置为 **Material** 相关的主题， ```MaterialCardView```组件要求运行在 **Material** 主题下。    
```xml
<com.google.android.material.card.MaterialCardView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:theme="@style/Theme.MaterialComponents.Light.NoActionBar"
    android:layout_margin="5dp"
    app:cardCornerRadius="4dp"
    android:layout_height="wrap_content">
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Material%20Design/Card/card_recyclerview.gif)

***  

### 2. AppBarLayout
&emsp;&emsp;在上面的案例中，**Toolbar** 被 **RecyclerView** 挡住了。**Toolbar** 和 **RecyclerView** 都是被放置在 **CoordinatorLayout** 中的，**CoordinatorLayout** 就是一个加强版的 **FrameLayout** ，**FrameLayout** 中的所有控件在不进行明确定位的情况下，默认都会摆放在布局的左上角，从而产生遮挡现象。  
&emsp;&emsp;**AppBarLayout** 实际上是一个垂直方向的 **LinearLayout**，它在内部做了很多滚动事件的封装，可以解决遮挡问题。  
&emsp;&emsp;  
将 **Toolbar** 嵌套到 **AppBarLayout** 中，然后给 **RecyclerView** 指定一个布局。  
&emsp;&emsp;使用```app:layout_behavior```属性指定一个布局行为。  
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MaterialCardViewTest">

    <androidx.coordinatorlayout.widget.CoordinatorLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <com.google.android.material.appbar.AppBarLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content">

            <androidx.appcompat.widget.Toolbar
                android:id="@+id/Tb_1"
                android:background="@color/colorPrimary"
                android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
                app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"/>

        </com.google.android.material.appbar.AppBarLayout>

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/recyclerView"
            app:layout_behavior="@string/appbar_scrolling_view_behavior"
            android:layout_width="match_parent"
            android:layout_height="match_parent"/>

        <com.google.android.material.floatingactionbutton.FloatingActionButton
            android:id="@+id/fab"
            android:layout_gravity="bottom|end"
            android:layout_margin="16dp"
            android:src="@drawable/ic_done"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

    </androidx.coordinatorlayout.widget.CoordinatorLayout>

</LinearLayout>
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Material%20Design/Card/AppBarLayout_1.gif)

**AppBarLayout** 还能实现Material Design效果，在 **Toolbar** 中添加```app:layout_scrollFlags```属性：  
&emsp;&emsp;```scroll```表示当 **RecyclerView** 向上滚动的时候，**Toolbar** 会跟着一起向上滚动并实现隐藏。  
&emsp;&emsp;```enterAlways```表示当 **RecyclerView** 向下滚动的时候， **Toolbar** 会跟着一起向下滚动并重新显示。  
&emsp;&emsp;```snap```表示当 **Toolbar** 还没有完全隐藏或显示的时候，会根据当前滚动的距离，自动选择是隐藏还是显示。  
```kotlin
<androidx.appcompat.widget.Toolbar
          android:id="@+id/Tb_1"
          android:background="@color/colorPrimary"
          android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
          app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
          android:layout_width="match_parent"
          app:layout_scrollFlags="scroll|enterAlways|snap"
          android:layout_height="?attr/actionBarSize"/>
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Material%20Design/Card/AppBarLayout_2.gif)

