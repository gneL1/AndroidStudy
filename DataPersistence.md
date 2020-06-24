# 数据存储
## 文件存储
### 将数据存储到文件中
&emsp;&emsp;```Context```类提供了一个```openFileOutput(String name, int mode)```方法，可以用于将数据存储到指定的文件中。  
&emsp;&emsp;第一个参数```name```是文件名，在文件创建的时候使用，文件默认存储到```/data/data/包名/files/```目录下。  
&emsp;&emsp;第二个参数```mode```是文件的操作模式，```MODE_PRIVATE```表示当指定相同文件名时，所写入的内容将会覆盖原文件中的内容。```MODE_APPEND```表示如果该文件已存在，则往文件
里面追加内容，不存在就创建新文件。  
1. 创建一个```FilePersistenceTest```   
```activity_file_persistence_test.xml```  
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".FilePersistenceTest">

    <EditText
        android:id="@+id/Edit_1"
        android:hint="输入"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

</LinearLayout>
```

&emsp;&emsp;```FilePersistenceTest```  
```kotlin
class FilePersistenceTest : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_file_persistence_test)
    }

    override fun onDestroy() {
        super.onDestroy()
        val inputText = Edit_1.text.toString()
        save(inputText)
    }

    /**
     * openFileOutput返回一个FileOutputStream对象
     * 借助FileOutputStream构建一个OutputStreamWriter对象
     * 再使用OutputStreamWriter对象构建一个BufferedWriter对象
     * 通过BufferedWriter将文本内容写入文件中
     * use函数是Kotlin提供的内置函数，保证在Lambda表达式中的代码全部执行完毕后自动将外层的流关闭
     * 不需要再编写一个finally手动关闭流
     */
    private fun save(inputText : String){
        try {
            val output = openFileOutput("data", Context.MODE_PRIVATE)
            val writer = BufferedWriter(OutputStreamWriter(output))
            writer.use {
                it.write(inputText)
            }
        }catch (e : IOException){
            e.printStackTrace()
        }
    }

}
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/DataPersistence/file_persistence_1.PNG)  
&emsp;&emsp;  
&emsp;&emsp;  
&emsp;&emsp;右下角```Device File Explorer```可以查看设备文件：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/DataPersistence/file_persistence_2.PNG)  

***

### 从文件中读取数据
&emsp;&emsp;```Context```类提供了一个```openFileOutput(String name)```方法，可以用于将数据存储到指定的文件中。  
&emsp;&emsp;接受一个参数```name```，即要读取的文件名，返回一个```FileInputStream```对象。  
&emsp;&emsp;修改```FilePersistenceTest```  
```kotlin
class FilePersistenceTest : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_file_persistence_test)
        val inputText = load()
        if(inputText.isNotEmpty()){
            Edit_1.setText(inputText)
            /**
             * 这里调用setSelection将输入光标移动到文本的末尾位置
             */
            Edit_1.setSelection(inputText.length)
            Toast.makeText(this,"获取成功",Toast.LENGTH_SHORT).show()
        }
    }

    /**
     * 通过FileInputStream构建一个InputStreamReader对象
     * 通过InputStreamReader构建一个BufferedReader对象
     * 通过BufferedReader将文件中的数据一行行读取出来，拼接到StringBuilder对象当中
     * 最后将读取的内容返回
     * forEachLine是Kotlin提供的一个内置函数，将读到的每行内容回调到Lambda表达式中
     */
    private fun load() : String{
        val content = StringBuilder()
        try {
            val input = openFileInput("data")
            val reader = BufferedReader(InputStreamReader(input))
            reader.use {
                reader.forEachLine {
                    content.append(it)
                }
            }
        }catch (e : IOException){
            e.printStackTrace()
        }
        return  content.toString()
    }

}
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/DataPersistence/file_persistence_3.PNG)  

***

## SharedPreferences
&emsp;&emsp;```SharedPreferences```使用键值对的方式存储数据。  
### 存储数据到SharedPreferences
&emsp;&emsp;要使用```SharedPreferences```存储数据，需要先获取```SharedPreferences```对象。有两种方式得到```SharedPreferences```对象：  
1. ```Context```类中的```getSharedPreferences(String name, int mode)```  
&emsp;&emsp;第一个参数用于指定```SharedPreferences```文件的名称，文件不存在则会创建一个。文件存放在```/data/data/包名/shared_prefs/```目录下。  
&emsp;&emsp;第二个参数用于指定操作模式，目前只有```MODE_PRIVATE```一种模式可选。  
2. ```Activity```类中的```getPreferences(int mode)```  
&emsp;&emsp;这个方法会自动将当前```Activity```的类名作为```SharedPreferences```的文件名。  
&emsp;&emsp;  
获取到```SharedPreferences```对象后，就可以向```SharedPreferences```文件中存储数据了：  
&emsp;&emsp;调用```SharedPreferences```对象的```edit()```方法获取一个```SharedPreferences.Editor```对象。  
&emsp;&emsp;向```SharedPreferences.Editor```对象中添加数据。  
&emsp;&emsp;调用```apply()```方法将添加的数据提交。  
&emsp;&emsp;  
&emsp;&emsp;  
&emsp;&emsp;创建一个```SharedPreferencesTest```：  
> **activity_shared_preferences_test.xml**  
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".SharedPreferencesTest">

    <Button
        android:id="@+id/Btn_Save"
        android:text="存储数据"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
    
</LinearLayout>
```

