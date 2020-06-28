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

***

## SQLite
### 1. 创建数据库
&emsp;&emsp;```SQLiteOpenHelper```是一个抽象类，要使用就需要创建一个自己的帮助类去继承它，并重写```onCreate()```和```onUpgrade()```方法，分别实现创建和升级数据库逻辑。  
&emsp;&emsp;```SQLiteOpenHelper```有两个实例方法```getReadableDatabase()```和```getWritableDatabase()```，这两个方法都可以创建或打开一个现有的数据库，数据库已存在则直接打开，否则创建一个新的数据库，并返回一个可对数据库进行读写操作的对象。  
&emsp;&emsp;当数据库不可写入（如磁盘空间已满），```getReadableDatabase()```返回的对象将只以可读的方式打开数据库，而```getWritableDatabase()```将出现异常。  
&emsp;&emsp;  
&emsp;&emsp;**常用构造方法：**  
&emsp;&emsp;```context```：上下文对象  
&emsp;&emsp;```name```：数据库名字  
&emsp;&emsp;```factory```：游标工厂，通常传入```null```  
&emsp;&emsp;```version```：当前数据库版本号，可用于对数据库进行升级操作  
```kotlin
SQLiteOpenHelper(@Nullable Context context, @Nullable String name, @Nullable CursorFactory factory, int version)
```  
&emsp;&emsp;构建出```SQLiteOpenHelper```实例后，再调用```getReadableDatabase()```或```getWritableDatabase()```就能够创建数据库了，此时重写的```onCreate()```方法也会得到执行，可以在
```onCreate()```方法里执行一些创建表的逻辑。  
&emsp;&emsp;数据库文件存放在```/data/data/包名/databases/```目录下。  

1. 新建```MyDatabaseHelper```类继承```SQLiteOpenHelper```  
&emsp;&emsp;```integer```表整型  
&emsp;&emsp;```real```表浮点型  
&emsp;&emsp;```text```表示文本类型  
&emsp;&emsp;```blob```表示二进制类型  
> 这里将 **id** 通过 **primary key** 设为主键，用 **autoincrement** 表示 **id** 是自增长的。  
> 把建表语句定义成一个字符串变量，在 **onCreate()** 方法中又调用了 **SQLiteDatabase** 的 **execSQL** 去执行建表语句，在数据库创建完成时创建 **Book** 表。  
```kotlin
class MyDatabaseHelper(val context: Context,name : String,version : Int) : SQLiteOpenHelper(context,name,null,version) {

    private val createBook = "create table Book (" +
            " id integer primary key autoincrement," +
            " author text," +
            " price real," +
            " pages integer," +
            " name text)"

    override fun onCreate(db: SQLiteDatabase?) {
        db?.execSQL(createBook)
        Toast.makeText(context,"创建数据库成功",Toast.LENGTH_SHORT).show()
    }

    override fun onUpgrade(db: SQLiteDatabase?, p1: Int, p2: Int) {
        TODO("Not yet implemented")
    }

}
```

2. 修改```activity_sqlite_test.xml```布局文件，增加一个按钮  
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".SqliteTest">

    <Button
        android:id="@+id/Btn_CreateDb"
        android:text="创建数据库"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

