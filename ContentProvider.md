# ContentProvider
&emsp;&emsp;```ContentProvider```主要用于在不同的应用程序之间实现数据共享的功能，允许一个程序访问另一个程序中的数据，同时保证被访问数据的安全性。  
&emsp;&emsp;```ContentProvider```可以选择只对哪一部分数据进行共享。

## 运行时权限
&emsp;&emsp;用户不需要在安装软件的时候一次性授权所有申请的权限，而是可以在软件的使用过程中再对某一项权限申请进行授权。  
&emsp;&emsp;比如一款相机应用在运行时申请了地理位置定位权限，用户拒绝了这个权限，也可以使用这个应用的其他功能。  
&emsp;&emsp;  
### 在程序运行时申请权限
1. 创建一个```PermissionTest```  
> **activity_permission_test.xml** 文件  
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".PermissionTest">

    <Button
        android:id="@+id/Btn_makeCall"
        android:text="打电话"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
    
</LinearLayout>
```
> **PermissionTest** 文件  
```kotlin
class PermissionTest : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_permission_test)

        Btn_makeCall.setOnClickListener {
            /**
             * 构建一个隐式的Intent,Intent的action指定为Intent.ACTION_CALL，这是一个内置的打电话的动作
             * Intent.ACTION_CALL表示直接拨打电话，必须声明权限
             * 而Intent.ACTION_DIAL表示打开拨号界面，是不需要声明权限的
             * 在data部分指定了协议是tel,号码是10086
             * 为了防止程序崩溃，需要将操作放在异常捕获代码块中
             */
            try {
                val intent = Intent(Intent.ACTION_CALL)
                intent.data = Uri.parse("tel:10086")
                startActivity(intent)
            }catch (e : SecurityException){
                e.printStackTrace()
            }
        }

    }
}
```

2. 修改```AndroidManifest.xml```文件  
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.contentprovidertest">

    <uses-permission android:name="android.permission.CALL_PHONE"/>
    
    ......
```
&emsp;&emsp;点击按钮的运行结果：  
&emsp;&emsp;```Permission Denial```表示权限被禁止，这是因为```Android 6.0```及以上系统在使用危险权限时必须进行运行时权限处理。  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/ContentProvider/permission_1.PNG)

***
**修改方案：**  
&emsp;&emsp;通过```ContextCompat.checkSelfPermission()```方法判断用户是否已经给过授权。  
&emsp;&emsp;```checkSelfPermission(@NonNull Context context, @NonNull String permission)```  
&emsp;&emsp;```context```：上下文  
&emsp;&emsp;```permission```：具体的权限名，比如打电话的权限名就是```Manifest.permission.CALL_PHONE```  
&emsp;&emsp;返回值是```int```型，通过返回值与```PackageManager.PERMISSION_GRANTED```做比较，相等说明用户已经授权，不相等表示用户没有授权。  
&emsp;&emsp;  
&emsp;&emsp;如果已经授权，则直接执行拨打电话的逻辑。  
&emsp;&emsp;如果没有授权，则需要调用```ActivityCompat.requestPermissions()```方法向用户申请授权。  
&emsp;&emsp;```requestPermissions(final @NonNull Activity activity,final @NonNull String[] permissions, final @IntRange(from = 0) int requestCode)```  
&emsp;&emsp;```activity```：```Activity```的实例  
&emsp;&emsp;```permissions```：String数组，把要申请的权限名放在数组中  
&emsp;&emsp;```requestCode```：请求码  
&emsp;&emsp;  
&emsp;&emsp;调用完```requestPermissions()```方法后，系统会弹出一个权限申请的对话框，用户可以选择同意或拒绝权限申请。  
&emsp;&emsp;不论同意还是拒绝，最终都会回调```onRequestPermissionsResult()```方法。  
&emsp;&emsp;```onRequestPermissionsResult(requestCode: Int,permissions: Array<out String>,grantResults: IntArray)```  
&emsp;&emsp;授权结果封装在```grantResults```参数中。  
&emsp;&emsp;  
&emsp;&emsp;判断授权结果，用户同意就调用```call()```方法拨打电话。用户拒绝则弹出一条失败提示。  

