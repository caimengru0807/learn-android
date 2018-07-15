本文主要介绍不同Android版本应用内安装apk的方法。首先将现有的Android版本进行分一下界限，Android 1.x~6.x 和Android 7.x 及Android 8.x。下面主要介绍下Android 7.0和Android 8.0 的适配方法。

#### Android 7.0
android 7.0引入了私有目录被限制访问”，“StrictMode API 政策”。” StrictMode API 政策” 是指禁止向你的应用外公开 file:// URI。 如果一项包含文件 file:// URI类型 的 Intent 离开你的应用，应用失败，并出现 FileUriExposedException 异常。这个时候就需要使用google 推荐的FileProvider来解决这个问题。

```
Caused by: android.os.FileUriExposedException: 
file:///storage/emulated/0/Download/myApp.apk exposed beyond app through Intent.getData()
```

#### Android 8.0
Android 8.0更新主要是对Intent隐式安装APK做了个安全管理：
需要在mainfist中添加一行权限：

```
<!--8.0安装apk需要权限-->
<uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES"/>
```

最后代码如下：

```
Intent intent = new Intent(Intent.ACTION_VIEW);
intent.setflags(Intent.FLAG_ACTIVITY_NEW_TASK);

// android 7.0以上
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
    String authority = getPackageName() + ".fileprovider";
    Uri apkuri = FileProvider.getUriForFile(context,authority,apkfile);
    intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
    install.setDataAndType(apkuri, "application/vnd.android.package-archive");
    
    // 解决华为mate 10 8.0手机锁屏时安装apk 报解析安装包失败错误
    List<ResolveInfo> resInfoList = getPackageManager().queryIntentActivities(intent, PackageManager.MATCH_DEFAULT_ONLY);
    for (ResolveInfo resInfo : resInfoList) {
        String packageName = resInfo.activityinfo.packageName;
        grantUriPermission(packageName,apkUri,Intent.FLAG_GRANT_WRITE_URI_PERMISSION | Intent.FLAG_GRANT_READ_URI_PERMISSION);
    }
} else {
    intent.setDataAndType(Uri.parse("file://" + apkfile.toString()), "application/vnd.android.package-archive");
}

startActivity(intent);
android.os.Process.killprocess(android.os.Process.myPid());


<manifest>
    ...
    <application>
        ...
        <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="${applicationId}.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            
             <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_path"/>
        </provider>
        ...
    </application>
</manifest>

参数	              说明
name	              “android.support.v4.content.FileProvider”固定值
authorities  	      一般是”${applicationId}.yourname”这种格式，对于每个app是唯一的
exported	          必须是false，表示不公开
grantUriPermissions	  必须是true，表示允许临时读写文件
resource	          在meta-data中定义资源路径

file_path.xml
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <files-path name="my_images" path="images/"/>
    ...
</paths>

标签	                标签代表路径	       path=”files”代表路径
root-path	            设备的根目录	/files/*
files-path	            内部存储空间应用私有目录下的 files/ 目录，等同于context.getFilesDir()	context.getFilesDir()+/files/*
cache-path	            内部存储空间应用私有目录下的 cache/ 目录，等同于context.getCacheDir()	context.getCacheDir()+/files/*
external-path	        外部存储空间根目录，等同于Environment.getExternalStorageDirectory()	Environment.getExternalStorageDirectory()+/files/*
external-files-path	    外部存储空间应用私有目录下的 files/ 目录，等同于context.getExternalFilesDirs()	context.getExternalFilesDirs()+/files/*
external-cache-path	    外部存储空间应用私有目录下的 cache/ 目录，等同于getExternalCacheDirs()	getExternalCacheDirs()+/files/*

<!--8.0安装apk需要权限-->
<uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES"/>

```

file://到content://的转换规则：

1.替换前缀：把file://替换成content://${android:authorities}。

2.匹配和替换

2.1.遍历的子节点，找到最大能匹配上文件路径前缀的那个子节点。

2.2.用path的值替换掉文件路径里所匹配的内容。

3.文件路径剩余的部分保持不变。
![image]https://github.com/caimengru0807/learn-android/blob/master/image/FileProvider_replace_rula.png)

参考文档：

```
http://www.aoaoyi.com//archives/840.html
https://juejin.im/post/5ad4499a6fb9a028b617fc1c
https://developer.android.google.cn/reference/android/support/v4/content/FileProvider
```