</LinearLayout>
```

3. 修改```SqliteTest```  
&emsp;&emsp;在```onCreate()```方法中构建了一个```MyDatabaseHelper```对象，通过构造方法的参数将数据库名指定为```BookStore.db```,版本指定为1。  
&emsp;&emsp;在点击事件里调用```getWritableDatabase()```方法，第一次点击按钮就会检测到当前没有```BookStore.db```这个数据库，然后创建该数据库并调用```MyDatabaseHelper```里的```onCreate()```方法。  
&emsp;&emsp;再次点击按钮，因为已经存在```BookStore.db```数据库，因此不会再创建一次。  
```kotlin
class SqliteTest : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_sqlite_test)
        val dbHelper = MyDatabaseHelper(this,"BookStore.db",1)
        Btn_CreateDb.setOnClickListener {
            dbHelper.writableDatabase
        }
    }
}
```

4. 查看数据库文件  
&emsp;&emsp;在插件商城里下载```Database Navigator```，下载完后重启Android Studio。  
&emsp;&emsp;打开```Device File Explorer```，在```/data/data/包名/databases/```目录下可以看到```BookStore.db```文件，```BookStore.db-journal```是一个为了让数据库能支持事务而产生的临时文件。对```BookStore.db```右键```SAVE AS```保存到本地。  
&emsp;&emsp;点击左侧```DB Browser```，没有的话就```Ctrl + Shift + A```快速搜索。点击左上角```+```号按钮，选择```SQLite```，在弹窗里的```Databases files```栏里修改成本地保存的```BookStore.db```，点击```OK```，此时可以在```DB Browser```中看到```Book```表。  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/DataPersistence/sqlite_1.PNG)

***

### 2. 升级数据库
&emsp;&emsp;修改```MyDatabaseHelper```，增加一张```Category```表用于记录图书的分类  
```kotlin
class MyDatabaseHelper(val context: Context,name : String,version : Int) : SQLiteOpenHelper(context,name,null,version) {
    
    ......
    
    private val createCategory = "create table Category(" +
            "id integer primary key autoincrement," +
            "category_name text," +
            "category_code integer)"

    override fun onCreate(db: SQLiteDatabase?) {
        db?.execSQL(createBook)
        db?.execSQL(createCategory)
        Toast.makeText(context,"创建数据库成功",Toast.LENGTH_SHORT).show()
    }

}
```
&emsp;&emsp;运行程序后点击按钮，不会弹出提示也没有创建```Category```表，因为```BookStore.db```已经存在了。  
&emsp;&emsp;如果将程序卸载后再重新运行，```BookStore.db```此时不存在，```Category```表是可以创建成功的，但这不是好的做法。  
&emsp;&emsp;正确的做法是使用```SQLiteOpenHelper```的升级功能：  
1. 修改```MyDatabaseHelper```   
&emsp;&emsp;在```onUpgrade()```方法里执行了两条```DROP```语句，如果数据库已存在```Book```表或```Category```表，就将这两张表删除，然后调用```onCreate()```方法重新创建表。  
```kotlin
......
override fun onUpgrade(db: SQLiteDatabase?, p1: Int, p2: Int) {
    db?.execSQL("drop table if exists Book")
    db?.execSQL("DROP TABLE IF EXISTS Category")
    onCreate(db)
}
```

2. 修改```SqliteTest```  
&emsp;&emsp;这里将版本号指定为2，表示对数据库进行升级了。  
```kotlin
class SqliteTest : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_sqlite_test)
//        val dbHelper = MyDatabaseHelper(this,"BookStore.db",1)
        val dbHelper = MyDatabaseHelper(this,"BookStore.db",2)
        Btn_CreateDb.setOnClickListener {
            dbHelper.writableDatabase
        }
    }
}
```
&emsp;&emsp;重新运行程序，点击按钮，创建成功。  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/DataPersistence/sqlite_2.PNG)

***

### 3. 添加数据
&emsp;&emsp;添加数据的方法是```insert(String table, String nullColumnHack, ContentValues values)```  
&emsp;&emsp;```table```：表名  
&emsp;&emsp;```nullColumnHack```：在未指定添加数据的情况下给某些可为空的列自动复制```NULL```，一般传入```null```  
&emsp;&emsp;```values```：```ContentValues```对象，提供了一系列```put()```方法重载，用于向```ContentValues```中添加数据  
&emsp;&emsp;  
&emsp;&emsp;修改```SqliteTest```：  
> 先获取 **SQLiteDatabase** 对象，然后使用 **ContentValues** 对要添加的数据进行组装  
> 这里没有给 **id** 赋值是因为之前在创建表的时候将 **id** 设置为自增长了，值会在入库的时候自动生成  
> 最后调用 **insert()** 将数据添加到表中  
```kotlin
class SqliteTest : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_sqlite_test)
        val dbHelper = MyDatabaseHelper(this,"BookStore.db",2)
        ......

        Btn_addData.setOnClickListener {
            val db = dbHelper.writableDatabase
            val values1 = ContentValues().apply {
                put("name","炎拳")
                put("author","藤本树")
                put("pages",214)
                put("price",24.0)
            }
            db.insert("Book",null,values1)

            val values2 = ContentValues().apply {
                put("name","彪马野郎")
                put("author","荒木")
                put("pages",244)
                put("price",28.0)
            }
            db.insert("Book",null,values2)
        }
    }
}
```
&emsp;&emsp;导出```BookStore.db```重新加载，在```DB BROWSER```里双击表，点击```No Filter```即可看到数据。  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/DataPersistence/sqlite_add_1.PNG)
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/DataPersistence/sqlite_add_2.PNG)

***

### 4. 更新数据
&emsp;&emsp;```update(String table, ContentValues values, String whereClause, String[] whereArgs)```  
&emsp;&emsp;```table```：表名  
&emsp;&emsp;```values```：```ContentValues```对象，把要更新的数据在这里组装进去  
&emsp;&emsp;```whereClause```：WHERE表达式（String），需数据更新的行。 若该参数为 ```null```, 就会修改所有行。```?```号是占位符  
&emsp;&emsp;```whereArgs```：WHERE选择语句的参数(String[]), 逐个替换 WHERE表达式中的```?```占位符  
&emsp;&emsp;  
&emsp;&emsp;修改```SqliteTest```：  
> **arrayOf()** 是Kotlin提供的便捷创建数组的内置方法  
```kotlin
Btn_updateData.setOnClickListener {
    val db = dbHelper.writableDatabase
    val values = ContentValues().apply {
        put("price",10.99)
    }
    db.update("Book",values,"name = ?", arrayOf("彪马野郎"))
}
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/DataPersistence/sqlite_update_1.PNG)