```kotlin
class PermissionTest : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_permission_test)

        Btn_makeCall.setOnClickListener {
            if(ContextCompat.checkSelfPermission(this,android.Manifest.permission.CALL_PHONE) != PackageManager.PERMISSION_GRANTED){
                ActivityCompat.requestPermissions(this, arrayOf(android.Manifest.permission.CALL_PHONE),1)
            }
            else{
                call()
            }
//            try {
//                val intent = Intent(Intent.ACTION_CALL)
//                intent.data = Uri.parse("tel:10086")
//                startActivity(intent)
//            }catch (e : SecurityException){
//                e.printStackTrace()
//            }
        }

    }

    override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<out String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        when(requestCode){
            1 -> {
                if(grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED){
                    call()
                }
                else{
                    Toast.makeText(this,"你拒绝了权限",Toast.LENGTH_SHORT).show()
                }
            }
        }
    }

    private fun call(){
        /**
         * 构建一个隐式的Intent,Intent的action指定为Intent.ACTION_CALL，这是一个内置的打电话的动作
         * Intent.ACTION_CALL表示直接拨打电话，必须声明权限
         * 而Intent.ACTION_DIAL表示打开拨号界面，是不需要声明权限的
         * 在data部分指定了协议是tel,号码是10086
         * 为了防止程序崩溃，需要将操作放在异常捕获代码块中
         */
        try {
            val intent = Intent(Intent.ACTION_CALL)
            intent.data = Uri.parse("tel:10086")
            startActivity(intent)
        }catch (e : SecurityException){
            e.printStackTrace()
        }
    }

}
```
&emsp;&emsp;当没有授权时点击按钮：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/ContentProvider/permission_2.PNG)

&emsp;&emsp;点击```Deny```拒绝授权：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/ContentProvider/permission_3.PNG)

&emsp;&emsp;点击```Allow```同意授权：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/ContentProvider/permission_4.PNG)

&emsp;&emsp;值得注意的是，如果用户连续两次都拒绝授权，之后就不会再弹出授权对话框。  

***

## 访问其他程序中的数据
### 1. ContentResolver的基本用法
&emsp;&emsp;想要访问```ContentProvider```中共享的数据，就要借助```ContentResolver```类，可以通过```Context```中的```getContentResolver()```方法获取该类的实例。  
&emsp;&emsp;```ContentResolver```提供了一系列方法来对数据进行增删改查的操作。  
&emsp;&emsp;```ContentResolver```的增删改查方法不接收表明参数，而是使用一个```Uri```参数代替，这个参数被称为内容URI。  
&emsp;&emsp;  
&emsp;&emsp;内容URI的组成：协议声明 + authority + path  
&emsp;&emsp;标准格式：```content://com.example.app.provider/table1```  
&emsp;&emsp;协议声明：```content://```  
&emsp;&emsp;authority：给不同的应用程序做区分，采用包名命名，如包名是```com.example.app```，那么authority命名为```com.example.app.provider```  
&emsp;&emsp;path：对同一应用程序的不同表做区分，如```/table1```  
&emsp;&emsp;  
&emsp;&emsp;得到内容URI字符串后，还需要将其解析成```Uri```对象才可以作为参数传入。  
```kotlin
val uri = Uri.parse("content://com.example.app.provider/table1")
```

1. 查询  
```kotlin
val cursor = ContentResolver.query(
    uri,
    projection,
    selection,
    selectionArgs,
    sortOrder
)
```
query()方法参数|对应SQK语句|描述
:-|:-|:-|
uri|from table_name|指定查询某个应用程序下的某一张表
projection|select column1,column2|指定查询的列名
selection|where column = value|指定where的约束条件
selectionArgs|-|为where中的占位符提供某个具体的值
sortOrder|order by column1,column2|指定查询结果的排序方式

&emsp;&emsp;查询完成后返回的是一个```Cursor```对象，通过移动游标的位置遍历```Cursor```的所有行。  
&emsp;&emsp;记得将```Cursor```对象关闭。  
```kotlin
while(cursor.moveToNext()){
  val column1 = cursor.getString(cursor.getColumnIndex("column1"))
  val column2 = cursor.getInt(cursor.getColumnIndex("column2"))
}
cursor.close()
```

2. 添加  
```kotlin
val values = contentValuesOf("column1" to "text","column2" to 1)
contentResolver.insert(uri,values)
```

