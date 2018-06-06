更新Android Stdudio后，经常会因为gradle版本和gradle Android插件版本不匹配导致出现问题，需查看gradle版本和gradle Android插件版本是否匹配。

在gradle/wrapper/gradle-wrapper.properties中修改gradle版本

```
...
distributionUrl = https\://services.gradle.org/distributions/gradle-4.4-all.zip
...
```



在最顶层的build.gradle文件中修改gradle插件版本

```
buildscript {
    repositories {
        // Gradle 4.1 and higher include support for Google's Maven repo using
        // the google() method. And you need to include this repo to download
        // Android plugin 3.0.0 or higher.
        google()
        ...
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.1.0'
    }
}
```

Plugin version | Required Gradle version
---|---
1.0.0 - 1.1.3   | 	  2.2.1 - 2.3
1.2.0 - 1.3.1   | 	  2.2.1 - 2.9
1.5.0           |     2.2.1 - 2.13
2.0.0 - 2.1.2   |     2.10 - 2.13
2.1.3 - 2.2.3   |     2.14.1+
2.3.0+          |     3.3+
3.0.0+          |     4.1+
3.1.0+          |     4.4+

官网地址：https://developer.android.google.cn/studio/releases/gradle-plugin#updating-plugin