***

### 5. 删除数据
&emsp;&emsp;```delete(String table, String whereClause, String[] whereArgs)```  
&emsp;&emsp;```table```：表名  
&emsp;&emsp;```whereClause```：WHERE表达式（String），需删除数据的行。 若该参数为 ```null```, 就会删除所有行。```?```号是占位符  
&emsp;&emsp;```whereArgs```：WHERE选择语句的参数(String[]), 逐个替换 WHERE表达式中的```?```占位符  
&emsp;&emsp;  
&emsp;&emsp;修改```SqliteTest```：  
```kotlin
Btn_deleteData.setOnClickListener {
    val db = dbHelper.writableDatabase
    db.delete("Book","pages > ?", arrayOf("220"))
}
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/DataPersistence/sqlite_delete_1.PNG)

***

### 6. 查询数据
&emsp;&emsp;```query(String table, String[] columns, String selection, String[] selectionArgs, String groupBy, String having, String orderBy)```  
&emsp;&emsp;调用```query()```方法后返回一个```Cursor```对象，查询到的所有数据从```Cursor```对象中取出。  

query()参数|对应SQL部分|描述
:-|:-|:-|
table|from table_name|指定查询的表名
columns|select column1,column2|指定查询的列名
selection|where column = value|指定where的约束条件
selectionArgs|-|为where的占位符提供具体的值
groupBy|group by column|指定需要group by的列
having|having column = value|对group by后的结果进一步约束
orderBy|order by column1,column2|指定查询结果的排序方式

&emsp;&emsp;修改```SqliteTest```：  
> 这里只使用第一个参数指明查询```Book```表，后面的参数全部为```null```，表示查询这张表的所有数据。  
> 调用```Cursor```对象的```moveToFirst()```方法，将数据的指针移动到第一行，通过```do/while()```进入循环,遍历查询的每一行数据。  
> 通过```Cursor```的```getColumnIndex()```方法获取某一列在表中对应的位置索引，然后将索引传入相应的取值方法中，如```getString()```。  
> 记得最后调用```close()```方法来关闭```Cursor```。  
```kotlin
class SqliteTest : AppCompatActivity() {
    companion object{
        val TAG = "SqliteTest"
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_sqlite_test)
        val dbHelper = MyDatabaseHelper(this,"BookStore.db",2)
        ......
        