3. 更新  
注意这里```arrayOf```里的```1```也是字符串。  
```kotlin
val values = contentValueOf("column1" to "")
contentResolver.update(uri,values,"column1 = ? and column2 = ?",arrayOf("text","1"))
```

4. 删除  
```kotlin
contentResolver.delete(uri,"column2 = ?",arrayOf("1"))
```

***

### 2. 读取系统联系人
1. 修改```activity_content_resolver.xml```布局：  
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".ContentResolverTest">

    <ListView
        android:id="@+id/Lv_1"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</LinearLayout>
```

2. 修改```ContentResolverTest```：  
```kotlin
class ContentResolverTest : AppCompatActivity() {

    private val contactsList = ArrayList<String>()
    private lateinit var adapter: ArrayAdapter<String>

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_content_resolver)

        //对ListView初始化
        adapter = ArrayAdapter(this,android.R.layout.simple_list_item_1,contactsList)
        Lv_1.adapter = adapter
        if(ContextCompat.checkSelfPermission(this,Manifest.permission.READ_CONTACTS) != PackageManager.PERMISSION_GRANTED){
            ActivityCompat.requestPermissions(this, arrayOf(Manifest.permission.READ_CONTACTS),1)
        }
        else{
            readContacts()
        }
    }

    override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<out String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        when(requestCode){
            1 -> {
                if (grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED){
                    readContacts()
                }
                else{
                    Toast.makeText(this,"你拒绝了授权",Toast.LENGTH_SHORT).show()
                }
            }
        }
    }

    /**
     * 使用ContentResolver的query()方法来查询系统的联系人数据
     * 使用?.操作符和apply操作符简化遍历代码
     */
    private fun readContacts(){
        //查询联系人数据
        contentResolver.query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI,null,null,null,null)?.apply {
            while (moveToNext()){
                //获取联系人姓名
                val displayName = getString(getColumnIndex(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME))
                //获取联系人手机号
                val number = getString(getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER))
                contactsList.add("$displayName\n$number")
            }
            adapter.notifyDataSetChanged()
            //关闭cursor对象
            close()
        }
    }
}
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/ContentProvider/contentresolver_1.PNG)

***

## 创建自定义ContentProvider
### 1. 创建ContentProvider的步骤
&emsp;&emsp;新建一个类继承```ContentProvider```，重写6个抽象方法。  
```kotlin
class MyProvider : ContentProvider() {

    override fun delete(uri: Uri, selection: String?, selectionArgs: Array<String>?): Int {
        TODO("Implement this to handle requests to delete one or more rows")
    }

    override fun getType(uri: Uri): String? {
        TODO(
            "Implement this to handle requests for the MIME type of the data" +
                    "at the given URI"
        )
    }

    override fun insert(uri: Uri, values: ContentValues?): Uri? {
        TODO("Implement this to handle requests to insert a new row.")
    }

    override fun onCreate(): Boolean {
        TODO("Implement this to initialize your content provider on startup.")
    }

    override fun query(
        uri: Uri, projection: Array<String>?, selection: String?,
        selectionArgs: Array<String>?, sortOrder: String?
    ): Cursor? {
        TODO("Implement this to handle query requests from clients.")
    }

    override fun update(
        uri: Uri, values: ContentValues?, selection: String?,
        selectionArgs: Array<String>?
    ): Int {
        TODO("Implement this to handle requests to update one or more rows.")
    }
}
```
&emsp;&emsp;```onCreate()```：初始化```ContentProvider```的时候调用，通常在这里完成数据库的创建和升级等操作。返回```true```表示```ContentProvider```初始化成功，
返回```false```则表示失败。  
&emsp;&emsp;```query()```：```uri```参数用于确定查询哪张表，```projection```用于查询那些列，查询的结果存放在```Cursor```对象中返回。  
&emsp;&emsp;```insert()```：```uri```用于确定要添加到的表，待添加的数据保存在```values```参数中。添加完成后，返回一个用于表示这条新纪录的```URI```。  
&emsp;&emsp;```update()```：被修改的行数作为返回值返回。  
&emsp;&emsp;```delete()```：被删除的行数作为返回值返回。  
&emsp;&emsp;```getType()```：根据传入的URI返回相应的```MIME```类型。  
&emsp;&emsp;  
&emsp;&emsp;  
#### URI的写法
&emsp;&emsp;一个标准的内容URI写法是：  
&emsp;&emsp;```content://com.example.app.provider/table1```  
&emsp;&emsp;即表示访问的是```com.example.app```这个应用的```table1```表中的数据。  
&emsp;&emsp;  
&emsp;&emsp;可以在这个内容URI的后面加上一个```id```：  
&emsp;&emsp;```content://com.example.app.provider/table1/1```  
&emsp;&emsp;访问的是```com.example.app```这个应用的```table1```表中```id```为 **1** 的数据。  
&emsp;&emsp;  
&emsp;&emsp;还可以使用通配符分别匹配两种格式的内容URI：  
&emsp;&emsp;```*```表示匹配任意长度的任意字符  
&emsp;&emsp;能够匹配任意表的内容URI：  
&emsp;&emsp;```content://com.example.app.provider/*```  
&emsp;&emsp;能够匹配任意一行数据的内容URI：  
&emsp;&emsp;```content://com.example.app.provider/table1/#```  

