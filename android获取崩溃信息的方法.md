## 1. Java Crash
通过UncaughtExceptionHandler来记录dump异常日志

---

## 2. Native Crash
1. 通过logcat来获取崩溃栈，需要崩溃时及时获取logcat，获取logcat的命令:

    adb shell logcat -v time > log.txt

2. android 7.0以上手机可以通过adb bugreport bugreport.zip 来获取到tombstones以及手机的一些状态信息
3. 通过集成google 开源项目breakpad来收集崩溃栈，下面主要介绍breakpad在android系统的集成。
breakpad官方地址为：https://chromium.googlesource.com/breakpad/breakpad，在github上的地址为https://github.com/google/breakpad。
###  集成到App
这个过程需要使用ndk-build编译出so文件，将so文件放到app的libs目录，并在java层初始化Breakpad，如以下三步：
- 编译Android各个平台的.so库
- 编写JNI层mapping文件
- 将.so文件和JNI mapping文件copy到项目

编译so库时遇到的问题
![image](https://github.com/caimengru0807/learn-android/blob/master/image/1.PNG)
解决方法：git clone https://chromium.googlesource.com/linux-syscall-support"
将 linux_syscall_support.h 拷贝到 breakpad/src/third_party/lss
![image](https://github.com/caimengru0807/learn-android/blob/master/image/2.PNG)
解决方法：breakpad最新代码需要使用NDK r11c 及以上版本编译

###  编译获得分析dmp文件的工具
这个过程主要是编译出2个工具：mindump_stack dump_syms。
由于breakpad的编译需要Linux下几个系统库，所以使用Ubuntu来进行编译。
在breakpad根目录下执行：

```
./configure && make
```
运行这个命令以后工具在生成在src/processor/mimdump_stack和src/tools/linux/dump_syms/dump_syms

###  分析Breakpad捕获到的dmp文件
假设：Breakpad捕获到的文件名为：crash.dmp, 你们项目中用到的.so 文件名为test.so，它们在同一个目录下。

获取symbol文件
./dump_sym test.so > test.so.sym
注： test.so.sym 是个文本文件
得到可读的堆栈跟踪文件: ./dump_stackwalk crash.dmp test.so.sym>crash.log
根据地址定位到对应的代码行数
./arm-linux-androideabi-addr2line -C -f -e libtest.so 0x1570ba
当项目中有多个so文件时，我们可以使用以下的sh脚本来进行解析：

```
#!/bin/bash  
  
#[usage]  
#将本脚本、dump_sys、minidump_stackwalk放在同级目录下，并创建libs文件夹，所有动态库放到libs文件夹内  
#./dump2txt.sh [dmp文件路径] [生成的txt文件路径]  
  
LIBRARY_DIRECTORY="libs"  
LIBRARY_EXTENDNAME=".sym"  
LIBRARY_KEYPOS=3  
  
Check()  
{  
    if [ $# -ne 2 ];then  
        echo please input two param,the first param is the dmp file path,the second param is txt file path  
        exit  
    fi  
  
    if [ ! -f "$1" ]; then  
        echo $1 is not exsit  
        exit  
    fi  
}  
  
DealLibrary()  
{  
    if [ ! -f "$LIBRARY_DIRECTORY/$1" ]; then  
        echo $LIBRARY_DIRECTORY/$1 is not exsit  
        return  
    fi  
  
    SYM_NAME=$1$LIBRARY_EXTENDNAME  
    ./dump_syms libs/$1 > $SYM_NAME  
      
    cat $SYM_NAME | while read line  
    do  
        LIBRARY_CODE=$line  
        ARR=($LIBRARY_CODE)  
        LIBRARY_CODE="${ARR[$LIBRARY_KEYPOS]}"  
        mkdir -p symbols/$1/$LIBRARY_CODE  
        mv $SYM_NAME symbols/$1/$LIBRARY_CODE  
        break  
    done  
}  
  
Main()  
{  
    #检查参数 $1:dmp文件路径;$2:生成的txt文件的路径  
    Check $1 $2  
    echo "start convert "$1" to "$2"...."  
  
    #创建解析dmp文件相关的目录以及文件  
    rm -rf symbols  
    for file in $LIBRARY_DIRECTORY/*  
    do  
        DealLibrary ${file:5}  
    done  
  
    #生成txt文件  
    ./minidump_stackwalk $1 symbols/ > $2  
    echo $2 is generated!!!!  
}  
Main $1 $2  
```
结构如下图：
![image](https://github.com/caimengru0807/learn-android/blob/master/image/3.PNG)
运行./dump2text.sh '/home/cai/crash/447e0686-d922-1304-0954048f-67affb6c.dmp' crash.txt即能生成txt的崩溃信息。