        Btn_queryData.setOnClickListener {
            val db = dbHelper.writableDatabase
            //查询Book表中所有数据
            val cursor = db.query("Book",null,null,null,null,null,null)
            if(cursor.moveToFirst()){
                do {
                    //遍历Cursor对象，取出数据并打印
                    val name = cursor.getString(cursor.getColumnIndex("name"))
                    val author = cursor.getString(cursor.getColumnIndex("author"))
                    val pages = cursor.getInt(cursor.getColumnIndex("pages"))
                    val price = cursor.getDouble(cursor.getColumnIndex("price"))
                    Log.d(TAG,"书名：$name")
                    Log.d(TAG,"作者：$author")
                    Log.d(TAG,"页数：$pages")
                    Log.d(TAG,"价格：$price")
                }while (cursor.moveToNext())
            }
            cursor.close()
        }
    }
}
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/DataPersistence/sqlite_query_1.PNG)

***

### 7. 使用SQL操作数据库
&emsp;&emsp;除了查询数据调用的是```SQLiteDatabase```的```rawQuery()```方法，其他操作都是调用的```execSQL()```方法。  
&emsp;&emsp;添加数据：  
```kotlin
val db = dbHelper.writableDatabase
db.execSQL("insert into Book (name,author,pages,price) values(?,?,?,?)",
        arrayOf("彪马野郎","荒木飞吕彦","244","28.0")
)

db.execSQL("insert into Book (name,author,pages,price) values(?,?,?,?)",
        arrayOf("炎拳","藤本树","214","24.0")
)
```

&emsp;&emsp;更新数据：  
```korlin
db.execSQL("update Book set price = ? where name = ?", arrayOf("10.99","彪马野郎"))
```

&emsp;&emsp;删除数据：
```kotlin
db.execSQL("delete from Book where pages > ?", arrayOf("220"))
```

&emsp;&emsp;查询数据：  
```kotlin
val cursor = db.rawQuery("select * from Book",null)
```

***

### 8. 使用事务
&emsp;&emsp;事务可以保证一系列的操作要么全部完成，要么一个都不会完成。  
&emsp;&emsp;调用```SQLiteDatabase```的```beginTransaction()```方法开启一个事务，然后在一个异常捕捉的代码块中执行具体的数据库操作，当所有操作都完成后，调用```setTransactionSuccessful()```表示事务已经执行成功了，最后在```finally```代码块中调用```endTransaction()```结束事务。  
&emsp;&emsp;由于事物的存在，中途出现异常会导致事物的失败，此时旧数据是删除不掉的。  
```kotlin
Btn_transaction.setOnClickListener {
    val db = dbHelper.writableDatabase
    //开启事务
    db.beginTransaction()
    try {
        db.delete("Book",null,null)
        if(true){
            //手动抛出一个异常，让事务失败
            throw NullPointerException()
        }
        val values = ContentValues().apply {
            put("name","一拳超人")
            put("author","村田雄介")
            put("pages",224)
            put("price",24.0)
        }
        db.insert("Book",null,null)
        //事务已经执行成功
        db.setTransactionSuccessful()
    }catch (e:Exception){
        e.printStackTrace()
    }finally {
        //结束事务
        db.endTransaction()
    }
}
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/DataPersistence/sqlite_transaction_1.PNG)

***

### 9. 升级数据库的正确写法
&emsp;&emsp;之前的写法，升级后原本的数据会全部丢失。  
&emsp;&emsp;每一个数据库版本都会对应一个版本号，当指定的数据库版本号大于当前数据库版本号，就会进入```onUpgrade()```方法中执行更新操作。  
&emsp;&emsp;每当升级一般数据库版本的时候，```onUpgrade()```方法都一定要写一个相应的```if```判断语句。    
&emsp;&emsp;  
&emsp;&emsp;第一版只需要创建一张```Book```表。第二版添加一张```Category```表。第三版要给```Book```表和```Category```表之间建立关系，需要在```Book```表中添加一个```category_id```字段。  
&emsp;&emsp;  
&emsp;&emsp;第一版：  
```kotlin
class MyDatabaseHelper(val context: Context,name : String,version : Int) : SQLiteOpenHelper(context,name,null,version) {