#### UriMatcher
&emsp;&emsp;可以实现匹配内容URI。  
&emsp;&emsp;```UriMatcher```提供了一个```addURI(String authority, String path, int code)```方法。  
&emsp;&emsp;第三个参数是一个自定义代码。  
```kotlin
private val bookDir = 0
private val authority = "com.example.datasave.provider"
val matcher = UriMatcher(UriMatcher.NO_MATCH)
matcher.addURI(authority,"book",bookDir)
```
&emsp;&emsp;当调用```UriMatcher```的```match(Uri uri)```方法时，将可以将一个```Uri```对象传入，返回值是能匹配这个Uri对象的自定义代码。  
&emsp;&emsp;通过自定义代码来判断访问的是哪张表的数据。  
```kotlin
when(uriMatcher.match(uri)){
    bookDir -> {
        ......
    }
}
```

#### MIME
&emsp;&emsp;一个内容URI对应的 **MIME** 字符串由3部分组成
1. 必须以```vnd```开头  
2. 如果内容URI以路径结尾，则后接```android.cursor.dir/```；如果内容URI以id结尾，则后接```android.cursor.item/```  
3. 最后接上```vnd.<authority>.<path>```  
&emsp;&emsp;  
```content://com.example.app.provider/table1```对应的 **MIME** 类型：  
&emsp;&emsp;```vnd.android.cursor.dir/vnd.com.example.app.provider.table1```  
&emsp;&emsp;  
```content://com.example.app.provider/table1/1```对应的 **MIME** 类型：  
&emsp;&emsp;```vnd.android.cursor.item/vnd.com.example.app.provider.table1```  

***

### 2. 跨程序数据共享
#### 1. 在之前数据存储的项目里
&emsp;&emsp;右键项目包 -> New -> Other -> Content Provier  
&emsp;&emsp;将```authority```指定为```包名.provider```  
&emsp;&emsp;```Exported```属性表示是否允许外部程序访问此```ContentProvider```  
&emsp;&emsp;```Enabled```属性表示是否启用这个```ContentProvider```  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/ContentProvider/contentProvider_1.PNG)

