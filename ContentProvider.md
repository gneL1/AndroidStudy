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
3. 最后接上```vnd.<authority>.<path>  
```content://com.example.app.provider/table1```对应的 **MIME** 类型：  
&emsp;&emsp;```vnd.android.cursor.dir/vnd.com.example.app.provider.table1```  

