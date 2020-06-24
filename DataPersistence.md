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