&emsp;&emsp;修改```DatabaseProvider```：  
```kotlin
class DatabaseProvider : ContentProvider() {

    /**
     * 前四个变量分别表示访问Book表中的所有数据、访问Book表中的单条数据、
     * 访问Category表中的所有数据、访问Category表中的单条数据
     */
    private val bookDir = 0
    private val bookItem = 1
    private val categoryDir = 2
    private val categoryItem = 3
    private val authority = "com.example.datasave.provider"
    private var dbHelper : MyDatabaseHelper? = null

    /**
     * 在by lazy代码块中对uriMatcher进行初始化操作
     * by lazy是一种懒加载技术，代码块中的代码一开始不执行，当UriMatcher变量首次被调用时才会执行，
     * 并将代码块中最后一行代码的返回值赋值给uriMatcher
     */
    private val uriMatcher by lazy {
        /**
         * 常量UriMatcher.NO_MATCH = 不匹配任何路径的返回码
         * 即初始化时不匹配任何东西
         * addURI(String authority, String path, int code)
         * 若内容URI = "content://com.example.datasave.provider/book",则返回注册码bookDir
         */
        val matcher = UriMatcher(UriMatcher.NO_MATCH)
        matcher.addURI(authority,"book",bookDir)
        matcher.addURI(authority,"book/#",bookItem)
        matcher.addURI(authority,"category",categoryDir)
        matcher.addURI(authority,"category/#",categoryItem)
        matcher
    }

    override fun delete(uri: Uri, selection: String?, selectionArgs: Array<String>?) = dbHelper?.let {
        val db = it.writableDatabase
        val deleteRows = when(uriMatcher.match(uri)){
            bookDir -> db.delete("Book",selection,selectionArgs)
            bookItem -> {
                val bookId = uri.pathSegments[1]
                db.delete("Book","id = ?", arrayOf(bookId))
            }
            categoryDir -> db.delete("Category",selection,selectionArgs)
            categoryItem -> {
                val categoryId = uri.pathSegments[1]
                db.delete("Category","id = ?", arrayOf(categoryId))
            }
            else -> 0
        }
        deleteRows
    }?: 0

    override fun getType(uri: Uri) = when(uriMatcher.match(uri)){
        bookDir -> "vnd.android.cursor.dir/vnd.$authority.book"
        bookItem -> "vnd.android.cursor.item/vnd.$authority.book"
        categoryDir -> "vnd.android.cursor.dir/vnd.$authority.category"
        categoryItem -> "vnd.android.cursor.item/vnd.$authority.category"
        else -> null
    }

    /**
     * insert()方法要求返回一个能够表示这条新增数据的URI
     * 需要调用Uri.parse()方法将一个内容URI解析成Uri对象
     * 此URL以新增数据的id结尾
     */
    override fun insert(uri: Uri, values: ContentValues?) = dbHelper ?.let {
        //添加数据
        val db = it.writableDatabase
        val uriReturn = when(uriMatcher.match(uri)){
            bookDir,bookItem -> {
                val newBookId = db.insert("Book",null,values)
                Uri.parse("content://$authority/book/$newBookId")
            }
            categoryDir,categoryItem -> {
                val newCategoryId = db.insert("Category",null,values)
                Uri.parse("content://$authority/category/$newCategoryId")
            }
            else -> null
        }
        uriReturn
    }

    /**
     * 因为代码块只有一行，所有用=符号代替{}符号
     * context不为null，才执行let函数体
     * 返回值 = 最后一行/return的表达式
     * context为空就执行?:操作符返回false初始化失败,不为空则通过let函数返回true表示ContentProvider初始化成功
     */
    override fun onCreate() = context?.let {
        //创建了一个MyDatabaseHelper实例
        dbHelper = MyDatabaseHelper(it,"BookStore.db",2)
        true
    } ?: false

    /**
     * 根据传入的Uri参数判断用户想要访问哪张表,再调用SQLiteDatabase的query()方法进行查询
     * 最后将Cursor对象返回
     * 访问单条数据调用了Uri对象的getPathSegments()方法，会将内容URI权限之后的部分以"/"符号进行分割
     * 分割后的结果放入字符串列表。第0个位置存放的是路径，第1个位置存放的是id
     */
    override fun query(
        uri: Uri, projection: Array<String>?, selection: String?,
        selectionArgs: Array<String>?, sortOrder: String?
    ) = dbHelper?.let {
        //查询数据
        var db = it.readableDatabase
        val cursor = when(uriMatcher.match(uri)){
            bookDir -> db.query("Book",projection,selection,selectionArgs,null,null,sortOrder)
            bookItem -> {
                val bookId = uri.pathSegments[1]
                db.query("Book",projection,"id = ?", arrayOf(bookId),null,null,sortOrder)
            }
            categoryDir -> db.query("Category",projection,selection,selectionArgs,null,null,sortOrder)
            categoryItem -> {
                val categoryId = uri.pathSegments[1]
                db.query("Category",projection,"id = ?", arrayOf(categoryId),null,null,sortOrder)
            }
            else -> null
        }
        cursor
    }

    override fun update(
        uri: Uri, values: ContentValues?, selection: String?,
        selectionArgs: Array<String>?
    ) = dbHelper?.let {
        //更新数据
        val db = it.writableDatabase
        val updateRows = when(uriMatcher.match(uri)){
            bookDir -> db.update("Book",values,selection,selectionArgs)
            bookItem -> {
                val bookId = uri.pathSegments[1]
                db.update("Book",values,"id = ?", arrayOf(bookId))
            }
            categoryDir -> db.update("Category",values,selection,selectionArgs)
            categoryItem -> {
                val categoryId = uri.pathSegments[1]
                db.update("Category",values,"id = ?", arrayOf(categoryId))
            }
            else -> 0
        }
        updateRows
    }?: 0
}
```

