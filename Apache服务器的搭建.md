# Apache服务器搭建
## 1. 下载Apache
打开官网[http://httpd.apache.org/](http://httpd.apache.org/)  

* 点击**Download**  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Apache%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA/apache_1.PNG)
&emsp;&emsp;  
&emsp;&emsp;  

* 点击**Files for Microsoft Windows**  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Apache%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA/apache_2.PNG)
&emsp;&emsp;  
&emsp;&emsp;  

* 点击**ApacheHaus**  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Apache%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA/apache_3.PNG)
&emsp;&emsp;  
&emsp;&emsp;  

* 下载64位版本  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Apache%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA/apache_4.PNG)
&emsp;&emsp;  
&emsp;&emsp;  

* 下载完后随便解压到一个文件夹  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Apache%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA/apache_5.PNG)
&emsp;&emsp;  
&emsp;&emsp;  

## 2. 配置Apache服务器
* 打开```conf```文件夹下的```httpd.conf```文件  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Apache%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA/apache_6.PNG)
&emsp;&emsp;  
&emsp;&emsp;  

* 搜索```Define SRVROOT```，修改目录位置与当前```Apache```安装的目录位置一致  
我这里是```Define SRVROOT "E:\Apache\Apache24"```  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Apache%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA/apache_7.PNG)
&emsp;&emsp;  
&emsp;&emsp;  

* 打开CMD，输入```netstat -ano```可以查看当前端口占用情况。  
输入```netstat -ano|findstr 80```可以查看指定端口80的占用情况。  
如果没有```0.0.0.0:80```，就说明80端口没有被占用，如果出现了则80端口被占用。  
下面的图是我配置完之后才显示80端口被占用的，在我实际配置过程中没有出现```0.0.0.0:80```。  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Apache%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA/apache_9.PNG)
&emsp;&emsp;  
&emsp;&emsp;  

* 如果80端口被占用，则修改```httpd.conf```里的端口为其他未被占用的端口。  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Apache%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA/apache_10.PNG)
&emsp;&emsp;  
&emsp;&emsp;  

* 在CMD里安装Apache的主服务，输入```"路径\bin\httpd.exe" -k install -n apache```，回车执行。  
&emsp;&emsp;这里的路径是Apache的安装目录，我这里就是```"E:\Apache\Apache24\bin\httpd.exe" -k install -n apache```，注意```""```双引号不能省略。  
&emsp;&emsp;命令意思是安装Windows可托管的Apache服务，其中"-n"后面参数是自定义Windows服务名称，这里将该服务名称命名为apache(也可以改成别的)。之后可使用Windows管理服务的命令来管理apache服务，如"net start/stop apache"（启动/停止服务）。  
&emsp;&emsp;```Errors reported here must be corrected before the service can be started.```意思是“此处报告的错误必须在服务开始前进行纠正”，可忽略不管。  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Apache%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA/apache_8.PNG)
&emsp;&emsp;  
&emsp;&emsp; 


## 3.启动Apache服务器
* 打开```bin```文件夹下的```ApacheMonitor.exe```  
&emsp;&emsp;```Errors reported here must be corrected before the service can be started.```意思是“此处报告的错误必须在服务开始前进行纠正”，可忽略不管。  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Apache%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA/apache_11.PNG)
&emsp;&emsp;  
&emsp;&emsp; 

* 右下角会出现小图标，双击打开界面后点击```Start```启动服务  
要关闭就点击```Stop```  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Apache%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA/apache_12.PNG)
&emsp;&emsp;  
&emsp;&emsp; 

* 打开浏览器，输入```http://localhost```或```127.0.0.1```，出现以下界面则表示Apache服务器配置成功。  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Apache%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA/apache_13.PNG)
&emsp;&emsp;  
&emsp;&emsp; 

* 关于Apache服务器下的各个文件的用途如下图。  
![图片示例](https://github.com/gneL1/AndroidStudy/blob/master/photos/Apache%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA/apache_14.PNG)
&emsp;&emsp;  
&emsp;&emsp; 