    private val createBook = "create table Book (" +
            " id integer primary key autoincrement," +
            " author text," +
            " price real," +
            " pages integer," +
            " name text)"

    override fun onCreate(db: SQLiteDatabase?) {
        db?.execSQL(createBook)
    }

    override fun onUpgrade(db: SQLiteDatabase?, oldVersion: Int, newVersion: Int) {

    }

}
```
&emsp;&emsp;第二版：  
&emsp;&emsp;第二版的时候，用户直接安装第二版程序，就会进入```onCreate()```方法，两张表一起创建。  
&emsp;&emsp;使用第二版的程序覆盖安装第一版，就会进入升级数据库的操作中，由于```Book```表已存在，只需要创建一张```Category```表。  
```kotlin
class MyDatabaseHelper(val context: Context,name : String,version : Int) : SQLiteOpenHelper(context,name,null,version) {

    private val createBook = "create table Book (" +
            " id integer primary key autoincrement," +
            " author text," +
            " price real," +
            " pages integer," +
            " name text)"


    private val createCategory = "create table Category(" +
            "id integer primary key autoincrement," +
            "category_name text," +
            "category_code integer)"

    override fun onCreate(db: SQLiteDatabase?) {
        db?.execSQL(createBook)
        db?.execSQL(createCategory)
    }

    override fun onUpgrade(db: SQLiteDatabase?, oldVersion: Int, newVersion: Int) {
        //如果用户数据库的旧版本号小于等于1，就只会创建一张Category表
        if(oldVersion <= 1){
            db?.execSQL(createCategory)
        }
    }

}
```
&emsp;&emsp;第三版：  
&emsp;&emsp;在```Book```表的建表语句中添加了一个```category_id```列，当用户直接安装第三版，这个新增的列就已经自动添加成功了。  
&emsp;&emsp;如果用户之前已经安装了某一版本的程序，现在需要覆盖安装，就会进入升级数据库的操作中。  
&emsp;&emsp;在```onUpgrade()```方法里，如果当前数据库的版本号是2，就会执行alter命令，为```Book```表新增一个```category_id```列。  
```kotlin
class MyDatabaseHelper(val context: Context,name : String,version : Int) : SQLiteOpenHelper(context,name,null,version) {

    private val createBook = "create table Book (" +
            " id integer primary key autoincrement," +
            " author text," +
            " price real," +
            " pages integer," +
            " name text," + 
            " category_id integer)"


    private val createCategory = "create table Category(" +
            "id integer primary key autoincrement," +
            "category_name text," +
            "category_code integer)"

    override fun onCreate(db: SQLiteDatabase?) {
        db?.execSQL(createBook)
        db?.execSQL(createCategory)
        Toast.makeText(context,"创建数据库成功",Toast.LENGTH_SHORT).show()
    }

    override fun onUpgrade(db: SQLiteDatabase?, oldVersion: Int, newVersion: Int) {
        //如果用户数据库的旧版本号小于等于1，就只会创建一张Category表
        if(oldVersion <= 1){
            db?.execSQL(createCategory)
        }
        if(oldVersion <= 2){
            db?.execSQL("alter table Book add column category_id integer")
        }
    }

}
```