&emsp;&emsp;```ContentProvider```一定要在```AndroidManifest.xml```文件中注册  
&emsp;&emsp;```<provider>```标签的```android:name```指定了```ContentProvider```的类名。    
&emsp;&emsp;```android:authoritier```指定了```DatabaseProvider```的```authority```。  
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.datasave">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <provider
            android:name=".DatabaseProvider"
            android:authorities="com.example.datasave.provider"
            android:enabled="true"
            android:exported="true"></provider>
        ......
```

***

#### 2. 在当前项目里  
&emsp;&emsp;修改```activity_provider_test.xml```布局文件  
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".ProviderTest">

    <Button
        android:id="@+id/Btn_add"
        android:text="添加"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <Button
        android:id="@+id/Btn_query"
        android:text="查询"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <Button
        android:id="@+id/Btn_update"
        android:text="更新"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <Button
        android:id="@+id/Btn_delete"
        android:text="删除"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

</LinearLayout>
```
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/ContentProvider/contentProvider_layout.PNG)

&emsp;&emsp;修改```ProviderTest```  
```kotlin
class ProviderTest : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_provider_test)

        var bookId : String? = null

        //添加数据
        Btn_add.setOnClickListener {
            val uri = Uri.parse("content://com.example.datasave.provider/book")
            val values = contentValuesOf("name" to "黑暗骑士","author" to "克里斯托弗·诺兰","pages" to 326,"price" to 74.0)
            val newUri = contentResolver.insert(uri,values)
            //insert()方法返回一个Uri对象，这个对象包含了新增数据的id
            //通过getPathSegments()方法将id取出
            bookId = newUri?.pathSegments?.get(1)
        }

        //查询数据
        Btn_query.setOnClickListener {
            val uri = Uri.parse("content://com.example.datasave.provider/book")
            contentResolver.query(uri,null,null,null,null)?.apply {
                while (moveToNext()){
                    val name = getString(getColumnIndex("name"))
                    val author = getString(getColumnIndex("author"))
                    val pages = getInt(getColumnIndex("pages"))
                    val price = getDouble(getColumnIndex("price"))
                    Log.d("ProviderTest：","书名：$name")
                    Log.d("ProviderTest：","作者：$author")
                    Log.d("ProviderTest：","页数：$pages")
                    Log.d("ProviderTest：","价格：$price")
                    Log.d("ProviderTest：","-----------------------------------------")
                }
                close()
            }
        }

        //更新数据
        Btn_update.setOnClickListener {
            bookId?.let {
                val uri = Uri.parse("content://com.example.datasave.provider/book/$it")
                val values = contentValuesOf("name" to "信条","pages" to 288,"price" to 54.1)
                contentResolver.update(uri,values,null,null)
            }
        }

        //删除数据
        Btn_delete.setOnClickListener {
            bookId?.let {
                val uri = Uri.parse("content://com.example.datasave.provider/book/$it")
                contentResolver.delete(uri,null,null)
            }
        }
    }
}
```
***

#### 3. 运行效果  
&emsp;&emsp;两个应用都要打开，如果被通信的程序已经关闭，则无法访问。  
&emsp;&emsp;不知道为什么，虚拟机无法实现通信，实机可以。  
&emsp;&emsp;无法访问的时候会报以下错误：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/ContentProvider/contentProvider_error_1.PNG)

&emsp;&emsp;直接查询：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/ContentProvider/contentProvider_2.PNG)

&emsp;&emsp;添加后查询：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/ContentProvider/contentProvider_3.PNG)

&emsp;&emsp;更新后查询：  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/ContentProvider/contentProvider_4.PNG)











