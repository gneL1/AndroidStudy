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