> **SharedPreferencesTest**  
```kotlin
class SharedPreferencesTest : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_shared_preferences_test)
        
        /**
         * 给按钮注册一个点击事件，
         * 在点击事件中通过getSharedPreferences()方法指定SharedPreferences的文件名为data,得到SharedPreferences.Editor对象
         * 向Editor对象中添加3条不同的数据
         * 调用apply()方法提交，完成数据存储
         */
        Btn_Save.setOnClickListener {
            val editor = getSharedPreferences("data", Context.MODE_PRIVATE).edit()
            editor.putString("name","Tom")
            editor.putInt("age",28)
            editor.putBoolean("married",false)
            editor.apply()
        }
    }
}
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/DataPersistence/sharedpreferences_1.PNG)  

***

### 从SharedPreferences中读取数据
&emsp;&emsp;```SharedPreferences```提供了许多```get```方法来读取存储的数据  
&emsp;&emsp;```get```方法接收两个参数。  
&emsp;&emsp;第一个参数是键，根据键来获取响应值。  
&emsp;&emsp;第二个参数是默认值，当传入的键找不到对应的值，会以默认值进行返回。  
&emsp;&emsp;  
&emsp;&emsp;  
&emsp;&emsp;增加一个按钮，修改SharedPreferencesTest```：  
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_shared_preferences_test)
    ......
    
    Btn_Read.setOnClickListener {
        val pref = getSharedPreferences("data",Context.MODE_PRIVATE)
        val name = pref.getString("name","")
        val age = pref.getInt("age",0)
        val married = pref.getBoolean("married",false)
        Log.d("MainActivity","name is $name")
        Log.d("MainActivity","age is $age")
        Log.d("MainActivity","married is $married")
    }
}
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/DataPersistence/sharedpreferences_2.PNG)  

***

### 实现一个记住密码的功能
&emsp;&emsp;实际项目中要结合加密算法对密码进行保护。  
1. ```activity_login.xml```  
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

    <LinearLayout
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="60dp">

        <CheckBox
            android:id="@+id/Cb_Remember"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

        <TextView
            android:text="记住密码"
            android:textSize="18sp"
            android:layout_width="wrap_content"
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

2. ```LoginActivity```  
```kotlin
class LoginActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_login)

        val pref = getPreferences(Context.MODE_PRIVATE)
        val isRemember = pref.getBoolean("remember_password",false)
        if(isRemember){
            //将账号和密码都设置到文本框
            val account = pref.getString("account","")
            val password = pref.getString("password","")
            Edit_Account.setText(account)
            Edit_Password.setText(password)
            Cb_Remember.isChecked = true
        }

        Btn_Login.setOnClickListener {
            val account = Edit_Account.text.toString()
            val password = Edit_Password.text.toString()
            if(account == "admin" && password == "123"){
                val editor = pref.edit()
                if(Cb_Remember.isChecked){
                    editor.putBoolean("remember_password",true)
                    editor.putString("account",account)
                    editor.putString("password",password)
                }
                else{
                    /**
                     * clear()方法能将SharedPreferences中的数据全部清除掉
                     */
                    editor.clear()
                }
                editor.apply()
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
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/DataPersistence/sharedpreferences_3.PNG)  
